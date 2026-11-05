# A2 + ALAC + B 整合候选结果（2026-11-05）

## 结论

A2、ALAC 和 B 已形成同源的 iPod 6G / 64 MB 本地候选，ZIP 完整性检查及主程序、44 个 codec 的同源对照通过。65 秒 ALAC 自然曲尾、单曲暂停、edit-list 以及 32 秒非零 seek/FIR 尾段在 SDL 中有针对性证据。seek/FIR 的首 80 个输出帧仍有 42 帧差异；之后的 1,456,262 帧与独立 suffix 参考逐字节一致。

这份候选尚未安装或在 iPod 上运行。DMA/I²S 实际排空、设备实时性、长期稳定性、听感、音质收益和续航均未验收。部分 sink-race 测试也没有触发其旧断言所期待的切频路径，不作通过声明。

## 候选与构建来源

- 候选包：[rockbox-ipod6g-A2-ALAC-B.zip](../out/AUDIO_CANDIDATE_2026-11-04-A2-ALAC-B/rockbox-ipod6g-A2-ALAC-B.zip)
- 导入与恢复说明：[README.md](../out/AUDIO_CANDIDATE_2026-11-04-A2-ALAC-B/README.md)
- 完整 ZIP 文件清单：[MANIFEST_unzip-l.txt](../out/AUDIO_CANDIDATE_2026-11-04-A2-ALAC-B/MANIFEST_unzip-l.txt)，由 `unzip -l` 生成。
- fresh ARM 构建产物目录：`/Users/zx75/Documents/Codex/2026-10-26/bang/work/audio-integrated-b-2026-11-04/build-ipod6g-2026-11-04-fir5/`。构建包装目录 `build-ipod6g-2026-11-05-package/` 的 Makefile 将 `BUILDDIR` 和 `OBJDIR` 指向 `build-ipod6g-2026-11-04-fir5/`。目标为 `ipod6g`、target id 71、64 MB、ARMv5；交叉编译器 GCC 9.5.0。ZIP 内 `rockbox-info.txt` 版本为 `66f3075f9eM-261105`。
- 构建启用 `ROCKBOX_EXPERIMENTAL_FIR`、`ROCKBOX_EXPERIMENTAL_FIR_TIMELINE`、`ROCKBOX_EXPERIMENTAL_FIR_PLAYBACK_EOF`、`ROCKBOX_EXPERIMENTAL_FIR_AUTOSWITCH`、`ROCKBOX_EXPERIMENTAL_FIR_BOUNDARY` 五个宏。
- 候选 ZIP 为 9,258,598 字节；含 433 个 ZIP 条目（409 个文件、24 个目录），顶层只有 `.rockbox/`。包内 `.rockbox/rockbox.ipod` 为 1,218,340 字节，另含 44 个 codec。候选 ZIP 与 `/Users/zx75/Documents/Codex/2026-10-26/bang/work/audio-integrated-b-2026-11-04/build-ipod6g-2026-11-04-fir5/rockbox.zip` 的只读 `cmp` 一致，大小均为 9,258,598 字节；包内主程序和 44 个 codec 与该 ARM 构建逐字节一致。当前 ZIP 的 `unzip -t` 通过，433 个条目 CRC 均正常。
- 首次在复制构建目录中执行打包时，生成路径读取了旧主程序；之后已从正确 integrated build cwd 重新执行 `make -j8 zip` 并以该包覆盖候选。第一次的错误 ZIP 没进入交付目录。
- 包不含用户主配置 `.rockbox/config.cfg`，但包含 `.rockbox/tagnavi.config`、`.rockbox/viewers.config`、插件配置、三个默认主题 `.cfg` 和两个默认 Adobe Helvetica 字体。不要据“不含主配置”推导为“包内没有配置”或“合并不会覆盖个人资源”；同名风险和导入/回退步骤见 README。

## SDL / 播放器验证

以下都是本地 SDL 或宿主证据，不是 iPod 实机验收。

| 用例 | 结果 | 范围说明 |
| --- | --- | --- |
| 44.1 kHz ALAC，自然 EOF 与 65 秒单曲暂停边界 | 两者各 2,866,500 帧，PCM 逐字节一致 | 仅覆盖该合成夹具及配置 |
| 88.2→44.1 kHz FIR，自然 EOF 与 65 秒单曲暂停边界 | 两者各 2,866,500 帧，PCM 逐字节一致 | 仅覆盖该合成夹具及配置 |
| 65 秒、24 位双声道 88.2 kHz ALAC，单段 v0 `elst`，自然曲尾 + FIR | 编辑区间 natural EOF 输出 2,867,542 个 44.1 kHz 帧；与 FFmpeg 按 edit list 解码后重编码的无 elst ALAC 参考逐字节一致 | 帧数受该 edit duration/timescale 影响；不外推到所有复用器输出 |
| 同一 edit-list/FIR 文件，seek 到 32 秒后自然 EOF | 输入为 v0 单段 `elst`（`media_time=3,412`、`duration=2,867,542`、movie timescale 44,100）。seek 请求 32,000 ms；最后一次 seek completion 时应用层报告 `elapsed=31,911 ms`（比目标早 89 ms）。该次 capture 为 1,745,158 帧，EOF marker 为 type 4 / 1,745,158 帧。取末尾 1,456,342 帧与无 elst suffix capture 对照：首 80 帧内 42 帧不同，差异帧从 0 到 58；之后 1,456,262 帧逐字节一致 | 80 帧作为 seek/FIR 边界差异窗口单独记录；不得表述为整个 suffix 全帧完全一致。两次通过的 SDL seek 复跑中，capture 总帧分别为 1,739,014 与 1,745,158，因为它也包含 seek 请求前已提交的 PCM；这两个总量均非固定预期值。固定比较对象是末尾 1,456,342 帧。测试脚本输出包含请求目标、报告 elapsed、marker 和帧差异窗口 |
| ALAC24 边界 STOP、SEEK、PAUSE/RESUME | 指定模拟器控制路径通过 | STOP 的函数/状态返回不证明最终 PCM 已静音、尾部已排空或 DMA/I²S 已停止 |
| 跨格式 drain 的 STOP、PAUSE/RESUME、SEEK | 指定 SDL 路径通过；捕获必须设 `ROCKBOX_TEST_PCM_CAPTURE_ALL=1`，否则首个 type=3 marker 会截断采集接缝 | STOP 控制返回不等于 PCM stop 验收；不据此宣称所有跨格式 PCM 尾部正确 |
| drain fault、ALAC 下一曲探测失败后 Play 重试、deferred queue 满路径 | 指定故障和队列场景通过 | 合成故障注入，不等同真实 I/O/设备故障覆盖 |
| action 18：sink 在 drain 前改变 | mismatch 保护按预期拒绝继续 drain | 这项保护用例通过 |

32 秒 seek 的独立验证脚本位于整合测试树：`/Users/zx75/Documents/Codex/2026-10-26/bang/work/audio-integrated-b-2026-11-04/validation/test_integrated_elst_seek.py`；测试 runner 的 action 101 位于同树 `simtest-source/tests/runner.h`。SDL 捕获启用了 `ROCKBOX_TEST_PCM_CAPTURE_ALL=1`，测试同时确认自然 EOF 和 FIR drain 成功。脚本结论只覆盖这份输入与这一采样率/seek 组合。

## 复用的 B 组件级证据

`docs/AUDIO_RESAMPLER_IMPLEMENTATION_RESULT.md` 中 2026-11-04 的 B 结果记录了宿主 writer-admission 与 PCM-output-gate 各 1,000 轮竞争验证，以及 buffer-quiesce 的 worker/reader/cancel/stale-generation 用例。集成前复核确认这些 B 生产文件与最新 B 版本逐字节一致，且 A2 增量没有修改这些文件；本轮没有重复运行这些组件用例。它们只补充组件级证据，不会把下述 SDL action 14/19/20/21/22 升级为通过。

## 未通过或未闭合的验证

- action 14/19 的旧 race 断言与当前缓冲重建围栏不匹配；观察到实际 sink 保持 44.1 kHz 且 drain 成功，预期的 48 kHz sink 变化没有发生，因此不计为 race 通过。
- action 20/21 的调用在 commit 前返回，不能证明实际切频已完成；action 22 没有进入 FIR 测试点。14/19/20/21/22 均不作为已通过的切频竞态证据。
- action 26 的旧 1 秒夹具不用于长曲/自然曲尾结论。长曲使用 action 13（上限 2,400 ticks）；action 0 的 400 tick 上限会在 65 秒曲目自然结束前停止，不能引用为 EOF 验证。
- edit-list seek 的 elapsed 更新值是 31,911 ms；本轮通过 PCM 尾段对照确认边界窗口后输出内容与 32 秒 suffix reference 一致，但该结果不能扩展为全部时间戳、容器或硬件播放位置的保证。
- SDL 对 PCM capture 的提交和 marker 记录不是硬件扬声器/I²S/DMA 采样。尤其 STOP 返回不证明静音时点、残余 PCM 或端到端排空。
- 2026-11-04 的 B SDL 全量构建记录有 libfaad `ps_dec.c` 两处 `-Wmaybe-uninitialized` 警告；它们是既有 AAC 代码背景，本次未扩大 AAC 范围。本次整合 ARM/SDL 的完整 warning log 未保留，因此不据此宣称整合构建没有额外警告；本轮仅强制重编了 SDL 测试副本 `apps/main.o` 并重链 `rockboxui`，该增量输出未出现警告。

## 设备、覆盖与恢复边界

没有连接/安装/写入设备，没有 ROLO，没有改 Bootloader、设备配置、音乐或主题。未做设备听音、实时负载、温度、续航或长时间稳定性测试。ARM 构建成功不证明 ARM 运行或设备兼容。不要将这份本地候选称为“已在设备通过”。

导入前备份现有 `.rockbox`，保留 `config.cfg`，检查并处理主题、字体、`tagnavi.config`、`viewers.config` 和插件配置同名文件；`15-Adobe-Helvetica.fnt` 与 `12-Adobe-Helvetica-Bold.fnt` 是明确的覆盖风险。主程序和 44 个 codec 必须成套更新，回退时也从同一份旧备份成套恢复。完整步骤见候选目录 README。本次只形成候选与文档，没有执行导入。
