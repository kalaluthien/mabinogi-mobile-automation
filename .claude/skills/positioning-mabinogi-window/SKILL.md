---
name: positioning-mabinogi-window
description: Report which monitor the 마비노기 모바일 (Mabinogi Mobile) window is on, then offer to flip it between the main monitor (foreground) and the BetterDisplay virtual monitor (background). Use for "마비노기 status / background / foreground", "where is mabinogi", or any request to move, park, or bring back the game window.
allowed-tools: Bash(scripts/mabinogi-window status), Bash(scripts/mabinogi-window toggle)
---

# Positioning the Mabinogi Mobile window

One entry point for the whole placement workflow: **report first, then offer to
flip.** The window has two locations — `foreground` (main monitor, focused) and
`background` (virtual monitor, off-screen but still running).

## Steps

1. **Report status.** Run this and show its output verbatim:

   ```
   scripts/mabinogi-window status
   ```

   The line ends with either `main monitor (foreground)` or
   `virtual monitor (background)`. If it instead errors (game not running, or
   Accessibility not granted), surface that error and **stop** — do not ask the
   question below.

2. **Offer the toggle.** Call `AskUserQuestion` with one single-select question.
   Label the move option with the *opposite* of the current location:

   - currently **foreground** → option 1 = "Send to background (virtual monitor)"
   - currently **background** → option 1 = "Bring to foreground (main monitor)"
   - option 2 = "Leave unchanged"

3. **Act on the choice.**

   - Move option chosen → run `scripts/mabinogi-window toggle` and report its
     output verbatim.
   - "Leave unchanged" → do nothing further; confirm the window was left where it is.
