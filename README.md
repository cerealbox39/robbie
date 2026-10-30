# Robbie

Robbie is a small macOS robot that shows live OpenCode state, tracks useful project time, answers OpenCode questions, and reviews held commands.

## What You Need

- macOS 14 or newer.
- Swift 6 command-line tools.
- OpenCode 1.19. Your shell setup gives plain `opencode` local port `4096` inside `Claude projects`.
- Notification permission if you want approval alerts.

## Run It

These commands use the full folder path, so they work no matter which folder Terminal is currently in.

Open a new Terminal tab inside `Claude projects` and run:

```sh
opencode
```

The shell setup automatically assigns the first free port from 4096 through 4105. Run plain `opencode` in each new Terminal tab. You can still choose ports manually:

```sh
opencode "/path/to/another/Claude project" --hostname 127.0.0.1 --port 4097
opencode "/path/to/a/third/Claude project" --hostname 127.0.0.1 --port 4098
```

Open a second Terminal window and start Robbie:

```sh
swift run --package-path "/Users/brie.chong/Claude projects/a-status-robot/front-end" Robbie
```

Robbie appears near the lower-right corner in a transparent 71 x 72 point window. Drag it anywhere. Click it to open the 330 x 521 Command Center. Robbie scans ports 4096 through 4105 and lists every OpenCode session under `Claude projects`. In VS Code, each row uses its integrated terminal's visible name. If no terminal can be matched, Robbie uses the project folder name. Starting a fresh session replaces that terminal's old idle row; active sessions and pending actions remain visible. Click a session to see its to-do details and timer. Command approvals remain queued and every decision returns to the exact port and session that requested it. Click outside the panel or use yellow to collapse it. Use red to quit Robbie.

Robbie reads live events plus OpenCode's saved questions and permissions. It can recover an action after a missed event or reconnect, including requests from helper sessions on the same port. In the expanded panel, questions show their real choices and optional custom-answer box. Command cards provide Reject, Once, and Always. Always asks for a second confirmation because future matching commands will run without asking.

Only OpenCode processes with a local listening port are visible to Robbie. Restart older plain OpenCode tabs that were opened before this port setup was added.

If a Terminal tab was already open when the setup changed, run `source ~/.zshrc` once before `opencode`. Robbie keeps checking empty ports and connects automatically when OpenCode starts.

Robbie shows Thinking while OpenCode generates or plans, Working while a tool runs, Paused after an interrupted response, Completed after successful output, and Error after failed output. It shows Needs You only for real OpenCode questions and command approvals. Wording in a successful answer never makes Robbie wait for an `ok` reply. Paused, Completed, and Error change to Idle after 10 minutes without new activity. This does not change the separate 15-minute project-time rule.

With several sessions, the small robot shows the most urgent state: Needs You, Error, Paused, Working, Thinking, Completed, Idle, then Disconnected. One unavailable port does not disconnect healthy sessions.

Alert mode has the highest priority. Any command approval or unanswered OpenCode question overrides the selected row and identifies the project and port that need action. When no alert exists, the robot and status pill follow the session row you clicked.

Robbie checks OpenCode's saved status, messages, running tools, and to-do list every second while connected. This repairs missed live events. Completed uses the supplied GIF as its reference: Robbie jumps with raised hands, running legs, a broad smile, and glowing diamond confetti. Reduce Motion keeps the celebration pose still.

The common poses are intentionally different: Completed uses compact-safe outward Y-shaped arms, Working has concentration eyes with a calm smile and laptop, and Thinking follows `robot_thinking.gif` with a hand under its chin, straight mouth, and glowing three-dot thought cloud.

When Robbie starts or reconnects, it restores the newest OpenCode session from `Claude projects` or any child project. A recent successful output returns as Completed, while an active Plan response returns as Thinking. Restarting Robbie does not restart the 10-minute Idle countdown.

When OpenCode has a to-do list, Robbie shows the current step number, task, priority, completed count, percentage, and the latest Started or Completed changes. Sessions without a to-do list show a clear empty message.

The project `opencode.json` asks before all shell commands and blocks a small set of especially dangerous tools. Restart OpenCode after changing this file because OpenCode reads configuration only at startup.

The parent `Claude projects/opencode.json` carries the same shell safety rules so child projects can send approval requests to Robbie. Existing OpenCode sessions must restart after this setting changes.

## Check It

Run the 132 repeatable logic checks:

```sh
swift run --package-path "/Users/brie.chong/Claude projects/a-status-robot/back-end" RobbieCoreChecks
```

Build the app:

```sh
swift build --package-path "/Users/brie.chong/Claude projects/a-status-robot/front-end"
swift build -c release --package-path "/Users/brie.chong/Claude projects/a-status-robot/front-end"
```

## Safety Boundary

OpenCode holds a command when its permission setting is `ask`. Robbie receives that real permission request and sends OpenCode `once`, `always`, or `reject` only after its own broker validates the exact request. Always requires a second confirmation.

Robbie does not intercept commands typed directly into Terminal. It also cannot review an OpenCode action configured as `allow`. Do not start OpenCode with `--auto` when using Robbie for approvals.

The local `Robbie Terminal Names` VS Code helper shares only each integrated terminal's visible name and shell process ID. It writes `~/Library/Application Support/Robbie/vscode-terminals.json`, sends nothing over the network, and never changes approval routing.

## Local Data

The redacted audit file is stored at `~/Library/Application Support/Robbie/audit.json`. Timer data is stored at `~/Library/Application Support/Robbie/timers.json`. Robbie has no analytics service and does not upload these files.
