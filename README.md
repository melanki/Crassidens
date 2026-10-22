# Crassidens

A desktop agentic development environment (ADE) for running a fleet of CLI coding agents in parallel, each in its own git worktree. Heavily inspired by the feature set and developer experience of [Orca](https://github.com/stablyai/orca).

> *Pseudorca crassidens*, the false killer whale: a smaller, more agile relative of the orca.

## Intent

I like Orca, but it isn't 100% what I want. Crassidens is my own build of the same idea: match Orca's core feature set and DX, then diverge where it falls short for me.

This is not a fork. Orca is the reference for behaviour and UX. It is MIT-licensed, so its code may be reused as long as its copyright notice is kept.

**Status:** empty. No stack chosen, no code yet.

## Reference: Orca

- Repo: [stablyai/orca](https://github.com/stablyai/orca), v1.4.237 (2026-10-21) at time of writing. Docs: [onorca.dev/docs](https://www.onorca.dev/docs). Its `AGENTS.md` is worth reading.
- Stack: Electron, React 19, Vite, Tailwind 4, xterm.js 6 (WebGL), Monaco, Tiptap, ssh2, Playwright. pnpm workspace.
- Agent model: any CLI agent that runs in a terminal (Claude Code, Codex, OpenCode, …) is an agent. Users bring their own subscriptions, and no models are bundled.

## Feature set

These are Orca's features, grouped into tiers. The tiers are a proposal, so confirm with me before reordering them.

### Core

- **Workspace per task:** each task gets a git worktree, a terminal and an agent, and one sidebar tracks them all.
- **Fan-out:** one prompt goes to N agents in separate worktrees. Compare the results and merge the winner.
- **Terminal:** WebGL rendering, unlimited splits, scrollback that survives restarts.
- **Agent-agnostic:** any CLI agent can be used, with a launch command configured per agent.
- **Diff review:** an inline diff per worktree. Annotate diff lines, send the comments back to the agent, and commit from the app.
- **Notifications:** alert when an agent finishes or needs input, and let threads be marked unread to come back to later.
- **Quick open:** search across worktrees, files, agents and commands.

### Next

- **Editor:** file explorer and Monaco editor with autosave. Drag files or images into an agent prompt.
- **Previews:** Markdown, images, PDFs.
- **CLI:** agents can drive the app themselves. Orca's commands include `orca worktree create`, `snapshot`, `click` and `fill`.
- **GitHub and Linear:** browse PRs, issues and boards in-app, and open a worktree from any task.
- **Accounts:** account switcher, plus a view of Claude and Codex usage and rate-limit resets.

### Later

- **Browser:** embedded Chromium with Design Mode. Clicking a UI element sends its HTML, CSS and a cropped screenshot into the agent prompt.
- **SSH worktrees:** agents run on a remote machine, with auto-reconnect and port forwarding.
- **Computer use:** agents can operate desktop apps.
- **Mobile companion:** monitor agents, get notified, and send follow-ups from a phone.

## Where Crassidens diverges from Orca

**Not yet defined.** This is why the project exists, so don't guess: ask me. Fill in this section as decisions are made, and date each one.

## Open decisions

- **Stack:** Orca's stack is the default. Record a reason here for any deviation.
- **Platforms:** macOS only at first, or cross-platform from day one?
- **License.**
- **CLI name.**

## For agents picking this up

1. Read this file, then Orca's README and `AGENTS.md` for reference behaviour.
2. Before designing anything that depends on how Crassidens diverges from Orca, ask me to fill in that section.
3. Record decisions here with dates. Move them to `docs/decisions/` once there are more than a handful.
4. Start with the Core tier.
