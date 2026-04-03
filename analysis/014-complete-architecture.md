# 014 - Complete Architecture Overview

## High-Level Architecture

OpenClaude is a full-stack TypeScript application with these main layers:

### 1. CLI Layer (bin/)
Entry point that bootstraps the application.

### 2. Application Layer (src/main.tsx)
Initializes the app, parses arguments, and routes to commands.

### 3. Command Layer (src/commands/)
Handles user commands like /help, /model, /clear.

### 4. Query Engine (src/QueryEngine.ts)
Manages AI conversations and tool orchestration.

### 5. Tools Layer (src/tools/)
Provides AI capabilities like file operations and bash execution.

### 6. Provider Layer (src/services/api/)
Communicates with AI providers (Anthropic, OpenAI, Ollama).

### 7. TUI Layer (src/ink/, src/components/)
Renders the terminal user interface with React/Ink.

## Data Flow

User Input → CLI Parser → Command Router → Command Handler → Tool/Provider → Response → TUI Output

## Key Integration Points

1. CLI → Main: Dynamic import of dist/cli.mjs
2. Main → Commands: Commander program with registered commands
3. Commands → Tools: Tool execution for AI capabilities
4. QueryEngine → Providers: AI API calls
5. Providers → External APIs: HTTP requests to AI services
6. State → Components: React state management

## Configuration Hierarchy

1. Built-in defaults
2. Environment variables (.env)
3. Global config (~/.claude/config.json)
4. Project config (.claude/config.json)
5. Command-line flags

## Security Architecture

- Permission system with allow/deny rules
- Sandbox execution for bash commands
- Input validation with Zod schemas
- Hook system for pre/post tool execution

## Extension Points

- Commands: Add new slash commands
- Tools: Add new AI capabilities
- Providers: Add new AI backends
- Plugins: Extend functionality without core changes

## Summary

OpenClaude uses a layered, modular architecture that enables:
- Multi-provider AI support
- Rich tool ecosystem
- Flexible configuration
- Session persistence
- Remote control via bridge
- Plugin extensibility