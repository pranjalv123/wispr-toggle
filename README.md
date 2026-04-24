# wispr-toggle

Toggle [Wispr Flow](https://wisprflow.ai/) hands-free mode from a single command — ideal for binding to a mouse button.

## Why

Wispr Flow ships with keyboard shortcuts for hands-free mode but no built-in way to toggle it from an arbitrary input device (e.g. a side button on a Logitech mouse). This wraps the app's `wispr-flow://` deeplinks behind a one-shot toggle.

## How it works

The app exposes `wispr-flow://start-hands-free` and `wispr-flow://stop-hands-free` deeplinks, but each only works in one direction — `start` no-ops unless the app is idle, `stop` no-ops unless it's currently in hands-free. To make a real toggle:

1. Read the latest `updateDictationStatus: <state>` line from `~/Library/Logs/Wispr Flow/main.log`.
2. If the state is `idle` / `dismissed`, fire `start-hands-free`. Otherwise, fire `stop-hands-free`.
3. Use `open -g` so Wispr Flow doesn't steal focus or unminimize its hub window.

## Install

```sh
git clone https://github.com/pranjalv123/wispr-toggle.git ~/ramp/wispr-toggle
ln -s ~/ramp/wispr-toggle/wispr-toggle ~/bin/wispr-toggle   # optional, if ~/bin is on $PATH
```

## Usage

### From the shell

```sh
wispr-toggle
```

### From a mouse button (Logi Options+, SteerMouse, etc.)

Most mouse drivers that offer "Run Shell Command" launch it inside Terminal.app, which is noisy. Instead, point the driver at the bundled app:

```
~/ramp/wispr-toggle/WisprToggle.app
```

It's a zero-UI wrapper (`LSUIElement=true`) that runs the script and exits. On first launch macOS may show a Gatekeeper prompt since the app is unsigned — right-click → Open once to approve.

## Caveats

- **Log lag (~45ms).** Status detection reads the app log; rapid double-clicks can race and double-fire `start`. A single click is fine.
- **Push-to-talk ambiguity.** If you press the toggle *while* holding push-to-talk, the script will call `stop-hands-free`, which no-ops (PTT isn't hands-free mode). The PTT keeps running; no harm done.
- **Floating dictation bubble.** Wispr Flow's on-screen dictation indicator still appears — that's the feature itself, not something this script can suppress.

## Deeplink reference

Documented for completeness; extracted from the app's main bundle.

| Deeplink | Effect |
| --- | --- |
| `wispr-flow://start-hands-free` | Start hands-free (requires state `idle`/`dismissed`) |
| `wispr-flow://stop-hands-free` | Stop hands-free (requires state `listening` + locked) |
| `wispr-flow://switch-mic?mic_name=<name>` | Switch active microphone |
| `wispr-flow://open` | Focus the hub window |
| `wispr-flow://open/<Page>` | Open a specific page (e.g. settings) |
