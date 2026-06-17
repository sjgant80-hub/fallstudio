# ◊ FallStudio

**The sovereign creative suite. Adobe Creative Cloud, replaced one HTML file at a time.**

- prime · **1409**
- version · **1.0.0**
- seal · **◊·κ=1** · MIT
- replaces · **Adobe Creative Cloud** (the suite hub / app launcher)
- repo · `sjgant80-hub/fallstudio`
- live · https://sjgant80-hub.github.io/fallstudio/

---

## For people who just want to make something

FallStudio is the menu. It catalogues eight sovereign tools that together replace the Adobe Creative Cloud bundle, then routes whatever you want to do to the right tool.

Open the page, press **Ctrl K**, type what you want to make in plain English — "edit this photo", "make a brochure", "logo animation", "voice memo", "3D product render", "organise my photos" — and FallStudio opens the right tool in a new tab with your intent pre-filled.

Every tool in the suite is:

- **one HTML file** — works from `file://`, no install
- **sovereign** — runs in your browser, nothing leaves the device
- **free** — MIT, no account, no telemetry, no upsell

### the suite

| ◊ | tool | replaces | what it does |
|---|---|---|---|
| ◧ | **FallMage** | Photoshop | raster image editor · crop, filter, paint, retouch |
| ▦ | **FallPDF** | Acrobat (writer) | author PDFs · letters, memos, invoices, briefs |
| ◊ | **FallVector** | Illustrator | vector illustration · logos, icons, SVG |
| ▤ | **FallPage** | InDesign | page layout · brochures, magazines, spreads |
| ♪ | **FallAudio** | Audition | record and edit audio · voice, podcasts, soundbeds |
| ▶ | **FallMotion** | After Effects | motion graphics · logo animation, lower thirds, loops |
| ⊞ | **FallAsset** | Bridge / Lightroom | asset library · catalogue, tag, rate, browse |
| ◉ | **FallScene** | Dimension | 3D product staging · GLB import, materials, render |

Live links:

- https://sjgant80-hub.github.io/fallmage/
- https://sjgant80-hub.github.io/fallpdf/
- https://sjgant80-hub.github.io/fallvector/
- https://sjgant80-hub.github.io/fallpage/
- https://sjgant80-hub.github.io/fallaudio/
- https://sjgant80-hub.github.io/fallmotion/
- https://sjgant80-hub.github.io/fallasset/
- https://sjgant80-hub.github.io/fallscene/

### Ω autopilot

Press **Ctrl K** anywhere in FallStudio. Type what you want to do.

Behind the scenes a tiered router runs:

- **T0** — keyword regex against your intent, no LLM, fully offline. Covers the common verbs ("photo", "pdf", "brochure", "animate", "3D", "voice", "organise", "vector"). This always works.
- **T2** — if you have Ollama running locally on `127.0.0.1:11434`, used as a free local LLM.
- **T3** — if you've added an Anthropic / OpenAI / Gemini / OpenRouter key in **Settings**, used to parse arbitrary intent into `{tool, hint}` JSON.

Pick a result with **Enter**. FallStudio opens the right tool in a new tab with `?intent=<your text>` so the tool can pre-populate.

### Installing as an app

FallStudio is a PWA. On mobile or desktop Chrome / Edge / Safari, use **Add to Home Screen** / **Install app**. After that it lives in your launcher like any native app, but it's still just one HTML file.

---

## For developers

### Architecture

- **single file** · `index.html` · all CSS and JS inline · no build step · no npm
- **vanilla JS** · no framework · no CDN · no dependencies
- **state** · `localStorage` for BYOK keys · suite registry hard-coded in JS
- **live status** · `fetch(url, {mode:'no-cors'})` per tool at boot, green dot if it responds
- **PWA** · manifest baked into a `data:` URL in `<head>`

### The suite registry

The 8 tools are declared in the `SUITE` constant at the top of the script. Each entry has `{id, icon, name, replaces, url, purpose, size, prime, features}`. To add a new tool: append to the array, append a matching entry to `ROUTES` for keyword routing.

### The router

```js
const ROUTES = [
  { tool:'fallmage',   verbs:/\b(image|photo|crop|filter|brush|paint|jpg|png|webp|...)\b/i },
  { tool:'fallpdf',    verbs:/\b(pdf|document|letter|memo|invoice|cv|...)\b/i },
  // ...
];
```

`routeT0(intent)` runs the regex set and returns `[{tool, match, hint}]`. `routeT3(intent)` sends a system prompt + intent to whichever BYOK provider is configured, expects JSON back, parses to `{tool, hint, why}`. The palette tries T0 first, falls back to T3 if available, then fuzzy name match.

### Cross-tool handoff

When a tool is launched from the palette, FallStudio opens:

```
<tool url>?intent=<your text>&from=fallstudio
```

Constituent tools that support it read `?intent` and pre-fill. FallStudio itself also reads `?intent` on arrival so other tools can hand back.

### postMessage API

Other estate tools can talk to FallStudio via `window.postMessage`:

```js
target.postMessage({target:'fallstudio', source:'mytool', action:'list'}, '*');
// → {tools:[{id, name, url}, ...]}

target.postMessage({target:'fallstudio', source:'mytool', action:'route', intent:'edit a photo'}, '*');
// → {hits:[{tool:'fallmage', match:'edit a photo', url:'...'}]}
```

`action:'ping'` returns `{ok:true, prime:1409, version:'1.0.0'}`.

### fallmesh

On boot FallStudio broadcasts on the `fall-signal` BroadcastChannel:

```js
{source:'fallstudio', type:'hello', prime:1409, version:'1.0.0', ts:<ms>}
```

Listens for `{type:'ping'}` and replies `{type:'pong', prime:1409}`.

### KONOMI

The sovereign-tier licence shim is baked in and inert:

```js
window.KONOMI = {active:true, tier:'sovereign', prime:1409, check:()=>({ok:true, tier:'sovereign'})};
```

### Cascade tiers

Standard estate Cascade (`T0` / `T2` / `T3`) is included. The tier badge in the header updates on boot. `Cascade.generate(sys, user, maxTok)` is callable from the console for ad-hoc routing.

### Manifest export

The footer link **manifest.json** downloads a versioned JSON of the suite registry — useful for cross-estate tools, fall-registry sync, or just inspection.

### Files

```
fallstudio/
├── index.html      ← the hub (single sovereign file)
├── README.md       ← this file
├── LICENSE         ← MIT
└── .nojekyll       ← Pages legacy deploy marker
```

### Deploy

GitHub Pages, legacy build type. The `.nojekyll` file is mandatory (Pages 404s without it on this estate). No Actions workflow needed.

### The 14-point gate

FallStudio is built against the shared estate doctrine:

1. one HTML file · sovereign · works from `file://`
2. <400KB
3. L1 FACE · domain-appropriate (the suite grid, not a generic dashboard)
4. L2 SWARM · Ω + 8 named operations (one per tool)
5. L3 CASCADE · T0 fully works offline · T2 / T3 optional
6. L4 BLOOM · intent → right action · no LLM-required path
7. L5 PERSIST · settings in `localStorage` · JSON manifest export
8. L6 SKIN · mobile-first · estate dark palette · brass + amber
9. L7 ASS · empty state shows all 8 tools immediately
10. KONOMI sovereign shim · baked · inert
11. fallmesh BroadcastChannel · `fall-signal` · prime 1409
12. PWA manifest · `data:` URL
13. README + MIT LICENSE
14. `.nojekyll` for Pages

---

## Licence

MIT · Copyright (c) 2026 Simon Gant · sjgant80-hub

Seal: **◊·κ=1**
