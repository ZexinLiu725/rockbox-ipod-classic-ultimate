# ROCKBOX ULTIMATE — iPod Classic 完全体

<p align="center"><strong>简体中文</strong> · <a href="README.en.md">English</a></p>

**二十年前的机器，不该用二十年前的妥协听歌。**
**A 20-year-old legend deserves better than 20-year-old compromises.**

这是给 iPod Classic 6G（ipod6g，64 MB）做的 Rockbox 定制版。它动的是原版 Rockbox 真正有问题的地方：ALAC 解码解错了数、采样率转换质量差、几个 DSP 和播放上的毛病。下面每条都有实测数字撑着。

A custom Rockbox build for the iPod Classic 6G (ipod6g, 64 MB). It fixes what stock Rockbox actually gets wrong: ALAC decoding errors, a poor sample-rate converter, and a handful of DSP and playback bugs. Every claim below is backed by measured numbers.

- 版本 / Version: `66f3075f9eM-261113`（A2 + ALAC + B 整合＋稳定性更新 / integrated build + stability update）
- 发布包 / Release: `rockbox-ipod6g-A2-ALAC-B-261113.zip`（9,261,810 字节，433 条目，44 codecs），在 Releases 里下载（当前版本 v261113-ultimate）/ download from Releases (current: v261113-ultimate)
- 许可 / License: GPL-2.0（继承自 Rockbox 上游 / inherited from upstream Rockbox）
- 状态 / Status: 作者已于 2026-11-13 真机装机，装后逐项核验通过，日常使用中 / Installed on the author's own iPod on 2026-11-13, verified file by file, in daily use.

---

## 先扫个盲：Rockbox 是什么，为什么要装 / New here? What Rockbox is, and why

Rockbox 是一套免费的开源系统，专门给老 MP3 播放器换上用的，2001 年就有人开始写，到现在二十多年了。你可以把它理解成给播放器换了个「大脑」：机器还是那台机器，但开机进去之后，界面、功能、能放的歌，全都换了一套。它不是苹果官方的东西，是全世界一群爱好者自己维护的，支持 iPod、闪迪、三星等很多老机型。

为什么好端端的要换系统？因为 iPod 的原厂系统限制太多：

- 传歌必须用 iTunes 同步，自己拖进去的文件它不认；
- 格式挑食，FLAC 这类常见的无损格式根本放不了；
- 均衡器只有几个固定预设，没法细调；
- 界面是十几年前的样子，主题、字体都换不了。

换上 Rockbox 之后，这些限制基本都没了：

- iPod 插上电脑就是个 U 盘，歌直接拖进去，打开就能播，不用再开 iTunes；
- 格式几乎全支持，MP3、FLAC、ALAC、APE、WAV、OGG 都能放，无损终于能无损地听；
- 有多段均衡器可以细调，还有无缝播放（现场专辑、古典乐曲目之间不断）、ReplayGain 音量平衡这些原厂没有的功能；
- 主题、字体、界面都能换，还能显示歌词。

那这一版和官方 Rockbox 又有什么区别？官方 Rockbox 救活了很多老机器，但它在 iPod Classic 上的音频部分有实打实的毛病：无损解码解错数、采样率转换质量差。这些问题藏了很多年，下一节就是这一版把它们一个个修掉的记录。

Rockbox is a free, open-source replacement firmware for older MP3 players, in development since 2001. Think of it as a new brain for the same hardware: the device stays, but the interface, features and supported formats are all replaced. It is a community project, not an Apple product, and it supports many classic players from iPod, SanDisk, Samsung and others.

Why bother? The stock iPod firmware is restrictive: music must go through iTunes sync (files you drag in yourself are ignored), formats are limited (no FLAC), the EQ is a few fixed presets, and the interface has been frozen for over a decade. With Rockbox, the iPod mounts as a plain USB drive — drag files in and play them. Nearly every format works (MP3, FLAC, ALAC, APE, WAV, OGG), there is a proper multi-band EQ, gapless playback, ReplayGain, lyrics display, and themes and fonts you can actually change.

So why this build, on top of Rockbox? Stock Rockbox revived these machines, but on the iPod Classic its audio path has real, long-standing bugs: the lossless decoder gets samples wrong, and the sample-rate converter is poor. The next section is the record of fixing them, one by one.

## 这一版改了什么 / What this build changes

### 1. ALAC：原版解错了三千八百多万个采样点，这版全对上了 / Bit-exact ALAC

原版 ALAC 解码器有 bug，还是那种听不出来、但一直在错的。四处全钉死了：24 位低字节拼装顺序错误、一类特殊 16 位帧的尾字节处理错误、32 位近乎静默、20 位电平错误。

拿一首真实的歌逐个采样点比对：38,715,000 个声道样本，原版约 3,860 万个和参考对不上，这一版逐个一致。容器到 codec 全链路收口，包括 edit list、曲尾和 seek：让它跳到 32 秒，实际落在 31,911 毫秒；落点之后除了最开始 80 帧里有 42 帧不同，往后 1,456,262 帧逐字节一致。

The stock ALAC decoder was silently wrong in four places: broken 24-bit low-byte reconstruction, a tail-byte bug on certain 16-bit frames, near-silent 32-bit output, and wrong 20-bit levels. All fixed and proven sample by sample: 38,715,000 channel samples of a real song, about 38.6 million wrong before, zero wrong after. Edit lists, end-of-track and seek are closed out too — a 32-second seek lands at 31,911 ms, and after an 80-frame boundary window, the following 1,456,262 frames are byte-identical to the reference.

### 2. 重采样：扔掉老插值，换带限 FIR / Band-limited FIR resampling

96/192 的歌要降到 48、88.2/176.4 要降到 44.1 才能出声。原版用的 Hermite 插值在高频留下镜像失真，镲片发毛、泛音发软。这一版换成带限 FIR：转换干净，瞬态不散，空气感归位。44.1/48 的原生文件不走转换，直通。

Hi-res files must be downsampled to the iPod's 44.1/48 kHz output. The stock Hermite interpolator smears the top end with imaging artifacts. This build uses a band-limited FIR resampler instead — clean conversion, tight transients. Native 44.1/48 kHz files pass through untouched.

### 3. DSP 与播放：一串小毛病一次清完 / Hardened DSP and playback

- 声道宽度调过 100% 后声场算歪，修了 / Channel width above 100% skewed the stereo image — fixed
- 增益开到极端时有个声道会反相，修了 / Extreme gain could flip a channel's phase — fixed
- ReplayGain 碰上没有峰值信息的歌会把正增益压没声，修了 / ReplayGain muted positive-gain tracks lacking peak data — fixed
- 解不了的歌原版容易卡死；现在有限次数跳过、重试，故障后按播放键能接着试 / Undecodable tracks get bounded skip/retry instead of a hang, and Play retries after a failure

### 作者本人实测 / From the author's own unit

这套固件就装在作者自己的 iPod 上，天天在用。装完逐个文件核对过，主程序和 44 个 codec 对版，个人设置一个没动。实际听下来，比原厂系统好出一截，尤其是无损文件的细节和声场。这是作者本人的耳朵结论，装上之后你可以自己验。

This build runs on the author's own iPod, in daily use, installed and verified file by file with personal settings untouched. To the author's ears it sounds clearly better than the stock Apple firmware, especially the detail and soundstage on lossless files. Install it and judge for yourself.

### 补记：BUGFIX 版（2026-11-05 同日小修）/ Bugfix update (same day)

完全体发布当天又收了三个小毛病，现在的下载版就是修好的版本，版本号没变（还是 `66f3075f9eM-261105`），认准 Release 名 `v261105-bugfix` 就行：

- **主菜单去掉录音入口**：iPod 6G 主菜单里不再显示录音，少一个用不上的入口 / The recording entry is gone from the 6G main menu.
- **主页不再重叠**：主页皮肤（SBS）改成全屏白底，并去掉了主页上的歌曲信息条，之前页面重叠、残留的毛病没了 / The home skin (SBS) now uses a full-screen white background and drops the song-info strip, fixing the overlapping / ghosted home screen.
- **内存不足不再死机**：播放时音频内存耗尽，原版会直接 PANIC 死机；现在改为停播并提示 `Audio memory exhausted`，机器还活着 / If audio memory runs out during playback, it no longer panics — playback stops with an `Audio memory exhausted` message instead.

还有一句字体的话：公开包里的字体是用思源黑体（Noto Sans CJK，OFL 开源授权）按原尺寸重新做的，和作者本机自用的那套字形略有不同，大小和间距一致，不影响使用。LanaPixel 像素字体因为授权没写清，这次没有收录；用到它的几个像素主题，菜单字体会回退成默认字体，其余一切正常。

The public package's fonts are rebuilt from Noto Sans CJK (OFL) at the original sizes — glyphs differ slightly from the author's own set, with identical metrics. The LanaPixel pixel font is not included (its license is unclear); pixel themes that referenced it fall back to the default font for menus.
### 补记：稳定性更新（2026-11-13）/ Stability update

版本号升到 `66f3075f9eM-261113`，认准 Release 名 `v261113-ultimate`。这版没有新功能，收的是几个稳定性口子：图片查看器与皮肤引擎各一处除零崩溃、两处内存不足时的硬崩溃、数据库标记契约，另外合入了上游后续两批更新。作者本机已装机，日常使用中。包的构成与首发完全体一致（自带 2 个 Helvetica 字体）；需要全套中文字体的，从 v261105-bugfix 包里把 .rockbox/fonts 文件夹拷过来合并即可。

The version string moves to `66f3075f9eM-261113`; look for the release named `v261113-ultimate`. No new features — this one closes stability holes: a divide-by-zero each in the JPEG viewer and the skin engine, two hard crashes on out-of-memory, and the database marker contract, plus upstream batches 2 and 3 merged in. It runs on the author's own iPod in daily use. Package composition matches the original ultimate package (2 bundled Helvetica fonts); for the full CJK font set, merge in the .rockbox/fonts folder from the v261105-bugfix package.


## 强烈推荐：导入音源最高只到 48 kHz / Feed it nothing above 48 kHz

真机实测（2026-11-05）：48 kHz 流畅；96 kHz 每几秒卡一次；192 kHz 卡成缓冲中的视频，声音和电平条一起顿。

原因两条。一，这台机器的输出天花板就是 44.1/48，96/192 进来最后都要降到 48，多出来的采样换不来任何可闻信息。二，FIR 转换质量好但吃算力，2 倍、4 倍下采样再叠上 24 位 ALAC 解码，2007 年的芯片算不过来，缓冲被吃空，就是卡。44.1/48 走直通，不转换，全部修复正是在这个区间火力最足。

最优音源：直接下 48 封顶的无损。比如 Apple Music 的无损档上限就是 24 位 / 48 kHz，母带链是第一梯队，文件天生落在直通区间，零转换、零卡顿。已有的 96/192 也不用扔，在电脑上用 libsoxr 最高档或 r8brain 这类工具离线转到 48 再导入，电脑不用实时，质量只高不低，转完彻底不卡。

On real hardware (2026-11-05): 48 kHz plays flawlessly; 96 kHz stutters every few seconds; 192 kHz stutters like a buffering video. The hardware ceiling is 44.1/48 kHz, so hi-res files must be downsampled before they can make a sound — and realtime FIR downsampling (2x for 96, 4x for 192) stacked on 24-bit ALAC decoding outruns the 2007 SoC and starves the playback buffer. Native 44.1/48 files pass straight through, which is where every fix in this build fires at full strength. Best source: lossless capped at 48 kHz, such as Apple Music's Lossless tier (24-bit/48 kHz). For 96/192 files you already own, convert offline to 48 kHz with a top-tier resampler (libsoxr VHQ / r8brain class) — same quality, zero stutter.

## 安装（新手版）/ Install (beginner-friendly)

不难，就三件事：先备份、再合并、最后核对。不用刷机，不动引导，装完不满意可以退回去。
Back up, merge, verify. No flashing, bootloader untouched, fully reversible.

开始前你需要：一台电脑（Mac 或 Windows 都行）、一根数据线、一台已经装好 Rockbox 的 iPod Classic 6G。还没装过 Rockbox 的，先按官方手册装好引导再回来。
You need: a computer, a USB cable, and an iPod Classic 6G already running Rockbox. New to Rockbox? Install the official bootloader first.

第 1 步：备份（最重要，别跳过）
把 iPod 插上电脑，它会像 U 盘一样出现。把里面的 .rockbox 文件夹整个复制到电脑上存好。这个文件夹是隐藏的：Mac 在访达里按 Cmd+Shift+. 显示它，Windows 在资源管理器里勾「隐藏的项目」。这个备份就是你的后悔药。
Copy the whole hidden .rockbox folder to your computer. This backup is your undo button.

第 2 步：解压
解压 Release 里的 rockbox-ipod6g-A2-ALAC-B-261113.zip，里面只有一个 .rockbox 文件夹，对了。
Unzip the release ZIP. Inside is a single .rockbox folder.

第 3 步：合并，别替换
把解压出的 .rockbox 拖到 iPod 根目录，就是能看到 Music 文件夹的那一层。系统问你怎么处理，选「合并 / Merge」，千万别选「替换 / Replace」。安装包里没有 config.cfg，你的主题、歌单、设置都不会丢。
Drag the unzipped .rockbox onto the iPod root and choose Merge — never Replace. The package contains no config.cfg, so your themes and settings stay.

第 4 步：弹出、开机、核对
安全弹出，拔线开机，进 Rockbox Info 看到版本号 66f3075f9eM-261113 就是装好了。放一首 44.1/48 kHz 的歌，能播、能暂停、能拖进度，齐活。
Eject safely, boot, and confirm version 66f3075f9eM-261105 in Rockbox Info.

想退回去？把第 1 步备份的 .rockbox 再拖回来合并一次，就回到原来的系统。
Rollback: merge your backup .rockbox back the same way.

### 懒人通道：让 Codex 帮你装 / Let Codex install it for you

把下面这段话原样发给 Codex（或任何能操作你电脑的 AI 编程助手），它会带着你装完。该备份的备份，该核对的核对，出错会停下来问你。
Send this prompt to Codex (or any AI agent that can operate your computer):

```text
请帮我把一个 Rockbox 定制固件包装到我的 iPod Classic 6G 上，严格按下面做：
1. 固件包在 GitHub 仓库 ZexinLiu725/rockbox-ipod-classic-ultimate 的 Release v261113-ultimate 里，文件名 rockbox-ipod6g-A2-ALAC-B-261113.zip，先下载并解压，确认里面只有一个 .rockbox 文件夹、大小 9,261,810 字节。
2. 把 iPod 插上电脑，先把 iPod 里现有的整个 .rockbox 文件夹完整复制一份到电脑上做备份，并打开备份确认 rockbox.ipod 和 codecs 都在；备份没确认好之前，不要往 iPod 写任何东西。
3. 把解压出的 .rockbox 以「合并」方式拷到 iPod 根目录，绝对不要整目录替换；如果遇到同名文件，默认保留我设备上的个人设置文件 config.cfg（安装包本来就不含它）。
4. 拷完逐项核对：.rockbox/rockbox.ipod 应为 1,226,276 字节，.rockbox/codecs 里应有 44 个 .codec 文件。
5. 安全弹出后让我拔线开机，进 Rockbox Info 确认版本是 66f3075f9eM-261113。
6. 任何一步对不上立刻停下来告诉我，不要自行尝试修复，更不要动引导程序（bootloader）。
装好之后提醒我：导入这台 iPod 的歌曲采样率最高只到 48 kHz，96/192 的歌先在电脑上转成 48 再导入，不然会卡。
```

英文版提示词在 Release 详情里。The English prompt is in the Release notes.

## 配套工具 / Companion tool

歌从哪来？推荐配 [Music Bridge](https://github.com/ZexinLiu725/music-bridge) 一起用：粘贴一个网易云歌单链接，它自动去 Apple Music 匹配无损（最高 48 kHz，正好是这台机器的直通区间），下载完单层平铺、按顺序编号导入 iPod。这版固件负责把歌放好，Music Bridge 负责把歌送进来。

Pair it with [Music Bridge](https://github.com/ZexinLiu725/music-bridge): paste a NetEase playlist link and it matches Apple Music lossless (capped at 48 kHz — this machine's pass-through sweet spot), then imports flat, in playlist order, into the iPod.

## 致谢 / Credits

基于 Rockbox 上游（ipod6g 目标）。谢谢 Rockbox 项目，谢谢所有在 Apple 转身离开之后，还让这台机器活着的人。
Built on upstream Rockbox. Thanks to the Rockbox project, and to everyone who kept the iPod Classic alive long after Apple walked away.

让每一台还在转的 Classic，都听到它本该听到的声音。
Every Classic still spinning deserves to hear what it was always capable of.
