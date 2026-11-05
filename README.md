# ROCKBOX ULTIMATE — iPod Classic 完全体

**二十年前的机器，不该用二十年前的妥协听歌。**
**A 20-year-old legend deserves better than 20-year-old compromises.**

这是给 iPod Classic 6G（ipod6g，64 MB）做的 Rockbox 定制版。它动的是原版 Rockbox 真正有问题的地方：ALAC 解码解错了数、采样率转换质量差、几个 DSP 和播放上的毛病。下面每条都有实测数字撑着。

A custom Rockbox build for the iPod Classic 6G (ipod6g, 64 MB). It fixes what stock Rockbox actually gets wrong: ALAC decoding errors, a poor sample-rate converter, and a handful of DSP and playback bugs. Every claim below is backed by measured numbers.

- 版本 / Version: `66f3075f9eM-261105`（A2 + ALAC + B 整合版 / integrated build）
- 发布包 / Release: `rockbox-ipod6g-A2-ALAC-B.zip`（9,258,598 字节，433 条目，44 codecs），在 Releases 里下载 / download from Releases
- 许可 / License: GPL-2.0（继承自 Rockbox 上游 / inherited from upstream Rockbox）
- 状态 / Status: 作者已于 2026-11-05 真机装机，装后逐项核验通过，日常使用中 / Installed on the author's own iPod on 2026-11-05, verified file by file, in daily use.

---

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
解压 Release 里的 rockbox-ipod6g-A2-ALAC-B.zip，里面只有一个 .rockbox 文件夹，对了。
Unzip the release ZIP. Inside is a single .rockbox folder.

第 3 步：合并，别替换
把解压出的 .rockbox 拖到 iPod 根目录，就是能看到 Music 文件夹的那一层。系统问你怎么处理，选「合并 / Merge」，千万别选「替换 / Replace」。安装包里没有 config.cfg，你的主题、歌单、设置都不会丢。
Drag the unzipped .rockbox onto the iPod root and choose Merge — never Replace. The package contains no config.cfg, so your themes and settings stay.

第 4 步：弹出、开机、核对
安全弹出，拔线开机，进 Rockbox Info 看到版本号 66f3075f9eM-261105 就是装好了。放一首 44.1/48 kHz 的歌，能播、能暂停、能拖进度，齐活。
Eject safely, boot, and confirm version 66f3075f9eM-261105 in Rockbox Info.

想退回去？把第 1 步备份的 .rockbox 再拖回来合并一次，就回到原来的系统。
Rollback: merge your backup .rockbox back the same way.

### 懒人通道：让 Codex 帮你装 / Let Codex install it for you

把下面这段话原样发给 Codex（或任何能操作你电脑的 AI 编程助手），它会带着你装完。该备份的备份，该核对的核对，出错会停下来问你。
Send this prompt to Codex (or any AI agent that can operate your computer):

```text
请帮我把一个 Rockbox 定制固件包装到我的 iPod Classic 6G 上，严格按下面做：
1. 固件包在 GitHub 仓库 liuzexin725-maker/rockbox-ipod-classic-ultimate 的 Release v261105-ultimate 里，文件名 rockbox-ipod6g-A2-ALAC-B.zip，先下载并解压，确认里面只有一个 .rockbox 文件夹、大小 9,258,598 字节。
2. 把 iPod 插上电脑，先把 iPod 里现有的整个 .rockbox 文件夹完整复制一份到电脑上做备份，并打开备份确认 rockbox.ipod 和 codecs 都在；备份没确认好之前，不要往 iPod 写任何东西。
3. 把解压出的 .rockbox 以「合并」方式拷到 iPod 根目录，绝对不要整目录替换；如果遇到同名文件，默认保留我设备上的个人设置文件 config.cfg（安装包本来就不含它）。
4. 拷完逐项核对：.rockbox/rockbox.ipod 应为 1,218,340 字节，.rockbox/codecs 里应有 44 个 .codec 文件。
5. 安全弹出后让我拔线开机，进 Rockbox Info 确认版本是 66f3075f9eM-261105。
6. 任何一步对不上立刻停下来告诉我，不要自行尝试修复，更不要动引导程序（bootloader）。
装好之后提醒我：导入这台 iPod 的歌曲采样率最高只到 48 kHz，96/192 的歌先在电脑上转成 48 再导入，不然会卡。
```

英文版提示词在 Release 详情里。The English prompt is in the Release notes.

## 致谢 / Credits

基于 Rockbox 上游（ipod6g 目标）。谢谢 Rockbox 项目，谢谢所有在 Apple 转身离开之后，还让这台机器活着的人。
Built on upstream Rockbox. Thanks to the Rockbox project, and to everyone who kept the iPod Classic alive long after Apple walked away.

让每一台还在转的 Classic，都听到它本该听到的声音。
Every Classic still spinning deserves to hear what it was always capable of.
