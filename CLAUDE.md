# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the Claude Code CLI source — a TypeScript application built with Bun that provides an AI-powered coding assistant in the terminal. It uses a custom fork of Ink (React for terminals) for its TUI.

## Build System

- **Runtime:** Bun
- **Language:** TypeScript with TSX (React components for terminal UI)
- **Feature gating:** Uses `bun:bundle` feature flags for dead code elimination at build time (e.g., `COORDINATOR_MODE`, `KAIROS`, `AGENT_TRIGGERS`)
- No `package.json` or `tsconfig.json` tracked in this repo — build infrastructure is external

## Architecture

### Core Loop

`src/QueryEngine.ts` is the central orchestrator — it manages the AI conversation loop, tool use/results, context management, message compaction, and permissions. All AI interactions flow through here.

### Entry Points

- `src/entrypoints/cli.tsx` — CLI bootstrap with fast paths (--version, --dump-system-prompt)
- `src/main.tsx` — Main program setup via Commander.js with 100+ CLI options, session management, feature detection
- `src/entrypoints/mcp.ts` — MCP server entry point

### Tool System (`src/tools/`)

45+ tools each in their own directory with a consistent structure:
- `*Tool.ts` — Tool implementation
- `prompt.ts` — Tool description/prompt for the AI
- `constants.ts` — Tool name constants
- `UI.tsx` — React component for terminal rendering

Central registry in `src/tools.ts` composes the tool set based on feature flags and permissions.

### Terminal UI (`src/ink/`)

Custom Ink fork providing React-based terminal rendering: components (Box, Text, Button, ScrollBox), event system (input, click, focus), layout engine (yoga-based), ANSI rendering, and vim keybinding support.

### Task System (`src/Task.ts`, `src/tasks/`)

Manages background work with typed tasks: `local_bash`, `local_agent`, `remote_agent`, `in_process_teammate`, `dream`. Task IDs use type prefixes (b=bash, a=agent, r=remote). States: pending → running → completed/failed/killed.

### Services (`src/services/`)

- **API:** Claude AI client, OAuth, bootstrap
- **MCP:** Client, registry, server approval
- **Analytics:** Growthbook feature gates, Datadog, event logging
- **Specialized:** LSP, voice (STT/TTS), token estimation, history compaction, memory extraction

### State & Context

- `src/state/AppState.tsx` / `AppStateStore.ts` — Root state with listener pattern
- `src/context.ts` — Git status, CLAUDE.md discovery, memory files, system context injection
- `src/context/` — React contexts for notifications, modals, overlays, prompts, voice

### Remote/Bridge (`src/bridge/`, `src/remote/`, `src/server/`)

Enables cloud execution (Claude Code Remote Control): WebSocket sessions, JWT auth, backoff/retry, capacity management, SDK message adapter.

### Skills (`src/skills/`)

Bundled skills in `src/skills/bundled/` (batch, claude-api, loop, remember, schedule, simplify, etc.). Loaded from user directories as well.

### Hooks (`src/hooks/`)

~87 React hooks for: file suggestions, keybindings, permissions (`useCanUseTool`), IDE integration, notifications, background task polling, auto-updates.

## Key Patterns

- **Feature gating:** Many subsystems are compile-time eliminated via `feature()` from `bun:bundle`. Check feature gates before assuming code is active.
- **Lazy imports:** Circular dependencies avoided via lazy `require()` — especially in tools, team operations, and message selection.
- **Dual mode:** Interactive TUI (default) vs headless/print mode (`-p`/`--print`) with different initialization paths.
- **Context injection:** System prompt is composed from multiple sources: CLAUDE.md files, memory files, git status, tool descriptions, feature-dependent sections.
- **Permission model:** Granular tool-level + filesystem-level permissions with enterprise policy support and user denial tracking.
