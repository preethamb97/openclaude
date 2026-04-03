# 009 - Remaining Files Overview

## Overview

This file provides an overview of the remaining important files in the OpenClaude codebase that weren't covered in detail in previous files.

## Core Source Files

### `src/context.ts` - Context Management

**Purpose**: Manages system and user context for AI conversations.

**Key Functions**:
- `getSystemContext()`: Gets git status, system info
- `getUserContext()`: Gets CLAUDE.md files, user preferences
- `getGitStatus()`: Gets current git branch and status

**Usage**:
```typescript
const systemContext = await getSystemContext()
const userContext = await getUserContext()

// Used in system prompt
const systemPrompt = [
  ...defaultSystemPrompt,
  ...(systemContext.gitStatus ? [systemContext.gitStatus] : []),
  ...(userContext.claudeMd ? [userContext.claudeMd] : [])
]
```

### `src/history.ts` - Command History

**Purpose**: Manages command history for up-arrow navigation.

**Key Functions**:
- `addToHistory()`: Add command to history
- `getHistory()`: Get history entries
- `getTimestampedHistory()`: Get history with timestamps

**Storage**: `~/.claude/history.jsonl`

**Features**:
- Paste content references
- Session-based history
- Lazy paste content resolution

### `src/ink.ts` - TUI Framework

**Purpose**: React/Ink integration for terminal UI.

**Exports**:
- `render()`: Render React component
- `createRoot()`: Create root for rendering
- `Box`, `Text`: UI components
- `useInput()`, `useApp()`: React hooks

**Usage**:
```typescript
import { render, Box, Text } from './ink.js'

render(
  <Box>
    <Text>Hello World</Text>
  </Box>
)
```

### `src/interactiveHelpers.tsx` - TUI Helpers

**Purpose**: Helper functions for interactive TUI sessions.

**Key Functions**:
- `showDialog()`: Show modal dialog
- `showSetupScreens()`: Show onboarding/trust dialogs
- `renderAndRun()`: Render and run TUI
- `getRenderContext()`: Get rendering context

**Features**:
- Onboarding flow
- Trust dialog
- API key approval
- Permission mode dialogs

## Service Files

### `src/services/api/` - API Integration

**Key Files**:
- `claude.ts`: Claude API client
- `bootstrap.ts`: Bootstrap data fetching
- `errors.ts`: Error handling

**Features**:
- Streaming responses
- Retry logic
- Error categorization
- Token tracking

### `src/services/analytics/` - Analytics

**Key Files**:
- `index.ts`: Event logging
- `growthbook.ts`: Feature flags
- `datadog.ts`: Datadog integration

**Features**:
- Event tracking
- Feature flag management
- Performance monitoring

### `src/services/mcp/` - MCP Integration

**Key Files**:
- `client.ts`: MCP client
- `config.ts`: MCP configuration
- `types.ts`: MCP types

**Features**:
- MCP server connection
- Tool/resource loading
- Configuration management

## Utility Files

### `src/utils/config.ts` - Configuration

**Purpose**: Manage global configuration.

**Key Functions**:
- `getGlobalConfig()`: Get config
- `saveGlobalConfig()`: Save config
- `checkHasTrustDialogAccepted()`: Check trust status

**Storage**: `~/.claude/config.json`

### `src/utils/git.ts` - Git Operations

**Purpose**: Git operations and status.

**Key Functions**:
- `getIsGit()`: Check if in git repo
- `getBranch()`: Get current branch
- `getDefaultBranch()`: Get default branch
- `findGitRoot()`: Find git root

### `src/utils/auth.ts` - Authentication

**Purpose**: Handle authentication.

**Key Functions**:
- `isClaudeAISubscriber()`: Check Claude.ai subscription
- `isUsing3PServices()`: Check if using 3rd party
- `getSubscriptionType()`: Get subscription type

## Component Files

### `src/components/` - React Components

**Key Components**:
- `Onboarding.tsx`: Onboarding flow
- `TrustDialog/`: Trust dialog
- `ApproveApiKey.tsx`: API key approval
- `ModelPicker.tsx`: Model selection

**Features**:
- Interactive UI
- Form handling
- State management

## Tool Files

### `src/tools/` - Tool Implementations

**Key Tools**:
- `BashTool/`: Shell command execution
- `FileReadTool/`: File reading
- `FileWriteTool/`: File writing
- `WebSearchTool/`: Web search
- `AgentTool/`: Sub-agent spawning

**Structure**:
```typescript
const Tool = {
  name: 'ToolName',
  description: 'What it does',
  inputSchema: z.object({...}),
  
  async call(args, context) {
    // Implementation
  }
}
```

## State Management

### `src/state/` - Application State

**Key Files**:
- `AppState.ts`: Main app state
- `store.ts`: State store
- `onChangeAppState.ts`: State change handler

**Features**:
- Global state management
- State persistence
- Change notifications

## Plugin System

### `src/plugins/` - Plugin System

**Key Files**:
- `bundled/index.ts`: Built-in plugins
- `pluginLoader.ts`: Plugin loading
- `pluginCliCommands.ts`: Plugin commands

**Features**:
- Dynamic plugin loading
- Plugin lifecycle management
- Command registration

## Key Patterns

### Async Operations

```typescript
// Most operations are async
const result = await someAsyncOperation()

// Error handling
try {
  await operation()
} catch (error) {
  handleError(error)
}
```

### Feature Flags

```typescript
// Conditional features
if (feature('FEATURE_NAME')) {
  // Feature-specific code
}
```

### Memoization

```typescript
// Cache expensive operations
const getCachedValue = memoize(async () => {
  return await expensiveOperation()
})
```

### Event System

```typescript
// Log events
logEvent('event_name', {
  key: 'value'
})

// Handle events
addEventListener('event', handler)
```

## File Organization

### Directory Structure

```
src/
├── components/      # React UI components
├── services/        # Backend services
├── tools/           # Tool implementations
├── utils/           # Utility functions
├── state/           # State management
├── plugins/         # Plugin system
├── bridge/          # Remote session management
├── commands/        # Slash commands
└── types/           # TypeScript types
```

### Naming Conventions

- **Files**: `camelCase.ts` or `PascalCase.tsx` for components
- **Functions**: `camelCase()`
- **Constants**: `UPPER_SNAKE_CASE`
- **Types**: `PascalCase`

### Import Patterns

```typescript
// Relative imports
import { something } from './utils/something.js'

// Absolute imports (with tsconfig paths)
import { something } from 'src/utils/something.js'

// External imports
import React from 'react'
import { z } from 'zod'
```

## Best Practices

### 1. Error Handling
Always handle errors gracefully:
```typescript
try {
  await operation()
} catch (error) {
  logError(error)
  showUserFriendlyMessage()
}
```

### 2. Type Safety
Use TypeScript for type safety:
```typescript
interface Config {
  key: string
  value: number
}

function process(config: Config) {
  // Type-safe operations
}
```

### 3. Async/Await
Use async/await for asynchronous operations:
```typescript
async function fetchData() {
  const response = await fetch(url)
  return response.json()
}
```

### 4. Feature Flags
Use feature flags for conditional code:
```typescript
if (feature('NEW_FEATURE')) {
  // New feature code
}
```

### 5. Memoization
Cache expensive operations:
```typescript
const getCached = memoize(async () => {
  return await expensiveOperation()
})
```

## Summary

The remaining files provide:
- **Context management** for AI conversations
- **History tracking** for user experience
- **TUI framework** for terminal interface
- **Service layer** for API, analytics, MCP
- **Utility functions** for common operations
- **Component library** for UI
- **Tool implementations** for AI capabilities
- **State management** for application state
- **Plugin system** for extensibility

Together, they form a complete, production-ready CLI application.
</task_progress>
</write_to_file>