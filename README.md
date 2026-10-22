# Crassidens

A native macOS app for running a fleet of CLI coding agents in parallel, each in its own git worktree, with an iOS companion. Heavily inspired by the feature set and developer experience of [Orca](https://github.com/stablyai/orca).

> *Pseudorca crassidens*, the false killer whale: a smaller, more agile relative of the orca.

## Intent

I like Orca, but it isn't 100% what I want. Crassidens is my own build of the same idea, focused on:

1. **Worktree-based agent views:** one view per worktree with its agents, terminals, diff, browser and Linear issue.
2. **A built-in browser:** shared by the user and the agents, with Design Mode and agent automation.
3. **Built-in Linear:** issues become worktrees, and issue status follows the work.
4. **Auto wiki:** a generated, continuously updated wiki of each repository, readable by people and agents.
5. **PR insights:** for any change, a summary of its intent, its system design impact and its risk. GitHub PR review is built around these summaries.

Only the Orca features the specification lists are in scope.

It is native Swift on macOS and iOS only. It is not a fork: Orca is the reference for behaviour and UX. Orca is MIT-licensed, so its code may be reused as long as its copyright notice is kept.

**Status:** the specification and system design are proposed. There is no code yet.

## Docs

- [Specification](docs/specification.md): what Crassidens does, with requirement IDs, tiers, the differences from Orca, and open questions.
- [System design](docs/system-design.md): how it is built, with processes, protocol, components, milestones, risks and dated decisions.

## Reference: Orca

- Repo: [stablyai/orca](https://github.com/stablyai/orca), v1.4.237 (2026-10-21) at time of writing. Docs: [onorca.dev/docs](https://www.onorca.dev/docs). Its `AGENTS.md` is worth reading.
- Orca's agent model: any CLI agent that runs in a terminal (Claude Code, Codex, OpenCode, …) is an agent. Users bring their own subscriptions, and no models are bundled.

## For agents picking this up

1. Read the specification, then the system design. Requirement IDs (WS-1, PI-2, …) are the shared vocabulary, so reference them in issues, commits and PRs.
2. Work milestone by milestone ([system design §14](docs/system-design.md#14-milestones)). Start with the M0 spikes, because several design choices depend on their results.
3. Record new decisions in [system design §16](docs/system-design.md#16-decisions) with a date. Don't guess at open questions; ask me.
