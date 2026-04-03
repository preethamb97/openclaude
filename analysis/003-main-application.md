# 003 - Main Application Entry Point (`src/main.tsx`)

## Overview

The `src/main.tsx` file is the **heart of OpenClaude**. It's a large file (~4000+ lines) that orchestrates everything: command parsing, session management, provider initialization, and the TUI (Terminal User Interface).

## Key Responsibilities

### 1. Command-Line Argument Parsing
```typescript
// Parses arguments like:
// openclaude --model sonnet --provider openai
// openclaude -p "explain this code"
// openclaude --resume abc123
```

### 2. Provider Initialization
- Initializes AI providers (OpenAI, Gemini, Ollama, etc.)
- Sets up the smart router
- Handles provider credentials

### 3. Session Management
- Creates and manages conversation sessions
- Handles session resume
- Manages session persistence

### 4. TUI Launch
- Creates the React/Ink TUI
- Sets up the REPL (Read-Eval-Print Loop)
- Handles user input

## Main Function Flow

```typescript
export async function main() {
  // 1. Security checks
  process.env.NoDefaultCurrentDirectoryInExePath = '1'
  
  // 2. Initialize warning handler
  initializeWarningHandler()
  
  // 3. Parse command-line arguments
  // ... extensive argument parsing
  
  // 4. Initialize providers
  await init()
  
  // 5. Run the main logic
  await run()
}
```

## Command-Line Options

The main function handles many CLI options:

| Option | Description |
|--------|-------------|
| `-p, --print` | Print response and exit (non-interactive) |
| `--model <model>` | Specify AI model (e.g., 'sonnet', 'gpt-4') |
| `--provider <provider>` | Specify provider (openai, gemini, ollama) |
| `--resume [id]` | Resume a conversation |
| `--continue` | Continue most recent conversation |
| `--mcp-config <file>` | Load MCP servers from JSON |
| `--dangerously-skip-permissions` | Bypass all permission checks |
| `--bare` | Minimal mode (skip hooks, LSP, plugins) |
| `--settings <file>` | Load settings from JSON file |

## Provider Initialization

```typescript
// Example: Initialize with OpenAI
const provider = 'openai'
const apiKey = process.env.OPENAI_API_KEY

// The smart router will select the best available provider
const router = new SmartRouter()
await router.initialize()
```

## Session Management

### Creating a New Session
```typescript
const sessionId = randomUUID()
// Session is stored in ~/.claude/sessions/
```

### Resuming a Session
```typescript
// Load conversation history
const messages = await loadConversationForResume(sessionId)
// Continue from where you left off
```

## Error Handling

The main function has extensive error handling:

```typescript
try {
  await run()
} catch (error) {
  if (error instanceof Error) {
    logError(error)
    // Show user-friendly error message
  }
}
```

## Key Imports

```typescript
// Core functionality
import { Command as CommanderCommand } from '@commander-js/extra-typings'
import chalk from 'chalk'
import React from 'react'

// Provider management
import { initializeGrowthBook } from './services/analytics/growthbook.js'
import { fetchBootstrapData } from './services/api/bootstrap.js'

// Session management
import { loadConversationForResume } from './utils/conversationRecovery.js'

// TUI components
import { renderAndRun } from './interactiveHelpers.js'
import { launchRepl } from './replLauncher.js'
```

## How It All Fits Together

1. **User runs**: `openclaude --model sonnet`
2. **CLI loads**: `bin/openclaude` executes
3. **Main starts**: `main()` function runs
4. **Arguments parsed**: Model set to 'sonnet'
5. **Provider initialized**: OpenAI/Anthropic/etc ready
6. **TUI launched**: React/Ink renders the REPL
7. **User interacts**: Types prompts, uses commands
8. **Commands handled**: Slash commands processed
9. **AI responds**: Provider generates responses
10. **Session saved**: Conversation persisted

## Performance Optimizations

- **Profile checkpoints**: Tracks startup performance
- **Parallel initialization**: Multiple providers start concurrently
- **Lazy loading**: Heavy modules loaded on-demand
- **Dead code elimination**: Unused features removed in build

## Debugging

```bash
# Enable debug mode
openclaude --debug

# Debug with specific categories
openclaude --debug api,hooks

# Verbose output
openclaude --verbose
```

## Common Patterns

### Checking Environment
```typescript
if (isEnvTruthy(process.env.CLAUDE_CODE_USE_OLLAMA)) {
  // Ollama-specific logic
}
```

### Feature Flags
```typescript
if (feature('COORDINATOR_MODE')) {
  // Coordinator mode specific code
}
```

### Async Operations
```typescript
// Most operations are async
await init()
await router.initialize()
await launchRepl()
```

## Summary

`src/main.tsx` is the central hub that:
- Parses CLI arguments
- Initializes providers and router
- Manages sessions
- Launches the TUI
- Handles the entire user interaction flow

It's designed to be robust, extensible, and performant, handling everything from simple one-liner queries to complex multi-turn conversations.
</task_progress>
</write_to_file>