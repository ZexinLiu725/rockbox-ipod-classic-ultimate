<p align="center"><a href="README.md">简体中文</a> · <strong>English</strong></p>

# ROCKBOX ULTIMATE — The Definitive iPod Classic Build

**A 20-year-old legend deserves better than 20-year-old compromises.**

A custom Rockbox build for the iPod Classic 6G (ipod6g, 64 MB). It fixes what stock Rockbox actually gets wrong: ALAC decoding errors, a poor sample-rate converter, and a handful of DSP and playback bugs. Every claim below is backed by measured numbers.

- Version: `66f3075f9eM-261105` (A2 + ALAC + B integrated build)
- Release package: `rockbox-ipod6g-A2-ALAC-B-BUGFIX.zip` (73,394,046 bytes, 583 entries, 44 codecs), download from Releases (current: v261105-bugfix)
- License: GPL-2.0 (inherited from upstream Rockbox)
- Status: Installed on the author's own iPod on 2026-11-05, verified file by file, in daily use.

---

## New here? What Rockbox is, and why

Rockbox is a free, open-source replacement firmware for older MP3 players, in development since 2001. Think of it as a new brain for the same hardware: the device stays, but the interface, features and supported formats are all replaced. It is a community project, not an Apple product, and it supports many classic players from iPod, SanDisk, Samsung and others.

Why bother? The stock iPod firmware is restrictive: music must go through iTunes sync (files you drag in yourself are ignored), formats are limited (no FLAC), the EQ is a few fixed presets, and the interface has been frozen for over a decade. With Rockbox, the iPod mounts as a plain USB drive — drag files in and play them. Nearly every format works (MP3, FLAC, ALAC, APE, WAV, OGG), there is a proper multi-band EQ, gapless playback, ReplayGain, lyrics display, and themes and fonts you can actually change.

So why this build, on top of Rockbox? Stock Rockbox revived these machines, but on the iPod Classic its audio path has real, long-standing bugs: the lossless decoder gets samples wrong, and the sample-rate converter is poor. The next section is the record of fixing them, one by one.

## What this build changes

### 1. Bit-exact ALAC

The stock ALAC decoder was silently wrong in four places: broken 24-bit low-byte reconstruction, a tail-byte bug on certain 16-bit frames, near-silent 32-bit output, and wrong 20-bit levels. All fixed and proven sample by sample: 38,715,000 channel samples of a real song, about 38.6 million wrong before, zero wrong after. Edit lists, end-of-track and seek are closed out too — a 32-second seek lands at 31,911 ms, and after an 80-frame boundary window (42 frames differ), the following 1,456,262 frames are byte-identical to the reference.

### 2. Band-limited FIR resampling

Hi-res files must be downsampled to the iPod's 44.1/48 kHz output. The stock Hermite interpolator smears the top end with imaging artifacts. This build uses a band-limited FIR resampler instead — clean conversion, tight transients. Native 44.1/48 kHz files pass through untouched.

### 3. Hardened DSP and playback

- Channel width above 100% skewed the stereo image — fixed
- Extreme gain could flip a channel's phase — fixed
- ReplayGain muted positive-gain tracks lacking peak data — fixed
- Undecodable tracks get bounded skip/retry instead of a hang, and Play retries after a failure

### From the author's own unit

This build runs on the author's own iPod, in daily use, installed and verified file by file with personal settings untouched. To the author's ears it sounds clearly better than the stock Apple firmware, especially the detail and soundstage on lossless files. Install it and judge for yourself.

### Bugfix update (2026-11-05, same day)

Three small bugs were caught on launch day. The current download is the fixed build — the version string is unchanged (`66f3075f9eM-261105`); look for the release named `v261105-bugfix`:

- **No more recording entry**: the 6G main menu no longer shows a recording entry it cannot use.
- **Home screen no longer overlaps**: the home skin (SBS) now uses a full-screen white background and drops the song-info strip, fixing the overlapping / ghosted home screen.
- **Out of memory no longer crashes**: if audio memory runs out during playback, it no longer panics — playback stops with an `Audio memory exhausted` message instead.

One word on fonts: the public package's fonts are rebuilt from Noto Sans CJK (OFL) at the original sizes — glyphs differ slightly from the author's own set, with identical metrics. The LanaPixel pixel font is not included (its license is unclear); pixel themes that referenced it fall back to the default font for menus, everything else works as before.

## Feed it nothing above 48 kHz

On real hardware (2026-11-05): 48 kHz plays flawlessly; 96 kHz stutters every few seconds; 192 kHz stutters like a buffering video, with the sound and the level meter hitching together.

Two reasons. First, the hardware ceiling is 44.1/48 kHz, so hi-res files must be downsampled before they can make a sound — the extra samples buy no audible information. Second, the FIR converter sounds better but costs compute: 2x or 4x downsampling stacked on 24-bit ALAC decoding outruns the 2007 SoC and starves the playback buffer. That is the stutter. Native 44.1/48 files pass straight through, which is where every fix in this build fires at full strength.

Best source: lossless capped at 48 kHz, such as Apple Music's Lossless tier (24-bit/48 kHz) — top-tier mastering chain, files land natively in the pass-through range, zero conversion, zero stutter. For 96/192 files you already own, convert offline to 48 kHz with a top-tier resampler (libsoxr VHQ / r8brain class) before importing — a computer does not work in real time, quality is as good or better, and playback is stutter-free for good.

## Install (beginner-friendly)

Back up, merge, verify. No flashing, bootloader untouched, fully reversible.

You need: a computer (Mac or Windows), a USB cable, and an iPod Classic 6G already running Rockbox. New to Rockbox? Install the official bootloader first, then come back.

**Step 1 — Back up (do not skip).** Plug the iPod in; it mounts like a USB drive. Copy the whole `.rockbox` folder to your computer. It is hidden: on Mac press Cmd+Shift+. in Finder to reveal it; on Windows enable "Hidden items" in Explorer. This backup is your undo button.

**Step 2 — Unzip.** Unzip `rockbox-ipod6g-A2-ALAC-B-BUGFIX.zip` from the Release. Inside is a single `.rockbox` folder.

**Step 3 — Merge, never Replace.** Drag the unzipped `.rockbox` onto the iPod's root (the level where you see the Music folder). When the system asks, choose **Merge** — never Replace. The package contains no config.cfg, so your themes, playlists and settings stay.

**Step 4 — Eject, boot, verify.** Eject safely, unplug, boot, and confirm version `66f3075f9eM-261105` in Rockbox Info. Play a 44.1/48 kHz song — play, pause and seek all working means you are done.

Rollback: merge your backup `.rockbox` back the same way.

### Let Codex install it for you

Send this prompt to Codex (or any AI agent that can operate your computer):

```text
Please install a custom Rockbox firmware package on my iPod Classic 6G, following these steps exactly:
1. The package is in the GitHub repo ZexinLiu725/rockbox-ipod-classic-ultimate, Release v261105-bugfix, file name rockbox-ipod6g-A2-ALAC-B-BUGFIX.zip. Download and unzip it, and confirm it contains a single .rockbox folder and is 73,394,046 bytes.
2. Plug in my iPod and first copy its entire existing .rockbox folder to my computer as a backup. Open the backup and confirm rockbox.ipod and the codecs are inside. Do not write anything to the iPod until the backup is confirmed.
3. Copy the unzipped .rockbox to the iPod root using Merge — never replace the whole folder. If a file conflict involves my personal config.cfg, keep the one on my device (the package does not contain one anyway).
4. Verify afterwards: .rockbox/rockbox.ipod must be 1,218,324 bytes, and .rockbox/codecs must contain 44 .codec files.
5. Eject safely, let me unplug and boot, and confirm the version in Rockbox Info is 66f3075f9eM-261105.
6. If anything does not match, stop and tell me immediately. Do not attempt repairs on your own, and never touch the bootloader.
When it is installed, remind me: songs imported to this iPod should be at most 48 kHz — convert 96/192 kHz files to 48 on the computer first, or they will stutter.
```

## Companion tool

Pair it with [Music Bridge](https://github.com/ZexinLiu725/music-bridge): paste a NetEase playlist link and it matches Apple Music lossless (capped at 48 kHz — this machine's pass-through sweet spot), downloads flat in playlist order, and imports into the iPod. This firmware handles playing it right; Music Bridge handles getting it there.

## Credits

Built on upstream Rockbox (ipod6g target). Thanks to the Rockbox project, and to everyone who kept the iPod Classic alive long after Apple walked away.

Every Classic still spinning deserves to hear what it was always capable of.
