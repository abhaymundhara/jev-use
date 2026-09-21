<p align="center"><img src="docs/banner.png" alt="jev-use" width="100%"></p>

# jev-use

Voice and typed computer use for macOS. You say what you want. Local Laya picks the next on-screen action. macOS performs it. No screenshots: the app reads the screen through the Accessibility tree.

## Quick start

Requires macOS 14.2+ and Xcode. No dependencies.

```sh
bash build.sh
open "$HOME/Applications/Desktop Voice.app"
```

In setup: start the local Laya server, then allow Accessibility, microphone and speech. Laya runs at `http://127.0.0.1:8770` when started from the [Laya project](https://brainfunctioncollapse.com/laya).

Hold **Control–Option–Space**, speak, release. **Escape** cancels. Or type a command in the widget, or from a shell: `scripts/say.sh "Open Finder"`.

## Examples

- "Open Obsidian, create a new note and type hello"
- "Go to youtube.com, search Rick Astley and play the first video"
- "Open 3 new tabs"
- "Scroll down three times"
- "Tile all the Brave windows so none are stacked"
- "In every Brave window, go to wikipedia.org and search for accessibility" — the optional planner works out the steps once, code repeats them in each window
- "Close the window", "Save", "New tab" — any item in the app's menu bar

## How it works

One loop, about 0.3–1.5 s per step:

1. **Read.** Walk the front app's Accessibility tree (~120 ms). Every element describes itself: what it is, its name, its value, where it sits, what it can do. No per-app code.
2. **Choose.** One local request to Laya (`/api/predict`): the goal, the numbered targets, the last ten actions and their effects. Laya selects an operation and a target. It never generates free text; typed text is a span of your sentence.
3. **Act.** Press, select, type, menu, key, scroll, open, arrange windows.
4. **Check.** Read the screen again. Report the real effect. Repeat until DONE, BLOCKED or WAIT.

Low-confidence and destructive picks stop and ask instead of acting.

## What is sent

To local Laya at `http://127.0.0.1:8770/api/predict`: your command, the app and window names, the on-screen targets with their labels and values, and recent actions. Nothing is sent to TypeSafe, and secure text fields are excluded. No screenshots. Speech uses Apple Speech. If the optional OpenRouter planner is enabled, it receives only the spoken text and app names.

Everything is logged locally: `log show --predicate 'subsystem == "local.jev-use"' --last 10m --info`

## Develop

```sh
swift test        # 10 tests
bash build.sh     # quit the app first
```
