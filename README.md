# mabinogi-mobile-automation

Park the **마비노기 모바일** (Mabinogi Mobile) window on a BetterDisplay virtual
monitor when idle, and pull it back to the main monitor to play — one command
each way.

```sh
# Read the state
scripts/mabinogi-window status       # (st)        which monitor is it on?
scripts/mabinogi-window screenshot   # (shot, ss)  capture just the game window, open in Preview (no move)

# Set the desired state (idempotent)
scripts/mabinogi-window foreground   # (fg)        -> main monitor, focused
scripts/mabinogi-window background   # (bg)        -> virtual monitor (keep running off-screen)
```

There is no `toggle`: to flip, read the state, then set the opposite.

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

### 4. (Optional) Enable zsh tab-completion

Run the installer to complete subcommands (`status`, `screenshot`, `foreground`,
`background`, and their aliases) after the command:

```sh
scripts/install-completion
exec zsh          # or open a new terminal
```

```sh
scripts/mabinogi-window <TAB>   # -> status  st  screenshot  shot  ss  foreground  fg  background  bg
```

What it does:

- Symlinks `completions/_mabinogi-window` into `~/.local/share/zsh/site-functions/`
  (a directory zsh loads completion functions from). The repo stays the source of
  truth; re-running just refreshes the link.
- Clears the cached `compdump` so the next shell rescans and registers the new
  completion. This matters because `compinit` only picks up new completions when
  its cache is stale — [prezto](https://github.com/sorin-ionescu/prezto), for
  example, reuses a dump under `~/.cache/prezto/` for 20 hours and otherwise
  never rescans. The installer clears both the prezto and plain-zsh caches.

If it warns that the target directory is not on `$fpath`, add the line it prints
to your `~/.zshrc` (before `compinit` runs) and re-run.

In Claude Code, `.claude/CLAUDE.md` routes window requests ("where is mabinogi",
"park it", "bring it back", "show mabinogi") straight to this script and
describes the report-then-flip behaviour. See it for the agent-facing signals and
the full mechanism.
