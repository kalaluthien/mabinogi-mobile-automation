---
name: positioning-mabinogi-window
description: Report which monitor the 마비노기 모바일 (Mabinogi Mobile) window is on, or move it to a desired state. Takes an optional argument — `foreground` (main monitor, focused) or `background` (virtual monitor, off-screen) — and moves there directly without asking. With no argument, report status then offer to flip. Use for "마비노기 status / background / foreground", "where is mabinogi", or any request to move, park, or bring back the game window.
allowed-tools: Bash(scripts/mabinogi-window status), Bash(scripts/mabinogi-window foreground), Bash(scripts/mabinogi-window background), Bash(scripts/mabinogi-window toggle)
---

# Positioning the Mabinogi Mobile window

One entry point for the whole placement workflow. The window has two locations —
`foreground` (main monitor, focused) and `background` (virtual monitor,
off-screen but still running).

The skill has two modes, chosen by whether an argument is given:

- **Argument given** → move directly to that desired state, no questions.
- **No argument** → report status, then offer to flip.

## Mode A — argument given (desired state)

The argument names the desired end state:

- `foreground` → run `scripts/mabinogi-window foreground`
- `background` → run `scripts/mabinogi-window background`

Run the matching subcommand and report its output verbatim. **Do not call
`AskUserQuestion`.** If the argument is anything else, tell the user the valid
values are `foreground` and `background`, and stop.

If the command errors (game not running, or Accessibility not granted), surface
that error and stop.

## Mode B — no argument (report then offer)

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
