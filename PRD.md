# RoboOps — Product Requirements Document
**Version:** 1.0  
**Date:** 14 October 2026  
**Target implementation:** macOS desktop companion  
**Coding environment:** OpenCode using the GPT Sol model

## 1. Product summary

RoboOps is an always-visible, expressive 3D robot companion for a developer's macOS desktop. It observes a supported coding-agent session, communicates the agent's real state through animated robot behavior, and requests approval for commands that are not covered by explicitly trusted rules.

The robot is the ambient user experience. A native macOS app owns the UI, state, settings, and notifications. A local integration/broker communicates with the coding agent and enforces command approvals. The terminal and agent remain responsible for coding work; RoboOps must not pretend to be the agent or invent progress.

**Core product promise:** See what your coding agent is doing at a glance, and safely approve commands without constantly switching back to Terminal.

## 2. Product goals

1. Keep a small, expressive robot visible during normal desktop work.
2. Make agent states understandable without reading logs.
3. Provide fast, actionable macOS notifications for approval requests.
4. Auto-allow only commands covered by narrow, explicit trusted rules; ask for approval for unknown or higher-risk actions.
5. Keep command execution and approval enforcement outside the UI, in a local control layer.
6. Make the MVP usable with OpenCode and an explicitly integrated terminal/script workflow.
7. Keep local coding workflows functional without a cloud dependency.

## 3. Non-goals for MVP

- Building a new coding agent or terminal emulator.
- Supporting every coding agent automatically.
- Inferring commands by scraping terminal screen text and treating that as a security boundary.
- A native WidgetKit widget; the floating companion is the primary surface.
- Cloud accounts, team collaboration, telemetry backend, or remote phone approval.
- Elaborate real-time 3D rendering if lightweight sprite/Lottie animation can meet quality and performance needs.

## 4. Target user and core use case

**Primary user:** A developer running OpenCode or another supported agent in a terminal on macOS.

**Main scenario:**
1. User starts an agent session through a supported integration.
2. RoboOps displays the robot and the current session state.
3. The robot animates based on actual agent events.
4. Commands covered by trusted policy may run without repeated prompts.
5. Unknown or restricted commands are held by the integration layer.
6. RoboOps shows a macOS notification and an in-app approval card.
7. User approves once, rejects, or opens a detailed review.
8. RoboOps records the decision and confirmed outcome.

## 5. Platform and recommended implementation

Build a native macOS application using **Swift + SwiftUI**, with AppKit where needed for window behavior.

- **Floating window:** `NSPanel` or another suitable AppKit window configuration, with SwiftUI content.
- **Robot animation:** Prefer sprite/image sequences, Lottie, or a lightweight 3D renderer. Choose the simplest reliable approach that supports action-specific loops and transparent backgrounds.
- **Notifications:** `UserNotifications`, with registered action categories where supported.
- **Local IPC:** Unix domain socket or another authenticated local IPC mechanism.
- **Persistence:** SQLite or another reliable local store for sessions, pending requests, trusted rules, and audit events.
- **Agent integration:** A versioned adapter interface. Implement OpenCode first where supported by its current hooks/events/permissions. If a capability is not supported, document it and use an explicit wrapper/script rather than faking support.
- **Model:** The developer is using GPT Sol through OpenCode. Treat this as the agent/model configuration, not as an assumption that the model itself provides a specific API. Keep the integration model-agnostic.

Before implementation, inspect the existing repository and follow its current architecture, package manager, conventions, and test setup. Do not replace working infrastructure without a clear reason.

## 6. Main UX surfaces

### 6.1 Compact floating companion (default)

A small, borderless or visually minimal floating window that stays above normal app windows where macOS permits it.

**Layout**
- Default footprint target: approximately 150–190 px wide by 130–180 px tall, adapting to display scale.
- Cute white 3D robot with glossy dark faceplate and cyan/blue glowing eyes.
- Dark navy visual language with electric-blue accents; state colors should be consistent.
- Small state label and dot: Working, Thinking, Needs You, Completed, Error, Disconnected, Paused, or Idle.
- No large permanent text panel.
- Click robot to expand; drag to reposition.
- Hover may reveal subtle controls; do not show a permanent toolbar.

**Window behavior**
- Do not steal keyboard focus on state updates or when notifications arrive.
- Remember position and size between launches.
- Recover gracefully if the saved position is outside the current display bounds.
- Respect hide/show and full-screen preferences. Be honest that macOS may prevent visibility above certain system/full-screen surfaces.
- Offer menu-bar controls for Show/Hide, Pause/Resume where supported, Settings, and Quit.
- Never intercept normal typing or block important app content.

### 6.2 Expanded companion panel

Target width: 340–400 px; height adapts to screen.

Show:
- Session status and connection state.
- Agent name, project directory, and branch when available.
- Current task or latest agent-provided summary.
- Elapsed time only when reliably measured.
- Recent event feed with timestamps.
- Pending approval card when applicable.
- Controls: Open Terminal, Pause/Resume if supported, Stop session, Settings.
- Clear error/reconnect information.

Do not invent task progress percentages, completion estimates, commands, or events. Distinguish agent-reported facts from app-derived facts.

### 6.3 Command review

Every review must show:
- Exact command or structured executable + arguments.
- Working directory.
- Agent-provided reason, if available.
- Risk classification and why it was assigned.
- Potential side effects where known; otherwise say “Effects not verified.”
- One-time Approve and Reject actions.
- Optional separate “Create trusted rule” action, never selected by default.
- Copy command and open relevant terminal/session controls.

High-risk actions must require a more deliberate in-app confirmation and must not be approved with a single notification tap.

### 6.4 Settings

Sections:
- **Appearance:** size, animation intensity, theme, reduced motion.
- **Position:** display, preferred corner, opacity if supported, always-on-top preference.
- **Behavior:** start at login, show/hide behavior, full-screen handling, menu-bar presence.
- **Notifications:** approval, error, completion, disconnect, quiet hours.
- **Trusted commands:** list/add/edit/revoke rules and scope.
- **Security:** execution policy, isolation options, audit retention, log redaction.
- **Agent connection:** OpenCode integration status, project/session details, reconnect.
- **Privacy:** local-only operation, clear history, diagnostic logging toggle.

### 6.5 Activity/audit history

Each record should include:
- Timestamp, session ID, project and agent.
- Exact command/action or a safe redacted representation.
- Decision source: trusted rule, user approval, rejection, or system block.
- Rule ID/version where applicable.
- Execution status, exit code, and relevant output where available.
- Any confirmed side effects, when known.

Keep history local by default. Provide configurable retention and a clear-history action. Never log secret values unnecessarily.

## 7. Robot visual identity and animation

### Art direction
- Cute, expressive, premium 3D desk-pet robot.
- White rounded shell, glossy black faceplate, bright cyan/blue eyes, small antenna, blue-lit details.
- Dark navy background or transparent background where feasible.
- Consistent proportions, lighting, camera angle, and scale across all states.
- Motion should be readable at small floating-window sizes.
- Use the supplied reference image as the character/style source if it is available in the project. Do not substitute unrelated robot designs.
- Prefer real action animation to static artwork with only a whole-image bob. If only static assets exist, implement an interim loop and document the limitation.

### Required states and actual actions
1. **Working — “ACTIVE”**
   - Furiously types on a tiny laptop: rapid alternating hand/arm motion, slight body bounce, occasional screen/key glow.
   - Loop should feel energetic but not frantic or distracting.
2. **Thinking — “IDLE” or “THINKING”**
   - Raises a hand to chin, glances upward, gently shifts posture.
   - Thought bubble above/right of head with three dots that animate sequentially.
3. **Needs You — “APPROVAL”**
   - Stops typing, turns attention toward the user, raises a hand or leans forward.
   - Orange speech/thought bubble with an exclamation mark pulses gently.
   - Must remain visually distinct until the pending request is resolved.
4. **Completed — “SUCCESS”**
   - Jumps or lifts arms in celebration; confetti falls or pops around it.
   - Play once or briefly, then transition to Idle/Ready. Avoid endless celebratory motion.
5. **Error — “ERROR”**
   - Looks worried/surprised; red warning symbol appears and gives a short pulse.
   - Do not imply an error if only an ordinary command returned a nonzero exit code; use actual error state semantics.
6. **Disconnected — “OFFLINE”**
   - Slumps or falls asleep; floating Z symbols rise and fade.
   - Clearly indicates the connection is lost, not that the agent is safely stopped.
7. **Paused — “PAUSED”**
   - Stops typing, rests hands on/near laptop, eyes close or become calm.
   - Blue pause symbol appears. Only show if pause has actually been confirmed.
8. **Idle — “READY”**
   - Calm sitting pose, subtle breathing, occasional blink and small glance.
   - No busy typing while there is no active work.

### Animation implementation requirements
- Use a finite-state animation controller keyed to the real session state.
- Each state should have a seamless loop where appropriate; Completed and some alert transitions may be one-shot.
- Keep animation frame rate and CPU usage reasonable. Pause or reduce animation when hidden, inactive, or Reduce Motion is enabled.
- Support accessibility/reduced-motion mode using static poses and restrained transitions.
- State changes must interrupt or transition from the prior animation predictably.
- Do not use a generic “typing” animation for Thinking, Idle, Disconnected, or Paused.
- Use transparent assets where possible so the robot can sit naturally on the desktop.

## 8. State model and transitions

Use explicit states:
- `idle`
- `working`
- `thinking`
- `approvalPending`
- `executingApprovedCommand`
- `paused`
- `completed`
- `error`
- `disconnected`

Keep connection state and session state separate internally. For example, a disconnected session may have had a last-known state of Working, but the UI must display the data as stale.

Expected transitions:
- Session starts → Working or Thinking only after a valid event.
- Agent reports progress → update feed and state based on the event.
- Command request arrives → evaluate trusted policy.
- Trusted policy matches → broker authorizes the exact request and records the decision.
- No trusted rule / policy requires review → hold command, create pending approval, show Needs You and notify.
- User approves → broker revalidates request ID, command, context, expiry, and policy before authorization.
- User rejects → invalidate request and return rejection to agent/integration.
- Request expires or command changes → invalidate; never execute based on stale approval.
- Agent reports task finished and process outcome confirms it → Completed.
- Agent or process fails → Error with truthful explanation.
- Connection lost → Disconnected; do not assume running child processes stopped.
- User requests stop → request termination, then report confirmed outcome only.
- Completed celebration → transition to Idle after a short one-shot animation if no new work exists.

## 9. Command approval and safety

### Default policy
- Auto-allow only narrow, explicitly configured trusted commands.
- Ask for approval for unknown commands.
- Block or require strong confirmation for destructive, privileged, sensitive-data, or production-impacting operations.
- A user's approval of one command does not automatically authorize future commands.

### Risk tiers
- **Trusted/low risk:** explicitly allowlisted, narrow commands such as selected read-only inspection or test/lint commands, within defined scope.
- **Review:** unknown commands, package installs, configuration changes, external network access, or actions outside the usual project scope.
- **Restricted/high risk:** deletion, force pushes, privileged execution, credential access, production changes, or sensitive data transfer. Block by default or require an explicit in-app confirmation according to policy.

Risk should not be assigned by command name alone. Consider arguments, working directory, executable resolution, environment, scripts, and side effects when possible.

### Non-negotiable controls
1. The UI is not the security boundary. Enforce policy in the broker/execution path.
2. Bind approvals to an immutable request ID and exact command/arguments, working directory, session, and relevant execution context.
3. If the command or context changes, invalidate the approval.
4. Reject duplicate, replayed, expired, or already-resolved approvals.
5. Prefer structured process execution over interpolated shell strings.
6. If a shell is required, treat shell syntax as high risk and handle it explicitly.
7. Use least privilege. Do not run the broker as root.
8. Do not assume a command can be intercepted just because its terminal output can be read.
9. Require a supported integration or explicit wrapper that places every relevant command behind the broker. If OpenCode cannot expose a required capability, document the limitation rather than pretending it is supported.
10. Redact secrets from notification text, activity feeds, and logs.
11. Keep the local workflow local by default.
12. If the broker is unavailable, fail closed for commands that require approval; do not silently bypass the gate.

### Trusted rule design
Each rule should define:
- Executable identity/path or supported command type.
- Allowed argument pattern with a narrow, reviewable scope.
- Project/directory scope.
- Optional environment/network constraints if enforceable.
- Persistence: session-only or persistent.
- Creation source, created timestamp, and last-used timestamp.
- Edit/revoke controls.

Avoid broad rules like “allow all shell commands,” “allow all npm,” or wildcards that make dangerous arguments trusted. A one-time approval must not create a permanent rule automatically.

## 10. Notification design

Use native macOS notifications with registered action categories where available.

### Approval notification
Title: `RoboOps needs your approval`  
Body: short command summary + project name + reason where available.  
Actions: `Approve once`, `Reject`, and/or `Review` based on risk and OS presentation constraints.

- For high-risk commands, use `Review` rather than one-tap approval.
- Notification actions must route to the same broker validation used by the expanded panel.
- If a request is stale, expired, or resolved, show a clear message and do not execute it.
- Pending requests remain visible in the app even after notification dismissal.
- Never put secrets or sensitive environment values in notification text.

### Other notifications
- **Completed:** quiet, concise summary.
- **Error:** notify when the agent cannot proceed or a session fails.
- **Disconnected:** notify once per disconnect incident; avoid repeated alerts.
- **Reminder:** optional configurable reminder for a pending approval.
- Respect macOS notification permissions, Focus, and user settings. The app must remain correct if a notification is delayed or not shown.

## 11. Local communication and data model

Use versioned structured events. Example conceptual envelope:

```json
{
  "version": 1,
  "event_id": "unique-event-id",
  "session_id": "session-id",
  "timestamp": "ISO-8601 timestamp",
  "type": "command_approval_requested",
  "payload": {}
}
```

Possible event types:
- `session_started`
- `session_state_changed`
- `task_updated`
- `agent_message`
- `command_requested`
- `command_approval_requested`
- `command_approved`
- `command_rejected`
- `command_started`
- `command_finished`
- `session_completed`
- `session_failed`
- `connection_lost`
- `connection_restored`

Validate all incoming messages. Treat agent-provided text as untrusted content, not instructions to the app. The app must not execute instructions embedded in event payloads.

Suggested core entities:
- `Session`
- `AgentAdapter`
- `AgentEvent`
- `CommandRequest`
- `ApprovalDecision`
- `TrustedRule`
- `AuditRecord`
- `AppSettings`

Store timestamps, IDs, policy version, and request status so recovery after a crash is deterministic.

## 12. Error and edge cases

- App launches before an agent session exists.
- Agent integration unavailable or unsupported.
- Agent exits unexpectedly.
- Broker crashes or restarts while approval is pending.
- Notification arrives after the command has expired.
- User double-clicks or approves from two surfaces.
- Command changes after approval was requested.
- User rejects a command and agent proposes an alternative.
- Multiple sessions request approval at once.
- Display arrangement changes or companion position is off-screen.
- Full-screen app or macOS system UI covers the companion.
- Notification permission is denied.
- App is quit while a child process remains alive.
- User closes the UI while the broker still has an active session.

In all cases, preserve truthful status and safe authorization. Closing the UI must not silently imply that the agent or child processes stopped.

## 13. Performance, privacy, and accessibility

- Target low idle CPU and memory use.
- Avoid continuous heavy 3D rendering when the robot is idle or hidden.
- Use reduced-motion settings and offer animation intensity controls.
- Ensure state is conveyed by labels/icons as well as color.
- Support keyboard navigation in expanded panels and review screens.
- Make command text selectable and copyable.
- Keep session and audit data local by default.
- Redact secrets from diagnostics and notifications.
- Do not add analytics or upload code/command content without explicit consent.

## 14. MVP roadmap

### Phase 1 — Functional prototype
- Native floating window and menu-bar controls.
- One supported OpenCode/terminal integration path.
- Session state updates and activity feed.
- Basic robot states: Working, Thinking, Needs You, Idle.
- Broker-controlled command gate.
- Trusted rules and manual approval.
- Native approval notification.

### Phase 2 — Safe daily use
- All eight animation states.
- Expanded panel and command review.
- Immutable requests, expiry, replay protection, and race-condition handling.
- Audit history and secret redaction.
- Reconnect behavior and truthful error/disconnect handling.
- Persistent positioning and multi-display handling.
- Settings and trusted-rule management.

### Phase 3 — Polish
- Action-specific animated assets with real motion, not only whole-image bobbing.
- Reduced-motion mode and performance tuning.
- Better risk explanations and per-project policies.
- Quiet hours and configurable notifications.
- Packaging, signing/notarization investigation, and clean install/uninstall flow.

### Later
- Additional agent adapters.
- Multiple concurrent sessions.
- Optional WidgetKit status widget.
- More detailed summaries and configurable robot personality.

Timeline estimates should be set after inspecting the repository and confirming the actual OpenCode integration capabilities.

## 15. Definition of done

### UI
- Robot remains visible across normal app switching.
- Robot can be dragged and does not get stranded off-screen.
- Compact and expanded views work reliably.
- No status update steals keyboard focus.
- Each state has a distinct, correct animation and label.
- Thinking includes an animated three-dot thought bubble.
- Working shows active typing; Idle and Disconnected do not pretend to type.

### Command safety
- Unknown commands cannot execute before approval.
- Trusted rules are narrow, inspectable, and revocable.
- Approval authorizes only the exact request.
- Expired, changed, duplicate, or replayed approvals cannot execute.
- High-risk actions use stronger review.
- The broker fails closed if authorization cannot be verified.

### Reliability
- App distinguishes agent failure, command failure, and lost connection.
- Completion is shown only when confirmed.
- Pending requests survive UI dismissal and recover safely after restart.
- Logs and notifications do not expose secrets.
- Local workflow does not require cloud connectivity.

## 16. Implementation instructions for OpenCode / GPT Sol

Work in small, verifiable steps. Do not attempt to generate the entire application in one pass.

1. Inspect the repository, identify the app type, toolchain, current structure, and tests. Summarize the findings briefly.
2. Propose a concise implementation plan with files/modules and key risks.
3. Implement the smallest vertical slice: native floating robot window, one mocked session-state source, and a clean state-driven UI.
4. Add unit tests for state transitions and policy decisions before wiring real command execution.
5. Implement the OpenCode adapter only against capabilities confirmed in the installed/current OpenCode version. If uncertain, inspect available documentation or local interfaces; do not invent hook names or API contracts.
6. Implement the local broker and command approval path. Keep authorization logic separate from UI callbacks.
7. Add notification actions and make them call the same validated broker API as in-app approval.
8. Add trusted rules, audit persistence, expiry/replay protection, and failure handling.
9. Add the animated robot states and reduced-motion support.
10. Run tests, build, and lint where available. Report what passed, what remains incomplete, and any macOS behavior that requires manual testing.

Implementation guardrails:
- Do not fake command interception.
- Do not bypass approvals when integration fails.
- Do not execute shell commands directly from a view or notification callback.
- Do not hardcode secrets.
- Do not claim a feature is complete unless it works and has been tested.
- Prefer simple, maintainable modules over premature abstractions.
- Ask for clarification only when a genuinely blocking product or security decision cannot be resolved from this PRD.

## 17. Suggested repository structure

Adapt to the existing repository instead of forcing this layout.

```text
RoboOps/
  App/
    RoboOpsApp.swift
    AppDelegate.swift
    WindowController.swift
  UI/
    Companion/
      CompactCompanionView.swift
      ExpandedCompanionView.swift
      RobotStateView.swift
    Approval/
      CommandReviewView.swift
    Activity/
      ActivityFeedView.swift
    Settings/
      SettingsView.swift
  Animation/
    RobotAnimationController.swift
    Assets/
  Domain/
    Session.swift
    AgentEvent.swift
    CommandRequest.swift
    ApprovalDecision.swift
    TrustedRule.swift
  Integration/
    AgentAdapter.swift
    OpenCodeAdapter.swift
    TerminalWrapperAdapter.swift
  Broker/
    CommandBroker.swift
    PolicyEngine.swift
    ApprovalValidator.swift
    ProcessRunner.swift
  Notifications/
    NotificationManager.swift
  Persistence/
    Database.swift
    AuditRepository.swift
    SettingsRepository.swift
  Tests/
    StateMachineTests.swift
    PolicyEngineTests.swift
    ApprovalValidatorTests.swift
```

## Final product principle

**RoboOps is a real-time, local-first desktop companion with a delightful robot personality and a strict command-authorization boundary.** The UI should be charming; the execution path must be boring, explicit, auditable, and safe.
