<div align="center">

# 🎬 vidkit

### *Three no-nonsense bash tools for cutting, scanning, and framing your videos — powered by ffmpeg.*

[![Shell Script](https://img.shields.io/badge/Shell_Script-121011?style=for-the-badge&logo=gnu-bash&logoColor=white)](https://www.gnu.org/software/bash/)
[![FFmpeg](https://img.shields.io/badge/FFmpeg-007808?style=for-the-badge&logo=ffmpeg&logoColor=white)](https://ffmpeg.org/)
[![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)](https://www.linux.org/)
[![macOS](https://img.shields.io/badge/macOS-000000?style=for-the-badge&logo=apple&logoColor=white)](https://www.apple.com/macos/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

[![Typing SVG](https://readme-typing-svg.herokuapp.com?font=Fira+Code&weight=600&size=22&duration=2500&pause=800&color=2AA6A0&background=00000000&center=true&vCenter=true&width=650&lines=vid-info+%E2%86%92+scan+your+video+library;vid-cut+%E2%86%92+trim+clips+or+split+chapters;vid-2-img+%E2%86%92+pull+frames+at+any+rate)](https://git.io/typing-svg)

</div>

---

## 📌 About

**vidkit** is a small toolbox of standalone bash scripts built on `ffmpeg` / `ffprobe` — no Python env, nothing to install beyond ffmpeg. Each tool does one job, has colored output, progress bars, safety confirmations, and a proper `--help`.

| Script | Does what | Version |
|---|---|:---:|
| 🔍 [`vid-info`](#-vid-info) | Scan a file or a whole folder → duration, size, bitrate, codecs, resolution, fps | `3.0.0` |
| ✂️ [`vid-cut`](#️-vid-cut) | Cut a clip by time range, or auto-split by embedded chapters | `1.2.0` |
| 🖼️ [`vid-2-img`](#️-vid-2-img) | Extract frames — by fps, by Nth frame, by time range, or all of them | `1.0.0` |

---

## ⚙️ Requirements

```bash
# Ubuntu / Debian
sudo apt install ffmpeg

# macOS
brew install ffmpeg

# jq is optional — only needed for `vid-info --json`
```

## 🚀 Installation

```bash
git clone https://github.com/diea-abdeltwab/vidkit.git
cd vidkit
chmod +x vid-info vid-cut vid-2-img

# optional: make them available anywhere on your system
sudo cp vid-info vid-cut vid-2-img /usr/local/bin/
```

---

## 🔍 `vid-info`

Scan a single file or an entire folder, sort/filter results, or export JSON for scripting.

```bash
vid-info video.mp4
# → duration, size, bitrate, resolution, fps, codec — for that one file

vid-info ~/Videos -r --min 100
# → recursively scans ~/Videos, but skips anything under 100 MB

vid-info ~/Videos --sort size --json | jq '.[].name'
# → JSON output piped into jq, sorted largest-first, printing just the filenames

vid-info ~/Videos -j 4
# → scans the folder using 4 parallel jobs instead of one-by-one
```

<details>
<summary><b>All options</b></summary>

| Flag | Description |
|---|---|
| `-r`, `--recursive` | Scan sub-folders too |
| `--min <MB>` | Skip files smaller than N MB (default: 0) |
| `--sort <key>` | Sort by `name` \| `size` \| `dur` \| `fps` |
| `-j`, `--jobs <N>` | Parallel scan jobs (default: auto, capped at 8) |
| `--json` | Raw JSON output, for piping into `jq` |
| `-v`, `--version` / `-h`, `--help` | Show version / help |

</details>

---

## ✂️ `vid-cut`

Frame-accurate cuts by default, or instant stream-copy with `--fast`. Can also explode a video into one file per embedded chapter.

```bash
vid-cut video.mp4 -s 0:30 -e 1:45
# → cuts from 00:30 to 01:45, re-encoded so the cut is frame-exact

vid-cut video.mp4 -s 1:20:00 -e 1:25:30 -o out.mp4
# → same idea with HH:MM:SS timestamps, saved as out.mp4

vid-cut video.mp4 -s 90 -e 150 --fast
# → cuts using raw seconds, no re-encode (snaps to the nearest keyframe — instant, slightly less precise)

vid-cut video.mp4 --chapters -o ./parts/
# → splits the video into one numbered file per embedded chapter, saved into ./parts/

vid-cut --info video.mp4
# → just prints duration/codec info + the chapter list, then exits (no cutting)
```

<details>
<summary><b>All options</b></summary>

| Flag | Description |
|---|---|
| `-s`, `--start TIME` | Start time (required for clip mode) |
| `-e`, `--end TIME` | End time (required for clip mode) |
| `-o`, `--output PATH` | Output file (clip mode) or folder (chapters mode) |
| `--chapters` | Split into one numbered file per chapter |
| `--fast` | Stream copy — no re-encode, snaps to keyframes |
| `-y`, `--overwrite` | Overwrite existing output |
| `--info <video>` | Show video info + chapters, then exit |
| `-v`, `--version` / `-h`, `--help` | Show version / help |

**Time formats accepted:** `HH:MM:SS`, `MM:SS`, or raw seconds.

</details>

---

## 🖼️ `vid-2-img`

Pull frames at a fixed fps, every Nth frame, or every single frame — with a live progress bar and a safety cap so you don't accidentally dump 50,000 images on disk.

```bash
vid-2-img video.mp4
# → default: 1 frame per second, saved as .jpg next to the video

vid-2-img video.mp4 --fps 5 --fmt png
# → 5 frames per second, PNG instead of JPG

vid-2-img video.mp4 --nth 10 -o ~/frames
# → every 10th frame only, saved into ~/frames

vid-2-img video.mp4 -s 0:30 -e 1:45 --quality 85
# → only extracts frames between 00:30–01:45, JPEG quality 85

vid-2-img video.mp4 --fps all --batch
# → dumps every single frame, skipping all confirmation prompts (careful — that's a lot of files)
```

<details>
<summary><b>All options</b></summary>

| Flag | Description |
|---|---|
| `--fps <N\|all>` | Frames per second, or `all` (default: 1) |
| `--nth <N>` | Every Nth frame (overrides `--fps`) |
| `-s`, `--start TIME` / `-e`, `--end TIME` | Limit extraction to a time range |
| `--fmt <jpg\|png>` | Output format (default: jpg) |
| `--quality <1-100>` | JPEG quality (default: 95) |
| `--max <N>` | Safety cap on frame count (default: 500, 0 = off) |
| `-o`, `--output DIR` | Output folder |
| `-r`, `--recursive` | Search sub-folders when given a folder |
| `--batch` | Skip all confirmation prompts |
| `--info <video>` | Show video info, then exit |
| `-v`, `--version` / `-h`, `--help` | Show version / help |

</details>

---

## 📜 License

Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details.

<div align="center">

## 👋 About the Author

**Diea Abdeltwab** — Data Engineer · Software Engineer · Cairo, Egypt 🇪🇬

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/diea-abdeltwab/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/diea-abdeltwab)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:dieaabdeltwab2024@gmail.com)

⭐ **If this saved you time, a star means a lot!**

</div>
