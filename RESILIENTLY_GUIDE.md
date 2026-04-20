# Hyperframes — Resiliently Adaptation Guide

> How to use the Hyperframes Student Kit to produce Resiliently YouTube videos.
> Adapted from the `aisoc-lesson-5-1` (talking head + motion graphics) and `may-shorts-19` (short-form vertical) projects.

---

## What Hyperframes Is

**HTML + GSAP → MP4.** You write plain HTML files with `data-*` attributes for timing, CSS for visuals, and GSAP timelines for animation. The Hyperframes CLI renders through headless Chromium into real H.264 video.

- **Not a video editor.** No GUI timeline. You edit code.
- **Not React/Remotion.** Plain HTML. Simple mental model.
- **Real output.** 1080p H.264 MP4s. Production-ready.
- **Free/open source.** Backed by HeyGen.

---

## How It Maps to Resiliently's Format

| Resiliently Video | Hyperframes Template | Why |
|---|---|---|
| **Pillar A/C talking head** (8-14 min, 16:9) | `aisoc-lesson-5-1` | Face-cam PiP in corner, motion graphics on left 2/3, sections synced to transcript. This is your primary template. |
| **Pillar B live code demo** | `clickup-demo` | Product demo format with screen recordings and UI overlays. |
| **Shorts/Clips** (30-60s, 9:16) | `may-shorts-19` | Vertical talking head with scene overlays and karaoke captions. |

---

## The Lesson Template (Pillar A/C — Your Workhorse)

Based on `aisoc-lesson-5-1`. 16:9 (1920×1080), face-cam bottom-right PiP, motion graphics on the left 2/3.

### Architecture

```
index.html (root, 1920×1080)
├── bg-image (brand background or dark gradient)
├── face-clip (PiP, bottom-right, 360×540 with rounded corners)
│   └── face-wrapper (1920×1080 source, scaled to PiP)
│       └── face-video (muted)
├── face-audio (separate track, same source file)
├── sec-01.html (track-index="1") — Scene 1 overlay
├── sec-02.html (track-index="1") — Scene 2 overlay
├── ...
└── sec-NN.html (track-index="1") — Last scene
```

### Key Patterns from the Lesson Template

1. **Face-cam stays in corner.** No fullscreen mode. Teacher + materials energy. The face-clip has rounded corners, drop shadow, and a cyan rim.
2. **Sections are sub-compositions.** Each section is a separate HTML file loaded via `data-composition-src`.
3. **Sections chain with `data-start="sec-prev"`.** No manual time calculation — reference the previous section's ID.
4. **All GSAP offsets are LOCAL.** Inside each section file, time starts at 0. The framework handles the global offset.
5. **`tl.set({}, {}, DURATION)` pads every timeline.** Prevents black frame flashes when timeline duration doesn't match `data-duration`.

### Resiliently Brand Adaptation

Replace AIS branding with Resiliently tokens:

```css
/* brand-tokens.css */
:root {
  --rsl-bg: #0a0f1a;         /* deep navy-black */
  --rsl-primary: #2dd4bf;     /* teal — trust, solution, brand */
  --rsl-accent: #f59e0b;      /* amber — warning, cost, the "gotcha" */
  --rsl-danger: #ef4444;      /* red — claim denied, exclusion triggered */
  --rsl-text: #f1f5f9;        /* light gray — body text */
  --rsl-text-dim: #64748b;    /* dim gray — labels, meta */
  --rsl-font-display: 'Inter', sans-serif;
  --rsl-font-mono: 'JetBrains Mono', monospace;
}
```

Color meanings (from the Motion Philosophy "symbolic palette" rule):
- **Teal** = solution / coverage / the underwriter's good news
- **Amber** = cost / warning / the thing you didn't expect
- **Red** = denial / exclusion / the policy doesn't cover this
- **Chrome/white** = premium / authority / the underwriter speaking

---

## Production Workflow for a Resiliently Video

### Step 1: Record face video
Standard talking head. 1920×1080, landscape. Record the full script in one take. Multiple takes is fine — you'll edit the audio.

### Step 2: Edit audio
Cut retakes, pauses, mistakes. Save as `<name>-edit.mp4`. This is the source of truth for all timing.

```bash
ffprobe -v error -show_entries format=duration -of csv=p=0 assets/<name>-edit.mp4
```

That number becomes `data-duration` on every element.

### Step 3: Transcribe
```bash
npx hyperframes transcribe assets/<name>-edit.mp4 --model small.en --json
```
Produces word-level timestamps for caption sync.

### Step 4: Write the storyboard
Map the script to sections (one section per video "act" or major topic). Each section gets:
- Start time (from transcript)
- Duration
- Visual metaphor (what appears on the motion graphics canvas)
- Key word anchors (specific words where visuals should land)

### Step 5: Build section compositions
For each section, create `compositions/sections/NN-<label>.html`:
- Static CSS positions elements at their hero frame
- GSAP `from()` for entrances, `to()` for exits
- All timing is LOCAL (0-based from section start)
- Pad with `tl.set({}, {}, DURATION)`

### Step 6: Lint + Preview
```bash
npx hyperframes lint
npx hyperframes preview    # http://localhost:3002
```
Scrub the timeline in browser. Check every section.

### Step 7: Render
```bash
npx hyperframes render --quality draft --output renders/draft.mp4
```
Draft is CRF 28 (fast). Watch it. Then:
```bash
npx hyperframes render --quality standard --output renders/final.mp4
```
Standard is CRF 18 (visually lossless).

---

## Resiliently-Specific Visual Vocabulary

### What to put on the motion graphics canvas (left 2/3)

| Content Type | Visual Treatment | When to Use |
|---|---|---|
| **Policy clause text** | Chrome gradient type, scale-in from center | Act 3 of Pillar C scripts |
| **Cost numbers** | Counter ticking up from 0 to final amount, amber color | Damage chain sections |
| **Coverage table** | Grid with ✅/❌/⚠️, rows appearing with stagger | Policy coverage breakdowns |
| **Timeline** | Horizontal bar with events, left-to-right reveal | Incident chronology |
| **Comparison columns** | Two-column split (before/after, with/without coverage) | Coverage comparisons |
| **Warning/Exclusion callout** | Red-bordered card with stamp effect | Coverage gaps |
| **NIS2/DORA checklist** | Numbered list with checkmark pop-ins | Compliance sections |
| **SBOM/dependency tree** | Animated node graph | Supply chain content |
| **Claim scenario** | Fictitious company card (name, sector, size) sliding in | Damage scenarios |

### What NOT to put on canvas

- Screenshots of actual policies (too small to read at PiP scale)
- Dense code (Pillar B is better as screen recording, not motion graphics)
- Stock photos (breaks the premium aesthetic)

---

## Starter Project Template

```
video-projects/resiliently-episode-NN/
├── hyperframes.json
├── meta.json
├── index.html
├── compositions/
│   ├── sections/
│   │   ├── 01-hook.html
│   │   ├── 02-incident.html
│   │   ├── 03-damage-chain.html
│   │   ├── 04-coverage-gaps.html
│   │   ├── 05-takeaways.html
│   │   └── 06-outro.html
│   └── brand/
│       └── ambient-bg.html
├── assets/
│   ├── brand-tokens.css
│   ├── <name>-edit.mp4
│   └── transcript.json
└── renders/
```

---

## Key Learnings from the Kit

### 1. "Layout Before Animation"
Build the hero frame first (where everything sits at maximum visibility). Then animate FROM offscreen TO that position. Not the other way around. This prevents overlap issues.

### 2. Face wrapper, never video element
Animate the wrapper div around the video. Animating `<video>` directly freezes frames during rendering.

### 3. Timeline padding is non-negotiable
Every sub-composition needs `tl.set({}, {}, DURATION)` at the end. Without it, Hyperframes shows a black frame when the timeline is shorter than `data-duration`.

### 4. Sections chain via `data-start="prev-section-id"`
No manual time math. The framework resolves it. Just chain sections in order.

### 5. No `Math.random()` or `Date.now()`
Breaks deterministic rendering. Use seeded pseudo-random if you need variation.

### 6. Audio is separate from video
The `<audio>` element is its own track. The `<video>` is muted. The mixer handles sync.

### 7. Rotate transition flavors
Never use the same transition type twice in a row. The kit includes: blur-crossfade, push-slide, flash-through-white, SDF-iris, zoom-through.

### 8. Data-feel beats decoration
Charts, counters, grids, timelines > floating icons and decorative shapes. "Evidence" reads better than "decoration."

---

## Installation Checklist

```bash
cd /Users/michaelguiao/Projects/active/hyperframes-editor
npm install
npx hyperframes doctor    # check Node, FFmpeg, Chrome

# Test with an existing project
cd video-projects/aisoc-lesson-5-1
npx hyperframes preview    # should open http://localhost:3002
```

---

## Files Created

| File | Purpose |
|------|---------|
| This file | Resiliently adaptation guide |
| `MOTION_PHILOSOPHY.md` | Gold-standard motion aesthetic (read before brainstorming) |
| `.claude/skills/short-form-video/` | Vertical short-form playbook |
| `.claude/skills/make-a-video/` | End-to-end video creation pipeline |
| `.claude/skills/hyperframes/` | Framework rules and composition patterns |
| `.claude/skills/gsap/` | GSAP animation reference |
