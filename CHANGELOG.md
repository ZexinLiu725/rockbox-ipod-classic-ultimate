# 更新日志 / Changelog

## [BUGFIX 版] — 2026-11-05

版本 / Version: `66f3075f9eM-261105`（同完全体基线，Release: `v261105-bugfix`）

### 修复 / Fixed
- iPod 6G 主菜单隐藏录音入口 / Recording entry removed from the iPod 6G main menu
- 主页皮肤（SBS）改全屏白底并去掉主页歌曲信息条，修复页面重叠与残留 / Home skin (SBS): full-screen white background, song-info strip removed — fixes overlapping and ghosted home-screen content
- 音频内存耗尽不再 PANIC 死机，改为停播并提示 `Audio memory exhausted` / Audio-memory exhaustion no longer panics; playback stops with an `Audio memory exhausted` message

### 发布包 / Packaging
- 公开包字体为思源黑体（Noto Sans CJK，OFL 授权）按原尺寸重制，与作者本机自用版字形略有不同 / Public-package fonts rebuilt from Noto Sans CJK (OFL) at the original metrics
- LanaPixel 像素字体因授权未明未收录，用到它的像素主题菜单回退默认字体 / LanaPixel excluded (license unclear); pixel themes fall back to the default font for menus

## [A2+ALAC+B 完全体] — 2026-11-05

版本 / Version: `66f3075f9eM-261105`（iPod Classic 6G / ipod6g, 64 MB）

### 新增 / Added
- 带限 FIR 重采样器：88.2/176.4 kHz → 44.1 kHz、96/192 kHz → 48 kHz，替代原装 Hermite 插值（带五宏开关整合）
- Band-limited FIR resampler replacing the stock Hermite interpolator

### 修复 / Fixed
- ALAC 全链路正确性：24 位低字节重建、特殊 16 位帧尾字节、32 位近静默、20 位电平错误；包边界 / edit list / seek / 截断流 / 自然曲尾收口
- Bit-exact ALAC decoding across bit depths; container-to-codec chain hardened
- DSP 声道宽度（>100%）整数语义、极端增益声道反相、ReplayGain 缺 peak 压静音
- 播放恢复：有界跳过 / 重试、故障后 Play 重试

### 真机结论 / Real-device findings (2026-11-05)
- 已装机并逐项核验（主程序 SHA-256 与候选一致、44 codecs、个人 config 未动）
- **44.1/48 kHz 流畅；96 kHz 几秒一卡、192 kHz 严重卡顿**——实时 FIR 超出本机算力
- **正式建议：导入音源最高 48 kHz**（如 Apple Music 无损档 24/48）；存量高清文件请离线转码到 48 后导入
- Installed and verified on hardware. 44.1/48 kHz smooth; 96/192 kHz stutter under realtime FIR load — feed it ≤48 kHz sources (offline-convert hi-res to 48 first).

## [A-ALAC v2] — 2026-10-30
- 一类特殊 16 位 ALAC 压缩帧尾字节处理错误修复（冻结包）

## [A-ALAC v1] — 2026-10-30
- 24 位 ALAC 解码、DSP 整数精度、ReplayGain 防削波等首批修复
