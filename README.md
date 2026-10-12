# seemath

[简体中文](README.zh-CN.md) · **English**

A collection of education skills that turn academic problems into **interactive lesson web pages** and **narrated explainer videos**.

## Install

**Recommended** — install with [skills](https://github.com/vercel-labs/skills) in one command:

```bash
npx skills add jinshihao1227/seemath1
```

To update to the latest version later:

```bash
npx skills update
```

> **Note:** `npx skills update` only refreshes skills you've **already** installed — it does **not** pull in skills newly added to the repo. When this repo gains a new skill (e.g. a new `seemath-*`), run `npx skills add jinshihao1227/seemath1` again to install it.

Or use it as a Claude Code plugin marketplace:

```
/plugin marketplace add jinshihao1227/seemath1
/plugin install seemath
```

Once installed, the skills activate on their trigger words, and can also be invoked manually.

## Skill: seemath-analytic-geometry

![seemath-analytic-geometry demo](seemath-analytic-geometry.gif)

Solves one analytic geometry (conic sections) problem into a self-contained interactive lesson page. Same three entry points as above (text / image / random). Built on a **2D Canvas board + KaTeX** with a generic, data-driven interactive engine: a parameter slider drives derived constructions (line∩conic, point-on-conic, central reflection, tangent…) and live readouts, with a theoretical-range bar or a fixed-value indicator.

**Problem types covered**: standard equation, chord length, dot-product range / fixed value, triangle-area extremum, fixed point, fixed value (slope product), locus, tangent, eccentricity — on ellipses / hyperbolas / parabolas / circles. All solved uniformly via "parametrized line `x = my + c` + system + Vieta's formulas + substitution."

> A correctness note baked into the kernel: open/closed interval endpoints are decided by whether a *real* line attains them, so the boxed answer always matches what the interactive tool shows (e.g. the ellipse `MA·MB` range is the **closed** `[-3, 7/4]` — slide θ to 0° and you read exactly −3).

**Trigger words**: analytic geometry, conic sections, ellipse, hyperbola, parabola, chord length, dot product range, fixed point, fixed value, locus, eccentricity, interactive analytic geometry solution page; 解析几何、圆锥曲线、椭圆、双曲线、抛物线、焦点弦、向量数量积取值范围、定点问题、定值问题、斜率之积、三角形面积最值、轨迹方程、离心率, etc.

### Dependency

The compute core `lib/analytic_kernel.py` depends on **sympy** (same as above).

### Generate from the command line (without Claude)

```bash
cd skills/seemath-analytic-geometry
python3 scripts/generate.py list                          # list registered problem types
python3 scripts/generate.py ellipse_dot_range ./sol.html  # ellipse · MA·MB range [-3, 7/4]
python3 scripts/generate.py parabola_dot_const ./sol.html # parabola focal chord · OA·OB ≡ -3
python3 scripts/generate.py all ./out_dir                 # all registered types
python3 lib/analytic_kernel.py                            # kernel built-in self-check
```

> Like above, no output path → writes to the **current working directory (cwd)**.

## Skill: seemath-math-video

![seemath-math-video demo](seemath-math-video.jpg)

Turns one math problem (geometry, algebra, functions, motion problems…) into a **16:9 1920×1080 explainer MP4**: Chinese voice-over (Zhipu **GLM-TTS**), bilingual zh + en subtitles (`.srt` too), and hand-drawn notebook-style canvas animation driven by the narration timeline. Input is a problem screenshot or plain text.

**The picture explains the step**: every narration line gets a "point → move → keep" action on the figure (equal segments slide onto each other, congruent triangles overlay, the 3D camera tweens to a top view, a cone unrolls into a sector…) — timed by `S.at(k, f)` (a fraction into line *k*), never by hard-coded seconds. A `motion` check rejects static lines.

**Pipeline** (one folder per video, created in the user's current directory):

```
script.json ──build_audio.py──► timeline.json + mix.wav + <name>.srt   (GLM-TTS + synthesized music)
anim.js + engine.js ──node render.mjs video──► <name>.mp4              (Playwright + ffmpeg, 30 fps)
```

**Guard rails**: the first scene shows the original problem with each condition boxed as it is read; `tts` text must be speakable Chinese (no digits / math symbols); polyphones are pinned (`长[cháng]`, shared `pron.json` lexicon) and `--check` must report 0 before any paid TTS call; a free `--preview` + contact-sheet review comes before real audio; `--asr` transcribes the audio back to catch misread letters.

**Trigger words**: math explainer video, walkthrough video, problem-solving video, micro-lesson; 讲解视频、解题视频、例题精讲、微课 etc.

### Dependency

- Python 3 with `numpy requests pypinyin pillow`; Node.js 18+ with `playwright` + `ffmpeg-static` (installed once in the workspace — `scripts/new_video.sh` writes the `package.json`); Google Chrome or Playwright Chromium.
- A Zhipu **`GLM_API_KEY`** provided by you, in `~/.config/math-problem-video/.env` (see `reference/glm-tts-setup.md`).
- **No key?** It falls back automatically (`TTS_ENGINE=auto`): [edge-tts](https://github.com/rany2/edge-tts) if installed (free neural voices, needs internet), else macOS `say` (offline, robotic). Force one with `TTS_ENGINE=glm|edge|say`.

### Run the pipeline by hand (without Claude)

```bash
bash skills/seemath-math-video/scripts/new_video.sh "$PWD" my_problem   # scaffold from template/
bash skills/seemath-math-video/scripts/setup_check.sh my_problem        # must print ALL OK
cd my_problem
python3 build_audio.py --check                                      # script + pronunciation lint (free)
python3 build_audio.py --preview && node render.mjs motion && node render.mjs stills auto
python3 build_audio.py && node render.mjs video 6                   # real TTS, then render (6 = parallel pages)
```

## How it works

1. **Get a problem spec** — normalize text, image, or random input into a structured problem description.
2. **Solve and assemble** — the analytic-geometry kernel computes exact results for interactive lesson pages; the math-video pipeline builds narration, subtitles, and timed animation.
3. **Self-check** — verify the computed answer, script, pronunciation, animation motion, and rendered preview before delivery.
4. **Deliver** — write the finished HTML page or MP4/SRT files to the user's current working directory.

## Project structure

seemath/
├── .claude-plugin/
├── index.html
└── skills/
    ├── seemath-analytic-geometry/
    └── seemath-math-video/

## License

[Apache-2.0](LICENSE)

## Author

JSH

## Star History

<a href="https://www.star-history.com/?repos=jinshihao1227%2Fseemath1&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=jinshihao1227/seemath1&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=jinshihao1227/seemath1&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=jinshihao1227/seemath1&type=date&legend=top-left" />
 </picture>
</a>
