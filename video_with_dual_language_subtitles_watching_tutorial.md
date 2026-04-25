# macOS：本地视频挂载中英双语字幕观看教程

> 适用场景：你已经**合法获得/自己拥有授权**的视频文件，并且已经拥有对应的中文字幕和英文字幕文件，希望在 macOS 上实现本地播放、自动加载字幕、中英双语同时显示，或把字幕封装进一个 MKV 文件中。  
>  
> 本教程不针对任何特定网站、账号、课程或个人路径，避免泄漏隐私。请不要下载、传播、破解、绕过 DRM 或使用未授权的版权内容。

---

## 目录

- [1. 最终目标](#1-最终目标)
- [2. 准备工具](#2-准备工具)
- [3. 建立统一文件夹结构](#3-建立统一文件夹结构)
- [4. 合法获取视频与字幕](#4-合法获取视频与字幕)
- [5. 统一整理视频和字幕文件名](#5-统一整理视频和字幕文件名)
- [6. 用 VLC 或 IINA 播放外挂字幕](#6-用-vlc-或-iina-播放外挂字幕)
- [7. 合并中英字幕为双语字幕](#7-合并中英字幕为双语字幕)
- [8. 让播放器自动加载双语字幕](#8-让播放器自动加载双语字幕)
- [9. 可选：封装成内置字幕 MKV](#9-可选封装成内置字幕-mkv)
- [10. 可选：做一个桌面快捷入口](#10-可选做一个桌面快捷入口)
- [11. 常见问题排查](#11-常见问题排查)

---

## 1. 最终目标

整理完成后，希望得到这样的结构：

```text
video_subtitle_project/
├── videos/          # 原始视频
├── subs_zh/         # 原始中文字幕
├── subs_en/         # 原始英文字幕
├── watch/           # 整理后用于观看的文件
└── mkv/             # 可选：封装后的 MKV 文件
```

`watch/` 文件夹中每一集应类似这样：

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

其中：

- `01.mp4`：第 1 集视频
- `01.zh-CN.srt`：第 1 集中文字幕
- `01.en.srt`：第 1 集英文字幕
- `01.bilingual.srt`：中英合并后的双语字幕
- `01.srt`：复制出来的默认双语字幕，方便 VLC/IINA 自动加载

---

## 2. 准备工具

### 2.1 安装 Homebrew

如果你的 Mac 已经安装过 Homebrew，可以跳过。

在终端执行：

```bash
brew --version
```

如果能显示版本号，说明已经安装。

如果没有安装，请到 Homebrew 官网查看最新安装方式：

```text
https://brew.sh/
```

---

### 2.2 安装播放器

推荐安装 IINA 或 VLC。

```bash
brew install --cask iina
brew install --cask vlc
```

说明：

- IINA：更符合 macOS 使用习惯
- VLC：兼容性强，适合测试各种视频和字幕

---

### 2.3 安装 FFmpeg

FFmpeg 用于把字幕封装进 MKV 文件。

```bash
brew install ffmpeg
```

检查是否安装成功：

```bash
ffmpeg -version
```

---

### 2.4 可选：安装 yt-dlp

如果你需要从自己有权访问的平台下载视频，可以使用 `yt-dlp`。请只用于合法授权内容。

```bash
python3 -m pip install -U yt-dlp
```

检查是否安装成功：

```bash
python3 -m yt_dlp --version
```

如果提示 Python 版本过旧，可以安装新版 Python：

```bash
brew install python
```

---

## 3. 建立统一文件夹结构

在终端执行：

```bash
mkdir -p ~/Desktop/video_subtitle_project/videos
mkdir -p ~/Desktop/video_subtitle_project/subs_zh
mkdir -p ~/Desktop/video_subtitle_project/subs_en
mkdir -p ~/Desktop/video_subtitle_project/watch
mkdir -p ~/Desktop/video_subtitle_project/mkv
```

然后手动放入文件：

```text
视频文件         → videos/
中文字幕文件     → subs_zh/
英文字幕文件     → subs_en/
```

例如：

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

## 4. 合法获取视频与字幕

### 4.1 获取视频

视频来源可以是：

- 自己录制的视频
- 课程官方允许下载的视频
- 已获得授权的公开视频
- 平台明确允许离线保存的内容

如果使用 `yt-dlp`，通用命令示例：

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

说明：

- `VIDEO_URL` 替换为你有权下载的视频页面链接
- `--cookies-from-browser chrome` 用于读取浏览器登录状态
- 如果不需要登录，可以去掉 `--cookies-from-browser chrome`
- `--playlist-items 1-25` 表示下载第 1 到第 25 个视频，可按实际情况修改

如果遇到下载失败：

```bash
unset http_proxy
unset https_proxy
unset all_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
unset ALL_PROXY
```

然后重新尝试。很多平台对代理、异常 IP、频繁请求比较敏感。

---

### 4.2 获取字幕

你可以使用已有字幕，也可以下载平台提供的字幕。

查看某个视频有哪些字幕：

```bash
python3 -m yt_dlp \
  --cookies-from-browser chrome \
  --list-subs \
  "VIDEO_URL"
```

只下载字幕，不下载视频：

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

注意：

- 不一定每个视频都有中英文字幕
- 自动字幕可能有错误
- 没有英文字幕时，需要自己翻译或使用语音识别工具生成
- 没有中文字幕时，需要自己翻译或生成

---

## 5. 统一整理视频和字幕文件名

如果视频和字幕名字不完全一致，播放器通常不会自动识别。  
推荐不要手动改名，而是用脚本复制到 `watch/` 文件夹，并统一命名。

### 5.1 检查数量

```bash
cd ~/Desktop/video_subtitle_project

echo "视频数量："
find videos -maxdepth 1 -type f | wc -l

echo "中文字幕数量："
find subs_zh -maxdepth 1 -type f | wc -l

echo "英文字幕数量："
find subs_en -maxdepth 1 -type f | wc -l
```

如果是 25 集，理想结果应该接近：

```text
25
25
25
```

---

### 5.2 创建整理脚本

创建 `organize_files.py`：

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

# 修改这里：课程/视频总数
TOTAL_EPISODES = 25

def extract_episode_number(filename: str):
    """
    尽量从文件名中提取集数编号。
    支持：
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
                print(f"警告：第 {n:02d} 集出现重复文件：")
                print(f"  已有：{result[n].name}")
                print(f"  新的：{file.name}")
            else:
                result[n] = file

    return result, unknown

videos, unknown_videos = collect(VIDEO_DIR, VIDEO_EXTS)
zh_subs, unknown_zh = collect(ZH_DIR, SUB_EXTS)
en_subs, unknown_en = collect(EN_DIR, SUB_EXTS)

print("\n========== 自动识别结果 ==========\n")

rows = []

for i in range(1, TOTAL_EPISODES + 1):
    v = videos.get(i)
    z = zh_subs.get(i)
    e = en_subs.get(i)

    status = "OK" if v and z and e else "MISSING"

    print(f"第 {i:02d} 集：{status}")
    print(f"  视频：{v.name if v else '缺失'}")
    print(f"  中文：{z.name if z else '缺失'}")
    print(f"  英文：{e.name if e else '缺失'}")
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

print("已生成匹配预览表：", csv_path)

if unknown_videos or unknown_zh or unknown_en:
    print("\n========== 无法识别编号的文件 ==========")

    if unknown_videos:
        print("\n无法识别编号的视频：")
        for f in unknown_videos:
            print("  ", f.name)

    if unknown_zh:
        print("\n无法识别编号的中文字幕：")
        for f in unknown_zh:
            print("  ", f.name)

    if unknown_en:
        print("\n无法识别编号的英文字幕：")
        for f in unknown_en:
            print("  ", f.name)

print("\n请先打开 match_preview.csv 检查匹配是否正确。")
print("确认正确后，再运行：")
print("python3 organize_files.py --copy")

if "--copy" not in sys.argv:
    raise SystemExit

print("\n========== 开始复制并统一命名 ==========\n")

for i in range(1, TOTAL_EPISODES + 1):
    v = videos.get(i)
    z = zh_subs.get(i)
    e = en_subs.get(i)

    if not (v and z and e):
        print(f"跳过第 {i:02d} 集，因为文件不完整。")
        continue

    nn = f"{i:02d}"

    shutil.copy2(v, WATCH_DIR / f"{nn}{v.suffix.lower()}")
    shutil.copy2(z, WATCH_DIR / f"{nn}.zh-CN{z.suffix.lower()}")
    shutil.copy2(e, WATCH_DIR / f"{nn}.en{e.suffix.lower()}")

    print(f"第 {nn} 集已整理完成")

print("\n全部完成。整理后的文件在：", WATCH_DIR)
PY
```

---

### 5.3 先预览匹配结果

```bash
python3 organize_files.py
```

脚本会生成：

```text
match_preview.csv
```

请打开这个 CSV，确认每一集的视频、中文字幕、英文字幕匹配正确。

---

### 5.4 确认无误后正式复制

```bash
python3 organize_files.py --copy
```

检查结果：

```bash
ls ~/Desktop/video_subtitle_project/watch | head -n 30
```

应类似：

```text
01.mp4
01.zh-CN.srt
01.en.srt
02.mp4
02.zh-CN.srt
02.en.srt
```

---

## 6. 用 VLC 或 IINA 播放外挂字幕

打开 `watch/` 文件夹：

```bash
open ~/Desktop/video_subtitle_project/watch
```

用 IINA 打开：

```bash
open -a IINA ~/Desktop/video_subtitle_project/watch/01.mp4
```

用 VLC 打开：

```bash
open -a VLC ~/Desktop/video_subtitle_project/watch/01.mp4
```

如果字幕没有自动出现：

### IINA

```text
菜单栏 → 字幕 → 打开字幕文件
```

选择：

```text
01.zh-CN.srt
```

或：

```text
01.en.srt
```

### VLC

```text
Subtitles → Add Subtitle File...
```

选择：

```text
01.zh-CN.srt
```

或：

```text
01.en.srt
```

注意：VLC/IINA 通常一次只显示一个字幕轨。  
如果想让中英文同时显示，需要把两个字幕合并成一个双语字幕文件。

---

## 7. 合并中英字幕为双语字幕

### 7.1 创建双语字幕合并脚本

创建 `merge_bilingual_srt.py`：

```bash
cd ~/Desktop/video_subtitle_project

cat > merge_bilingual_srt.py <<'PY'
from pathlib import Path
import re

ROOT = Path.home() / "Desktop" / "video_subtitle_project"
WATCH = ROOT / "watch"

# 修改这里：视频总集数
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
        print(f"缺少中文字幕：{zh_path.name}")
        return

    if not en_path.exists():
        print(f"缺少英文字幕：{en_path.name}")
        return

    zh = parse_srt(zh_path)
    en = parse_srt(en_path)

    n = min(len(zh), len(en))

    if len(zh) != len(en):
        print(f"警告：第 {nn} 集中英文字幕条数不同：中文 {len(zh)} 条，英文 {len(en)} 条，将按较短数量合并")

    lines = []

    for idx in range(n):
        # 默认使用英文字幕的时间轴。
        # 如果你的中文字幕时间轴更准确，可以改为：time_line = zh[idx]["time"]
        time_line = en[idx]["time"]

        en_text = " ".join(en[idx]["text"]).strip()
        zh_text = " ".join(zh[idx]["text"]).strip()

        lines.append(str(idx + 1))
        lines.append(time_line)

        # 上英文，下中文
        if en_text:
            lines.append(en_text)
        if zh_text:
            lines.append(zh_text)

        lines.append("")

    out_path.write_text("\n".join(lines), encoding="utf-8")
    print(f"已生成：{out_path.name}")

for i in range(1, TOTAL_EPISODES + 1):
    merge_one(i)

print("\n全部完成。双语字幕已生成在 watch 文件夹中。")
PY
```

---

### 7.2 运行脚本

```bash
python3 merge_bilingual_srt.py
```

生成：

```text
01.bilingual.srt
02.bilingual.srt
...
25.bilingual.srt
```

---

### 7.3 测试双语字幕

用 VLC 打开第 1 集：

```bash
open -a VLC ~/Desktop/video_subtitle_project/watch/01.mp4
```

然后：

```text
Subtitles → Add Subtitle File...
```

选择：

```text
01.bilingual.srt
```

如果显示为：

```text
English sentence
中文字幕
```

说明双语字幕合并成功。

---

## 8. 让播放器自动加载双语字幕

很多播放器会自动加载与视频同名的 `.srt` 文件。

因此可以把：

```text
01.bilingual.srt
```

复制成：

```text
01.srt
```

单集测试：

```bash
cd ~/Desktop/video_subtitle_project/watch

cp 01.bilingual.srt 01.srt
open -a VLC 01.mp4
```

如果第 1 集成功自动显示双语字幕，再批量复制：

```bash
cd ~/Desktop/video_subtitle_project/watch

for i in $(seq -w 1 25); do
  cp "${i}.bilingual.srt" "${i}.srt"
done
```

之后结构会类似：

```text
01.mp4
01.srt
01.zh-CN.srt
01.en.srt
01.bilingual.srt
```

其中 `01.srt` 是默认自动加载字幕。

---

## 9. 可选：封装成内置字幕 MKV

如果不想依赖外挂字幕，可以把视频、中文字幕、英文字幕封装到一个 MKV 文件里。

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
    -metadata:s:s:0 title="中文" \
    -metadata:s:s:1 language=eng \
    -metadata:s:s:1 title="English" \
    "mkv/${i}.with-subtitles.mkv"
done
```

生成：

```text
mkv/
├── 01.with-subtitles.mkv
├── 02.with-subtitles.mkv
└── ...
```

说明：

- 这种方式是“软封装”，字幕可以打开、关闭、切换
- 不会把字幕烧死在画面上
- 视频和音频使用 `copy`，通常不会重新编码，速度较快

如果想把双语字幕也封装进去：

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
    -metadata:s:s:0 title="中英双语" \
    "mkv/${i}.bilingual.mkv"
done
```

---

## 10. 可选：做一个桌面快捷入口

如果不想每次打开终端，可以创建一个桌面快捷脚本。

### 10.1 一键打开课程文件夹

```bash
cat > ~/Desktop/打开本地视频课程.command <<'SH'
#!/bin/zsh
open "$HOME/Desktop/video_subtitle_project/watch"
SH

chmod +x ~/Desktop/打开本地视频课程.command
```

以后双击桌面上的：

```text
打开本地视频课程.command
```

即可打开 `watch/` 文件夹。

---

### 10.2 一键播放第 1 集

```bash
cat > ~/Desktop/播放第1集.command <<'SH'
#!/bin/zsh
open -a VLC "$HOME/Desktop/video_subtitle_project/watch/01.mp4"
SH

chmod +x ~/Desktop/播放第1集.command
```

如果你更喜欢 IINA，把 `VLC` 改成 `IINA`：

```bash
open -a IINA "$HOME/Desktop/video_subtitle_project/watch/01.mp4"
```

---

## 11. 常见问题排查

### 11.1 为什么我打开视频没有字幕？

常见原因：

1. 你打开的是 `videos/` 里的原始视频，而不是 `watch/` 里的整理后视频
2. 字幕文件名和视频文件名不对应
3. 使用了 macOS 自带 QuickTime，外挂字幕支持较弱
4. 播放器没有自动加载字幕，需要手动添加

建议：

```bash
open ~/Desktop/video_subtitle_project/watch
```

然后用 VLC 或 IINA 打开：

```text
01.mp4
```

确保旁边有：

```text
01.srt
01.zh-CN.srt
01.en.srt
```

---

### 11.2 为什么只有英文字幕，没有中文字幕？

因为播放器当前只选择了英文字幕轨。  
如果想同时显示中英，需要加载：

```text
01.bilingual.srt
```

或默认字幕：

```text
01.srt
```

---

### 11.3 VLC 能不能像下载工具一样下载在线视频？

不建议这么做。  
VLC 主要用于播放本地文件和加载字幕，不适合作为批量下载工具。

推荐分工：

```text
下载阶段：合法授权前提下使用 yt-dlp
整理阶段：用脚本统一命名
观看阶段：使用 VLC 或 IINA
长期保存：可选封装为 MKV
```

---

### 11.4 遇到 HTTP 412、403、无法下载怎么办？

这通常与平台风控、代理、IP、登录状态、请求频率有关。

可尝试：

```bash
unset http_proxy
unset https_proxy
unset all_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
unset ALL_PROXY
```

并确认：

- 浏览器已经登录对应平台账号
- 不使用异常代理或机房节点
- 不频繁重复请求
- 等几分钟后再试
- 只下载自己有权访问和保存的内容

---

### 11.5 字幕时间轴对不上怎么办？

可以使用播放器临时调整字幕延迟：

- VLC：`G` / `H` 调整字幕延迟
- IINA：菜单栏里调整字幕延迟

如果要永久修正，可以用 Aegisub 或 Subtitle Edit 修改字幕时间轴。

---

### 11.6 字幕乱码怎么办？

优先确认字幕是 UTF-8 编码。  
如果乱码，可以尝试用编辑器重新保存为 UTF-8。

macOS 命令行可以查看文件编码：

```bash
file 01.zh-CN.srt
```

---

## 推荐工作流总结

```text
1. 合法获得视频和中英文字幕
2. 放入 videos / subs_zh / subs_en
3. 运行 organize_files.py 预览匹配
4. 确认 match_preview.csv 正确
5. 运行 organize_files.py --copy
6. 用 VLC/IINA 测试单集外挂字幕
7. 运行 merge_bilingual_srt.py 生成双语字幕
8. 把 bilingual.srt 复制为同名 .srt
9. 双击视频即可自动显示双语字幕
10. 可选：用 FFmpeg 封装成 MKV
```

---

## 注意事项

- 不要把个人账号 cookies、下载链接中带有个人追踪参数的 URL、真实用户名路径提交到 GitHub
- 不要提交受版权保护的视频文件
- 不要提交未经授权的课程资源
- 如果要开源，只建议提交：
  - 教程文档
  - 脚本模板
  - 空文件夹结构
  - 示例文件名
- 字幕和视频是否可以公开发布，取决于原始内容的授权协议
