# 001 - Project Overview: OpenClaude

> **Note**: This is the starting point for understanding OpenClaude. For a complete guide, see [COMPLETE_ORCHESTRATION_GUIDE.md](./COMPLETE_ORCHESTRATION_GUIDE.md).

## What is OpenClaude?

OpenClaude is a **CLI tool** that allows you to use **Claude Code with any LLM** (Large Language Model). It's built on top of Anthropic's Claude Code but opens it up to work with multiple AI providers including:

- **OpenAI** (GPT models)
- **Google Gemini**
- **Ollama** (local models like Llama, Mistral)
- **DeepSeek**
- **200+ other models**

## Key Features

1. **Multi-Provider Support**: Works with any OpenAI-compatible API
2. **Smart Routing**: Automatically selects the best provider based on latency, cost, and health
3. **Local Model Support**: Works with Ollama and Atomic Chat (Apple Silicon)
4. **Bridge System**: Remote session management and control
5. **Plugin Architecture**: Extensible via plugins and skills
6. **Rich TUI**: Beautiful terminal interface built with React/Ink

## Tech Stack

- **Language**: TypeScript/JavaScript
- **Runtime**: Node.js (>=20.0.0) with Bun as build tool
- **UI Framework**: React 19 with Ink (React for CLI)
- **Build System**: Bun bundler
- **Package Manager**: npm/bun

## Project Structure

```
openclaude/
├── bin/              # CLI entry point
├── src/              # Main source code
│   ├── bridge/       # Remote session management
│   ├── commands/     # Slash commands
│   ├── components/   # React/Ink UI components
│   ├── services/     # Backend services
│   ├── tools/        # Tool implementations
│   └── utils/        # Utility functions
├── scripts/          # Build and utility scripts
├── docs/             # Documentation
├── vscode-extension/ # VS Code extension
└── analysis/         # This analysis folder
```

## Entry Points

1. **CLI**: `bin/openclaude` - The main CLI entry point
2. **Main**: `src/main.tsx` - The application's main logic
3. **Commands**: `src/commands.ts` - All available slash commands

## How It Works

1. User runs `openclaude` in terminal
2. CLI loads and checks for built version
3. Main application initializes providers
4. Smart router selects best available provider
5. User interacts with the TUI
6. Commands are processed and routed to appropriate handlers

## Python Providers

The project includes Python files for:
- **atomic_chat_provider.py**: Apple Silicon local model support
- **ollama_provider.py**: Ollama integration
- **smart_router.py**: Intelligent request routing

## Key Concepts

- **Provider**: An AI service (OpenAI, Gemini, Ollama, etc.)
- **Tool**: A capability (file read, bash, web search, etc.)
- **Command**: A slash command (/help, /model, etc.)
- **Bridge**: Remote session management
- **Plugin**: Extension system

## License

MIT License - Open source and free to use
</task_progress>
</write_to_file>