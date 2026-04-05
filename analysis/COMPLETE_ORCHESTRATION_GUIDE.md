# Complete OpenClaude Orchestration Guide

> **📘 How to Use This Guide**
> 
> This guide explains how OpenClaude works from start to finish. It's designed for developers who want to understand:
> - How the application starts and processes user input
> - How commands and tools work
> - How AI providers are integrated
> - How to extend the system
> 
> **No prior knowledge of the codebase required!** We explain each concept with examples.

## Table of Contents
1. [What is OpenClaude?](#what-is-openclaude)
2. [High-Level Architecture](#high-level-architecture)
3. [Complete Flow: From Start to Finish](#complete-flow-from-start-to-finish)
4. [Key Components Deep Dive](#key-components-deep-dive)
5. [Data Flow Diagrams](#data-flow-diagrams)
6. [Common Scenarios](#common-scenarios)
7. [Troubleshooting Guide](#troubleshooting-guide)

---

## What is OpenClaude?

**OpenClaude** is a command-line tool that lets you use AI-powered code assistance with **any AI model** you choose. Think of it as a universal adapter for AI models - instead of being locked into one provider (like Anthropic's Claude), you can use OpenAI's GPT, Google's Gemini, or even run models locally with Ollama.

### Why Does This Exist?

Anthropic's Claude Code is powerful but only works with Claude models. OpenClaude extends it so you can:
- Use cheaper models (like Ollama running locally - **free!**)
- Use different models for different tasks
- Run everything offline if needed
- Switch providers without changing your workflow

### Key Capabilities Explained

| Capability | What It Means | Example Use Case |
|------------|---------------|------------------|
| **Multi-Provider Support** | Connect to any AI service | Use GPT-4 for complex tasks, Ollama for simple ones |
| **Smart Routing** | Automatically picks the best provider | Sends quick questions to fast models, complex ones to smart models |
| **Local Model Support** | Run AI on your own machine | Work offline, no API costs, complete privacy |
| **Rich TUI** | Beautiful terminal interface | Easy to read responses with syntax highlighting |
| **Bridge System** | Control from mobile/web | Start work on desktop, continue on phone |
| **Plugin Architecture** | Add custom features | Create your own commands and tools |

### Simple Example

```bash
# Using Claude (paid, cloud-based)
$ openclaude --model claude-sonnet "Explain this code"

# Using OpenAI (paid, cloud-based)  
$ openclaude --provider openai --model gpt-4 "Explain this code"

# Using Ollama (FREE, runs on your machine!)
$ openclaude --provider ollama --model llama3:8b "Explain this code"
```

All three commands work the same way - OpenClaude handles the differences!

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER                                     │
│                    (Terminal / Mobile)                           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CLI ENTRY POINT                               │
│                    bin/openclaude                                │
│                                                                  │
│  • Checks for dist/cli.mjs                                       │
│  • Dynamically imports compiled code                             │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MAIN APPLICATION                              │
│                    src/main.tsx                                  │
│                                                                  │
│  • Security setup                                                │
│  • Signal handlers                                               │
│  • Parse early arguments                                         │
│  • Determine interactive/non-interactive                         │
│  • Set client type                                               │
│  • Load settings                                                 │
│  • Call run()                                                    │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    COMMANDER PROGRAM                             │
│                                                                  │
│  • Creates Commander instance                                    │
│  • Registers all commands                                        │
│  • Sets up preAction hook                                        │
│  • Parses argv                                                   │
│  • Routes to command handler                                     │
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
│                                                                  │
│  • Manages conversation context                                  │
│  • Builds system prompt                                          │
│  • Handles message history                                       │
│  • Orchestrates tool execution                                   │
│  • Manages token limits                                          │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AI PROVIDER                                   │
│  Anthropic, OpenAI, Gemini, Ollama, etc.                        │
│                                                                  │
│  Sends request and receives streaming response                  │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    OUTPUT RENDERING                              │
│                    React/Ink TUI                                 │
│                                                                  │
│  • Format response                                               │
│  • Apply syntax highlighting                                     │
│  • Render in TUI                                                 │
│  • Stream to stdout (non-interactive)                            │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       USER OUTPUT                                │
│                  (Terminal / Mobile / API)                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Complete Flow: From Start to Finish

### Phase 1: Startup (bin/openclaude → main.tsx)

```
1. User runs: openclaude [options] [prompt]
   │
   ▼
2. bin/openclaude executes
   │
   ├─ Checks if dist/cli.mjs exists
   ├─ If yes: imports and runs it
   └─ If no: shows error with build instructions
   │
   ▼
3. main.tsx loads (compiled as dist/cli.mjs)
   │
   ├─ Security: Prevents Windows PATH hijacking
   ├─ Initializes warning handler
   ├─ Sets up signal handlers (SIGINT, exit)
   │
   ▼
4. Early argument processing
   │
   ├─ Check for cc:// or cc+unix:// URL (direct connect)
   ├─ Check for deep link URI (--handle-uri)
   ├─ Check for assistant mode (assistant [sessionId])
   ├─ Check for SSH mode (ssh <host> [dir])
   ├─ Determine interactive vs non-interactive
   │  └─ Non-interactive if: -p/--print, --init-only, --sdk-url, or !isTTY
   │
   ▼
5. Set client type
   │
   ├─ github-action (if GITHUB_ACTIONS env)
   ├─ sdk-typescript (if CLAUDE_CODE_ENTRYPOINT=sdk-ts)
   ├─ sdk-python (if CLAUDE_CODE_ENTRYPOINT=sdk-py)
   ├─ sdk-cli (if CLAUDE_CODE_ENTRYPOINT=sdk-cli)
   ├─ claude-vscode (if CLAUDE_CODE_ENTRYPOINT=claude-vscode)
   ├─ local-agent (if CLAUDE_CODE_ENTRYPOINT=local-agent)
   ├─ claude-desktop (if CLAUDE_CODE_ENTRYPOINT=claude-desktop)
   ├─ remote (if session ingress token provided)
   └─ cli (default)
   │
   ▼
6. Load settings from flags (if provided)
   │
   ├─ --settings <file-or-json>
   └─ --setting-sources <sources>
   │
   ▼
7. Call run()
```

### Phase 2: Commander Setup (run() function)

```
1. Create Commander program
   │
   ├─ Configure help (sorted options)
   ├─ Enable positional options
   ├─ Set name: "claude"
   ├─ Set description
   ├─ Add [prompt] argument
   │
   ▼
2. Add all CLI options
   │
   ├─ Debug options: -d, --debug, --debug-file, --verbose
   ├─ Output options: -p/--print, --output-format, --json-schema
   ├─ Input options: --input-format, --replay-user-messages
   ├─ Model options: --model, --provider, --fallback-model
   ├─ Session options: -c/--continue, -r/--resume, --fork-session
   ├─ Tool options: --tools, --allowed-tools, --disallowed-tools
   ├─ MCP options: --mcp-config, --strict-mcp-config
   ├─ Permission options: --permission-mode, --dangerously-skip-permissions
   ├─ System prompt options: --system-prompt, --append-system-prompt
   ├─ Agent options: --agent, --agents
   ├─ Worktree options: --worktree, --tmux
   ├─ And many more...
   │
   ▼
3. Set up preAction hook
   │
   ├─ Load MDM settings
   ├─ Wait for keychain prefetch
   ├─ Call init() - initializes plugins, skills, MCP
   ├─ Set process title
   ├─ Initialize logging sinks
   ├─ Load inline plugins (if --plugin-dir)
   ├─ Run migrations
   ├─ Load remote managed settings (non-blocking)
   └─ Load policy limits (non-blocking)
   │
   ▼
4. Add subcommands
   │
   ├─ mcp (MCP server management)
   ├─ plugin (Plugin management)
   ├─ auth (Authentication)
   ├─ doctor (Diagnostics)
   └─ ... (many more)
   │
   ▼
5. Parse argv and route to handler
```

### Phase 3: Action Handler (default command)

```
1. Parse action handler options
   │
   ├─ Extract all parsed options
   ├─ Handle --bare mode (sets CLAUDE_CODE_SIMPLE=1)
   ├─ Ignore "code" as prompt (shows tip)
   │
   ▼
2. Handle assistant mode (KAIROS feature)
   │
   ├─ Check if .claude/settings.json has assistant: true
   ├─ Check GrowthBook gate
   ├─ Check trust dialog accepted
   ├─ Initialize assistant team if enabled
   │
   ▼
3. Process tool and permission options
   │
   ├─ Parse --tools, --allowed-tools, --disallowed-tools
   ├─ Initialize tool permission context
   ├─ Handle overly broad permissions
   ├─ Strip dangerous permissions for auto mode
   │
   ▼
4. Process MCP configuration
   │
   ├─ Parse --mcp-config (files or JSON strings)
   ├─ Validate MCP configs
   ├─ Apply policy filtering
   ├─ Handle enterprise MCP config
   ├─ Handle Claude in Chrome MCP
   ├─ Handle Computer Use MCP (Chicago)
   │
   ▼
5. Process Claude in Chrome
   │
   ├─ Check if enabled via CLI or auto-enable
   ├─ Verify claude.ai subscriber (unless ant)
   ├─ Setup Chrome MCP server
   ├─ Add Chrome tools to allowed list
   │
   ▼
6. Process system prompt options
   │
   ├─ --system-prompt or --system-prompt-file
   ├─ --append-system-prompt or --append-system-prompt-file
   ├─ Add teammate system prompt addendum (if applicable)
   │
   ▼
7. Load MCP configs
   │
   ├─ Start loading MCP configs early (non-blocking)
   ├─ Fetch claude.ai MCP configs (if eligible)
   ├─ Apply policy filtering
   │
   ▼
8. Load tools
   │
   ├─ Call getTools(toolPermissionContext)
   ├─ Apply coordinator mode filtering (if applicable)
   ├─ Add SyntheticOutputTool (if JSON schema provided)
   │
   ▼
9. Call setup()
   │
   ├─ Initialize working directory
   ├─ Handle worktree setup (if --worktree)
   ├─ Set up session directories
   ├─ Configure git worktrees
   ├─ Process.chdir() if worktree
   │
   ▼
10. Load commands and agents (parallel with setup)
    │
    ├─ getCommands(cwd) - loads all commands
    ├─ getAgentDefinitionsWithOverrides(cwd) - loads agents
    │
    ▼
11. Handle branch from here based on mode
    │
    ├─ Interactive mode → launch REPL
    ├─ Non-interactive mode (-p) → runHeadless
    └─ Other modes → specific handlers
```

### Phase 4: Interactive Mode (REPL)

```
1. Show setup screens (if first time)
   │
   ├─ Trust dialog
   ├─ API key approval
   ├─ Permission mode dialog
   │
   ▼
2. Launch REPL
   │
   ├─ Create React/Ink root
   ├─ Render REPL component
   ├─ Set up input handling
   ├─ Start deferred prefetches
   │
   ▼
3. REPL loop
   │
   ├─ Wait for user input
   ├─ Process input
   │  ├─ Check for slash commands
   │  ├─ Check for special keys (Ctrl+C, etc.)
   │  └─ Send to AI if regular input
   │
   ├─ Create QueryEngine
   ├─ Submit message to QueryEngine
   ├─ Stream responses
   ├─ Update UI
   └─ Repeat
   │
   ▼
4. Session persistence
   │
   ├─ Save messages to transcript
   ├─ Save session metadata
   └─ Enable --resume
```

### Phase 5: Non-Interactive Mode (-p/--print)

```
1. Process input
   │
   ├─ Get input from stdin or prompt
   ├─ Handle input format (text or stream-json)
   │
   ▼
2. Create QueryEngine
   │
   ├─ Configure with tools, commands, etc.
   ├─ Set up system prompt
   ├─ Set up initial messages
   │
   ▼
3. Submit message
   │
   ├─ Process user input
   ├─ Build system prompt
   ├─ Call AI provider
   ├─ Handle tool execution
   ├─ Stream responses
   │
   ▼
4. Format output
   │
   ├─ Based on --output-format (text, json, stream-json)
   ├─ Include hook events (if --include-hook-events)
   ├─ Include partial messages (if --include-partial-messages)
   │
   ▼
5. Exit
   │
   └─ process.exit(exitCode)
```

---

## Key Components Deep Dive

### 1. QueryEngine (src/QueryEngine.ts)

The QueryEngine manages the complete AI conversation lifecycle.

```typescript
class QueryEngine {
  // Configuration
  config: QueryEngineConfig
  
  // State
  mutableMessages: Message[]
  abortController: AbortController
  permissionDenials: SDKPermissionDenial[]
  totalUsage: NonNullableUsage
  
  // Main method
  async *submitMessage(
    prompt: string | ContentBlockParam[],
    options?: { uuid?: string; isMeta?: boolean }
  ): AsyncGenerator<SDKMessage, void, unknown> {
    // 1. Process user input
    const { messages: messagesFromUserInput, shouldQuery } = 
      await processUserInput({...})
    
    // 2. Push new messages
    this.mutableMessages.push(...messagesFromUserInput)
    
    // 3. Build system prompt
    const systemPrompt = await this.buildSystemPrompt()
    
    // 4. Call AI provider
    for await (const message of query({
      messages: this.mutableMessages,
      systemPrompt,
      tools: this.config.tools,
      canUseTool: this.config.canUseTool
    })) {
      // 5. Handle different message types
      if (message.type === 'assistant') {
        this.mutableMessages.push(message)
        yield message
        
        // 6. Execute tools if requested
        const toolCalls = this.extractToolCalls(message)
        for (const toolCall of toolCalls) {
          const result = await this.executeTool(toolCall)
          this.mutableMessages.push(result)
          yield result
        }
      }
    }
  }
}
```

### 2. Commands System (src/commands.ts)

The commands system manages all slash commands.

```typescript
// Command types
type Command = {
  name: string
  type: 'local' | 'prompt' | 'local-jsx'
  description: string
  action?: (args: string) => Promise<void>  // for 'local'
  content?: string | ((args: string) => string)  // for 'prompt'
  render?: (args: string) => React.ReactNode  // for 'local-jsx'
}

// Command loading
const COMMANDS = memoize((): Command[] => [
  // 80+ built-in commands
  addDir, advisor, agents, branch, btw, chrome, clear, color,
  compact, config, copy, desktop, context, cost, diff, dream,
  doctor, effort, exit, fast, files, heapDump, help, ide, init,
  keybindings, mcp, memory, mobile, model, plugin, provider,
  resume, session, skills, status, tag, tasks, theme, vim,
  // ... and many more
])

// Dynamic commands from skills, plugins, MCP, workflows
async function getCommands(cwd: string): Promise<Command[]> {
  const allCommands = await loadAllCommands(cwd)
  // Filter by availability and enabled status
  return allCommands.filter(
    _ => meetsAvailabilityRequirement(_) && isCommandEnabled(_)
  )
}
```

### 3. Tools System (src/tools.ts)

The tools system provides AI capabilities.

```typescript
// Tool interface
interface Tool<Input, Output> {
  name: string
  description: string
  inputSchema: Input  // Zod schema
  outputSchema?: Output
  
  call(args: Input, context: ToolUseContext): Promise<ToolResult<Output>>
  checkPermissions(input: Input, context: ToolUseContext): Promise<PermissionResult>
  validateInput(input: Input, context: ToolUseContext): Promise<ValidationResult>
  
  isEnabled(): boolean
  isReadOnly(input: Input): boolean
  isDestructive(input: Input): boolean
}

// All base tools
function getAllBaseTools(): Tools {
  return [
    AgentTool, TaskOutputTool, BashTool,
    GlobTool, GrepTool, ExitPlanModeV2Tool,
    FileReadTool, FileEditTool, FileWriteTool,
    NotebookEditTool, WebFetchTool, TodoWriteTool,
    WebSearchTool, TaskStopTool, AskUserQuestionTool,
    SkillTool, EnterPlanModeTool,
    // ... 50+ tools (many feature-gated)
  ]
}

// Tool filtering
function getTools(permissionContext: ToolPermissionContext): Tools {
  const tools = getAllBaseTools()
  return filterToolsByDenyRules(tools, permissionContext)
    .filter(tool => tool.isEnabled())
}

// Tool pool assembly (includes MCP tools)
function assembleToolPool(
  permissionContext: ToolPermissionContext,
  mcpTools: Tools
): Tools {
  const builtInTools = getTools(permissionContext)
  const allowedMcpTools = filterToolsByDenyRules(mcpTools, permissionContext)
  
  // Sort for cache stability, deduplicate
  return uniqBy(
    [...builtInTools].sort(byName).concat(allowedMcpTools.sort(byName)),
    'name'
  )
}
```

### 4. Smart Router (python/smart_router.py)

The smart router selects the best AI provider.

```python
class SmartRouter:
    def __init__(self):
        self.providers = build_default_providers()
        self.strategy = os.getenv("ROUTER_STRATEGY", "balanced")
        self.fallback_enabled = os.getenv("ROUTER_FALLBACK", "true") == "true"
    
    async def initialize(self):
        """Ping all providers and build initial latency scores."""
        await asyncio.gather(
            *[self._ping_provider(p) for p in self.providers]
        )
    
    async def route(self, messages, claude_model="claude-sonnet"):
        """Route request to best provider."""
        # Select best provider based on strategy
        provider = min(
            [p for p in self.providers if p.healthy and p.is_configured],
            key=lambda p: p.score(self.strategy)
        )
        
        # Map Claude model to provider's model
        model = self.get_model_for_provider(provider, claude_model)
        
        return {
            "provider": provider.name,
            "model": model,
            "api_key": provider.api_key,
        }
    
    async def record_result(self, provider_name, success, duration_ms):
        """Update provider scores based on actual performance."""
        # Update latency, error rate, health
```

### 5. Bridge System (src/bridge/)

The bridge system enables remote session management.

```typescript
// Main bridge loop
async function runBridgeLoop(
  config: BridgeConfig,
  environmentId: string,
  environmentSecret: string,
  api: BridgeApiClient,
  spawner: SessionSpawner,
  logger: BridgeLogger,
  signal: AbortSignal
): Promise<void> {
  const activeSessions = new Map<string, SessionHandle>()
  
  while (!signal.aborted) {
    // Poll for work
    const work = await api.pollForWork(environmentId, environmentSecret)
    
    if (work) {
      // Decode work secret
      const secret = decodeWorkSecret(work.secret)
      
      // Spawn session
      const session = spawner.spawn({
        sdkUrl: secret.api_base_url,
        model: work.data.model
      }, config.dir)
      
      activeSessions.set(work.id, session)
    }
    
    // Check active sessions
    for (const [id, session] of activeSessions) {
      if (session.processExited) {
        activeSessions.delete(id)
      }
    }
    
    // Wait before next poll
    await sleep(config.pollInterval)
  }
}
```

---

## Data Flow Diagrams

### Message Flow

```
User Input
    │
    ▼
┌─────────────────┐
│  Input Parser   │
│  (main.tsx)     │
└─────────────────┘
    │
    ▼
┌─────────────────┐     ┌─────────────────┐
│  Command Router │────▶│  Slash Command  │
│  (commands.ts)  │     │  Handler        │
└─────────────────┘     └─────────────────┘
    │ (if not command)
    ▼
┌─────────────────┐
│  Query Engine   │
│  (QueryEngine)  │
└─────────────────┘
    │
    ▼
┌─────────────────┐     ┌─────────────────┐
│  AI Provider    │────▶│  Tool Executor  │
│  (API call)     │     │  (tools.ts)     │
└─────────────────┘     └─────────────────┘
    │
    ▼
┌─────────────────┐
│  Response       │
│  Formatter      │
└─────────────────┘
    │
    ▼
User Output
```

### Tool Execution Flow

```
AI requests tool use
    │
    ▼
┌─────────────────┐
│  Permission     │
│  Check          │
└─────────────────┘
    │
    ├─ Allow → Execute tool
    ├─ Deny → Return error
    └─ Ask → Prompt user
    │
    ▼
┌─────────────────┐
│  Tool           │
│  Execution      │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│  Result         │
│  Processing     │
└─────────────────┘
    │
    ▼
Return to AI
```

### Session Persistence Flow

```
Session Start
    │
    ▼
┌─────────────────┐
│  Create Session │
│  ID (UUID)      │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│  User Messages  │
│  + AI Responses │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│  Save to        │
│  Transcript     │
│  (~/.claude/    │
│   projects/     │
│   <cwd>/        │
│   sessions/     │
│   <id>.jsonl)   │
└─────────────────┘
    │
    ▼
Session End
    │
    ▼
Resume Later
    │
    ▼
┌─────────────────┐
│  Load           │
│  Transcript     │
└─────────────────┘
    │
    ▼
Continue Conversation
```

---

## Common Scenarios

### Scenario 1: Simple Interactive Session

```bash
# User runs OpenClaude interactively
$ openclaude

# What happens:
1. bin/openclaude loads dist/cli.mjs
2. main.tsx runs, determines interactive mode
3. run() creates Commander program
4. preAction hook calls init()
5. Action handler runs setup()
6. launchRepl() starts the TUI
7. User types prompts, gets AI responses
8. Session is saved for --resume
```

### Scenario 2: Non-Interactive One-Liner

```bash
# User runs OpenClaude with -p flag
$ openclaude -p "What is 2+2?"

# What happens:
1. bin/openclaude loads dist/cli.mjs
2. main.tsx detects -p flag, sets non-interactive
3. run() creates Commander program
4. preAction hook calls init()
5. Action handler creates QueryEngine
6. QueryEngine.submitMessage("What is 2+2?")
7. AI responds, output printed to stdout
8. process.exit(0)
```

### Scenario 3: Using a Specific Model

```bash
# User specifies model
$ openclaude --model sonnet

# What happens:
1. --model flag parsed in Commander
2. Model stored in state
3. QueryEngine uses specified model
4. AI provider receives model name
5. Response generated with that model
```

### Scenario 4: Using Local Ollama

```bash
# User wants to use local Ollama
$ openclaude --provider ollama --model llama3:8b

# What happens:
1. --provider flag parsed
2. Provider set to 'ollama'
3. Smart router (or direct call) routes to Ollama
4. Ollama API called at http://localhost:11434
5. Local model generates response
6. No API costs!
```

### Scenario 5: Resuming a Session

```bash
# User resumes previous session
$ openclaude --resume

# What happens:
1. --resume flag parsed
2. Session picker shown (interactive) or latest loaded
3. Transcript loaded from ~/.claude/projects/<cwd>/sessions/<id>.jsonl
4. Messages restored to QueryEngine
5. Conversation continues from where it left off
```

### Scenario 6: Using MCP Servers

```bash
# User loads MCP servers
$ openclaude --mcp-config mcp.json

# What happens:
1. --mcp-config parsed
2. MCP config file read and validated
3. MCP servers started (stdio transport)
4. MCP tools and resources discovered
5. Tools added to tool pool
6. AI can now use MCP tools
```

---

## Troubleshooting Guide

### Issue: "dist/cli.mjs not found"

```bash
# Solution: Build the project
bun run build

# Or run directly with Bun
bun run dev
```

### Issue: "No API key found"

```bash
# Solution: Set API key in environment
export ANTHROPIC_API_KEY=sk-ant-...

# Or use --settings to provide key
openclaude --settings '{"apiKey": "sk-ant-..."}'
```

### Issue: "Permission denied" for tools

```bash
# Solution: Use --dangerously-skip-permissions (only in trusted environments)
openclaude --dangerously-skip-permissions

# Or configure permissions in ~/.claude/config.json
{
  "permissions": {
    "allow": ["Bash(git:*)", "Read", "Edit", "Write"]
  }
}
```

### Issue: Slow startup

```bash
# Solution: Use --bare mode for minimal startup
openclaude --bare

# Or profile startup
CLAUDE_CODE_PROFILE=true openclaude
```

### Issue: Ollama not connecting

```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# Start Ollama if not running
ollama serve

# Pull a model if needed
ollama pull llama3:8b
```

### Issue: Session not resuming

```bash
# Check session directory
ls ~/.claude/projects/$(pwd)/sessions/

# List recent sessions
openclaude --resume

# Manually specify session ID
openclaude --resume <session-id>
```

---

## Summary

OpenClaude's architecture is designed to be:

1. **Modular**: Each component has a clear responsibility
2. **Extensible**: Easy to add new commands, tools, and providers
3. **Robust**: Extensive error handling and recovery
4. **Performant**: Parallel initialization, lazy loading, caching
5. **Secure**: Permission system, sandboxing, validation

The complete flow from user input to AI response involves:
- CLI parsing → Commander routing → Command handling → Query Engine → AI Provider → Tool execution → Response formatting → Output

Understanding this flow helps with debugging, extending, and effectively using OpenClaude.