# Mabinogi Mobile automation

Move the **마비노기 모바일** (Mabinogi Mobile) window between the physical main
monitor and a BetterDisplay virtual monitor on this Mac, so the game keeps
running off-screen when not in use.

Anything the user says about where the window sits ("마비노기 status /
background / foreground", "where is mabinogi", "move / park / bring back
mabinogi") routes to the one `positioning-mabinogi-window` skill under
`.claude/skills/`. That skill has two modes, chosen by whether it gets an
argument naming the desired state:

- **`foreground` / `background` argument** (aliases `fg` / `bg`,
  case-insensitive) → move directly to that state, no question
  (`positioning-mabinogi-window background` parks it, `... foreground` brings it
  back).
- **No argument** → **report-then-flip**:
  1. `scripts/mabinogi-window status` — report which monitor the window is on.
  2. `AskUserQuestion` — offer to flip to the *other* monitor (option labelled
     from the current location) or leave it unchanged.
  3. On "flip", `scripts/mabinogi-window toggle`; on "leave", nothing.

Direct, non-interactive subcommands still exist for scripting: `status`,
`foreground` (main, focused), `background` (virtual, no focus), `toggle` (flip
based on current location). This file holds the environment, prerequisites, and
mechanism the skill relies on.

## Environment (verify before relying on it — displays/IDs change on reconnect)

- **App**: 마비노기 모바일 — an iOS-app-on-Apple-Silicon wrapper.
  - Process name: `ProductName` (the wrapper's generic name).
  - Bundle id: `com.nexon.devcat.mm`.
  - Target it by process name `ProductName`; identity is stable while it runs.
- **Displays** (logical points, as of 2026-07-19):
  - Main `LG HDR 4K` — origin (0,0), 1920×1080 pts (4K panel, HiDPI-scaled).
  - Virtual `Virtual 16:9` (BetterDisplay) — origin (1920,0), 1920×1080 pts,
    placed to the **right** of main. This is the "background" parking spot.
- The script reads live display bounds via CoreGraphics (`swift`) every run, so
  it survives resolution/arrangement changes: "main" = the OS main display,
  "background" = the first secondary (virtual) display. It does **not** hardcode
  the (1920,0) offset.

## Prerequisite — Accessibility permission (one-time, manual)

Moving another app's window uses macOS System Events, which requires
**Accessibility** permission for whatever runs the script (the terminal —
**iTerm2** here). Without it the script exits with a clear error.

- Grant: **System Settings → Privacy & Security → Accessibility → enable iTerm2**
  (and Terminal, if used). Then re-run.
- This cannot be scripted around; it is an intentional macOS security gate. If a
  move fails with "assistive access" / error `-1719`, this is the cause.

## How it works (mechanism, for when the script needs changing)

1. `swift` + CoreGraphics `CGDisplayBounds` → each display's origin/size in points.
2. Pick the target rectangle (main display, or first secondary for background).
3. System Events `set position of window` to center the window on that rectangle;
   `set frontmost` only for foreground.

The script is a small state machine over these primitives: `load_window_frame`
reads the window's position, `locate_center` maps it to `foreground` /
`background` / `offscreen`, `move_to <location>` performs a move, and `toggle`
composes read + locate + move to flip to the opposite side. `status` and
`toggle` are the two the skill calls.

Do not reach for BetterDisplay's CLI to move the window — BetterDisplay manages
*displays*, not app *windows*. The virtual monitor is BetterDisplay's job; moving
the window across it is System Events' job.

## BetterDisplay setup (context)

- BetterDisplay.app is installed at `/Applications/BetterDisplay.app`.
- A virtual screen "Virtual 16:9" (3840×2160 native, 1920×1080 pts) is enabled,
  not mirrored, arranged to the right of the main display.
- The `betterdisplaycli` command-line tool is **not** installed. It is not needed
  for window moving; install it (BetterDisplay menu → "Install Command Line
  Tool") only if you later want to script the virtual display's lifecycle.
