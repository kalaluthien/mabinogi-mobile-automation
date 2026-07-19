# Mabinogi Mobile automation

Move the **마비노기 모바일** window between the main monitor and a BetterDisplay
virtual monitor, so the game keeps running off-screen when idle.

## Window requests → `scripts/mobinogi-window`

Anything about where the window sits ("where is mabinogi", "park it", "bring it
back", "show mabinogi") routes to this self-contained script. Subcommands:
`status`/`st`, `screenshot`/`shot`/`ss`, `foreground`/`fg`, `background`/`bg`.

- **Wants a state** → run `foreground` (main, focused) or `background` (virtual,
  no focus). Idempotent: a no-op when already there.
- **Wants to see it** → `screenshot`: captures just the game window, opens it in
  Preview, no move. Works while parked.
- **Vague** → report-then-flip: `status`, then `AskUserQuestion` offering to move
  to the other monitor or leave it; on move, run the opposite state.

No `toggle` — commands are reads (`status`, `screenshot`) or set-desired-state
(`foreground`, `background`). To flip, read then set the opposite.

## Gotchas

- Moves need **Accessibility permission** for the terminal (System Settings →
  Privacy & Security → Accessibility). Error `-1719` / "assistive access" means
  it is missing; cannot be scripted around.
- **Never move the window with BetterDisplay's CLI** — it manages displays, not
  windows. Moving the window is System Events' job.

Setup and app/display/mechanism details live in `README.md` and the script's own
header comments — read `scripts/mobinogi-window` before changing it. Display
IDs/bounds change on reconnect; verify before relying on them.
