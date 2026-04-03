# OpenClaude: Complete Implementation Guide

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [System Architecture](#2-system-architecture)
3. [Core Algorithms & Data Structures](#3-core-algorithms--data-structures)
4. [Complete System Design](#4-complete-system-design)
5. [REST APIs & Integration Points](#5-rest-apis--integration-points)
6. [Step-by-Step Implementation Guide](#6-step-by-step-implementation-guide)
7. [Configuration & Deployment](#7-configuration--deployment)
8. [Testing & Debugging](#8-testing--debugging)

---

## 1. Project Overview

### What is OpenClaude?

OpenClaude is a **CLI tool** that enables using **Claude Code with any LLM** (Large Language Model). It's built on top of Anthropic's Claude Code but extends it to work with multiple AI providers including:

- **OpenAI** (GPT models)
- **Google Gemini**
- **Ollama** (local models like Llama, Mistral)
- **DeepSeek**
- **200+ other models**

### Key Features

1. **Multi-Provider Support**: Works with any OpenAI-compatible API
2. **Smart Routing**: Automatically selects the best provider based on latency, cost, and health
3. **Local Model Support**: Works with Ollama and Atomic Chat (Apple Silicon)
4. **Bridge System**: Remote session management and control
5. **Plugin Architecture**: Extensible via plugins and skills
6. **Rich TUI**: Beautiful terminal interface built with React/Ink

### Tech Stack

- **Language**: TypeScript/JavaScript
- **Runtime**: Node.js (>=20.0.0) with Bun as build tool
- **UI Framework**: React 19 with Ink (React for CLI)
- **Build System**: Bun bundler
- **Package Manager**: npm/bun

---

## 2. System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER INPUT                                │
│                  (Terminal / Mobile / API)                        │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CLI ENTRY POINT                              │
│                    bin/openclaude                                 │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MAIN APPLICATION                              │
│                    src/main.tsx                                  │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    COMMANDER PROGRAM                             │
│                    Command routing & parsing                     │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    COMMAND HANDLER                               │
│                    src/commands/*.ts                             │
└─────────────────────────────────────────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                │                               │
                ▼                               ▼
┌──────────────────────────┐    ┌──────────────────────────┐
│     TOOL EXECUTION       │    │   DIRECT OPERATION       │
│  (File ops, Bash, etc.)  │    │  (Status, Config, etc.)  │
└──────────────────────────┘    └──────────────────────────┘
                │                               │
                └───────────────┬───────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    QUERY ENGINE                                  │
│                    src/QueryEngine.ts                            │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AI PROVIDER                                   │
│  Anthropic, OpenAI, Gemini, Ollama, etc.                        │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    OUTPUT RENDERING                              │
│                    React/Ink TUI                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Core Components

| Component | Purpose | Location |
|-----------|---------|----------|
| **CLI** | Entry point | `bin/openclaude` |
| **Main** | Application logic | `src/main.tsx` |
| **Commands** | Slash commands | `src/commands.ts` |
| **Tools** | AI capabilities | `src/tools.ts` |
| **Query Engine** | Conversation loop | `src/QueryEngine.ts` |
| **Bridge** | Remote control | `src/bridge/` |
| **Providers** | AI backends | `*.py`, `src/services/api/` |

---

## 3. Core Algorithms & Data Structures

### 3.1 Smart Router Algorithm

The smart router selects the best AI provider based on multiple factors:

```python
# smart_router.py - Core routing algorithm
class SmartRouter:
    def __init__(self):
        self.providers: dict[str, ProviderState] = {}
        self.strategy: RouterStrategy = "balanced"
    
    async def route(self, messages, claude_model="claude-sonnet"):
        """
        Route request to best available provider.
        
        Algorithm:
        1. Filter healthy providers
        2. Calculate score for each provider:
           - Latency score: 1 / avg_latency_ms
           - Cost score: 1 / cost_per_token
           - Health score: success_rate
           - Availability score: 1 if available, 0 if not
        
        3. Weight scores based on strategy:
           - "latency": 0.7 * latency + 0.1 * cost + 0.1 * health + 0.1 * availability
           - "cost": 0.1 * latency + 0.7 * cost + 0.1 * health + 0.1 * availability
           - "balanced": 0.3 * latency + 0.3 * cost + 0.2 * health + 0.2 * availability
        
        4. Select provider with highest score
        5. If all unhealthy, try fallback or raise error
        """
        healthy = [p for p in self.providers.values() if p.healthy]
        
        if not healthy:
            return await self._fallback(messages)
        
        scores = []
        for provider in healthy:
            score = self._calculate_score(provider)
            scores.append((provider, score))
        
        best = max(scores, key=lambda x: x[1])[0]
        return await self._call_provider(best, messages)
    
    def _calculate_score(self, provider):
        """Calculate weighted score based on strategy."""
        weights = self._get_weights()
        
        latency_score = 1 / max(provider.avg_latency_ms, 1)
        cost_score = 1 / max(provider.cost_per_token, 0.0001)
        health_score = provider.success_rate
        availability_score = 1.0 if provider.available else 0.0
        
        return (
            weights["latency"] * latency_score +
            weights["cost"] * cost_score +
            weights["health"] * health_score +
            weights["availability"] * availability_score
        )
```

### 3.2 Token Budget Algorithm

Manages token usage to prevent exceeding limits:

```typescript
// Token budget tracking
interface TokenBudget {
  total: number;
  used: number;
  remaining: number;
  warningThreshold: number;  // e.g., 0.8 (80%)
  blockingThreshold: number; // e.g., 0.95 (95%)
}

function checkTokenBudget(budget: TokenBudget): BudgetCheckResult {
  const usagePercent = budget.used / budget.total;
  
  if (usagePercent >= budget.blockingThreshold) {
    return {
      shouldContinue: false,
      reason: 'blocking_limit',
      message: 'Token budget exceeded. Use /compact to reduce context.'
    };
  }
  
  if (usagePercent >= budget.warningThreshold) {
    return {
      shouldContinue: true,
      reason: 'warning',
      message: `Token usage at ${(usagePercent * 100).toFixed(1)}%. Consider compacting.`
    };
  }
  
  return { shouldContinue: true, reason: 'ok' };
}
```

### 3.3 Context Compaction Algorithm

Reduces token usage while preserving important context:

```typescript
// Context compaction strategy
interface CompactionStrategy {
  maxTokens: number;
  preserveRecent: number;  // Keep last N messages
  preserveTools: boolean;  // Keep tool definitions
  summarizeOld: boolean;   // Summarize old messages
}

async function compactContext(
  messages: Message[],
  strategy: CompactionStrategy
): Promise<Message[]> {
  const totalTokens = countTokens(messages);
  
  if (totalTokens <= strategy.maxTokens) {
    return messages; // No compaction needed
  }
  
  // 1. Always keep recent messages
  const recent = messages.slice(-strategy.preserveRecent);
  const older = messages.slice(0, -strategy.preserveRecent);
  
  // 2. Summarize older messages if enabled
  let summarized: Message[];
  if (strategy.summarizeOld) {
    summarized = await summarizeMessages(older);
  } else {
    summarized = older;
  }
  
  // 3. Combine and return
  return [...summarized, ...recent];
}
```

### 3.4 Permission Check Algorithm

Validates tool permissions before execution:

```typescript
// Permission validation
interface PermissionRule {
  tool: string;
  pattern: string;  // e.g., "Bash(git:*)"
  action: 'allow' | 'deny';
}

function checkPermission(
  tool: string,
  input: any,
  rules: PermissionRule[]
): PermissionResult {
  // 1. Check deny rules first (higher priority)
  for (const rule of rules.filter(r => r.action === 'deny')) {
    if (matchesPattern(tool, input, rule.pattern)) {
      return {
        behavior: 'deny',
        message: `Operation denied by rule: ${rule.pattern}`
      };
    }
  }
  
  // 2. Check allow rules
  for (const rule of rules.filter(r => r.action === 'allow')) {
    if (matchesPattern(tool, input, rule.pattern)) {
      return { behavior: 'allow', updatedInput: input };
    }
  }
  
  // 3. Default: require user approval
  return {
    behavior: 'ask',
    message: 'Permission required. Approve? (y/n)'
  };
}
```

### 3.5 Session Management Data Structures

```typescript
// Session state management
interface Session {
  id: string;
  createdAt: Date;
  updatedAt: Date;
  messages: Message[];
  metadata: {
    model: string;
    provider: string;
    totalTokens: number;
    totalCost: number;
  };
}

interface AppState {
  // Conversation state
  messages: Message[];
  sessionId: string;
  
  // Tool state
  toolPermissionContext: ToolPermissionContext;
  activeTools: Tool[];
  
  // MCP state
  mcp: {
    clients: MCPServerConnection[];
    tools: Tool[];
    resources: ServerResource[];
  };
  
  // UI state
  fastMode: boolean;
  effortValue: 'low' | 'medium' | 'high' | 'max';
  
  // History state
  fileHistory: FileHistoryState;
  attribution: AttributionState;
}
```

---

## 4. Complete System Design

### 4.1 Command System Design

Commands follow a consistent pattern:

```typescript
// Command interface
interface Command {
  name: string;
  aliases?: string[];
  description: string;
  type: 'local' | 'prompt' | 'local-jsx';
  source: 'builtin' | 'plugin' | 'skills' | 'mcp';
  
  // For 'local' type
  action?: (args: string) => Promise<void>;
  
  // For 'prompt' type
  content?: string | ((args: string) => string);
  
  // For 'local-jsx' type
  render?: (args: string) => React.ReactNode;
}

// Example: Clear command
const clearCommand: Command = {
  name: 'clear',
  description: 'Clear the conversation',
  type: 'local',
  action: async () => {
    messages.length = 0;
    // Persist cleared state
    await saveSession(sessionId, []);
  }
};

// Example: Model picker command
const modelCommand: Command = {
  name: 'model',
  description: 'Change the AI model',
  type: 'local-jsx',
  render: (args) => <ModelPicker onSelect={handleModelSelect} />
};
```

### 4.2 Tool System Design

Tools provide AI capabilities:

```typescript
// Tool interface
interface Tool<Input, Output> {
  name: string;
  description: string;
  inputSchema: Input;  // Zod schema for validation
  outputSchema?: Output;
  
  // Core methods
  call(args: Input, context: ToolUseContext): Promise<ToolResult<Output>>;
  checkPermissions(input: Input, context: ToolUseContext): Promise<PermissionResult>;
  validateInput(input: Input, context: ToolUseContext): Promise<ValidationResult>;
  
  // UI methods
  renderToolUseMessage(input: Partial<Input>): React.ReactNode;
  renderToolResultMessage(output: Output): React.ReactNode;
  
  // Metadata
  isEnabled(): boolean;
  isReadOnly(input: Input): boolean;
  isConcurrencySafe(input: Input): boolean;
  isDestructive(input: Input): boolean;
}

// Example: BashTool
const BashTool = {
  name: 'Bash',
  description: 'Execute shell commands',
  inputSchema: z.object({
    command: z.string(),
    timeout: z.number().optional()
  }),
  
  async call(args, context) {
    const { command, timeout = 30000 } = args;
    
    return new Promise((resolve, reject) => {
      exec(command, { timeout }, (error, stdout, stderr) => {
        if (error) {
          resolve({ error: stderr });
        } else {
          resolve({ output: stdout });
        }
      });
    });
  },
  
  isReadOnly(input) {
    // Check if command is read-only
    const readOnlyCommands = ['cat', 'ls', 'grep', 'find'];
    return readOnlyCommands.some(cmd => input.command.startsWith(cmd));
  }
};
```

### 4.3 Query Engine Design

The query engine manages AI conversations:

```typescript
// Query engine configuration
interface QueryEngineConfig {
  cwd: string;
  tools: Tools;
  commands: Command[];
  mcpClients: MCPServerConnection[];
  agents: AgentDefinition[];
  canUseTool: CanUseToolFn;
  getAppState: () => AppState;
  setAppState: (f: (prev: AppState) => AppState) => void;
  initialMessages?: Message[];
  customSystemPrompt?: string;
  appendSystemPrompt?: string;
  userSpecifiedModel?: string;
  fallbackModel?: string;
  thinkingConfig?: ThinkingConfig;
  maxTurns?: number;
  maxBudgetUsd?: number;
  taskBudget?: { total: number };
}

// Query engine class
class QueryEngine {
  private config: QueryEngineConfig;
  private mutableMessages: Message[];
  private abortController: AbortController;
  private totalUsage: NonNullableUsage;
  
  async *submitMessage(
    prompt: string | ContentBlockParam[],
    options?: { uuid?: string; isMeta?: boolean }
  ): AsyncGenerator<SDKMessage, void, unknown> {
    // 1. Process user input
    const { messages: newMessages, shouldQuery } = await processUserInput({
      input: prompt,
      mode: 'prompt',
      context: this.config,
      messages: this.mutableMessages
    });
    
    // 2. Add to message history
    this.mutableMessages.push(...newMessages);
    
    // 3. Build system prompt
    const systemPrompt = await this.buildSystemPrompt();
    
    // 4. Call AI provider
    for await (const message of query({
      messages: this.mutableMessages,
      systemPrompt,
      tools: this.config.tools,
      canUseTool: this.config.canUseTool
    })) {
      // 5. Handle different message types
      if (message.type === 'assistant') {
        this.mutableMessages.push(message);
        yield message;
        
        // 6. Execute tools if requested
        const toolCalls = this.extractToolCalls(message);
        for (const toolCall of toolCalls) {
          const result = await this.executeTool(toolCall);
          this.mutableMessages.push(result);
          yield result;
        }
      }
    }
  }
}
```

### 4.4 Bridge System Design

The bridge enables remote session management:

```typescript
// Bridge configuration
interface BridgeConfig {
  apiBaseUrl: string;
  sessionIngressUrl: string;
  dir: string;
  spawnMode: 'single-session' | 'same-dir' | 'worktree';
  maxSessions: number;
  debugFile?: string;
}

// Bridge main loop
async function runBridgeLoop(
  config: BridgeConfig,
  environmentId: string,
  environmentSecret: string,
  api: BridgeApiClient,
  spawner: SessionSpawner,
  logger: BridgeLogger,
  signal: AbortSignal
): Promise<void> {
  const activeSessions = new Map<string, SessionHandle>();
  
  while (!signal.aborted) {
    // 1. Poll for work
    const work = await api.pollForWork(environmentId, environmentSecret);
    
    if (work) {
      // 2. Decode work secret
      const secret = decodeWorkSecret(work.secret);
      
      // 3. Spawn session
      const session = spawner.spawn({
        sdkUrl: secret.api_base_url,
        model: work.data.model
      }, config.dir);
      
      activeSessions.set(work.id, session);
    }
    
    // 4. Check active sessions
    for (const [id, session] of activeSessions) {
      if (session.processExited) {
        activeSessions.delete(id);
      }
    }
    
    // 5. Wait before next poll
    await sleep(config.pollInterval);
  }
}
```

---

## 5. REST APIs & Integration Points

### 5.1 Anthropic API Integration

```typescript
// src/services/api/claude.ts
export async function callAnthropicAPI(
  messages: Message[],
  model: string,
  systemPrompt: string,
  tools: Tool[]
): Promise<AsyncIterable<StreamChunk>> {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const stream = await client.messages.create({
    model,
    max_tokens: 4096,
    system: systemPrompt,
    messages: formatMessagesForAPI(messages),
    tools: formatToolsForAPI(tools),
    stream: true
  });
  
  return stream;
}
```

### 5.2 OpenAI API Integration

```typescript
// OpenAI-compatible provider
export async function callOpenAIAPI(
  messages: Message[],
  model: string,
  systemPrompt: string,
  tools: Tool[]
): Promise<AsyncIterable<StreamChunk>> {
  const client = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
    baseURL: process.env.OPENAI_BASE_URL || 'https://api.openai.com/v1'
  });
  
  const stream = await client.chat.completions.create({
    model,
    messages: formatMessagesForOpenAI(messages, systemPrompt),
    tools: formatToolsForOpenAI(tools),
    stream: true
  });
  
  return stream;
}
```

### 5.3 Ollama API Integration

```python
# ollama_provider.py
async def ollama_chat(
    model: str,
    messages: list[dict],
    system: str | None = None,
    max_tokens: int = 4096,
    temperature: float = 1.0,
) -> dict:
    """Send chat request to Ollama."""
    async with httpx.AsyncClient(timeout=120.0) as client:
        # Convert messages to Ollama format
        ollama_messages = anthropic_to_ollama_messages(messages)
        
        # Add system prompt
        if system:
            ollama_messages.insert(0, {
                "role": "system",
                "content": system
            })
        
        # Make request
        resp = await client.post(
            f"{OLLAMA_BASE_URL}/api/chat",
            json={
                "model": model,
                "messages": ollama_messages,
                "stream": False,
                "options": {
                    "num_predict": max_tokens,
                    "temperature": temperature
                }
            }
        )
        resp.raise_for_status()
        data = resp.json()
        
        # Convert to Anthropic format
        return {
            "id": f"msg_ollama_{data.get('created_at', 'unknown')}",
            "type": "message",
            "role": "assistant",
            "content": [{
                "type": "text",
                "text": data["message"]["content"]
            }],
            "model": model,
            "stop_reason": "end_turn",
            "usage": {
                "input_tokens": data.get("prompt_eval_count", 0),
                "output_tokens": data.get("eval_count", 0),
            },
        }
```

### 5.4 MCP Protocol Integration

```typescript
// MCP client implementation
export class MCPClient {
  private connections: Map<string, MCPServerConnection> = new Map();
  
  async connect(serverConfig: MCPServerConfig): Promise<MCPServerConnection> {
    const transport = new StdioClientTransport({
      command: serverConfig.command,
      args: serverConfig.args,
      env: serverConfig.env
    });
    
    const client = new Client({
      name: "openclaude",
      version: "1.0.0"
    });
    
    await client.connect(transport);
    
    // Discover tools
    const tools = await client.listTools();
    
    // Discover resources
    const resources = await client.listResources();
    
    const connection: MCPServerConnection = {
      client,
      transport,
      tools: tools.tools,
      resources: resources.resources
    };
    
    this.connections.set(serverConfig.name, connection);
    return connection;
  }
  
  async callTool(
    serverName: string,
    toolName: string,
    args: any
  ): Promise<any> {
    const connection = this.connections.get(serverName);
    if (!connection) {
      throw new Error(`MCP server ${serverName} not connected`);
    }
    
    const result = await connection.client.callTool({
      name: toolName,
      arguments: args
    });
    
    return result;
  }
}
```

---

## 6. Step-by-Step Implementation Guide

### Step 1: Project Setup

```bash
# 1. Clone repository
git clone https://github.com/anthropics/claude-code.git
cd claude-code

# 2. Install dependencies
npm install
# or
bun install

# 3. Set up environment variables
cp .env.example .env
# Edit .env with your API keys

# 4. Build the project
bun run build

# 5. Run in development mode
bun run dev
```

### Step 2: Configure AI Providers

```bash
# .env file configuration

# Anthropic (default)
ANTHROPIC_API_KEY=sk-ant-...

# OpenAI
OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1

# Ollama (local)
OLLAMA_BASE_URL=http://localhost:11434
PREFERRED_PROVIDER=ollama

# Gemini
GEMINI_API_KEY=...

# Smart Router
ROUTER_MODE=smart
ROUTER_STRATEGY=balanced
```

### Step 3: Implement Custom Command

```typescript
// src/commands/my-command/index.ts
import type { Command } from '../../commands.js';

const myCommand: Command = {
  name: 'mycommand',
  aliases: ['mc'],
  description: 'My custom command',
  type: 'local',
  source: 'builtin',
  
  async action(args: string) {
    console.log('My command executed with args:', args);
    // Implement your logic here
  }
};

export default myCommand;
```

### Step 4: Implement Custom Tool

```typescript
// src/tools/MyTool/MyTool.ts
import { z } from 'zod';
import type { Tool, ToolUseContext } from '../../Tool.js';

const inputSchema = z.object({
  param1: z.string(),
  param2: z.number().optional()
});

type Input = z.infer<typeof inputSchema>;

export const MyTool: Tool<Input, any> = {
  name: 'MyTool',
  description: 'Does something custom',
  inputSchema,
  
  async call(args: Input, context: ToolUseContext) {
    // Implement tool logic
    const result = await doSomething(args.param1);
    return { result };
  },
  
  async checkPermissions(input: Input, context: ToolUseContext) {
    // Check if operation is allowed
    return { behavior: 'allow' as const, updatedInput: input };
  },
  
  isEnabled() {
    return true;
  },
  
  isReadOnly(input: Input) {
    return true;
  }
};
```

### Step 5: Add MCP Server

```json
// ~/.claude/config.json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["path/to/server.js"],
      "env": {
        "API_KEY": "..."
      }
    }
  }
}
```

### Step 6: Configure Permissions

```json
// ~/.claude/config.json
{
  "permissions": {
    "allow": [
      "Read",
      "Edit",
      "Write",
      "Bash(npm test)",
      "Bash(git commit:*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(curl:*)"
    ]
  }
}
```

### Step 7: Run and Test

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
```

---

## 7. Configuration & Deployment

### 7.1 Configuration Hierarchy

Configuration is loaded in this order (later overrides earlier):

1. **Built-in defaults** - Hardcoded in source
2. **Environment variables** - `.env` file or system env
3. **Global config** - `~/.claude/config.json`
4. **Project config** - `.claude/config.json` (in project root)
5. **Command-line flags** - Arguments passed to CLI

### 7.2 Environment Variables

```bash
# Provider selection
PREFERRED_PROVIDER=anthropic  # or openai, ollama, gemini

# Anthropic
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_BASE_URL=https://api.anthropic.com

# OpenAI
OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1

# Ollama
OLLAMA_BASE_URL=http://localhost:11434

# Models
BIG_MODEL=claude-sonnet-4-20250514
SMALL_MODEL=claude-haiku-4-20250414

# Router
ROUTER_MODE=smart
ROUTER_STRATEGY=balanced

# Features
CLAUDE_CODE_DEBUG=false
CLAUDE_CODE_VERBOSE=false
```

### 7.3 Global Configuration

```json
// ~/.claude/config.json
{
  "theme": "dark",
  "editorMode": "vim",
  "verbose": false,
  "autoCompactEnabled": true,
  "permissions": {
    "allow": ["Read", "Edit", "Write"],
    "deny": ["Bash(rm -rf /)"]
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path"]
    }
  }
}
```

### 7.4 Build and Deploy

```bash
# Build for production
bun run build

# Type check
bun run typecheck

# Run tests
bun run test

# Package for distribution
npm pack

# Install globally
npm install -g .
```

---

## 8. Testing & Debugging

### 8.1 Running Tests

```bash
# Run all tests
bun run test

# Run specific test file
bun run test src/utils/model/model.test.ts

# Run with coverage
bun run test:coverage

# Run in watch mode
bun run test:watch
```

### 8.2 Debug Mode

```bash
# Enable debug logging
openclaude --debug

# Debug with specific categories
openclaude --debug api,hooks,tools

# Verbose output
openclaude --verbose

# Debug file
CLAUDE_CODE_DEBUG_FILE=/tmp/claude-debug.log openclaude
```

### 8.3 Common Debugging Scenarios

```bash
# Check provider connection
openclaude --debug api

# Test tool execution
openclaude --debug tools

# Monitor token usage
openclaude --verbose

# Check MCP server
openclaude --debug mcp
```

### 8.4 Performance Profiling

```bash
# Enable profiling
CLAUDE_CODE_PROFILE=true openclaude

# View profile
open ~/.claude/profile.json

# Analyze startup time
openclaude --debug startup
```

---

## Summary

OpenClaude is a production-ready CLI application with:

1. **Multi-provider AI support** - Works with OpenAI, Gemini, Ollama, etc.
2. **Rich tool ecosystem** - File ops, bash, web search, agents
3. **Flexible configuration** - Environment, global, project, CLI flags
4. **Session persistence** - Resume conversations across sessions
5. **Remote control** - Bridge system for mobile/web access
6. **Plugin extensibility** - Add custom commands, tools, MCP servers

### Key Implementation Points

1. **Start with CLI entry** - `bin/openclaude` → `dist/cli.mjs`
2. **Main application** - `src/main.tsx` handles initialization
3. **Commands** - Register in `src/commands.ts`
4. **Tools** - Implement in `src/tools/`
5. **Providers** - Configure in `.env` or `config.json`
6. **Testing** - Use `bun run test`

### Next Steps

1. **Read existing analysis** - Files in `analysis/` folder
2. **Explore source code** - Use analysis as guide
3. **Run locally** - `bun run dev`
4. **Add features** - Commands, tools, providers
5. **Deploy** - `bun run build && npm pack`

---

**Document Version**: 1.0
**Last Updated**: 2026-04-03
**Files Covered**: Complete codebase analysis