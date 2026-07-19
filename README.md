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

1. Enable a BetterDisplay virtual screen (arranged as a secondary display).
2. Grant **Accessibility** to your terminal: System Settings → Privacy &
   Security → Accessibility → enable iTerm2 (and Terminal). Required for the
   script to move another app's window; without it you get a clear error.

Mabinogi Mobile runs as an iOS-app wrapper (process `ProductName`, bundle
`com.nexon.devcat.mm`). The script reads live display geometry via CoreGraphics,
so it adapts when you rearrange or rescale monitors.

See `.claude/CLAUDE.md` for the agent-facing signals ("마비노기 background/
foreground") and the full mechanism.
