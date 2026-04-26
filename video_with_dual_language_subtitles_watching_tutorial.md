
# macOS Local Video Bilingual Subtitle Guide

A practical guide for watching local videos with Chinese-English bilingual subtitles on macOS.

This project shows how to:

- Organize local video and subtitle files
- Automatically match videos with Chinese and English subtitles
- Merge Chinese and English `.srt` files into one bilingual subtitle file
- Make VLC or IINA automatically load subtitles
- Optionally package subtitles into MKV files using FFmpeg

> This guide is intended only for videos and subtitles that you legally own or are authorized to use.  
> Do not download, distribute, crack, bypass DRM, or share unauthorized copyrighted content.

---

## 1. Project Goal

After setup, your folder structure should look like this:

```text
video_subtitle_project/
├── videos/          # Original video files
├── subs_zh/         # Original Chinese subtitles
├── subs_en/         # Original English subtitles
├── watch/           # Organized files for watching
└── mkv/             # Optional packaged MKV files
````

Each episode in the `watch/` folder should look like this:

```text
01.mp4
01.zh-CN.srt
01.en.srt
01.bilingual.srt
01.srt

02.mp4
02.zh-CN.srt
02.en.srt
02.bilingual.srt
02.srt
```

Explanation:

* `01.mp4`: Episode 1 video
* `01.zh-CN.srt`: Chinese subtitle
* `01.en.srt`: English subtitle
* `01.bilingual.srt`: merged bilingual subtitle
* `01.srt`: default subtitle file for automatic loading

---

## 2. Requirements

### Homebrew

Check whether Homebrew is installed:

```bash
brew --version
```

If not installed, visit:

```text
https://brew.sh/
```

---

### IINA or VLC

```bash
brew install --cask iina
brew install --cask vlc
```

Recommended usage:

* **IINA**: better macOS user experience
* **VLC**: stronger compatibility for testing videos and subtitles

---

### FFmpeg

```bash
brew install ffmpeg
```

Check installation:

```bash
ffmpeg -version
```

---

### Optional: yt-dlp

Only use `yt-dlp` for content you are legally authorized to access and save.

```bash
python3 -m pip install -U yt-dlp
```

Check installation:

```bash
python3 -m yt_dlp --version
```

If Python is too old:

```bash
brew install python
```

---

## 3. Create Folder Structure

Run:

```bash
mkdir -p ~/Desktop/video_subtitle_project/videos
mkdir -p ~/Desktop/video_subtitle_project/subs_zh
mkdir -p ~/Desktop/video_subtitle_project/subs_en
mkdir -p ~/Desktop/video_subtitle_project/watch
mkdir -p ~/Desktop/video_subtitle_project/mkv
```

Then place your files manually:

```text
Video files          → videos/
Chinese subtitles    → subs_zh/
English subtitles    → subs_en/
```

Example:

```text
videos/
├── Lecture 01 xxx.mp4
├── Lecture 02 xxx.mp4
└── ...

subs_zh/
├── Lecture 1 Chinese.srt
├── Lecture 2 Chinese.srt
└── ...

subs_en/
├── Lecture 1 English.srt
├── Lecture 2 English.srt
└── ...
```

---

## 4. Legally Obtain Videos and Subtitles

### Download videos

Example command:

```bash
cd ~/Desktop/video_subtitle_project

python3 -m yt_dlp \
  --cookies-from-browser chrome \
  --yes-playlist \
  --playlist-items 1-25 \
  -f "bv*+ba/b" \
  --merge-output-format mp4 \
  -o "videos/%(playlist_index)02d - %(title).100s.%(ext)s" \
  "VIDEO_URL"
```

Notes:

* Replace `VIDEO_URL` with your authorized video URL.
* `--cookies-from-browser chrome` uses your Chrome login session.
* If login is not required, remove `--cookies-from-browser chrome`.
* `--playlist-items 1-25` downloads items 1 to 25.

If download fails, try clearing proxy variables:

```bash
unset http_proxy
unset https_proxy
unset all_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
unset ALL_PROXY
```

---

### Download subtitles

Check available subtitles:

```bash
python3 -m yt_dlp \
  --cookies-from-browser chrome \
  --list-subs \
  "VIDEO_URL"
```

Download subtitles only:

```bash
python3 -m yt_dlp \
  --cookies-from-browser chrome \
  --skip-download \
  --write-subs \
  --write-auto-subs \
  --sub-langs "zh.*,en.*,zh-CN,en-US" \
  --convert-subs srt \
  -o "subs/%(playlist_index)02d - %(title).100s.%(ext)s" \
  "VIDEO_URL"
```

Notes:

* Not every video has both Chinese and English subtitles.
* Auto-generated subtitles may contain errors.
* If a subtitle is missing, you may need to translate or generate it yourself.

---

## 5. Standardize Video and Subtitle Filenames

If video and subtitle filenames do not match, players usually cannot load subtitles automatically.

Instead of renaming manually, use the script below.

---

### Check file counts

```bash
cd ~/Desktop/video_subtitle_project

echo "Number of videos:"
find videos -maxdepth 1 -type f | wc -l

echo "Number of Chinese subtitles:"
find subs_zh -maxdepth 1 -type f | wc -l

echo "Number of English subtitles:"
find subs_en -maxdepth 1 -type f | wc -l
```

For 25 episodes, the ideal result should be close to:

```text
25
25
25
```

---

## 6. Create `organize_files.py`

Create the script:

```bash
cd ~/Desktop/video_subtitle_project

cat > organize_files.py <<'PY'
from pathlib import Path
import re
import shutil
import csv
import sys

ROOT = Path.home() / "Desktop" / "video_subtitle_project"

VIDEO_DIR = ROOT / "videos"
ZH_DIR = ROOT / "subs_zh"
EN_DIR = ROOT / "subs_en"
WATCH_DIR = ROOT / "watch"

WATCH_DIR.mkdir(parents=True, exist_ok=True)

VIDEO_EXTS = {".mp4", ".mkv", ".webm", ".mov"}
SUB_EXTS = {".srt", ".ass", ".vtt"}

# Change this according to your project
TOTAL_EPISODES = 25

def extract_episode_number(filename: str):
    """
    Try to extract the episode number from a filename.
    Supports:
    01 - xxx.mp4
    Lecture 1 xxx.srt
    Lec 01 xxx.srt
    第1讲 xxx.srt
    1.xxx.srt
    """
    patterns = [
        r'^(?P<n>\d{1,3})\s*[-_. ]',
        r'\b[Ll]ecture\s*(?P<n>\d{1,3})\b',
        r'\b[Ll]ec\s*(?P<n>\d{1,3})\b',
        r'第\s*(?P<n>\d{1,3})\s*[讲集课节]',
        r'\b(?P<n>\d{1,3})\b',
    ]

    for pattern in patterns:
        m = re.search(pattern, filename)
        if m:
            n = int(m.group("n"))
            if 1 <= n <= TOTAL_EPISODES:
                return n

    return None

def collect(folder: Path, exts):
    result = {}
    unknown = []

    for file in sorted(folder.iterdir()):
        if not file.is_file():
            continue
        if file.suffix.lower() not in exts:
            continue

        n = extract_episode_number(file.name)

        if n is None:
            unknown.append(file)
        else:
            if n in result:
                print(f"Warning: duplicate files found for Episode {n:02d}:")
                print(f"  Existing: {result[n].name}")
                print(f"  New: {file.name}")
            else:
                result[n] = file

    return result, unknown

videos, unknown_videos = collect(VIDEO_DIR, VIDEO_EXTS)
zh_subs, unknown_zh = collect(ZH_DIR, SUB_EXTS)
en_subs, unknown_en = collect(EN_DIR, SUB_EXTS)

print("\n========== Automatic Matching Result ==========\n")

rows = []

for i in range(1, TOTAL_EPISODES + 1):
    v = videos.get(i)
    z = zh_subs.get(i)
    e = en_subs.get(i)

    status = "OK" if v and z and e else "MISSING"

    print(f"Episode {i:02d}: {status}")
    print(f"  Video: {v.name if v else 'Missing'}")
    print(f"  Chinese: {z.name if z else 'Missing'}")
    print(f"  English: {e.name if e else 'Missing'}")
    print()

    rows.append([
        f"{i:02d}",
        v.name if v else "",
        z.name if z else "",
        e.name if e else "",
        status
    ])

csv_path = ROOT / "match_preview.csv"
with open(csv_path, "w", newline="", encoding="utf-8-sig") as f:
    writer = csv.writer(f)
    writer.writerow(["episode", "video", "zh_subtitle", "en_subtitle", "status"])
    writer.writerows(rows)

print("Generated matching preview table:", csv_path)

if unknown_videos or unknown_zh or unknown_en:
    print("\n========== Files Whose Episode Numbers Could Not Be Recognized ==========")

    if unknown_videos:
        print("\nVideos with unrecognized episode numbers:")
        for f in unknown_videos:
            print("  ", f.name)

    if unknown_zh:
        print("\nChinese subtitles with unrecognized episode numbers:")
        for f in unknown_zh:
            print("  ", f.name)

    if unknown_en:
        print("\nEnglish subtitles with unrecognized episode numbers:")
        for f in unknown_en:
            print("  ", f.name)

print("\nPlease open match_preview.csv first and check whether the matches are correct.")
print("After confirming everything is correct, run:")
print("python3 organize_files.py --copy")

if "--copy" not in sys.argv:
    raise SystemExit

print("\n========== Start copying and renaming files ==========\n")

for i in range(1, TOTAL_EPISODES + 1):
    v = videos.get(i)
    z = zh_subs.get(i)
    e = en_subs.get(i)

    if not (v and z and e):
        print(f"Skipping Episode {i:02d} because the files are incomplete.")
        continue

    nn = f"{i:02d}"

    shutil.copy2(v, WATCH_DIR / f"{nn}{v.suffix.lower()}")
    shutil.copy2(z, WATCH_DIR / f"{nn}.zh-CN{z.suffix.lower()}")
    shutil.copy2(e, WATCH_DIR / f"{nn}.en{e.suffix.lower()}")

    print(f"Episode {nn} organized successfully.")

print("\nAll done. The organized files are in:", WATCH_DIR)
PY
```

Preview the matching result:

```bash
python3 organize_files.py
```

After checking `match_preview.csv`, copy and rename files:

```bash
python3 organize_files.py --copy
```

Check the result:

```bash
ls ~/Desktop/video_subtitle_project/watch | head -n 30
```

---

## 7. Play External Subtitles

Open the `watch/` folder:

```bash
open ~/Desktop/video_subtitle_project/watch
```

Open with IINA:

```bash
open -a IINA ~/Desktop/video_subtitle_project/watch/01.mp4
```

Open with VLC:

```bash
open -a VLC ~/Desktop/video_subtitle_project/watch/01.mp4
```

If subtitles do not appear automatically:

### IINA

```text
Menu Bar → Subtitles → Open Subtitle File
```

Choose:

```text
01.zh-CN.srt
```

or:

```text
01.en.srt
```

### VLC

```text
Subtitles → Add Subtitle File...
```

Choose:

```text
01.zh-CN.srt
```

or:

```text
01.en.srt
```

If you want Chinese and English subtitles to appear at the same time, merge them into one bilingual subtitle file.

---

## 8. Create `merge_bilingual_srt.py`

```bash
cd ~/Desktop/video_subtitle_project

cat > merge_bilingual_srt.py <<'PY'
from pathlib import Path
import re

ROOT = Path.home() / "Desktop" / "video_subtitle_project"
WATCH = ROOT / "watch"

# Change this according to your project
TOTAL_EPISODES = 25

def parse_srt(path: Path):
    text = path.read_text(encoding="utf-8-sig", errors="replace")
    blocks = re.split(r"\n\s*\n", text.strip())
    result = []

    for block in blocks:
        lines = block.strip().splitlines()
        if len(lines) < 2:
            continue

        time_idx = None
        for i, line in enumerate(lines):
            if "-->" in line:
                time_idx = i
                break

        if time_idx is None:
            continue

        time_line = lines[time_idx].strip()
        content = lines[time_idx + 1:]
        content = [line.strip() for line in content if line.strip()]

        result.append({
            "time": time_line,
            "text": content
        })

    return result

def merge_one(i: int):
    nn = f"{i:02d}"
    zh_path = WATCH / f"{nn}.zh-CN.srt"
    en_path = WATCH / f"{nn}.en.srt"
    out_path = WATCH / f"{nn}.bilingual.srt"

    if not zh_path.exists():
        print(f"Missing Chinese subtitle: {zh_path.name}")
        return

    if not en_path.exists():
        print(f"Missing English subtitle: {en_path.name}")
        return

    zh = parse_srt(zh_path)
    en = parse_srt(en_path)

    n = min(len(zh), len(en))

    if len(zh) != len(en):
        print(f"Warning: Episode {nn} has different numbers of subtitle entries: Chinese {len(zh)}, English {len(en)}. Merging according to the smaller number.")

    lines = []

    for idx in range(n):
        # Use the English subtitle timeline by default.
        # If the Chinese subtitle timeline is more accurate, change this to:
        # time_line = zh[idx]["time"]
        time_line = en[idx]["time"]

        en_text = " ".join(en[idx]["text"]).strip()
        zh_text = " ".join(zh[idx]["text"]).strip()

        lines.append(str(idx + 1))
        lines.append(time_line)

        # English above, Chinese below
        if en_text:
            lines.append(en_text)
        if zh_text:
            lines.append(zh_text)

        lines.append("")

    out_path.write_text("\n".join(lines), encoding="utf-8")
    print(f"Generated: {out_path.name}")

for i in range(1, TOTAL_EPISODES + 1):
    merge_one(i)

print("\nAll done. Bilingual subtitles have been generated in the watch folder.")
PY
```

Run:

```bash
python3 merge_bilingual_srt.py
```

This generates:

```text
01.bilingual.srt
02.bilingual.srt
...
25.bilingual.srt
```

---

## 9. Make Bilingual Subtitles Load Automatically

Many players automatically load an `.srt` file with the same name as the video.

Test with one episode:

```bash
cd ~/Desktop/video_subtitle_project/watch

cp 01.bilingual.srt 01.srt
open -a VLC 01.mp4
```

If it works, batch-copy all bilingual subtitles:

```bash
cd ~/Desktop/video_subtitle_project/watch

for i in $(seq -w 1 25); do
  cp "${i}.bilingual.srt" "${i}.srt"
done
```

After this, each episode should have:

```text
01.mp4
01.srt
01.zh-CN.srt
01.en.srt
01.bilingual.srt
```

---

## 10. Optional: Package Subtitles into MKV

Package Chinese and English subtitle tracks into MKV:

```bash
cd ~/Desktop/video_subtitle_project

for i in $(seq -w 1 25); do
  ffmpeg -y \
    -i "watch/${i}.mp4" \
    -i "watch/${i}.zh-CN.srt" \
    -i "watch/${i}.en.srt" \
    -map 0 -map 1 -map 2 \
    -c:v copy \
    -c:a copy \
    -c:s srt \
    -metadata:s:s:0 language=zho \
    -metadata:s:s:0 title="Chinese" \
    -metadata:s:s:1 language=eng \
    -metadata:s:s:1 title="English" \
    "mkv/${i}.with-subtitles.mkv"
done
```

Package bilingual subtitle into MKV:

```bash
cd ~/Desktop/video_subtitle_project

for i in $(seq -w 1 25); do
  ffmpeg -y \
    -i "watch/${i}.mp4" \
    -i "watch/${i}.bilingual.srt" \
    -map 0 -map 1 \
    -c:v copy \
    -c:a copy \
    -c:s srt \
    -metadata:s:s:0 language=zho \
    -metadata:s:s:0 title="Chinese-English Bilingual" \
    "mkv/${i}.bilingual.mkv"
done
```

Notes:

* This is soft packaging.
* Subtitles can still be turned on, turned off, or switched.
* Subtitles are not burned into the video.
* Video and audio are copied without re-encoding.

---

## 11. Optional: Desktop Shortcuts

### Open the watch folder

```bash
cat > ~/Desktop/Open_Local_Video_Course.command <<'SH'
#!/bin/zsh
open "$HOME/Desktop/video_subtitle_project/watch"
SH

chmod +x ~/Desktop/Open_Local_Video_Course.command
```

Double-click:

```text
Open_Local_Video_Course.command
```

---

### Play Episode 1

```bash
cat > ~/Desktop/Play_Episode_1.command <<'SH'
#!/bin/zsh
open -a VLC "$HOME/Desktop/video_subtitle_project/watch/01.mp4"
SH

chmod +x ~/Desktop/Play_Episode_1.command
```

If you prefer IINA, replace `VLC` with `IINA`:

```bash
open -a IINA "$HOME/Desktop/video_subtitle_project/watch/01.mp4"
```

---

## 12. Troubleshooting

### No subtitles appear

Possible reasons:

1. You opened the original video in `videos/`, not the organized video in `watch/`.
2. Subtitle filenames do not match the video filenames.
3. You are using QuickTime, which has weak external subtitle support.
4. The player did not automatically load subtitles.

Recommended:

```bash
open ~/Desktop/video_subtitle_project/watch
```

Then open `01.mp4` with VLC or IINA.

Make sure these files are next to it:

```text
01.srt
01.zh-CN.srt
01.en.srt
```

---

### Only English subtitles appear

The player is probably only using the English subtitle track.

To display both languages, load:

```text
01.bilingual.srt
```

or:

```text
01.srt
```

---

### VLC cannot download online videos

VLC is mainly for local playback and subtitle loading.

Recommended workflow:

```text
Download stage: use yt-dlp only when legally authorized
Organization stage: use scripts to standardize filenames
Viewing stage: use VLC or IINA
Long-term storage: optionally package files as MKV
```

---

### HTTP 412, 403, or download failure

This may be caused by platform risk control, proxy settings, login status, IP issues, or frequent requests.

Try:

```bash
unset http_proxy
unset https_proxy
unset all_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
unset ALL_PROXY
```

Also check:

* You are logged in to the platform in your browser.
* You are not using an unusual proxy or data-center IP.
* You are not sending requests too frequently.
* You only download content you are authorized to access and save.

---

### Subtitle timing is not aligned

Temporary adjustment:

* VLC: press `G` / `H`
* IINA: adjust subtitle delay from the menu bar

Permanent adjustment:

* Use Aegisub or Subtitle Edit to edit the subtitle timeline.

---

### Subtitles are garbled

Make sure the subtitle file is encoded in UTF-8.

Check encoding:

```bash
file 01.zh-CN.srt
```

If garbled, reopen the subtitle file in a text editor and save it as UTF-8.

---

## 13. Recommended Workflow

```text
1. Legally obtain video files and Chinese/English subtitles.
2. Put them into videos / subs_zh / subs_en.
3. Run organize_files.py to preview matching.
4. Check match_preview.csv.
5. Run organize_files.py --copy.
6. Test external subtitles with VLC or IINA.
7. Run merge_bilingual_srt.py.
8. Copy bilingual subtitles as same-name .srt files.
9. Open the video and automatically display bilingual subtitles.
10. Optional: package subtitles into MKV files.
```

---

## 14. Important Notes

Do not upload the following to GitHub:

* personal account cookies
* video URLs containing personal tracking parameters
* real username paths
* copyrighted video files
* unauthorized course materials
* subtitle files that you do not have permission to share

If you want to open-source this project, it is recommended to upload only:

* tutorial documents
* script templates
* empty folder structures
* example filenames

Whether videos or subtitles can be publicly shared depends on the license of the original content.

---

## License

This project is for educational and personal study purposes only.

Please follow the license terms of all original video and subtitle materials.
