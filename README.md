# Motion Reel: a Claude Code skill for motion graphics

`/motion-reel` is a Claude Code skill that makes beat-synced motion graphics entirely in code.

Give Claude a product URL. It pulls the real brand from the site (screenshots, logo, fonts, colours) and writes a style guide and a shotlist. Then it **stops and waits for your OK**. After you approve, it builds the film, synthesizes a score with UI sounds on the measured beat grid, and critiques its own renders until every score reaches 8/10. You get H.264 MP4s in 16:9, 1:1, 4:5 and 9:16.

Each film is a pure function of time: `window.seek(t)` paints frame *t* in headless Chrome, and ffmpeg encodes the frames. Every render comes out frame-for-frame identical. There are no timers, no CSS transitions and no `Math.random`, and all motion uses closed-form springs.

> Based on the free Motion Reel Kit. Weekly AI workflow updates: https://www.weekly10x.com

---

## Install

### Requirements
- **Claude Code**
- **Node 22+** and **ffmpeg**: `brew install node ffmpeg`
- **Python 3.9+** with numpy, scipy, soundfile, librosa and pillow (the music, beat grid and mix)
- **Chromium** through Playwright. On macOS 13 or older, where newer Playwright has no Chromium, the scripts fall back to your installed **Google Chrome**.

### Option A: the installer (recommended)
```bash
git clone https://github.com/statiiiix/motion-graphic.git
cd motion-graphic
sh install.sh
```
The installer copies the skill and its presets to `~/.claude/skills/motion-reel`, so `/motion-reel` works in **every** project. It also installs Playwright, Chromium and the Python audio libraries. To install into a single project, run `sh install.sh --project /path/to/project`.

### Option B: as a Claude Code plugin
Inside Claude Code:
```text
/plugin marketplace add statiiiix/motion-graphic
/plugin install motion-reel@motion-graphic
```
Then install the toolchain yourself, once:
```bash
npm i -g playwright && npx playwright install chromium
python3 -m pip install --user numpy scipy soundfile librosa pillow
```

Both options need a **new** Claude Code session afterwards, because skills load when a session starts.

---

## How to use it

Open any project in Claude Code and type `/motion-reel` followed by what you want:

```text
/motion-reel a 15-second launch reel for https://yoursite.com
```
```text
/motion-reel a 9:16 teaser for https://yoursite.com in the style of <link to a launch video you love>. Take its rhythm and transitions, not its content.
```
```text
/motion-reel 20-second launch reel for https://example.com in 16:9 and 9:16. Use preset blank, synthesize the music, no voiceover.
```
```text
/motion-reel fill in preset blank from https://yoursite.com (colours, fonts, promise, CTA), save it as presets/<my-brand>, then make a 15-second reel with it.
```
```text
/motion-reel critique the last render: contact sheet, phone test at 360 px, loop seam. Fix the 3 worst problems and repeat until every score is 8+.
```

Claude asks for anything missing in one round of questions. It shows you the shotlist before it builds anything. Each film is written to `videos/<slug>/`, and the finished files land in `videos/<slug>/renders/<format>.mp4`.

### What happens when you run it

| Step | What Claude does |
|---|---|
| 1. Intake | Fills `brief.md` and asks for missing inputs (duration, formats, music, voiceover) in one round |
| 2. Assets | Captures real screenshots, logo, fonts and colours from your URL with Playwright |
| 3. Style guide | Palette, type and rhythm. From a reference film it borrows the *grammar*, never the content |
| 4. Beat grid | Synthesizes an original score, or uses yours, and measures the beats with librosa into `beats.json` |
| 5. Shotlist | Writes every shot with its timing, text, motion and SFX, then **stops for your OK** |
| 6. Build | Builds the scenes on springs, with hits landing on the beat |
| 7. Critique loop | Contact sheets, phone test at 360 px, loop seam. A fresh critic scores 8 criteria and Claude fixes the 3 worst problems. At least 3 rounds, until every score is 8+ |
| 8. Finals | 60 fps with motion blur, SFX, a -14 LUFS mix, every format |

Optional voiceover uses the **Fish Audio MCP**. Connect it in Claude Code first; you can use your own cloned voice.

---

## Presets: your brand in one file

A preset fills in the brand, colours, fonts, tempo, formats and voiceover, so Claude only asks about what's left.

1. Copy `skills/motion-reel/presets/blank` to `presets/<your-brand>` in your project, or to `~/.claude/skills/motion-reel/presets/<your-brand>` to use it in every project.
2. Fill in `preset.jsonc`. Every field is commented.
3. Say "use preset <your-brand>" when you run `/motion-reel`.

Included presets: `blank` (an empty template) and `lukas-yt` (a worked example with Geist font, 120 BPM, 16:9 + 9:16 and own-voice narration).

---

## What's in this repo

| Path | What it is |
|---|---|
| `skills/motion-reel/SKILL.md` | The skill: the full 10-step pipeline Claude follows |
| `skills/motion-reel/engine/` | The film engine: `index.html`, `core.js`, `type.js`, `film.js`, `lib/motion.js` (springs, with tests) |
| `skills/motion-reel/scripts/` | Pipeline scripts: `init.sh`, `render.mjs`, `capture.mjs`, `music.py`, `beats.py`, `sfx.mjs`, `sync.mjs`, `mix.py`, `vo.py`, `review.py`, `preset.mjs` |
| `skills/motion-reel/reference/` | `RULES.md`, `ENGINE.md`, `AUDIO.md`, and `CRITIQUE.md` (the critic's prompt) |
| `skills/motion-reel/templates/` | Brief, timeline, style guide, shotlist and review-log templates |
| `skills/motion-reel/presets/` | `blank` and `lukas-yt` |
| `.claude-plugin/` | Plugin and marketplace manifests (Option B install) |
| `CLAUDE.md` | The motion house rules. Copy it into a project root to apply them to everything made there |
| `prompts/` | The 14 chapter prompts that built this skill, the director's brief template and the critique scorecard |
| `examples/blank-preset-5s-test.mp4` | A 5-second test render |
| `install.sh` / `test-render.sh` | Installer and smoke test |

### The house rules (short version)
- **Springs only.** Presets: snappy (UI), default (cards, camera), heavy (big type) and playful (mascots only). Use no easing curves for anything that enters, exits or retargets.
- **No clichés:** no centered title on a gradient, no everything-fades-in, no crossfades, no glows, spins, glitches or light leaks, and no dead time.
- **Something new every 2–4 seconds.** Hook by 2 s, end card ≤ 2 s.
- **Real product UI, logos and fonts.** Never redraw UI that already exists.
- **Sound on the grid.** Hits sit on measured beats. The mix is -14 LUFS.

The full rules are in [`skills/motion-reel/reference/RULES.md`](skills/motion-reel/reference/RULES.md).

---

## Check your setup

From this folder:
```bash
npm install && npm test
```
The test renders 5 seconds with the blank preset to `videos/_test-<time>/renders/16x9.mp4`. To test your own preset, run `sh test-render.sh <your-preset> 5`.

## Troubleshooting

**`/motion-reel` doesn't show up.** Start a **new** Claude Code session. Check the file exists: `ls ~/.claude/skills/motion-reel/SKILL.md`.

**"Playwright does not support chromium on mac13".** Newer Playwright has dropped older macOS. Install Google Chrome; the render and capture scripts fall back to it automatically.

**"playwright not found" when rendering.** Re-run `sh install.sh`, or run `npm i -D playwright` inside the film folder.

**pip refuses to install** ("externally-managed-environment"). Use the system Python: `/usr/bin/python3 -m pip install --user -r requirements.txt`.

## License notes
The Geist font in `skills/motion-reel/presets/lukas-yt/fonts` is under the SIL Open Font License (`OFL-Geist.txt`).
