# ROCKBOX ULTIMATE — iPod Classic 完全体

**二十年前的机器，不该用二十年前的妥协听歌。**
**A 20-year-old legend deserves better than 20-year-old compromises.**

为 iPod Classic 6G（ipod6g，64 MB）深度重铸的 Rockbox 音频完全体：
ALAC 解码逐样本对齐参考、带限 FIR 重采样取代老旧 Hermite 插值、播放生命周期全面加固——
每一帧，都算数。

The definitive audio build of Rockbox for the iPod Classic 6G (ipod6g, 64 MB):
bit-exact ALAC decoding, a band-limited FIR resampler replacing the stock Hermite
interpolator, and a hardened playback lifecycle — *every single frame, accounted for.*

- 版本 / Version: `66f3075f9eM-261105`（A2 + ALAC + B 整合版 / integrated build）
- 发布包 / Release: `rockbox-ipod6g-A2-ALAC-B.zip`（9,258,598 字节，433 条目，44 codecs）
- 许可 / License: GPL-2.0（继承自 Rockbox 上游 / inherited from upstream Rockbox）
- 状态 / Status: 2026-11-05 已装机，装后逐项核验通过（主程序 SHA-256 与候选逐字节一致、44 codecs 齐全、个人 config.cfg 未动）/ Installed on real hardware 2026-11-05 and verified file-by-file after install.

---

## 为什么它不一样 / Why this build hits different

### 1. ALAC：逐样本的正确 / Bit-exact ALAC

原装 ALAC 解码藏着一串静默的错误：24 位解码低字节重建顺序错误、
一类特殊 16 位压缩帧尾字节处理错误、32 位 ALAC 近乎静默、20 位电平错误。
本版全部修复并逐样本验证：真实歌曲 38,715,000 个声道样本，
修复前 3,860 万个与参考不符，修复后 **全部一致**。

The stock ALAC decoder silently got samples wrong — a broken 24-bit low-byte
reconstruction, a tail-byte bug on certain 16-bit frames, near-silent 32-bit
output, wrong levels on 20-bit files. All fixed, all proven sample-by-sample:
38,715,000 channel samples of a real song, 38.6 million wrong before,
**zero wrong after**.

- 容器到 codec 全链路收口：包边界、edit list（elst）、seek 落点、截断流、自然曲尾
- 32 秒非零 seek 实测：落点 31,911 ms（包边界量化），边界 80 帧窗口后 1,456,262 帧与参考逐字节一致

### 2. 重采样：FIR 带限，告别发毛的高频 / Band-limited FIR resampling

96/192 kHz 走 48 kHz、88.2/176.4 kHz 走 44.1 kHz——原装 Hermite 插值
在高频留下镜像失真，镲片发毛、泛音发软。本版换上带限 FIR 重采样器：
转换干净，瞬态不散，空气感归位。

Hi-res files must be downsampled to the iPod's 44.1/48 kHz output. The stock
Hermite interpolator smears the top end with imaging artifacts. This build
replaces it with a proper band-limited FIR resampler — clean conversion,
tight transients, air restored. (44.1/48 kHz native files pass through untouched.)

## 强烈推荐：导入音源最高只到 48 kHz / STRONGLY RECOMMENDED: feed it nothing above 48 kHz

**这是本版最重要的使用建议，没有之一。**
**This is the single most important usage note in this release.**

本机实测（2026-11-05，真机）：48 kHz 曲目流畅正常；96 kHz 每几秒卡顿一次；
192 kHz 卡成「视频缓冲」状，声音与电平表一起顿挫。
原因很实在，一条一条说清：

1. **这台机器的输出天花板就是 48 kHz。** iPod Classic 的硬件采样率家族只有
   44.1/48 两系，96/192 的歌最终都必须降到 48 才能发声——多出来的采样，
   在这台机器上换不来任何可闻信息，只换来一道实时换算工序。
2. **实时 FIR 换算比这颗老芯片跑得快。** 96 是 2 倍下采样、192 是 4 倍下采样，
   还叠着 24 位 ALAC 解码。播放缓冲被算力吃空，就是你听到的卡顿：
   不是文件坏了，也不是系统装坏了，是芯片算一口、播一口，算不过来。
3. **44.1/48 的文件走直通。** 不触发重采样，CPU 轻装上阵——这才是这台机器
   设计内的主场，也是本版全部正确性修复（ALAC、seek、曲尾）火力全开的区间。

**那最优音源从哪来？直接下最高 48 的无损。**
以 Apple Music 为例：它的「无损」档上限正好是 24-bit / 48 kHz，
母带处理链（Apple Digital Masters 一系）是业界第一梯队，文件天生落在
本机的直通区间——零换算、零卡顿、零损耗，比先下 192 再在任何地方降频都干净。
存量已有的 96/192 文件也不必扔：在电脑上用高质量重采样
（libsoxr 最高档 / r8brain 一类，浮点运算＋抖动）离线转到 48 再导入，
电脑不用实时、不用定点，转换误差压到设备底噪以下几十 dB，
听感与直下 48 无异，且彻底不卡。

On real hardware (2026-11-05): 48 kHz plays flawlessly; 96 kHz stutters every
few seconds; 192 kHz stutters like a buffering video, meters and audio alike.
Three plain reasons:

1. **The hardware ceiling is 48 kHz.** The iPod Classic's sample-rate families
   top out at 44.1/48 — a 96/192 file must be downsampled to 48 before it can
   make a sound. The extra samples buy zero audible information on this device.
2. **Realtime FIR conversion outruns this aging SoC.** 2x downsampling for 96,
   4x for 192, stacked on 24-bit ALAC decoding — the playback buffer starves
   and you hear it stutter. Not a bad file, not a bad install: the chip simply
   cannot compute as fast as it plays.
3. **44.1/48 files pass straight through.** No resampling, minimal CPU — the
   device's home turf, where every correctness fix in this build fires at full
   strength.

Best source: lossless capped at 48 kHz (e.g. Apple Music's Lossless tier,
24-bit/48 kHz, from a mastering-grade delivery chain) — native to the
passthrough range, no conversion, no stutter, no loss. For 96/192 files you
already own, convert offline on a computer with a top-tier resampler
(libsoxr VHQ / r8brain class, floating-point + dither) down to 48 kHz:
not realtime, not fixed-point, error buried tens of dB below the device's
own noise floor — audibly identical to a native 48 download, and stutter-free.

### 3. DSP 与播放：细节全线加固 / Hardened DSP & playback

- 声道宽度（>100%）整数运算语义修正，声场摆正
- 极端增益下输出环绕导致声道反相、负数左移未定义行为、抖动状态位宽——逐一修复
- ReplayGain 缺 peak 元数据时正增益压静音的坑已填
- 播放恢复：不可解码曲目有界跳过、I/O 错误有界重试、失败原因区分、故障后 Play 重试

## 安装 / Install

**先备份，后动手。备份没验完，不要写设备。**
**Back up first. No verified backup, no install.**

1. 备份 iPod 上整个 `.rockbox` 文件夹到电脑，并确认备份完整。
   Back up the entire `.rockbox` folder from your iPod and verify it.
2. 解压发布包，确认顶层只有 `.rockbox/`。
   Unzip the release; the only top-level folder must be `.rockbox/`.
3. 以「合并」方式把 `.rockbox` 拷入 iPod 根目录，**不要整目录替换**；
   保留你自己的 `config.cfg`（本包不含它）。
   Merge — never replace — into the iPod root. Keep your own `config.cfg` (not included).
4. 主程序 `rockbox.ipod` 与 44 个 codec 必须成套更新，不要混搭其他版本。
   The main binary and all 44 codecs ship as a set. Never mix versions.
5. 安全弹出、重启，确认 Rockbox Info 显示 `66f3075f9eM-261105`。
   Eject safely, reboot, confirm version `66f3075f9eM-261105` in Rockbox Info.

注意：包内含默认主题配置、两个默认字体与插件配置，同名文件合并前先比对。
Note: bundled default theme/font/plugin configs may share names with yours — diff before merging.

回退：从第 1 步的备份成套恢复整个原 `.rockbox` 即可，引导程序未动。
Rollback: restore the whole original `.rockbox` from your backup. The bootloader is untouched.

## 验证边界（不吹的部分）/ Verified — and honestly not

屌炸天的介绍，也要配屌炸天的诚实：

- 已验证：SDL/模拟器逐字节回归（自然曲尾、暂停、edit list、seek）、ARM 同源构建、
  候选包与构建产物逐字节一致、真机装机与装后文件核验。
- 真机实测边界（2026-11-05）：44.1/48 kHz 流畅；96 kHz 约几秒一卡、192 kHz 严重卡顿
  ——实时 FIR 重采样超出本机算力，故上方「最高导入 48」为正式使用建议。
- 未宣称：长时间稳定性、续航、DMA/I2S 实时排空与主观音质收益未经系统测量；
  部分切频竞态用例未计为通过，细节见仓库内整合结果单。

- Verified: byte-exact simulator regressions (natural EOF, pause, edit lists, seek),
  a from-source ARM build, release ZIP identical to the build output,
  and a real-device install verified file-by-file.
- Not claimed: long-term stability, battery life, DMA/I2S drain timing and subjective
  sound-quality gains are not benchmarked; some sample-rate-switch race cases are
  documented as not passed. See the integration report in this repo.

## 致谢 / Credits

基于 Rockbox 上游（ipod6g 目标，GCC 9.5.0 构建）。
Built on upstream Rockbox. Thanks to the Rockbox project and everyone who kept
the iPod Classic alive long after Apple walked away.

*让每一台还在转的 Classic，都听到它本该听到的声音。*
*Every Classic still spinning deserves to hear what it was always capable of.*
