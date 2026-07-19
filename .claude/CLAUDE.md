# Mabinogi Mobile automation

Move the **마비노기 모바일** (Mabinogi Mobile) window between the physical main
monitor and a BetterDisplay virtual monitor on this Mac, so the game keeps
running off-screen when not in use.

## When the user talks about the window — run `scripts/mabinogi-window`

Anything the user says about where the window sits ("마비노기 status /
background / foreground", "where is mabinogi", "move / park / bring back
mabinogi", "show / check mabinogi") is served by the self-contained
`scripts/mabinogi-window` script. Pick the subcommand from the request:

- **Wants a specific state** ("park it" / background, "bring it back" / foreground)
  → run `scripts/mabinogi-window background` or `... foreground` directly, no
  question. Moves are idempotent — a no-op (beyond re-focusing for foreground)
  when the window is already there.
- **Wants to see it without moving it** ("show / check mabinogi") → run
  `scripts/mabinogi-window screenshot`: captures just the game window (by window
  id — no desktop) and opens the PNG in Preview, without moving or focusing it.
  Works even while the window is parked off-screen.
- **Asks vaguely where it is / whether to move it** → **report-then-flip**:
  1. `scripts/mabinogi-window status` — report which monitor the window is on.
  2. `AskUserQuestion` — offer to move to the *other* monitor (option labelled
     from the current location) or leave it unchanged.
  3. On "move", run the opposite desired state — `scripts/mabinogi-window
     foreground` if it is on the virtual monitor, `... background` if on the
     main monitor; on "leave", nothing.

The script has no `toggle`: every command is either a **read** (`status`,
`screenshot`) or a **set-desired-state** (`foreground`, `background`). To flip,
read the state, then set the opposite — never an imperative flip.

Full subcommand list: `status` (which monitor), `screenshot` (capture just the
game window, open in Preview, no move), `foreground` (main, focused),
`background` (virtual, no focus). Aliases: `st`, `shot` / `ss`, `fg`, `bg`. The
rest of this file holds the environment, prerequisites, and mechanism the script
relies on.

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
`background` / `offscreen`, and `move_to <location>` sets the window to a desired
state (idempotent — a no-op when already there). `screenshot` composes
`window_id` + `screencapture -l` to grab just the app window. There is no flip
primitive: callers read with `status`, then set the opposite with `move_to`.

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
