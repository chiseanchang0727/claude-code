# Query Loop

Reference: [src/query.ts](../../src/query.ts)

`query()` is the core agentic loop that QueryEngine delegates to. It's a `while(true)` async generator that repeats until the model stops calling tools or a limit is hit. Each iteration = one API round-trip + tool execution.

## Contents

- [execution-flow.md](./execution-flow.md) — State machine pattern, full 12-step execution flow, iteration diagrams, state management, exit reasons
- [async-prefetch.md](./async-prefetch.md) — Start async work early, consume when ready, never block
- [withhold-then-recover.md](./withhold-then-recover.md) — Hold back recoverable errors, attempt recovery, only surface if all fails
- [tombstone.md](./tombstone.md) — Retract orphaned messages when streaming fails and falls back to non-streaming
- [streaming-tool-execution.md](./streaming-tool-execution.md) — Tools start executing while the model is still streaming
- [yieldMissingToolResultBlocks.md](./yieldMissingToolResultBlocks.md) — Safety net ensuring every tool_use gets a matching tool_result
- [stop-hooks.md](./stop-hooks.md) — What fires after the model stops (memory extraction, auto-dream, prompt suggestion)
- [token-budget.md](./token-budget.md) — Nudge-to-continue logic with diminishing returns detection
- [tool-orchestration.md](./tool-orchestration.md) — Sequential fallback for tool execution with concurrency partitioning
