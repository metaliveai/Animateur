---
name: openclaw
description: Work with OpenClaw, the local-first personal AI assistant platform. Use when an agent needs to install, run, debug, configure, modify, test, or explain OpenClaw Gateway, CLI, agents, channels, nodes, skills, plugins, Control UI, WebChat, Canvas/A2UI, source builds, pnpm development workflows, security settings, or OpenClaw skill loading behavior.
---

# OpenClaw

## Start Here

Use local files first when working inside an OpenClaw checkout: `README.md`, `AGENTS.md`, `CONTRIBUTING.md`, `SECURITY.md`, `package.json`, `docs/`, `src/`, `packages/`, `extensions/`, `skills/`, `ui/`, and `apps/`.

If no checkout is present, use the official docs and repository:

- https://github.com/openclaw/openclaw
- https://docs.openclaw.ai

OpenClaw is a long-running Gateway plus agents, channels, tools, nodes, skills, plugins, and UI surfaces. Treat changes as operational software, not a simple library.

## Common Architecture

Keep these concepts straight:

- Gateway: the long-lived control plane and daemon. It owns provider/channel connections and exposes the WebSocket API.
- CLI: `openclaw ...` commands for onboarding, gateway control, messages, agent runs, pairing, skills, diagnostics, and updates.
- Agents: isolated sessions/workspaces that execute tasks and use configured skills/tools.
- Channels: WhatsApp, Telegram, Slack, Discord, Signal, iMessage, WebChat, and similar message surfaces.
- Nodes: macOS, iOS, Android, or headless devices connected to the Gateway with explicit capabilities.
- Canvas/A2UI: visual surfaces served through the Gateway for agent-controlled UI.
- Skills: AgentSkills-compatible folders with `SKILL.md` frontmatter and instructions.
- Plugins: packages that can add tools, hooks, channels, routes, and skills.

One Gateway normally owns a host. Avoid starting duplicate gateways against the same state unless using a dev profile.

## Install And Run

Preferred user install:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

`pnpm` and `bun` install flows may exist, but Node remains the normal daemon runtime. Prefer Node 24. Verify the current minimum in `package.json` before changing Node requirements.

Useful operator commands:

```bash
openclaw onboard --install-daemon
openclaw gateway --port 18789 --verbose
openclaw agent --message "Ship checklist" --thinking high
openclaw doctor
openclaw skills list
```

On Windows, prefer WSL2 for serious Gateway work unless the current docs or user setup says otherwise.

## Source Development

For a local checkout, prefer `pnpm`.

```bash
pnpm install
pnpm openclaw setup
pnpm ui:build
pnpm gateway:watch
```

For a built `dist/`:

```bash
pnpm build
pnpm ui:build
```

Notes:

- `pnpm openclaw ...` runs the TypeScript entry through the repo tooling.
- `pnpm gateway:watch` is the normal dev loop and restarts on relevant source/config changes.
- `pnpm gateway:watch` does not rebuild `dist/control-ui`; rerun `pnpm ui:build` after Control UI changes or use the repo's UI dev command if available.
- Inspect `package.json` before choosing test, lint, formatting, or app build commands. Do not invent scripts.

## Safe Dev Profile

Use a dev profile for debugging to avoid touching the user's real Gateway state.

```bash
pnpm gateway:dev
```

The dev profile uses isolated state such as `~/.openclaw-dev` and a shifted default Gateway port. Prefer it for experiments, channel mocks, plugin debugging, and config migrations.

## Skills In OpenClaw

OpenClaw skills are directories containing `SKILL.md` with YAML frontmatter. Common locations, from highest to lower precedence, include:

- `<workspace>/skills`
- `<workspace>/.agents/skills`
- `~/.agents/skills`
- `~/.openclaw/skills`
- bundled skills
- configured extra directories

When creating or editing OpenClaw skills:

- Use concise frontmatter with at least `name` and `description`.
- Use snake_case names if following OpenClaw docs exactly.
- Keep frontmatter keys single-line unless local parser support proves otherwise.
- Put per-agent skills in that agent's workspace when behavior should not affect every agent.
- Start a new session or restart the Gateway when changes are not picked up automatically.
- Verify with `openclaw skills list`.

Do not mix up "working on OpenClaw source" with "writing a skill for OpenClaw agents". If the user asks for a skill, create a skill folder under the appropriate skills location. If the user asks to change OpenClaw behavior, edit the source/plugin/UI/config surface that owns that behavior.

## Security Posture

OpenClaw can connect to real messaging surfaces and can execute tools on the host. Treat inbound channel messages and group chats as untrusted input.

Before broadening access:

- Check `SECURITY.md` and official security docs.
- Run `openclaw doctor` when changing config or DM/channel policies.
- Preserve default pairing or allowlist behavior for public/direct-message channels unless the user explicitly opts into open inbound access.
- Avoid exposing browser, canvas, nodes, cron, Discord, Slack, gateway, or host execution tools to untrusted sessions without sandboxing.
- Use per-session or non-main sandbox settings for group/channel traffic when available.
- Never log tokens, OAuth data, pairing codes, private channel identifiers, or message content unnecessarily.

## Change Guidance

When changing Gateway/runtime code:

- Find the owning package/module before editing. OpenClaw spans `src/`, `packages/`, `extensions/`, `apps/`, `ui/`, and plugin areas.
- Preserve typed protocol/schema validation when adding WebSocket methods, events, or payloads.
- Add or update tests near the changed behavior when the repo has an existing test pattern.
- Keep migrations backwards-compatible for existing user config/state.

When changing channels:

- Use existing adapter patterns for auth, send/receive, retry, chunking, typing/presence, media handling, and allowlists.
- Preserve pairing/approval behavior for unknown direct-message senders.
- Validate real channel changes with mocks or a dev profile before touching live credentials.

When changing skills or plugins:

- Check plugin metadata, config gates, environment requirements, and skill load precedence.
- Keep skill instructions short and operational.
- Avoid giving skills broad shell instructions that can be exploited by untrusted input.

When changing UI:

- Identify whether the surface is Control UI, WebChat, Canvas/A2UI, or a companion app.
- Rebuild or run the appropriate UI workflow after edits.
- Check Gateway integration, routing/auth assumptions, and WebSocket event handling.

## Validation

Prefer validation that matches the touched surface:

- Install/source setup: `pnpm install`, `pnpm openclaw setup`, `pnpm build`, `pnpm ui:build`.
- Runtime loop: `pnpm gateway:watch` or `pnpm gateway:dev`.
- Diagnostics: `openclaw doctor`.
- Skills: `openclaw skills list`, then a fresh agent session that should trigger the skill.
- Agent path: `openclaw agent --message "..."`.
- Gateway/API path: run the Gateway, connect a local client, and inspect logs/events.
- UI path: run the Gateway/UI dev flow, open the surface, and check browser console plus Gateway logs.

If a command would touch live channels, send messages, install daemons, alter autostart services, or expose remote access, ask the user first unless they explicitly requested that exact operation.

## Boundaries

Do not run onboarding, install daemons, pair devices, send real messages, change remote exposure, or loosen security policy as part of ordinary code inspection. Prefer the dev profile and local test surfaces until the user asks for production setup.
