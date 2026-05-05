# Snap-Cut HMI — Project Context

## What is this?
Industrial HMI (Human-Machine Interface) for a plastic pellet cutting machine.
Two cutting units (Front / Rear), each with a blade, home sensor, end sensor.
The machine cuts plastic rods into pellets (spheres). Output goes into a basket.

## Target Hardware
- Screen: 7" touchscreen, **1024×600 px**, physical size **15.5 × 8.7 cm**
- Browser: Chromium kiosk mode (no chrome, fullscreen)
- The HMI must fit exactly on this screen — no scrolling

## Files
| File | Purpose |
|------|---------|
| `hmi2.html` | **Main source** — always edit this |
| `index.html` | Copy of hmi2.html for GitHub Pages — always run `cp hmi2.html index.html` after edits |
| `logo.svg` | Primary logo (blade from right) |
| `logo-dark.svg` | White wordmark for dark backgrounds |
| `logo-ltr.svg` | Blade from left variant |
| `logo-b.svg` | Sphere-as-hero concept |
| `logo-green.svg` | Green recycling variant (fixed: single smooth arc ring) |
| `logo-pro.svg` | Premium variant (dome highlight, seam line, richer sphere) |
| `logo-preview.html` | Shows all logos side by side |

## GitHub
- Repo: `git@github.com:elileon/snap.git` (main branch)
- GitHub Pages: serves `index.html` from root of main branch

## Screen Sizing Logic (`hmi2.html`)
```
html, body: normal flow, overflow-x hidden
.hmi-wrapper: width = calc(1024px × --hmi-scale), height = calc(600px × --hmi-scale)
.hmi-frame: 1024×600px, transform: scale(--hmi-scale), transform-origin: top left
```
- `fitHMI()` script: loads saved scale from `localStorage['hmi-dev-scale']`, defaults to `min(1.0, viewport-fit)`
- Sim bar has `−` / `%` / `+` buttons (±5%) to calibrate once on dev screens → saved to localStorage
- On actual 7" device: scale = 1.0 = exactly 15.5 cm

## HMI Architecture
### State machine
`state` ∈ `{idle, running, fault, estop}`

### Key variables
```javascript
let state = 'idle';
let sensorOn = true;
let fCuts = 0, rCuts = 0, totalCuts = 0, basketCuts = 0;
let fBladeCuts = 0, rBladeCuts = 0;   // blade life counters
let woSec = 0, basketSec = 0;          // timers in seconds
let faultUnit = '';                     // 'FRONT' | 'REAR' | ''
let basketNum = 1;
```

### Global Settings (`gSet`)
```javascript
const gSet = {
  cutTimeout:     { val: 800,   unit: ' ms'   },
  retractTimeout: { val: 1500,  unit: ' ms'   },
  bladeLimit:     { val: 10000, unit: ' cuts' },
  bladeWarn:      { val: 10,    unit: '%'     },
  basketTare:     { val: 5.8,   unit: ' kg'   },
  basketRemind:   { val: 60, min: 5, max: 480, step: 5, unit: ' min' },
};
```

### Timer loop
- `tick()` runs every 1000ms: increments `woSec`, `basketSec`, updates clock
- `simTick()` runs every 200ms: increments cut counters, schedules next cut
- `renderBladeStatus()` called every 25 simTicks

### Weigh reminder
Triggered when `basketSec >= gSet.basketRemind.val * 60` AND state = running.
The "Log Basket" button breathes (CSS animation `basket-breathe`) + bell icon rings (`bell-ring`).

## Navigation / Screens
Left nav bar (icons + labels):
- **RUN** — main operating screen (default)
- **MANUAL** — manual blade jog
- **PRODUCTION** — production stats
- **MAINT.** — maintenance (sensor health, blade life, filter log)
- **SETTINGS** — global settings

## Design Language (ISA-101)
- Light theme is default (was added late in the session)
- Colors: `--cyan` for primary actions, `--green` for running/ok, `--amber` for warnings, `--red` for faults
- Alarm colors NEVER used for decoration
- No blinking except unacknowledged alarms / weigh reminder

## Terminology (Hebrew→English)
| Hebrew | English used in UI |
|--------|--------------------|
| חיתוך | Cut |
| קצב חיתוך / Cut Rate | **Cut Cycle** (everywhere) |
| c/min | c/min (NOT cyc/min) |
| rejects weight | **Cut Pieces Weight** |
| סל | Basket |

## Blade Status
- `fBladeCuts` / `rBladeCuts`: increments on every cut in `simTick()`
- Warning threshold: `bladeLimit × (1 - bladeWarn/100)`
- Rendered by `renderBladeStatus()` (called every 25 ticks)

## Log Basket Modal
- Shows **Front unit cuts only** (`fCuts`), not combined `basketCuts`
- Subtitle: "Front unit · cut pieces weight"
- Resets `basketCuts` and `basketSec` on confirm

## Maintenance Screen
- Sensor health: compact single-row-per-unit layout (Front / Rear), Home | End sensors with `|` separator
- Filter Change Log: capped at 2 entries
- Blade life bars for Front and Rear

## Simulator Bar (dev only, below HMI frame)
Outside `.hmi-frame`, always visible in dev:
- Sensor On/Off, Fault Front/Rear, E-Stop, Reset to IDLE, Open New WO, Weigh Remind, Scale −/+

## Logo Design
Mark: circular dark badge (r=56, center 76,80) containing:
1. **Cylinder** (dark blue gradient) — raw plastic rod input from top
2. **Steel blade** (metallic gradient, enters from right) — the cut
3. **Sphere** (radial gradient) — the clean output pellet

Wordmark: `SNAP·CUT` in Arial Black 900 weight, `·` in cyan/blue accent
Tagline: `SMART CUT. CLEAN DROP.` in spaced small caps

## Common Commands
```bash
# After editing hmi2.html:
cp hmi2.html index.html
git add hmi2.html index.html
git commit -m "description"
git push

# Preview server (already in .claude/launch.json):
# python3 -m http.server 8080 --directory /Users/eli/workspace/snap-cut
```
