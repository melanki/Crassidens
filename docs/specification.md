# Crassidens — Specification

**Status:** proposal, 2026-10-22. How it is built: [system-design.md](system-design.md).

Crassidens is a native macOS app for running many CLI coding agents in parallel, each in its own git worktree. [Orca](https://github.com/stablyai/orca) (v1.4.237) is the reference for the core experience. This document lists what Crassidens must do and where it deliberately differs.

## 1. Product focus

1. **Worktree-based agent views.** The app is organised around worktrees. Each worktree has one view holding its agents, terminals, diff, browser and Linear issue.
2. **Built-in browser.** Agents and the user share a browser per worktree, with Design Mode and agent automation.
3. **Built-in Linear.** Issues become worktrees, and issue status follows the worktree's lifecycle.
4. **Auto wiki.** A generated, continuously updated wiki of each repository, readable by both people and agents. Orca has nothing like this.
5. **PR insights and PR tooling.** For any change, a summary of its intent, its system design impact and its risk. GitHub PRs are reviewed through these summaries. Orca has nothing like this.

Items 1–3 are **Core**. Items 4–5 are **Next**. Only the Orca features this spec lists are in scope; the rest are left out on purpose (§6).

## 2. Scope

**In:** macOS app, iOS companion, a CLI that drives the app, and an optional relay/push server.

**Out:**

- Windows, Linux, Android.
- Bundled models or model routing. Agents use the user's own subscriptions, and so do the wiki and insights (§4.4).
- Being an agent. Crassidens hosts CLI agents and doesn't replace them.
- Cloud-hosted agents and multi-user collaboration.
- A general-purpose IDE. The editor is for review and small edits, with no LSP in v1.

**Platforms:** macOS 26+ on Apple silicon, iOS 26+.

## 3. Concepts

| Term | Meaning |
|---|---|
| Repository | A local git repo added to Crassidens. |
| Workspace | One unit of work: a worktree, its branch, its sessions, its browser and an optional linked Linear issue. |
| Agent profile | How to launch one CLI agent: command, arguments, environment, and how its state is reported. |
| Session | A process running in a PTY (an agent or a shell). Owned by the session host, not the UI. |
| Annotation | A review comment on a diff line, queued to be sent to a session. |
| Wiki | Generated documentation for one repository, pinned to a commit. |
| Insight | A generated analysis of one change (a base…head range): intent, system design impact, risk. |
| Host / Companion | The Mac running the sessions / the iOS app paired with it. |

## 4. Functional requirements

IDs are stable; reference them from issues and PRs.

### 4.1 Core

#### Workspaces and agent views

- **WS-1** Add a repository by path. Detect its default branch and remote.
- **WS-2** Create a workspace from a name, a prompt or a Linear issue (LN-2).
  - Crassidens creates a branch and a worktree under a configurable root, by default `~/Development/worktrees/<repo>/<branch>`.
  - The base branch is selectable and defaults to the freshly fetched default branch.
- **WS-3** Per-repo settings live in `.crassidens.yml` at the repo root:
  - a setup script, run after a worktree is created
  - a teardown script, run before a worktree is removed
  - a list of gitignored files to copy into each new worktree (e.g. `.env`)
  - default sessions

  If the file is absent, Crassidens reads Orca's `orca.yaml` and `.worktreeinclude` instead.
- **WS-4** Removing a workspace removes its worktree. It refuses if there are uncommitted changes, unless forced. Deleting the branch is optional.
- **WS-5** **Workspace view.** One view per workspace combining:
  - its sessions (terminal tabs and splits)
  - its diff
  - its browser
  - its linked issue

  The layout is saved per workspace. Switching workspace switches the whole view.
- **WS-6** **Overview.** Every workspace across all repos, shown as a grid of live session previews with agent state, unread state, diff stats and PR state. From the overview you can focus a workspace, reply to a waiting agent or archive a workspace without opening it.
- **WS-7** Sessions survive app quit, crash and update. A session ends only when closed or when the process exits. After a reboot, each workspace restores its scrollback and offers to relaunch its agents.

#### Agents

- **AG-1** Any CLI agent can run through an agent profile. Built-in profiles: Claude Code, Codex, OpenCode. Custom profiles are a command plus arguments and environment.
- **AG-2** Each session has a state:
  - `working`
  - `needs-input`, with a reason (permission or question)
  - `done`
  - `exited(code)`

  The state comes from the agent's own hooks where the agent supports them, and from terminal signals otherwise ([system-design §5](system-design.md#5-agent-state)).
- **AG-3** Start an agent in a workspace with an initial prompt. If the workspace has a linked issue, the prompt is pre-filled from it.

#### Terminal

- **TM-1** GPU-rendered terminal with tabs per workspace and unlimited horizontal and vertical splits.
- **TM-2** Scrollback is held by the session host and survives app restarts (WS-7).
- **TM-3** Find in scrollback, clickable links, OSC 8 hyperlinks, bracketed paste and true colour.

#### Review

- **RV-1** A per-workspace diff against the merge base, covering both committed and uncommitted changes, with a file tree and inline or side-by-side layout.
- **RV-2** Annotate diff lines, then send the selected annotations to a session as a single message with file and line references.
- **RV-3** Stage, commit and push from the app, and open a PR on GitHub.
- **RV-4** Edit files in place with autosave.

#### Browser

- **BR-1** One browser per workspace, inside the workspace view. Each workspace has its own cookies and storage. Localhost URLs printed by a session are offered as one-click opens.
- **BR-2** Design Mode: picking an element sends its HTML, computed CSS and a cropped screenshot into a session's prompt.
- **BR-3** Agents can drive the browser through the CLI:
  - `open`, `snapshot` (the page as text, with element references)
  - `click`, `fill`, `press`
  - `screenshot`
  - `logs` (console output and failed requests)
- **BR-4** The user can watch agent-driven browsing live and take over at any point.

#### Linear

- **LN-1** Browse Linear in the app: my issues, team cycles, project boards and search.
- **LN-2** Create a workspace from an issue. The branch uses Linear's suggested branch name, the issue is linked to the workspace, and its description is offered as the agent's initial prompt.
- **LN-3** An issue panel in the workspace view shows the description, comments and status, and you can comment from it.
- **LN-4** Issue status follows the workspace: started → In Progress, PR opened → In Review, PR merged → Done.
  - The status mapping is set per team, and each transition can be turned off.
  - By default only "started" is on. Linear's own GitHub integration already handles the PR transitions for teams that use it.
- **LN-5** Create an issue from the app, optionally seeded from an annotation or a browser selection.

#### Notifications, navigation and CLI

- **NT-1** Send a macOS notification when a session moves to `needs-input` or `done`. This is configurable per profile. Clicking the notification focuses the session.
- **NT-2** Each session has an unread state. A session can be marked unread manually.
- **NT-3** Quick open searches workspaces, files, sessions, Linear issues and commands.
- **CL-1** The `crassidens` CLI controls the running host:
  - workspaces: create, list, remove
  - sessions: spawn, send input, status
  - browser (BR-3)
  - wiki (WK-4)
  - agent hooks (AG-2)
- **CL-2** Agents running inside sessions can use the CLI. The environment identifies their workspace and session, and an agent skill file describing the commands ships with the app.

### 4.2 Next

#### PR insights

- **PI-1** For any change (a workspace diff, a local branch or a GitHub PR), generate an insight with three parts:
  - **Intent:** what the change is for and why. Inferred from the diff, commit messages, PR description and linked Linear issue.
  - **System design impact:**
    - which components are affected, linked to their wiki pages (WK-1)
    - changes to public interfaces, data or schema, and configuration
    - how well the change fits existing patterns
    - any new abstractions or dependencies
    - whether it shifts how the code is structured or coupled
  - **Risk:** `low`, `medium` or `high`.
    - Every reason cites evidence (a file and line range), for example migrations, auth or security-sensitive code, concurrency, or untested changed code.
    - Ends with the hunks to review first, in order.
- **PI-2** The insight renders above the diff in the review view. Every claim links to the hunk it refers to.
- **PI-3** Insights are cached per (base, head, prompt version). When head moves, the insight is marked stale and regenerates on request or automatically (configurable).
- **PI-4** Posting an insight as a GitHub PR comment or a Linear comment is an explicit user action. It is never automatic.

#### GitHub PR tooling

- **GH-1** A PR inbox shows open PRs across the added repos that I authored or that request my review. Each row shows the insight's intent line and its risk level.
- **GH-2** The PR view leads with the insight (PI-1): intent, then system design impact, then risk with its review focus. The full diff comes after it, in the RV-1 layout. Insights for PRs requesting my review are generated when the PR is opened, or ahead of time (configurable).

GitHub issues and project boards are out of scope.

#### Auto wiki

- **WK-1** Generate a wiki for each repository from its default branch. It has:
  - an overview
  - an architecture page
  - one page per major component
  - key flows
  - a glossary

  Every claim links to source (a file and line range) at the commit the page was generated from. Diagrams are Mermaid.
- **WK-2** The wiki stays current. When the default branch moves, only the pages whose source files changed are regenerated. Each page shows its commit and whether it is stale.
- **WK-3** Browse and search the wiki in the app, with diagrams rendered.
- **WK-4** Agents can read the wiki through `crassidens wiki search|show`, so they can use it as context without re-reading the codebase.
- **WK-5** The wiki and insights are generated by running one of the user's agent profiles headless and read-only. Each run records the profile, the duration and the cost reported by the agent.

#### Files and previews

- **FP-1** A file explorer per workspace. Dragging a file or image onto a session inserts it into the prompt.
- **FP-2** Previews for Markdown (rich editing), images and PDF.

### 4.3 Later

- **CP-1** iOS companion, paired by scanning a QR code shown on the Mac.
- **CP-2** The companion can:
  - list workspaces and sessions with their state
  - view live terminal output
  - send input and follow-up prompts
  - answer permission requests
  - read insights
- **CP-3** Push notifications for the same state changes as NT-1.
- **CP-4** Works outside the local network through a relay. Traffic is end-to-end encrypted, so the relay can't read session content.
- **US-1** Show Claude Code and Codex usage with rate-limit reset times, and switch accounts.
- **RM-1** SSH workspaces: the worktree and sessions run on a remote host, while the UI stays local.

## 5. Non-functional requirements

- **Performance:** cold start to an interactive window in under 1 s. Heavy output in one session must not stall input or rendering in any other. Wiki and insight runs never block the UI.
- **Reliability:** a UI crash never kills a session (WS-7). App updates restart the UI but leave the session host running.
- **Security:**
  - Tokens live in the Keychain.
  - The local IPC socket is accessible to the current user only.
  - Wiki and insight runs can't write to the repository.
  - Companion traffic is end-to-end encrypted.
  - No telemetry.
- **Accessibility:** keyboard-first. VoiceOver covers all non-terminal UI.
- **Localisation:** English only.

## 6. Divergences from Orca

| Area | Orca | Crassidens |
|---|---|---|
| Scope | A broad feature set | The five focus areas, plus only the Orca features this spec lists. Left out: fan-out and orchestration, computer use, GitHub issues and project boards, Jira |
| Auto wiki | — | WK-1…5 |
| PR insights | — | PI-1…4 |
| GitHub | PR status, checks, comments, merge, issues, Projects | PR tooling built on insights (GH-1, GH-2) |
| Linear | Issues, comments, projects, views; create a worktree from an issue | The same, plus status that follows the workspace (LN-4) |
| Agent hooks | Written into the user's global agent config | Injected per session where the agent allows it |
| Push notifications | The push service sees titles and bodies in plaintext | End-to-end encrypted and decrypted on the phone |
| Runtime | Electron, TypeScript | Native Swift with AppKit/SwiftUI, plus web views only where the web is clearly better |
| Platforms | macOS, Windows, Linux, iOS, Android | macOS and iOS |
| Embedded browser | Chromium, driven over CDP | WebKit (`WKWebView`), driven by injected scripts |
| Relay | Required for the companion | Optional. A direct connection works on the local network or Tailscale |

## 7. Open questions

- **Wiki storage:** local only (proposed), or committed to the repo (for example `docs/wiki/`) so it is versioned and shared?
- **Default profile for wiki and insights:** which agent and model? Cost scales with repo size for WK-1 and with diff size for PI-1.
- **GitHub reviews:** should the PR tooling also submit reviews (line comments, approve, request changes) from the app, or only read, apart from PI-4?
- Which agent profiles ship beyond the three in AG-1?
- License.
- A short alias for the `crassidens` CLI.
