# seemath

**简体中文** · [English](README.md)

教育类技能集合：把学科问题转成**可交互的教学网页**和**带配音的讲解视频**。

## 安装

**推荐** —— 用 [skills](https://github.com/vercel-labs/skills) 一行命令安装：

```bash
npx skills add jinshihao1227/seemath1
```

后续更新到最新版：

```bash
npx skills update
```

> **注意**：`npx skills update` 只会把**已安装**的技能刷新到最新版，**不会**自动拉取仓库新增的技能。当本仓库新增了技能（如新的 `seemath-*`）时，再次运行 `npx skills add jinshihao1227/seemath1` 即可装上。

或作为 Claude Code 插件市场使用：

```
/plugin marketplace add jinshihao1227/seemath1
/plugin install seemath
```

安装后，技能会随触发词自动激活，也可以手动调用。

## 技能：seemath-analytic-geometry

![seemath-analytic-geometry 演示](seemath-analytic-geometry.gif)

把一道解析几何（圆锥曲线）题解成一个自包含的交互教学网页。三入口与上面一致（文字 / 图片 / 随机）。基于 **2D Canvas 画板 + KaTeX** 的通用数据驱动交互引擎：一个参数滑块驱动派生构造（直线∩曲线、曲线上动点、中心对称、切线…）与实时读数，并配理论范围条或定值指示。

**覆盖题型**：求标准方程、弦长、向量数量积取值范围 / 定值、三角形面积最值、定点、定值（斜率之积）、轨迹、切线、离心率——涵盖椭圆 / 双曲线 / 抛物线 / 圆。统一用"设含参直线 `x=my+c` + 联立 + 韦达 + 换元"。

> kernel 内置的一个正确性细节：区间端点的开 / 闭由"是否有真实直线取到"判定，所以框出的答案始终与交互工具一致（例如椭圆 `MA·MB` 的范围是**闭区间** `[-3, 7/4]`——把 θ 拖到 0° 屏幕上正好读到 −3）。

**触发词**：解析几何、圆锥曲线、椭圆、双曲线、抛物线、焦点弦、向量数量积取值范围、定点问题、定值问题、斜率之积、三角形面积最值、轨迹方程、离心率；analytic geometry, conic sections, ellipse, hyperbola, parabola, chord length, dot product range, fixed point, locus, interactive analytic geometry solution page 等。

### 依赖

计算核心 `lib/analytic_kernel.py` 依赖 **sympy**（同上）。

### 命令行直接生成（不经过 Claude）

```bash
cd skills/seemath-analytic-geometry
python3 scripts/generate.py list                          # 列出已注册题型
python3 scripts/generate.py ellipse_dot_range ./sol.html  # 椭圆 · MA·MB 范围 [-3, 7/4]
python3 scripts/generate.py parabola_dot_const ./sol.html # 抛物线焦点弦 · OA·OB ≡ -3
python3 scripts/generate.py all ./out_dir                 # 全部已注册题型
python3 lib/analytic_kernel.py                            # kernel 内置自检
```

> 同上，不传输出路径时默认写到**当前工作目录（cwd）**。

## 技能：seemath-math-video

![seemath-math-video 演示](seemath-math-video.jpg)

把一道数学题（几何、代数、函数、行程问题……）做成 **16:9、1920×1080 的讲解视频 MP4**：智谱 **GLM-TTS** 中文配音、中英双语字幕（同时输出 `.srt`）、按旁白时间轴驱动的手绘笔记本风格 canvas 动画。输入可以是题目截图或文字。

**图会"讲题"**：每句旁白在图上都有一个"指 → 动 → 留"的动作（相等线段滑过去重合、全等三角形叠上去、3D 相机转到俯视、圆锥侧面展开成扇形……），用 `S.at(k, f)`（第 k 句开始后 f 比例处）定时，绝不写死秒数；`motion` 检查会拦下静止不动的句子。

**流水线**（每个视频一个文件夹，建在用户当前目录）：

```
script.json ──build_audio.py──► timeline.json + mix.wav + <name>.srt   （GLM-TTS + 合成音乐）
anim.js + engine.js ──node render.mjs video──► <name>.mp4              （Playwright + ffmpeg，30 fps）
```

**护栏**：第一幕展示原题并逐条框出条件；`tts` 字段只能是能念出来的中文（无数字/数学符号）；多音字必须固定读音（`长[cháng]`，共享 `pron.json` 词表），`--check` 为 0 才允许调用付费 TTS；先免费 `--preview` + 截图拼图审查版面，再生成真配音；`--asr` 把配音转回文字核对字母读音。

**触发词**：讲解视频、解题视频、例题精讲、微课、数学题视频；math explainer video、walkthrough video 等。

### 依赖

- Python 3 + `numpy requests pypinyin pillow`；Node.js 18+ + `playwright` + `ffmpeg-static`（在工作目录装一次，`scripts/new_video.sh` 会生成 `package.json`）；Google Chrome 或 Playwright Chromium。
- 由你自己提供的智谱 **`GLM_API_KEY`**，写在 `~/.config/math-problem-video/.env`（见 `reference/glm-tts-setup.md`）。
- **没有 key？** 自动兜底（`TTS_ENGINE=auto`）：装了 [edge-tts](https://github.com/rany2/edge-tts) 就用它（免费神经网络音色，需联网），否则用 macOS 自带的 `say`（离线，机械感明显）。也可用 `TTS_ENGINE=glm|edge|say` 指定。

### 手动跑流水线（不经过 Claude）

```bash
bash skills/seemath-math-video/scripts/new_video.sh "$PWD" my_problem   # 从 template/ 建项目
bash skills/seemath-math-video/scripts/setup_check.sh my_problem        # 必须输出 ALL OK
cd my_problem
python3 build_audio.py --check                                      # 脚本 + 读音检查（免费）
python3 build_audio.py --preview && node render.mjs motion && node render.mjs stills auto
python3 build_audio.py && node render.mjs video 6                   # 真配音，然后渲染（6 = 并行页数）
```

## 工作原理

1. **得到 problem spec** —— 把文字、图片或随机输入归一成结构化题目描述。
2. **求解并组装** —— 解析几何 kernel 精确计算并生成交互教学页；数学视频流水线生成配音、字幕和按时间轴驱动的动画。
3. **自检** —— 检查答案、脚本、读音、动画动作和渲染预览。
4. **交付** —— 将 HTML 页面或 MP4/SRT 文件写入用户当前工作目录。

## 目录结构

seemath/
├── .claude-plugin/
├── index.html
└── skills/
    ├── seemath-analytic-geometry/
    └── seemath-math-video/

## License

[Apache-2.0](LICENSE)

## 作者

JSH

## Star History

<a href="https://www.star-history.com/?repos=jinshihao1227%2Fseemath1&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=jinshihao1227/seemath1&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=jinshihao1227/seemath1&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=jinshihao1227/seemath1&type=date&legend=top-left" />
 </picture>
</a>
