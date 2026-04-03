# OpenClaude Quick Reference Guide

## Project Structure

```
openclaude/
├── bin/
│   └── openclaude              # CLI entry point
├── src/
│   ├── main.tsx                # Main application logic
│   ├── commands.ts             # Command registration
│   ├── tools.ts                # Tool registration
│   ├── QueryEngine.ts          # AI conversation engine
│   ├── context.ts              # Context management
│   ├── history.ts              # Command history
│   ├── ink.ts                  # TUI framework
│   ├── bridge/                 # Remote session management
│   ├── commands/               # Command implementations
│   ├── components/             # React/Ink UI components
│   ├── services/               # Backend services
│   ├── tools/                  # Tool implementations
│   ├── utils/                  # Utility functions
│   ├── state/                  # State management
│   └── plugins/                # Plugin system
├── analysis/                   # This analysis
├── package.json                # Dependencies
└── tsconfig.json               # TypeScript config
```

## Key Files Reference

### Entry Points

| File | Purpose |
|------|---------|
| `bin/openclaude` | CLI bootstrap script |
| `src/main.tsx` | Main application initialization |
| `src/commands.ts` | Command registration and routing |

### Core Systems

| File | Purpose |
|------|---------|
| `src/QueryEngine.ts` | AI conversation loop and tool orchestration |
| `src/tools.ts` | Tool registration and management |
| `src/context.ts` | System and user context management |
| `src/history.ts` | Command history persistence |

### Services

| File | Purpose |
|------|---------|
| `src/services/api/claude.ts` | Anthropic API client |
| `src/services/mcp/client.ts` | MCP protocol client |
| `src/services/analytics/index.ts` | Event logging |

### Bridge System

| File | Purpose |
|------|---------|
| `src/bridge/bridgeMain.ts` | Main bridge loop |
| `src/bridge/bridgeApi.ts` | Bridge API client |
| `src/bridge/sessionRunner.ts` | Session spawning |

## Common Commands

### Running OpenClaude

```bash
# Interactive mode
openclaude

# One-liner mode
openclaude -p "explain this code"

# With specific model
openclaude --model sonnet

# With local provider
openclaude --provider ollama

# Resume session
openclaude --resume abc123

# Continue most recent
openclaude --continue
```

### Slash Commands (in TUI)

```bash
/help          # Show help
/model         # Change AI model
/clear         # Clear conversation
/resume        # Resume session
/compact       # Compact context
/config        # Edit configuration
/doctor        # Run diagnostics
```

## Configuration Locations

| File | Purpose |
|------|---------|
| `.env` | Environment variables |
| `~/.claude/config.json` | Global configuration |
| `.claude/config.json` | Project-specific config |

## Key Environment Variables

```bash
# Provider
PREFERRED_PROVIDER=anthropic  # or openai, ollama, gemini

# Anthropic
ANTHROPIC_API_KEY=sk-ant-...

# OpenAI
OPENAI_API_KEY=sk-...

# Ollama
OLLAMA_BASE_URL=http://localhost:11434

# Models
BIG_MODEL=claude-sonnet-4-20250514
SMALL_MODEL=claude-haiku-4-20250414

# Router
ROUTER_MODE=smart
ROUTER_STRATEGY=balanced
```

## Tool Categories

### File Operations
- `FileReadTool` - Read file contents
- `FileWriteTool` - Write/create files
- `FileEditTool` - Edit existing files
- `GlobTool` - Find files by pattern
- `GrepTool` - Search file contents

### Code Operations
- `BashTool` - Execute shell commands
- `NotebookEditTool` - Edit Jupyter notebooks
- `LSPTool` - Language Server Protocol

### Web Operations
- `WebFetchTool` - Fetch web pages
- `WebSearchTool` - Search the web
- `WebBrowserTool` - Control browser

### Agent Operations
- `AgentTool` - Spawn sub-agents
- `SkillTool` - Execute skills/commands
- `TaskStopTool` - Stop running tasks

## Python Providers

| File | Purpose |
|------|---------|
| `ollama_provider.py` | Ollama integration |
| `atomic_chat_provider.py` | Atomic Chat (Apple Silicon) |
| `smart_router.py` | Intelligent request routing |

## Testing

```bash
# Run all tests
bun run test

# Run specific test
bun run test src/utils/model/model.test.ts

# Run with coverage
bun run test:coverage

# Run in watch mode
bun run test:watch
```

## Debugging

```bash
# Enable debug logging
openclaude --debug

# Debug with categories
openclaude --debug api,hooks,tools

# Verbose output
openclaude --verbose

# Debug file
CLAUDE_CODE_DEBUG_FILE=/tmp/claude-debug.log openclaude
```

## Build Commands

```bash
# Build for production
bun run build

# Type check
bun run typecheck

# Development mode
bun run dev

# Run tests
bun run test

# Package for distribution
npm pack
```

## Common Patterns

### Adding a New Command

```typescript
// src/commands/my-command/index.ts
import type { Command } from '../../commands.js';

const myCommand: Command = {
  name: 'mycommand',
  description: 'My custom command',
  type: 'local',
  action: async (args) => {
    console.log('Command executed');
  }
};

export default myCommand;
```

### Adding a New Tool

```typescript
// src/tools/MyTool/MyTool.ts
import { z } from 'zod';
import type { Tool, ToolUseContext } from '../../Tool.js';

const inputSchema = z.object({
  param1: z.string(),
});

export const MyTool: Tool<Input, any> = {
  name: 'MyTool',
  description: 'Does something custom',
  inputSchema,
  
  async call(args: Input, context: ToolUseContext) {
    // Implement tool logic
    return { result: 'success' };
  },
  
  isEnabled() {
    return true;
  }
};
```

### Adding MCP Server

```json
// ~/.claude/config.json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["server.js"],
      "env": { "API_KEY": "..." }
    }
  }
}
```

## Architecture Layers

```
┌─────────────────────────────────────┐
│  CLI Entry (bin/openclaude)         │
├─────────────────────────────────────┤
│  Main Application (src/main.tsx)    │
├─────────────────────────────────────┤
│  Commands (src/commands.ts)         │
├─────────────────────────────────────┤
│  Tools (src/tools.ts)               │
├─────────────────────────────────────┤
│  Query Engine (src/QueryEngine.ts)  │
├─────────────────────────────────────┤
│  Providers (Python/TypeScript)      │
└─────────────────────────────────────┘
```

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

## Summary

This quick reference provides:
- **Project structure** overview
- **Key files** and their purposes
- **Common commands** for running OpenClaude
- **Configuration** locations and variables
- **Tool categories** and examples
- **Testing** and **debugging** commands
- **Build** and **deploy** commands
- **Common patterns** for extending functionality

Use this as a quick lookup guide while working with the codebase.