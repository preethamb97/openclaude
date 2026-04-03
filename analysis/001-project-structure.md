# OpenClaude Codebase Analysis - Part 1: Project Structure

## Overview
OpenClaude is a full-stack TypeScript/JavaScript application that provides an AI-powered code assistant with CLI, TUI (Terminal User Interface), and backend services.

## Root Directory Structure
```
openclaude/
├── bin/                    # CLI entry point scripts
├── docs/                   # Documentation
├── scripts/                # Build and utility scripts
├── src/                    # Main source code
├── analysis/               # This analysis (you are here)
├── package.json            # Dependencies and scripts
├── tsconfig.json           # TypeScript configuration
└── README.md              # Project documentation
```

## Source Code Organization (`src/`)

### Core Entry Points
- **`main.tsx`**: Main application entry point, initializes the app
- **`commands.ts`**: Command registration and routing system
- **`tools.ts`**: Tool registration and management
- **`context.ts`**: Context management for the application

### Major Subsystems

#### 1. **CLI Layer** (`src/cli/`)
- `exit.ts`: Exit handling
- `print.ts`: Output formatting
- `remoteIO.ts`: Remote I/O handling
- `structuredIO.ts`: Structured output
- `update.ts`: Update management

#### 2. **Commands** (`src/commands/`)
Each command has its own directory with:
- `index.ts`: Command definition
- Implementation files

Commands include:
- `add-dir`: Add directory to context
- `agents`: Agent management
- `assistant`: Assistant features
- `branch`: Git branch operations
- `bridge`: Bridge connections
- `chrome`: Chrome integration
- `clear`: Clear context
- `color`: Color settings
- `compact`: Context compaction
- `config`: Configuration
- `context`: Context management
- `cost`: Cost tracking
- `diff`: File differences
- `doctor`: Diagnostics
- `effort`: Effort tracking
- `exit`: Exit command
- `export`: Export data
- `fast`: Fast mode
- `feedback`: User feedback
- `files`: File operations
- `help`: Help system
- `hooks`: Hook management
- `ide`: IDE integration
- `init`: Initialization
- `install`: Installation
- `issue`: Issue tracking
- `keybindings`: Key binding configuration
- `login/logout`: Authentication
- `mcp`: MCP server management
- `memory`: Memory management
- `mobile`: Mobile features
- `model`: Model selection
- `permissions`: Permission management
- `plan`: Planning features
- `plugin`: Plugin management
- `provider`: Provider configuration
- `resume`: Session resumption
- `review`: Code review
- `session`: Session management
- `skills`: Skills management
- `status`: Status display
- `stats`: Statistics
- `tasks`: Task management
- `theme`: Theme management
- `upgrade`: Upgrade handling
- `usage`: Usage statistics
- `vim`: Vim mode

#### 3. **Components** (`src/components/`)
React/Ink UI components:
- Design system components
- Dialog components
- Display components
- Input components

#### 4. **Services** (`src/services/`)
- `api/`: API client implementations
- `analytics/`: Analytics tracking
- `mcp/`: MCP protocol implementation
- `compact/`: Context compaction
- `plugins/`: Plugin system
- `skills/`: Skills system
- `tools/`: Tool orchestration

#### 5. **Tools** (`src/tools/`)
Each tool has its own directory:
- `AgentTool/`: Agent spawning
- `BashTool/`: Shell command execution
- `FileReadTool/`: File reading
- `FileWriteTool/`: File writing
- `FileEditTool/`: File editing
- `GlobTool/`: File pattern matching
- `GrepTool/`: Text search
- `WebFetchTool/`: Web requests
- `WebSearchTool/`: Web search
- And many more...

#### 6. **Bridge** (`src/bridge/`)
Remote connection management:
- `bridgeApi.ts`: API client
- `bridgeConfig.ts`: Configuration
- `bridgeMain.ts`: Main bridge logic
- `sessionRunner.ts`: Session execution
- `workSecret.ts`: Secret management

#### 7. **State Management** (`src/state/`)
- `AppState.ts`: Application state
- `store.ts`: State store
- `onChangeAppState.ts`: State change handlers

#### 8. **Utilities** (`src/utils/`)
- `config.ts`: Configuration utilities
- `errors.ts`: Error handling
- `git.ts`: Git operations
- `messages.ts`: Message handling
- `permissions/`: Permission utilities
- `plugins/`: Plugin utilities
- `settings/`: Settings management

#### 9. **Types** (`src/types/`)
TypeScript type definitions:
- `message.ts`: Message types
- `permissions.ts`: Permission types
- `tools.ts`: Tool types
- `command.ts`: Command types

#### 10. **Ink (TUI)** (`src/ink/`)
Terminal UI framework:
- `root.tsx`: Root component
- `hooks/`: React hooks
- `components/`: UI components
- `events/`: Event handling

## Build System

### Scripts (`scripts/`)
- `build.ts`: Build script
- `provider-*.ts`: Provider management
- `system-check.ts`: System diagnostics

### Configuration Files
- `package.json`: NPM configuration
- `tsconfig.json`: TypeScript configuration
- `bun.lock`: Bun lockfile

## Key Architectural Patterns

1. **Command Pattern**: All user actions are commands with consistent interface
2. **Tool Pattern**: AI capabilities are tools with input/output schemas
3. **Provider Pattern**: Multiple AI providers with unified interface
4. **Component Pattern**: React/Ink components for TUI
5. **Service Pattern**: Backend services for API, analytics, etc.
6. **State Management**: Centralized state with change handlers

## Data Flow

```
User Input
    ↓
CLI Parser
    ↓
Command Router
    ↓
Command Handler
    ↓
Tool Executor (if needed)
    ↓
AI Provider (if needed)
    ↓
Response Formatter
    ↓
TUI/CLI Output
```

## Next Steps
See subsequent parts for detailed analysis of each subsystem.
</task_progress>
</write_to_file>