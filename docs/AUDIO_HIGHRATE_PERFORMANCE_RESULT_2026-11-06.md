# 96/192 kHz 播放卡顿排查结果（2026-11-06）

> **存档状态：本线已结案（2026-11-06）。** 结案理由：①根因未经设备实测证实，补齐证据需要在设备上复现卡顿并采集缓冲读数；②作者曲库已全部整理到 ≤48 kHz，新下载在源头锁定 48 kHz，日常使用不再遇到此问题；③本机输出上限即 48 kHz，机内实时重采样的音质上限不高于电脑端离线转换（结论见正文）。如需重启本线，先按文末「设备侧最小复核」清单备齐数据再议。正文为排查报告原文，仅将本地绝对路径改为相对表述。

## 结论

已从 B 源码确认：在 iPod 6G 内置输出、播放频率设为自动时，96 kHz 与 192 kHz 都会选 48 kHz 输出，并进入 B 的 2×/4× FIR 抽取路径。192 kHz 路径多经过一个 141-tap 前置级，再经过 181-tap 输出级。ARM926 构建的反汇编确认 FIR 内层使用 `SMLAL` 累加指令。这使高采样率的解码与 DSP 负载成为卡顿的合理嫌疑。

**根因仍未证实，也没有提交高采样率性能修复。**目前没有实际导入 ZIP 的身份、异常曲目的 codec/位深/码率、播放频率设置或卡顿当时的设备缓冲与时钟读数。源码路径、静态 tap 数、ARM 构建和模拟器都不能证明 iPod 上的实时余量。

用户报告的现象为：最新导入的 B 固件播放 96/192 kHz 曲目时声音和界面严重卡顿；44.1/48 kHz 正常；EQ 关闭。曲目位深、码率和 codec 尚未确认。

## 固件与路径身份

- 先前两个候选都报告版本字符串 `66f3075f9eM-261105`，但主程序不同、ALAC codec 相同。版本字符串不足以判断设备实际运行的是哪一个；实际导入的 ZIP 尚未定位。
- 本次全新构建使用一份来自整合树的独占源码副本，并带入 Bugfix 候选的两项生产代码变化：`apps/playback.c` 的音频缓冲分配失败退出处理、`apps/root_menu.c` 的录音入口隐藏。
- `firmware/export/config/ipod6g.h` 的硬件采样率能力含 44.1/48 kHz，不含 96/192 kHz；设备支持播放频率设置。`apps/playback.c:6311–6328` 的自动路由会优先选设备支持的源频率；否则，源频率能被 4,000 整除时回退到 48 kHz。因此 96/192 kHz 在自动模式下都选 48 kHz。若用户手动固定输出频率，路由可能不同，设备设置尚未知。

| 源频率 | 自动输出 | B 重采样路径 |
| --- | --- | --- |
| 44.1 kHz | 44.1 kHz | 输入与输出相同，重采样级关闭 |
| 48 kHz | 48 kHz | 输入与输出相同，重采样级关闭 |
| 96 kHz | 48 kHz | 2×，181 taps |
| 192 kHz | 48 kHz | 4×，141-tap 前置级 + 181-tap 输出级 |

具体 FIR 分支和 tap 选择见 `lib/rbcodec/dsp/resample.c:403–423`；内层运算见 `lib/rbcodec/dsp/resample_fir.h:23–81`。ARM926 反汇编在 `build-arm/lib/rbcodec/dsp/resample.o` 的 `fir_step` 中找到 `SMLAL`。FIR 状态在该构建占 20,616 字节 BSS（两个 DSP 状态槽）；这是静态内存占用信息，不是实时性能测量。

## 资源与缓冲线索

- iPod 6G 的常规 CPU 频率为 54 MHz，boost 上限为 216 MHz（`firmware/target/arm/s5l8702/system-target.h:27–36`）。这只是目标配置值，不代表卡顿时运行频率。
- codec 装载/运行及 PCM 缓冲低水位路径会触发 boost；`apps/pcmbuf.c:404–425` 还会根据 PCM 填充量调整 codec 线程优先级。源码显示有取消 boost 的等待路径，因此不能断言 boost 永远不恢复。
- `apps/debug_menu.c:400–516` 的 `View buffering thread` 可在设备侧读到 PCM 用量、文件缓冲的 `alloc/real/usefl`、`data_rem`、CPU 频率、boost 比例、`pcmbufdesc` 和 `watermark`。这些值能辅助区分解码/DSP 跟不上与读盘补给不足，但仍不能单独给出 codec CPU 百分比。
- 高采样率 ALAC 解码成本会随实际文件格式和编码复杂度变化；当前不知道异常曲目是否为 ALAC，也不知道位深和码率。存储读盘、调度饥饿、输出路径或 B 的边界逻辑仍未排除。单凭“96/192 卡、44.1/48 正常”不能确认 CPU 饱和。

## 本地构建与诊断包

- 全新 iPod 6G / 64 MB ARM926 构建使用 ARM GCC 9.5.0；`make -j8` 与 `make -j8 zip` 均成功。
- 构建显式启用 5 个 B 宏：`ROCKBOX_EXPERIMENTAL_FIR`、`ROCKBOX_EXPERIMENTAL_FIR_TIMELINE`、`ROCKBOX_EXPERIMENTAL_FIR_PLAYBACK_EOF`、`ROCKBOX_EXPERIMENTAL_FIR_AUTOSWITCH`、`ROCKBOX_EXPERIMENTAL_FIR_BOUNDARY`。
- 基础 ZIP 的 `unzip -t` 通过，含主程序、44 个 codec（含 `alac.codec`）。诊断包另外合入完整 Bugfix 主题 overlay：139 个文件，其中含 25 个 `.fnt` 文件；连同基础包字体，ZIP 共含 27 个 `.fnt`。部分 overlay 字体为个人用途衍生素材。包仅供原用户本地使用，不得再分发或公开发布；该诊断包仅在本地留存，未导入设备、未随本文入库。
- 若未来导入，包内 `.rockbox` 与设备同路径文件会发生覆盖；其中有多个主题 `.cfg`、`tagnavi.config`、`viewers.config`，但没有用户主配置 `config.cfg`。先备份设备 `.rockbox` 并保留新增/覆盖清单，再做文件级合并；回退时主程序与 codec 应从同一旧构建成套恢复。详见包内 `README.md`。本次没有导入设备。
- 因这份源码副本没有 Git 元数据，包内 `rockbox-info.txt` 版本字段为 `-261106`，不能用来声称它等同于先前候选。
- 当前主机 SDL 构建未完成：曾在宿主 `logf`/系统头文件预处理处失败；局部构建设置调整后又在 `codecs.make:124` 报 `missing separator`。没有改动生产源码来绕过该失败。本次没有运行高采样率 ALAC 的 SDL PCM 对照，也没有把宿主或 QEMU 计时用作实时证据。
- 没有安装或写入设备，没有改 Bootloader、设备配置、音乐或原始候选 ZIP；没有做设备听音或实时负载验收。

## 设备侧最小复核

要确认根因，需要同一次复现的少量设备数据：

1. 给出实际导入的 ZIP 文件名/路径；仅报版本字符串无法区分先前同标记的候选。
2. 对一首异常 96 kHz 和一首异常 192 kHz 曲目，记录 codec/container、位深、码率、声道数，并确认 `Settings > Playback Settings > Frequency` 是自动还是固定值。无需发送音频文件。
3. 播放时打开 `Debug > View buffering thread`，在 44.1/48 kHz 正常控制曲目与 96/192 kHz 卡顿期间各记录几次：`pcm` 用量/总量、`alloc`、`real`、`usefl`、`data_rem`、`cpu freq`、`boost`、`pcmbufdesc`、`watermark`。文本读数即可，无需截图。

若高采样率时文件缓冲保持充足而 PCM 持续见底，处理负载/调度嫌疑会上升；若文件缓冲先耗尽，则优先查读盘补给。两种结果仍需结合实际曲目和包身份再作判断。
