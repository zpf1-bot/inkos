# AGENTS.md - InkOS Development Guide

This document provides guidelines for agents working on the InkOS codebase.

## Project Overview

InkOS is a multi-agent novel production system written in TypeScript. It consists of:
- `@actalk/inkos-core`: Core library with models, agents, pipeline, and LLM integration
- `@actalk/inkos`: CLI tool for running the novel production pipeline

## Build Commands

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm build

# Build a specific package
cd packages/core && pnpm build
cd packages/cli && pnpm build

# Development mode (watch)
pnpm dev

# Type checking
pnpm typecheck
```

## Test Commands

```bash
# Run all tests
pnpm test

# Run tests in a specific package
cd packages/core && pnpm test
cd packages/cli && pnpm test

# Run a single test file
pnpm vitest run src/__tests__/logger.test.ts
cd packages/core && npx vitest run src/__tests__/logger.test.ts

# Run tests in watch mode
cd packages/core && npx vitest
```

## Lint Commands

```bash
# No ESLint configured - typecheck is the primary validation
pnpm typecheck
```

## Code Style Guidelines

### TypeScript Configuration

- Target: ES2022
- Module: Node16 with `.js` extension in imports
- Strict mode: enabled
- Always use explicit return types for exported functions

### Imports

- Use ES modules with `.js` extension (e.g., `import { X } from "./x.js"`)
- Group imports: external packages first, then internal modules
- Use `import type` for type-only imports when possible

```typescript
import { z } from "zod";
import type { Logger } from "../utils/logger.js";
import { createLogger } from "../utils/logger.js";
```

### Naming Conventions

- **Files**: kebab-case (e.g., `state-manager.ts`, `detection-runner.ts`)
- **Classes**: PascalCase (e.g., `BaseAgent`, `PipelineRunner`)
- **Functions/variables**: camelCase (e.g., `createLogger`, `chatCompletion`)
- **Interfaces/types**: PascalCase with descriptive names (e.g., `AgentContext`, `LLMResponse`)
- **Constants**: camelCase (e.g., `minLevel`, `maxTokens`)

### Type Annotations

- Use `readonly` for function parameters that should not be mutated
- Use explicit return types for all exported functions
- Prefer interfaces for object shapes, type aliases for unions/primitives
- Use `zod` for runtime validation schemas

```typescript
export interface AgentContext {
  readonly client: LLMClient;
  readonly model: string;
  readonly projectRoot: string;
}

export function createLogger(options: {
  readonly tag: string;
  readonly sinks: ReadonlyArray<LogSink>;
}): Logger { ... }
```

### Error Handling

- Use descriptive error messages in Chinese (project target audience)
- Wrap LLM errors with context using `wrapLLMError()` function
- Use custom error classes for specific error types
- Include helpful troubleshooting suggestions in error messages

```typescript
export class PartialResponseError extends Error {
  readonly partialContent: string;
  constructor(partialContent: string, cause: unknown) {
    super(`Stream interrupted after ${partialContent.length} chars: ${String(cause)}`);
    this.name = "PartialResponseError";
    this.partialContent = partialContent;
  }
}
```

### Logging

- Use the built-in logger utility (`createLogger`)
- Use appropriate log levels: debug, info, warn, error
- Include context objects for debugging

```typescript
logger.info("starting chapter write", { chapterId, title });
logger.warn("detection failed, skipping", { error: String(err) });
```

### Schema Validation

- Use Zod for all configuration and data validation
- Export both schema and inferred type
- Use descriptive error messages

```typescript
export const BookConfigSchema = z.object({
  id: z.string().min(1),
  title: z.string().min(1),
  platform: PlatformSchema,
});

export type BookConfig = z.infer<typeof BookConfigSchema>;
```

### Testing

- Test files: `src/__tests__/**/*.test.ts`
- Use Vitest with `describe`, `it`, `expect`
- Use `vi.useFakeTimers()` for time-dependent tests
- Mock external dependencies appropriately

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";

describe("createLogger", () => {
  beforeEach(() => { vi.useFakeTimers(); });
  afterEach(() => { vi.useRealTimers(); });

  it("creates a logger with correct behavior", () => {
    const sink = createStderrSink({ minLevel: "info" });
    expect(sink).toBeDefined();
  });
});
```

### General Patterns

- No semicolons at end of statements
- Use arrow functions for callbacks
- Prefer `for...of` over `for...in`
- Use nullish coalescing `??` and optional chaining `?.`
- Use template literals for string interpolation

```typescript
const level = options.minLevel ?? "info";
const content = chunks.join("");
const entry = { ...baseCtx, ...ctx };
```

### Agent Development

Agents should extend `BaseAgent` and implement:
- `name` getter property
- Core logic using `this.chat()` or `this.chatWithSearch()`

```typescript
export abstract class BaseAgent {
  protected readonly ctx: AgentContext;

  constructor(ctx: AgentContext) {
    this.ctx = ctx;
  }

  protected async chat(
    messages: ReadonlyArray<LLMMessage>,
    options?: { readonly temperature?: number; readonly maxTokens?: number },
  ): Promise<LLMResponse> { ... }

  abstract get name(): string;
}
```

## Package Structure

```
packages/core/
├── src/
│   ├── agents/       # AI agent implementations
│   ├── llm/         # LLM provider abstraction
│   ├── models/      # Data schemas and types
│   ├── notify/     # Notification integrations
│   ├── pipeline/   # Core pipeline logic
│   ├── state/      # State management
│   ├── utils/      # Utilities
│   └── __tests__/  # Test files
└── vitest.config.ts

packages/cli/
├── src/
│   └── index.ts    # CLI entry point
└── vitest.config.ts
```

## Environment Variables

Create `.env` based on `.env.example`:

```bash
INKOS_LLM_PROVIDER=openai
INKOS_LLM_API_KEY=your-api-key
INKOS_LLM_BASE_URL=https://api.openai.com/v1
INKOS_LLM_MODEL=gpt-4o
```

## Common Tasks

### Adding a new agent

1. Create `src/agents/my-agent.ts`
2. Extend `BaseAgent` class
3. Add exports to `src/index.ts`
4. Add tests in `src/__tests__/`

### Adding a new model/schema

1. Create `src/models/my-model.ts`
2. Define Zod schema and TypeScript type
3. Export from `src/models/index.ts` (if exists) and `src/index.ts`

### Running the CLI

```bash
cd packages/cli
pnpm build
node dist/index.js --help
```
