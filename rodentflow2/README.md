# RodentFlow 2.0 项目页 · 使用说明

整个文件夹就是一个页面，直接放进你的 `seesaw-earth.github.io` 仓库即可，不需要任何构建工具。

```
rodentflow2/
├── index.html            页面（样式和脚本都在里面）
├── assets/               网站小图标
├── videos/               转码后的视频放这里（每个视频 3 个文件）
└── tools/
    ├── encode-all.sh     一条命令把 8 个录屏全部转好
    └── encode.sh         转单个视频
```

## 1. 视频区是怎么工作的

标题下面就是视频区：左边是说明文字，右边是一个设备外框在播放，下面一排缩略图。

- **默认自动播放**：一个视频播完自动换下一个，缩略图条跟着滑过去，当前那张有进度条。短于 7 秒的视频会重复播放到至少 7 秒再换（在 `index.html` 里改 `MIN_SECONDS_PER_CLIP`）。
- **用户手动选**：点任意缩略图 → 播放那个视频并一直循环，自动切换关闭；控制条上的 `Auto-advance` 可以再打开。缩略图条可以手指滑、鼠标拖、键盘左右键。
- **看大图**：点视频本身或 ⤢，打开带进度条的大播放器。
- **iPhone 录屏自动套 iPhone 外框，Mac 录屏自动套 MacBook 外框**，切换时外框会变形过去；设备背后的光晕颜色取自视频画面。
- 每个视频旁边显示：用的是哪种追踪（颜色和页面其他部分一致）、测的是哪些指标、视频里的彩色点分别代表什么。

## 2. 加入你的视频（三步）

**① 装 ffmpeg**（只需一次）：`brew install ffmpeg`

**② 转码**：在 `rodentflow2/` 文件夹里运行

```bash
bash tools/encode-all.sh ~/Downloads/APP_demo/3x
```

它会把下表 8 个录屏转好，每个在 `videos/` 里生成 3 个文件：`名字.mp4`（视频）、`名字.jpg`（第一帧，加载前显示）、`名字-thumb.jpg`（缩略图）。第一次运行大约几分钟。

**③ 本地预览**：在 `rodentflow2/` 里运行 `python3 -m http.server`，**用 Chrome** 打开 <http://localhost:8000>。
（Python 自带的这个小服务器不支持 Safari 播放视频需要的"分段读取"，所以本地 Safari 可能不播；放到 GitHub Pages 上之后 Safari 和 iPhone 都正常。）

### 我推荐的 8 个和顺序

| # | 原文件 | 转码后名字 | 设备 | 追踪 | 为什么放这里 |
|---|---|---|---|---|---|
| 1 | `two_mice.mp4` | `two-mice` | iPhone | 两只 | 2.0 最大的新功能，第一眼就要看到 |
| 2 | `open_field.mp4` | `open-field` | iPhone | 单只 6 点 | 最基础的测试，看懂骨架颜色 |
| 3 | `3_chamber.mp4` | `three-chamber` | iPhone | 单只 6 点 | "用鼻尖计时"的好处最直观 |
| 4 | `new_object.mp4` | `novel-object` | iPhone | 单只 6 点 | 同上，另一个经典测试 |
| 5 | `Y_maze.mp4` | `y-maze` | iPhone | 质心 | 展示老方法依然在 |
| 6 | `TST.mp4` | `tail-suspension` | iPhone | 悬尾 | 多格自动识别，很有画面感 |
| 7 | `MAC_head_direction.mp4` | `mac-head-direction` | Mac | 单只 6 点 | 切到 Mac：外框变形，说明 Mac 版 |
| 8 | `MAC_elevated_maze.mp4` | `mac-plus-maze` | Mac | 质心 | 第二个 Mac 例子 |

**"追踪"一栏是我按文件名猜的，请核对**，不对的话改 `CLIPS` 里的 `tracker`（见下一节）。

备选（在 `tools/encode-all.sh` 的清单里，去掉行首 `#` 就会转）：`MAC_Social`、`MAC_openfield`、`MAC_barne_maze`、`head_direction`、`elevated_maze`、`barne`、`light_dark_room`、`gait`、`Cylinder`、`TST_single`。

## 3. 换视频、改顺序、改文字

全部在 `index.html` 底部脚本最上面的 `CLIPS` 里（搜索 `const CLIPS`），一个视频一段：

```js
{ file: "open-field", device: "phone", tracker: "pose",
  title: "Open field",
  text: "Six keypoints on every frame. …",
  measures: ["Distance", "Time in center", "Speed", "Head direction"] },
```

| 字段 | 意思 |
|---|---|
| `file` | 转码时起的名字（不带 `.mp4`） |
| `device` | `"phone"`（竖屏 iPhone 录屏）或 `"mac"`（横屏 Mac 录屏） |
| `tracker` | `"centroid"` 质心、`"pose"` 单只 6 点、`"dyad"` 两只、`"tst"` 悬尾。决定颜色、标签和"In the video"图例 |
| `title` / `text` / `measures` | 标题、说明、指标小标签 |
| `thumbPos` | 可选。缩略图显示画面的哪一部分，比如 `"50% 20%"`（偏上）、`"50% 80%"`（偏下）。默认 `"50% 30%"` |

- **改顺序**：上下挪整段。
- **加一个**：先用 `bash tools/encode.sh 原文件 新名字` 转码，再在 `CLIPS` 里加一段。数量不限，序号自动更新。
- 某个视频还没放进 `videos/` 时，页面会在那个位置显示要放的文件名，不会出错。

## 4. 转码参数（`tools/encode.sh`）

```bash
bash tools/encode.sh ~/Downloads/APP_demo/3x/open_field.mp4 open-field --start 1 --dur 12
```

| 参数 | 作用 |
|---|---|
| `--start` / `--dur` | 从第几秒开始、截多长。录屏开头常有点"开始录制"的操作，可以剪掉前 0.5–1 秒 |
| `--thumb 5` | 缩略图取第 5 秒的画面（默认取 40% 处） |
| `--speed 2` | 把 2 倍速直接做进文件 |
| `--width` | 默认 iPhone 录屏 720 宽、Mac 录屏 1600 宽 |
| `--fps 60` | 默认 30 帧。录屏是 60 帧，30 帧解码量减半、文件更小；界面滚动很多时可以用 60 |
| `--crf 21` | 画质，数字越小越清晰、文件越大（默认 23） |
| `--fade 0` | 默认会把结尾 0.3 秒淡入开头，循环接缝看不出来；不想要就设 0 |

想调整某一个视频，就把对应参数写在 `encode-all.sh` 清单那一行后面，例如：

```
open_field.mp4          open-field      --start 1 --dur 12 --thumb 6
```

## 5. 为什么不卡

- 你的原始录屏是 1320×2868、60 帧、8–11 Mbps；转码后 720 宽、30 帧、H.264（所有 iPhone / Mac / 浏览器都用硬件解码），文件约为原来的 1/5 到 1/10，`faststart` 边下边播。
- **任何时候只解码一个视频**。下一个在后台提前加载，所以自动切换没有黑屏，交叉淡入。
- 视频区滚出屏幕、切到别的标签页、打开大图时，自动暂停。
- 缩略图是 480 宽的小图片，不是视频。
- 系统开了"减少动态效果"或"省流量"：不自动播放，显示第一帧和播放按钮。iPhone 低电量模式会禁止自动播放：显示播放按钮，点一下之后自动切换照常进行。

## 6. 放到网站上

1. 把 `rodentflow2/` 整个文件夹复制到 `seesaw-earth.github.io` 仓库的根目录。
2. `git add . && git commit -m "Add RodentFlow 2.0 project page" && git push`
3. 一两分钟后打开：<https://seesaw-earth.github.io/rodentflow2/>

> 不要把文件夹叫 `RodentFlow`：`/RodentFlow/` 这个地址已经被 `seesaw-earth/RodentFlow` 仓库的用户手册占用。如果想换名字，比如 `rodentflow-2`，同时改掉 `index.html` 里的 `canonical`、`og:url`、`og:image` 和 BibTeX 里的网址。

在首页加个入口：打开个人网站的 `index.html`，在 RodentFlow 那张卡片的链接里加一行：

```html
<a href="rodentflow2/">Project page</a>
```

链接分享到 iMessage / Twitter / Slack 时的预览图用的是 `videos/mac-head-direction.jpg`（横图最合适）；换了 Mac 视频的话改 `og:image`。

## 7. 发布前请核对

- **每个视频的 `tracker` 和说明文字**：我没看到视频内容，是按文件名写的。
- **许可证**：个人首页写的是 "Source (MIT)"，但 2.0 代码包里的 `LICENSE` 是 AGPL-3.0（Ultralytics 也是 AGPL）。项目页没有写许可证，只链接到 GitHub；请确认仓库里写的是哪一个。
- **Mac 版**：页面写了 "iPhone, iPad and Mac"。如果 Mac 版还没上架，把 Hero 和 Runs on 两处改一下。
- **身份识别图表**：数字来自模拟测试，图上已注明 "Simulated"。有真实录像的验证结果后，换成真实数字更有说服力。
- **App Store 链接**：`apps.apple.com/app/id6818278240`。
- **BibTeX**：有论文或 DOI 之后，换成论文的引用格式，并在 Hero 加一个 "Paper" 按钮。
