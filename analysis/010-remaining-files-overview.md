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

### `src/setup.ts` - Session Setup

**Purpose**: Initializes session state and working directory.

**Key Functions**:
- `setup()`: Main setup function
- Handles worktree initialization
- Sets up session directories
- Configures git worktrees

### `src/query.ts` - Query Execution

**Purpose**: Core query execution for AI interactions.

**Key Functions**:
- `query()`: Main query function
- Handles streaming responses
- Manages tool execution loop

### `src/QueryEngine.ts` - AI Conversation Engine

**Purpose**: Manages the complete AI conversation lifecycle.

**Key Classes**:
- `QueryEngine`: Main engine class

**Key Functions**:
- `submitMessage()`: Submit a message and get streaming response
- Manages message history
- Handles tool orchestration
- Manages token budgets

**Configuration**:
```typescript
type QueryEngineConfig = {
  cwd: string
  tools: Tools
  commands: Command[]
  mcpClients: MCPServerConnection[]
  agents: AgentDefinition[]
  canUseTool: CanUseToolFn
  getAppState: () => AppState
  setAppState: (f: (prev: AppState) => AppState) => void
  initialMessages?: Message[]
  customSystemPrompt?: string
  appendSystemPrompt?: string
  userSpecifiedModel?: string
  fallbackModel?: string
  maxTurns?: number
  maxBudgetUsd?: number
  // ... more options
}
```

### `src/Tool.ts` - Tool Type Definitions

**Purpose**: Core tool type definitions and interfaces.

**Key Types**:
- `Tool<Input, Output>`: Tool interface
- `Tools`: Array of tools
- `ToolUseContext`: Context for tool execution
- `ToolPermissionContext`: Permission context

### `src/Task.ts` - Task Management

**Purpose**: Task management for long-running operations.

### `src/tasks.ts` - Task Utilities

**Purpose**: Utility functions for task management.

### `src/cost-tracker.ts` - Cost Tracking

**Purpose**: Tracks API costs during sessions.

### `src/costHook.ts` - Cost Tracking Hooks

**Purpose**: React hooks for cost tracking.

### `src/dialogLaunchers.tsx` - Dialog Launchers

**Purpose**: Launches various dialogs in the TUI.

**Key Functions**:
- `showSetupScreens()`: Show onboarding
- `launchResumeChooser()`: Resume session picker
- `launchAssistantInstallWizard()`: Assistant setup
- `launchTeleportResumeWrapper()`: Teleport resume

### `src/replLauncher.tsx` - REPL Launcher

**Purpose**: Launches the main REPL (Read-Eval-Print Loop).

**Key Functions**:
- `launchRepl()`: Main REPL launcher
- Sets up the interactive session
- Manages the TUI rendering

### `src/projectOnboardingState.ts` - Onboarding State

**Purpose**: Manages onboarding state for new users.

## Service Files

### `src/services/api/` - API Integration

**Key Files**:
- `claude.ts`: Claude API client
- `bootstrap.ts`: Bootstrap data fetching
- `errors.ts`: Error handling and categorization
- `filesApi.ts`: File download API
- `logging.ts`: API logging
- `referral.js`: Referral tracking

**Features**:
- Streaming responses
- Retry logic
- Error categorization
- Token tracking
- Usage accumulation

### `src/services/analytics/` - Analytics

**Key Files**:
- `index.ts`: Event logging
- `config.js`: Analytics configuration
- `growthbook.ts`: Feature flag management
- `sink.ts`: Event sink management

**Features**:
- Event tracking
- Feature flag management
- Performance monitoring
- Statsig integration

### `src/services/mcp/` - MCP Integration

**Key Files**:
- `client.ts`: MCP client and server management
- `config.ts`: MCP configuration parsing
- `types.ts`: MCP type definitions
- `utils.ts`: MCP utility functions
- `officialRegistry.ts`: Official MCP registry
- `claudeai.ts`: Claude.ai MCP configs

**Features**:
- MCP server connection
- Tool/resource loading
- Configuration management
- Policy filtering

### `src/services/compact/` - Context Compaction

**Key Files**:
- `snipCompact.js`: History snipping
- `snipProjection.js`: Snip projection

**Features**:
- Context compaction
- History snipping
- Token reduction

### `src/services/lsp/` - Language Server Protocol

**Key Files**:
- `manager.ts`: LSP server management

**Features**:
- LSP server lifecycle
- Language intelligence

### `src/services/plugins/` - Plugin System

**Key Files**:
- `pluginCliCommands.ts`: Plugin CLI commands
- `pluginLoader.ts`: Plugin loading utilities

**Features**:
- Plugin installation
- Plugin management
- Plugin lifecycle

### `src/services/policyLimits/` - Policy Limits

**Purpose**: Manages enterprise policy limits.

**Key Functions**:
- `loadPolicyLimits()`: Load policy limits
- `isPolicyAllowed()`: Check policy allowance
- `refreshPolicyLimits()`: Refresh policy limits

### `src/services/remoteManagedSettings/` - Remote Settings

**Purpose**: Manages remote enterprise settings.

**Key Functions**:
- `loadRemoteManagedSettings()`: Load remote settings
- `refreshRemoteManagedSettings()`: Refresh settings

### `src/services/settingsSync/` - Settings Sync

**Purpose**: Synchronizes settings across devices.

**Key Functions**:
- `uploadUserSettingsInBackground()`: Upload settings

### `src/services/skillSearch/` - Skill Search

**Purpose**: Provides skill search functionality.

**Key Files**:
- `localSearch.js`: Local skill search

### `src/services/PromptSuggestion/` - Prompt Suggestions

**Purpose**: Provides prompt suggestions.

**Key Files**:
- `promptSuggestion.ts`: Prompt suggestion logic

### `src/services/tips/` - Tips System

**Purpose**: Provides helpful tips to users.

**Key Files**:
- `tipRegistry.ts`: Tip registration

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
- `getWorktreeCount()`: Get worktree count

### `src/utils/auth.ts` - Authentication

**Purpose**: Handle authentication.

**Key Functions**:
- `isClaudeAISubscriber()`: Check Claude.ai subscription
- `isUsing3PServices()`: Check if using 3rd party
- `getSubscriptionType()`: Get subscription type
- `prefetchAwsCredentialsAndBedRockInfoIfSafe()`: Prefetch AWS credentials
- `prefetchGcpCredentialsIfSafe()`: Prefetch GCP credentials

### `src/utils/model/` - Model Management

**Key Files**:
- `model.ts`: Model selection and parsing
- `modelCapabilities.ts`: Model capability definitions
- `modelStrings.ts`: Model string initialization
- `deprecation.ts`: Model deprecation warnings
- `ollamaModels.ts`: Ollama model prefetching
- `providers.ts`: Provider utilities

**Features**:
- Model parsing and normalization
- Model capability tracking
- Provider detection
- Deprecation warnings

### `src/utils/permissions/` - Permission Utilities

**Key Files**:
- `PermissionMode.ts`: Permission mode definitions
- `PermissionSetup.ts`: Permission setup
- `permissions.ts`: Permission checking
- `filesystem.ts`: Filesystem permissions

**Features**:
- Permission mode management
- Permission checking
- Sandbox configuration
- Auto-mode handling

### `src/utils/settings/` - Settings Management

**Key Files**:
- `settings.ts`: Settings loading and management
- `settingsCache.ts`: Settings caching
- `validation.ts`: Settings validation
- `constants.ts`: Settings constants
- `changeDetector.ts`: Settings change detection
- `mdm/`: MDM settings handling

**Features**:
- Settings loading from multiple sources
- Settings validation
- Settings caching
- MDM integration
- Change detection

### `src/utils/plugins/` - Plugin Utilities

**Key Files**:
- `pluginLoader.ts`: Plugin loading
- `installedPluginsManager.ts`: Plugin management
- `pluginDirectories.ts`: Plugin directory management
- `orphanedPluginFilter.ts`: Orphaned plugin filtering
- `cacheUtils.ts`: Plugin cache utilities
- `managedPlugins.ts`: Managed plugin handling
- `loadPluginCommands.ts`: Plugin command loading

**Features**:
- Plugin discovery and loading
- Plugin cache management
- Plugin command registration
- Orphaned plugin cleanup

### `src/utils/skills/` - Skill Utilities

**Key Files**:
- `skillChangeDetector.ts`: Skill change detection

### `src/utils/swarm/` - Agent Swarm Utilities

**Key Files**:
- `teammatePromptAddendum.ts`: Teammate prompt addendum
- `backends/teammateModeSnapshot.ts`: Teammate mode snapshot
- `reconnection.ts`: Swarm reconnection

**Features**:
- Agent swarm coordination
- Teammate mode management
- Team context management

### `src/utils/claudeInChrome/` - Chrome Integration

**Key Files**:
- `setup.ts`: Chrome integration setup
- `common.ts`: Common utilities
- `prompt.ts`: Chrome prompt hints
- `gates.ts`: Feature gates

**Features**:
- Chrome browser integration
- MCP server setup
- Feature gating

### `src/utils/teleport/` - Teleport Utilities

**Purpose**: Teleport session management.

**Key Files**:
- `teleport.ts`: Main teleport logic
- `api.ts`: Teleport API
- `sessionRestore.ts`: Session restoration

### `src/utils/deepLink/` - Deep Link Handling

**Purpose**: Handle deep links and URL schemes.

**Key Files**:
- `protocolHandler.ts`: URL protocol handler
- `banner.ts`: Deep link banner

### `src/utils/sandbox/` - Sandbox Utilities

**Purpose**: Sandbox management.

**Key Files**:
- `sandbox-adapter.ts`: Sandbox adapter

### `src/utils/telemetry/` - Telemetry Utilities

**Key Files**:
- `pluginTelemetry.ts`: Plugin telemetry
- `skillLoadedEvent.ts`: Skill loaded events

### `src/utils/hooks/` - Hook Utilities

**Key Files**:
- `hookEvents.ts`: Hook event management
- `hookHelpers.ts`: Hook helpers

### `src/bootstrap/state.ts` - Bootstrap State

**Purpose**: Global bootstrap state management.

**Key Functions**:
- `getSessionId()`: Get session ID
- `getOriginalCwd()`: Get original working directory
- `setIsRemoteMode()`: Set remote mode
- `setMainLoopModelOverride()`: Override model
- Various state setters and getters

### `src/state/AppState.ts` - App State Types

**Purpose**: Application state type definitions.

### `src/state/AppStateStore.tsx` - App State Store

**Purpose**: Application state store with React context.

### `src/state/onChangeAppState.ts` - State Change Handlers

**Purpose**: Handle application state changes.

### `src/state/store.ts` - State Store

**Purpose**: State store implementation.

## New Subsystems

### `src/buddy/` - Buddy Companion System

**Purpose**: Provides a companion sprite in the TUI.

**Key Files**:
- `companion.ts`: Companion logic
- `CompanionSprite.tsx`: Sprite component
- `feature.ts`: Feature detection
- `observer.ts`: Observer pattern
- `prompt.ts`: Prompt utilities
- `sprites.ts`: Sprite definitions
- `types.ts`: Type definitions
- `useBuddyNotification.tsx`: Notification hook

**Features**:
- Animated companion sprite
- Notification system
- Feature gating

### `src/coordinator/` - Coordinator Mode

**Purpose**: Coordinator mode for agent orchestration.

**Key Files**:
- `coordinatorMode.ts`: Coordinator mode implementation

**Features**:
- Multi-agent coordination
- Worker management
- Task distribution

### `src/memdir/` - Memory Directory

**Purpose**: Persistent memory system.

**Key Files**:
- `memdir.ts`: Memory directory management
- `paths.ts`: Memory path utilities

**Features**:
- Persistent memory storage
- Memory retrieval
- Path management

### `src/migrations/` - Data Migrations

**Purpose**: Data migrations for version upgrades.

**Key Files**:
- `migrateAutoUpdatesToSettings.ts`
- `migrateBypassPermissionsAcceptedToSettings.ts`
- `migrateEnableAllProjectMcpServersToSettings.ts`
- `migrateFennecToOpus.ts`
- `migrateLegacyOpusToCurrent.ts`
- `migrateOpusToOpus1m.ts`
- `migrateReplBridgeEnabledToRemoteControlAtStartup.ts`
- `migrateSonnet1mToSonnet45.ts`
- `migrateSonnet45ToSonnet46.ts`
- `resetAutoModeOptInForDefaultOffer.ts`
- `resetProToOpusDefault.ts`

**Features**:
- Version-based migrations
- Settings migrations
- Model string migrations

### `src/constants/` - Application Constants

**Purpose**: Centralized application constants.

**Key Files**:
- `oauth.ts`: OAuth configuration
- `product.ts`: Product configuration
- `tools.ts`: Tool constants
- `xml.ts`: XML constants

### `src/context/` - Context Management

**Purpose**: Context-related utilities.

**Key Files**:
- `stats.ts`: Statistics context

### `src/entrypoints/` - Entry Points

**Purpose**: Application entry points.

**Key Files**:
- `agentSdkTypes.ts`: SDK type definitions
- `init.ts`: Initialization
- `initSinks.ts`: Sink initialization

### `src/hooks/` - React Hooks

**Purpose**: Custom React hooks.

**Key Files**:
- `useCanUseTool.ts`: Tool usage hook

### `src/keybindings/` - Keybindings

**Purpose**: Keybinding management.

### `src/screens/` - UI Screens

**Purpose**: UI screen components.

### `src/server/` - Server Functionality

**Purpose**: Server-related functionality.

**Key Files**:
- `createDirectConnectSession.ts`: Direct connect session creation
- `parseConnectUrl.ts`: Connect URL parsing

### `src/tasks/` - Task Management

**Purpose**: Task management utilities.

### `src/vim/` - Vim Mode

**Purpose**: Vim mode support.

### `src/voice/` - Voice Mode

**Purpose**: Voice interaction support.

## CLI Layer

### `src/cli/` - CLI Implementation

**Key Files**:
- `exit.ts`: Exit handling
- `ndjsonSafeStringify.ts`: NDJSON stringification
- `print.ts`: Output formatting for non-interactive mode
- `remoteIO.ts`: Remote I/O handling
- `structuredIO.ts`: Structured output handling
- `update.ts`: Update management
- `handlers/`: CLI request handlers
- `transports/`: CLI transport implementations

## Python Providers

### `python/` - Python Provider Files

**Key Files**:
- `ollama_provider.py`: Ollama integration
- `atomic_chat_provider.py`: Atomic Chat (Apple Silicon) integration
- `smart_router.py`: Intelligent request routing
- `tests/`: Python provider tests

## Build System

### `scripts/` - Build and Utility Scripts

**Key Files**:
- `build.ts`: Main build script
- `provider-bootstrap.ts`: Provider bootstrap
- `provider-discovery.ts`: Provider discovery
- `provider-launch.ts`: Provider launch
- `provider-recommend.ts`: Provider recommendations
- `system-check.ts`: System diagnostics
- `verify-no-phone-home.sh`: Phone home verification
- `verify-no-phone-home.ts`: Phone home verification script
- `no-telemetry-plugin.ts`: Telemetry plugin

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

### Lazy Loading

```typescript
// Lazy require to avoid circular dependencies
const getModule = () => require('./module.js') as typeof import('./module.js')
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
├── skills/          # Skill system
├── bridge/          # Remote session management
├── commands/        # Slash commands
├── buddy/           # Buddy companion
├── coordinator/     # Coordinator mode
├── memdir/          # Memory directory
├── migrations/      # Data migrations
├── constants/       # Application constants
├── context/         # Context management
├── entrypoints/     # Entry points
├── hooks/           # React hooks
├── keybindings/     # Keybindings
├── screens/         # UI screens
├── server/          # Server functionality
├── tasks/           # Task management
├── vim/             # Vim mode
├── voice/           # Voice mode
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

// Conditional imports (feature flags)
const module = feature('FEATURE') 
  ? require('./module.js') as typeof import('./module.js')
  : null
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

### 6. Lazy Loading
Use lazy loading for heavy modules:
```typescript
const heavyModule = await import('./heavy-module.js')
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
- **New subsystems** for advanced features (buddy, coordinator, memory, migrations)

Together, they form a complete, production-ready CLI application with advanced features like:
- Agent swarms and coordination
- Persistent memory
- Remote session management
- Voice interaction
- Vim mode
- Buddy companion
- Data migrations
- Feature flags
- MCP integration
- Chrome integration