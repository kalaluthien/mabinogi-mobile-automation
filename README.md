# mabinogi-mobile-automation

Park the **마비노기 모바일** (Mabinogi Mobile) window on a BetterDisplay virtual
monitor when idle, and pull it back to the main monitor to play — one command
each way.

```sh
scripts/mabinogi-window background   # -> virtual monitor (keep running off-screen)
scripts/mabinogi-window foreground   # -> main monitor, focused
scripts/mabinogi-window status       # which monitor is it on?
```

## Setup

### 1. Create the BetterDisplay virtual screen

Install [BetterDisplay](https://github.com/waydabber/BetterDisplay), then add a
virtual screen and configure it as below.

![BetterDisplay virtual screen configuration](docs/images/betterdisplay-virtual-screen.png)

- **Connect this virtual screen** — on. (Can be toggled on demand any time.)
- **Virtual screen name** — `Virtual 16:9` (any name; used only for
  identification).
- **Aspect ratio** — `16 x 9`, so the parked window keeps the game's shape.
- Leave *Enable resolutions over 8K*, *Use custom resolution list*, and *Rotated
  orientation* **off** — the defaults are fine and 8K+ virtual screens can cause
  stability issues.

### 2. Arrange it as a secondary display

In **System Settings → Displays → Arrange…**, place `Virtual 16:9` to the
**right** of the main display (do *not* mirror). This is the off-screen parking
spot; the script reads live geometry, so the exact position only needs to be a
separate, non-mirrored secondary — it does not have to be pixel-exact.

![macOS Arrange Displays with Virtual 16:9 to the right](docs/images/macos-arrange-displays.png)

### 3. Grant Accessibility to your terminal

**System Settings → Privacy & Security → Accessibility → enable iTerm2** (and
Terminal, if you use it). Required for the script to move another app's window;
without it you get a clear error (`assistive access` / error `-1719`).

Mabinogi Mobile runs as an iOS-app wrapper (process `ProductName`, bundle
`com.nexon.devcat.mm`). The script reads live display geometry via CoreGraphics,
so it adapts when you rearrange or rescale monitors.

See `.claude/CLAUDE.md` for the agent-facing signals ("마비노기 background/
foreground") and the full mechanism.
