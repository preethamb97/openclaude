# 013 - Configuration Options Reference

## Overview

This document provides a complete reference of all configuration options available in OpenClaude.

## Configuration Hierarchy

Configuration is loaded in this order (later overrides earlier):

1. **Built-in defaults** - Hardcoded in source
2. **Environment variables** - `.env` file or system env
3. **Global config** - `~/.claude/config.json`
4. **Project config** - `.claude/config.json` (in project root)
5. **Command-line flags** - Arguments passed to CLI

## Environment Variables

### Provider Configuration

```bash
# Which AI provider to use
# Options: anthropic, openai, gemini, ollama, atomic-chat, github, bedrock, vertex
PREFERRED_PROVIDER=anthropic

# Anthropic (Claude) configuration
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_BASE_URL=https://api.anthropic.com

# OpenAI configuration
OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1

# Gemini configuration
GEMINI_API_KEY=...
GEMINI_BASE_URL=https://generativelanguage.googleapis.com

# Ollama configuration
OLLAMA_BASE_URL=http://localhost:11434

# Atomic Chat configuration (Apple Silicon)
ATOMIC_CHAT_BASE_URL=http://127.0.0.1:1337

# GitHub Copilot configuration
GITHUB_TOKEN=ghp_...

# AWS Bedrock configuration
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=us-east-1
CLAUDE_CODE_USE_BEDROCK=true

# Google Vertex configuration
GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
CLAUDE_CODE_USE_VERTEX=true
```

### Model Configuration

```bash
# Models for different request sizes
BIG_MODEL=claude-sonnet-4-20250514
SMALL_MODEL=claude-haiku-4-20250414

# Model override for current session
MODEL_OVERRIDE=claude-opus-4-20250514
```

### Router Configuration

```bash
# Smart router mode
# Options: smart, fixed
ROUTER_MODE=smart

# Router strategy
# Options: latency, cost, balanced
ROUTER_STRATEGY=balanced

# Enable fallback on failure
ROUTER_FALLBACK=true
```

### Feature Flags

```bash
# Enable/disable specific features
CLAUDE_CODE_DISABLE_CLAUDE_MDS=false
CLAUDE_CODE_DISABLE_TERMINAL_TITLE=false
CLAUDE_CODE_SIMPLE=false
CLAUDE_CODE_BARE=false
CLAUDE_CODE_EAGER_FLUSH=false

# Debug and logging
CLAUDE_CODE_DEBUG=false
CLAUDE_CODE_DEBUG_FILE=/path/to/debug.log
CLAUDE_CODE_VERBOSE=false

# Telemetry
CLAUDE_CODE_DISABLE_TELEMETRY=false

# Sandbox
CLAUDE_CODE_SANDBOX_ENABLED=false
CLAUDE_CODE_SANDBOX_ALLOW_UNSANDBOXED=false
```

### Session Configuration

```bash
# Session persistence
CLAUDE_CODE_NO_SESSION_PERSISTENCE=false

# Session ingress (for remote sessions)
CLAUDE_CODE_SESSION_ACCESS_TOKEN=...
CLAUDE_CODE_WEBSOCKET_AUTH_FILE_DESCRIPTOR=...

# Remote mode
CLAUDE_CODE_REMOTE=false
CLAUDE_CODE_ENVIRONMENT_KIND=local
```

### MCP Configuration

```bash
# MCP server configuration
CLAUDE_CODE_MCP_CONFIG=/path/to/mcp.json
CLAUDE_CODE_STRICT_MCP_CONFIG=false
```

## Global Configuration (`~/.claude/config.json`)

### Structure

```json
{
  "theme": "dark",
  "hasCompletedOnboarding": true,
  "lastOnboardingVersion": "0.1.7",
  "hasCompletedClaudeInChromeOnboarding": false,
  "apiKeyHelper": "echo $MY_API_KEY",
  "customApiKeyResponses": {
    "sk-ant-...abc": "approved"
  },
  "tldrEnabled": false,
  "preferredNotifChannel": "iterm2",
  "verbose": true,
  "editorMode": "vim",
  "autoCompactEnabled": true,
  "mcpServers": {},
  "mcpContextUris": [],
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(git commit:*)"
    ],
    "deny": [
      "Bash(rm -rf /)"
    ]
  },
  "migrationVersion": 11
}
```

### Configuration Keys

#### UI Settings

```json
{
  "theme": "dark",  // Options: dark, light, system
  "editorMode": "vim",  // Options: vim, emacs
  "preferredNotifChannel": "iterm2",  // Options: iterm2, kitty, ghostty, bell, none
  "verbose": true,  // Enable verbose output
  "tldrEnabled": false  // Enable TLDR summaries
}
```

#### Onboarding

```json
{
  "hasCompletedOnboarding": true,
  "lastOnboardingVersion": "0.1.7",
  "hasCompletedClaudeInChromeOnboarding": false
}
```

#### Authentication

```json
{
  "apiKeyHelper": "echo $MY_API_KEY",  // Command to get API key
  "customApiKeyResponses": {
    "sk-ant-...abc": "approved"  // Approved API keys
  }
}
```

#### Features

```json
{
  "autoCompactEnabled": true,  // Enable auto-compaction
  "autoModeOptIn": false,  // Enable auto mode
  "advisorEnabled": false,  // Enable advisor mode
  "fastModeEnabled": false  // Enable fast mode
}
```

#### MCP Servers

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["server.js"],
      "env": {
        "API_KEY": "..."
      }
    }
  },
  "mcpContextUris": [
    "file:///path/to/context.md"
  ]
}
```

#### Permissions

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(git commit:*)",
      "Read",
      "Write",
      "Edit"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(curl:*)"
    ]
  }
}
```

## Project Configuration (`.claude/config.json`)

Project-specific overrides:

```json
{
  "mcpServers": {
    "project-server": {
      "command": "python",
      "args": ["server.py"]
    }
  },
  "permissions": {
    "allow": [
      "Bash(npm run test:*)"
    ]
  },
  "context": {
    "include": [
      "src/**/*.ts",
      "README.md"
    ],
    "exclude": [
      "node_modules/**",
      "dist/**"
    ]
  }
}
```

## Command-Line Flags

### Global Options

```bash
# Help
openclaude --help
openclaude -h

# Version
openclaude --version

# Debug mode
openclaude --debug
openclaude -d

# Verbose mode
openclaude --verbose

# Bare mode (minimal)
openclaude --bare
```

### Model Options

```bash
# Specify model
openclaude --model claude-sonnet-4-20250514
openclaude --model sonnet  # Alias

# Specify provider
openclaude --provider openai
openclaude --provider ollama

# Fallback model
openclaude --fallback-model claude-haiku-4-20250414

# Effort level
openclaude --effort low
openclaude --effort medium
openclaude --effort high
openclaude --effort max

# Thinking mode
openclaude --thinking enabled
openclaude --thinking adaptive
openclaude --thinking disabled

# Max thinking tokens (deprecated)
openclaude --max-thinking-tokens 10000
```

### Session Options

```bash
# Resume session
openclaude --resume
openclaude --resume abc123

# Continue most recent
openclaude --continue
openclaude -c

# Session ID
openclaude --session-id 550e8400-e29b-41d4-a716-446655440000

# Session name
openclaude --name "My Session"
openclaude -n "My Session"

# Fork session
openclaude --fork-session --resume abc123

# Disable persistence
openclaude --no-session-persistence
```

### Output Options

```bash
# Print mode (non-interactive)
openclaude -p "explain this code"
openclaude --print "explain this code"

# Output format
openclaude -p "test" --output-format text
openclaude -p "test" --output-format json
openclaude -p "test" --output-format stream-json

# Input format
openclaude --input-format text
openclaude --input-format stream-json

# JSON schema for structured output
openclaude --json-schema '{"type":"object","properties":{"name":{"type":"string"}}}'
```

### Context Options

```bash
# Add directories to context
openclaude --add-dir /path/to/dir1 /path/to/dir2

# System prompt
openclaude --system-prompt "You are a helpful assistant"
openclaude --system-prompt-file /path/to/prompt.txt

# Append to system prompt
openclaude --append-system-prompt "Focus on TypeScript"
openclaude --append-system-prompt-file /path/to/append.txt
```

### Tools Options

```bash
# Allow specific tools
openclaude --allowed-tools "Bash(git:*) Edit Read"

# Disallow specific tools
openclaude --disallowed-tools "Bash(rm:*) WebFetch"

# Specify available tools
openclaude --tools "Bash,Edit,Read,Glob,Grep"
openclaude --tools ""  # Disable all tools
openclaude --tools "default"  # Use all default tools
```

### Permissions Options

```bash
# Permission mode
openclaude --permission-mode default
openclaude --permission-mode bypassPermissions
openclaude --permission-mode plan
openclaude --permission-mode auto

# Skip permission prompts
openclaude --dangerously-skip-permissions

# Allow dangerous skip
openclaude --allow-dangerously-skip-permissions
```

### MCP Options

```bash
# MCP configuration
openclaude --mcp-config /path/to/mcp.json
openclaude --mcp-config '{"servers":{...}}'

# Strict MCP config (only use specified)
openclaude --strict-mcp-config
```

### Plugin Options

```bash
# Plugin directory
openclaude --plugin-dir /path/to/plugins
openclaude --plugin-dir /path1 --plugin-dir /path2

# Disable slash commands
openclaude --disable-slash-commands
```

### Budget Options

```bash
# Max turns
openclaude --max-turns 10

# Max budget
openclaude --max-budget-usd 1.00

# Task budget
openclaude --task-budget 100000
```

### Other Options

```bash
# Provider-specific
openclaude --provider openai --model gpt-4

# Agent
openclaude --agent reviewer

# Betas
openclaude --betas prompt-caching-2024-07-31

# Settings file
openclaude --settings /path/to/settings.json
openclaude --settings '{"theme":"dark"}'

# Setting sources
openclaude --setting-sources user,project

# Chrome integration
openclaude --chrome
openclaude --no-chrome

# Files to download
openclaude --file file_abc:doc.txt file_def:image.png
```

## Configuration Examples

### Example 1: Using Ollama

```bash
# .env file
PREFERRED_PROVIDER=ollama
OLLAMA_BASE_URL=http://localhost:11434
BIG_MODEL=codellama:34b
SMALL_MODEL=llama3:8b

# Usage
openclaude --provider ollama --model llama3:8b
```

### Example 2: Using OpenAI

```bash
# .env file
OPENAI_API_KEY=sk-...
PREFERRED_PROVIDER=openai

# Usage
openclaude --provider openai --model gpt-4
```

### Example 3: Using Gemini

```bash
# .env file
GEMINI_API_KEY=...
PREFERRED_PROVIDER=gemini

# Usage
openclaude --provider gemini --model gemini-pro
```

### Example 4: Production Setup

```json
// ~/.claude/config.json
{
  "theme": "dark",
  "verbose": false,
  "autoCompactEnabled": true,
  "permissions": {
    "allow": [
      "Read",
      "Edit",
      "Write",
      "Bash(npm test)",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm:*)",
      "Bash(curl:*)",
      "WebFetch"
    ]
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"]
    }
  }
}
```

### Example 5: Development Setup

```json
// .claude/config.json (project-specific)
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(npx:*)",
      "Read",
      "Write",
      "Edit"
    ]
  },
  "context": {
    "include": [
      "src/**/*.ts",
      "test/**/*.ts"
    ],
    "exclude": [
      "node_modules/**",
      "dist/**",
      "coverage/**"
    ]
  }
}
```

## Permission Patterns

### Bash Commands

```bash
# Allow all git commands
"Bash(git:*)"

# Allow specific git commands
"Bash(git commit:*)"
"Bash(git push:*)"

# Allow npm scripts
"Bash(npm run test)"
"Bash(npm run build:*)"

# Deny dangerous commands
"Bash(rm -rf:*)"
"Bash(curl:*)"
```

### Tool Access

```bash
# Allow all tools
"Read"
"Write"
"Edit"
"Glob"
"Grep"
"Bash"

# Deny specific tools
"!WebFetch"
"!Bash"
```

## Summary

OpenClaude's configuration system is:

1. **Hierarchical** - Defaults → Env → Global → Project → CLI flags
2. **Flexible** - Multiple ways to configure the same thing
3. **Secure** - Permission system with allow/deny rules
4. **Extensible** - MCP servers, plugins, custom providers
5. **User-friendly** - Sensible defaults, helpful flags

The configuration system allows you to:
- Use any AI provider
- Customize permissions per-project
- Add tools via MCP
- Control costs with budgets
- Optimize for your workflow
</task_progress>
</write_to_file>