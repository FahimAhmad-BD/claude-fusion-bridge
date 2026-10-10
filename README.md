# Claude Fusion Bridge

A small Autodesk Fusion add-in that lets Claude Code, or any local script, drive Fusion through a folder. You write a Python job file, the add-in runs it inside Fusion with the full Fusion API, and you get back the printed output, any error traceback and screenshots of the viewport.

It is free to use and needs nothing beyond Fusion itself. It opens no network port, so only programs on the same computer can reach it.

## How it works

```
your tool ──writes──> Documents/ClaudeFusion/bridge/inbox/<job>.py
Fusion add-in ──runs the job on Fusion's main thread──>
           ──writes─> Documents/ClaudeFusion/bridge/outbox/<job>.json   (+ <job>_<name>.png)
           ──moves──> Documents/ClaudeFusion/bridge/done/<job>.py
heartbeat: Documents/ClaudeFusion/bridge/status.json (every 5 s while Fusion runs)
```

Names available inside a job:

| Name | Meaning |
|---|---|
| `adsk`, `app`, `ui` | Fusion API modules and application objects |
| `design()` | The active `adsk.fusion.Design` |
| `snapshot(name="view", width=1600, height=1000, fit=True)` | Saves a PNG of the viewport to the outbox and returns its path |
| `state` | A dict that persists between jobs until Fusion restarts |

The result JSON contains `ok`, `stdout`, `error`, `images`, `started` and `seconds`.

## Install (once)

1. Clone this repository anywhere outside a synced folder, for example `C:\dev\claude-fusion-bridge`.
2. In Fusion: **Utilities > Add-Ins > Scripts and Add-Ins**, click **+** (script or add-in from my computer) and select the `ClaudeFusionBridge` folder.
3. Select **ClaudeFusionBridge**, click **Run**, and turn on **Run on Startup**.

Install only one copy. Two bridge add-ins watching the same inbox would run every job twice.

## Use

From Git Bash (Windows) or any POSIX shell:

```bash
bash client/fusion_run.sh jobs/ping.py
```

It prints the result JSON and exits 0 on success, 1 if the job raised an error, 2 if the bridge isn't running, and 3 on timeout. A modal dialog open in Fusion blocks jobs until it is closed.

To use it from Claude Code, tell Claude where this repository is cloned and ask it to drive Fusion through `client/fusion_run.sh`.

## Safety

Every file placed in the inbox runs with your user rights inside Fusion. Keep the bridge folder local, and only let tools you trust write to it.

## Origin

Written for the Duccky v1.0 robot project and published here as a standalone tool.

Author: Fahim Ahmad. License: Apache 2.0.
