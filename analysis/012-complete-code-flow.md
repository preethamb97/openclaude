# 012 - Complete Code Flow and Integration Points

## Overview

This document traces the complete code flow from user input to AI response, showing how all components integrate together.

## Complete Flow Diagram

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
│                                                                  │
│  • Checks for dist/cli.mjs                                      │
│  • Dynamically imports compiled code                            │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MAIN APPLICATION                              │
│                    src/main.tsx                                  │
│                                                                  │
│  main() function:                                               │
│  • Security setup                                               │
│  • Signal handlers                                              │
│  • Parse early arguments                                        │
│  • Determine interactive/non-interactive                        │
│  • Set client type                                              │
│  • Load settings                                                │
│  • Call run()                                                   │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    COMMANDER PROGRAM                             │
│                                                                  │
│  run() function:                                                │
│  • Creates Commander instance                                   │
│  • Registers all commands                                       │
│  • Sets up preAction hook                                       │
│  • Parses argv                                                  │
│  • Routes to command handler                                    │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    COMMAND HANDLER                               │
│                    src/commands/*.ts                             │
│                                                                  │
│  • Receives parsed arguments                                    │
│  • Validates input                                              │
│  • Creates necessary context                                    │
│  • Calls appropriate service/tool                               │
└─────────────────────────────────────────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                │                               │
                ▼                               ▼
┌──────────────────────────┐    ┌──────────────────────────┐
│     TOOL EXECUTION       │    │   DIRECT OPERATION       │
│                          │    │                          │
│  • BashTool             │    │  • Status check          │
│  • FileReadTool         │    │  • Config update         │
│  • FileWriteTool        │    │  • Session management    │
│  • WebSearchTool        │    │  • History access        │
│  • etc.                 │    │  • etc.                  │
└──────────────────────────┘    └──────────────────────────┘
                │                               │
                └───────────────┬───────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    QUERY ENGINE                                  │
│                    src/QueryEngine.ts                            │
│                                                                  │
│  For AI-powered commands:                                       │
│  • Manages conversation context                                 │
│  • Builds system prompt                                         │
│  • Handles message history                                      │
│  • Orchestrates tool execution                                  │
│  • Manages token limits                                         │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AI PROVIDER                                   │
│                    src/services/api/*.ts                         │
│                                                                  │
│  • Anthropic (Claude)                                           │
│  • OpenAI (GPT)                                                 │
│  • Gemini (Google)                                              │
│  • Ollama (Local)                                               │
│  • etc.                                                         │
│                                                                  │
│  Sends request and receives streaming response                  │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RESPONSE PROCESSING                           │
│                                                                  │
│  • Parse AI response                                            │
│  • Extract tool calls                                           │
│  • Execute tools                                                │
│  • Send tool results back to AI                                 │
│  • Repeat until AI is done                                      │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    OUTPUT RENDERING                              │
│                    src/ink/ + src/components/                    │
│                                                                  │
│  • Format response                                              │
│  • Apply syntax highlighting                                    │
│  • Render in TUI                                                │
│  • Stream to stdout (non-interactive)                           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       USER OUTPUT                                │
│                  (Terminal / Mobile / API)                        │
└─────────────────────────────────────────────────────────────────┘
```

## Detailed Flow: User Types "explain this code"

### Step 1: Input Parsing

```typescript
// src/main.tsx - main()
const cliArgs = process.argv.slice(2)
// cliArgs = ["-p", "explain this code"]

const hasPrintFlag = cliArgs.includes('-p')
// hasPrintFlag = true

const isNonInteractive = hasPrintFlag
// isNonInteractive = true
```

### Step 2: Commander Setup

```typescript
// src/main.tsx - run()
const program = new CommanderCommand()
  .name('claude')
  .argument('[prompt]', 'Your prompt')
  .option('-p, --print', 'Print response and exit')
  .action(async (prompt, options) => {
    // This is the default action for prompts
    await handlePrompt(prompt, options)
  })
```

### Step 3: Command Routing

```typescript
// When user runs: openclaude -p "explain this code"
// Commander parses:
// - prompt = "explain this code"
// - options.print = true

// Routes to default action handler
program.parseAsync(['node', 'openclaude', '-p', 'explain this code'])
```

### Step 4: Prompt Handler

```typescript
// src/main.tsx - handlePrompt()
async function handlePrompt(prompt: string, options: any) {
  // 1. Create QueryEngine
  const engine = new QueryEngine({
    cwd: process.cwd(),
    tools: getTools(permissionContext),
    canUseTool,
    getAppState: () => appState,
    setAppState: updateAppState,
    userSpecifiedModel: options.model,
    verbose: options.verbose
  })
  
  // 2. Submit message
  const messages = engine.submitMessage(prompt)
  
  // 3. Process responses
  for await (const message of messages) {
    // Output each message
    if (options.print) {
      // Non-interactive: print to stdout
      process.stdout.write(formatMessage(message))
    }
  }
}
```

### Step 5: Query Engine Processing

```typescript
// src/QueryEngine.ts - submitMessage()
async *submitMessage(prompt: string) {
  // 1. Build system context
  const systemContext = await getSystemContext()
  const userContext = await getUserContext()
  
  // 2. Build system prompt
  const systemPrompt = buildSystemPrompt({
    tools: this.config.tools,
    systemContext,
    userContext,
    model: this.config.userSpecifiedModel
  })
  
  // 3. Create user message
  const userMessage = createUserMessage(prompt)
  this.mutableMessages.push(userMessage)
  
  // 4. Call AI provider
  for await (const response of query({
    messages: this.mutableMessages,
    systemPrompt,
    tools: this.config.tools,
    canUseTool: this.config.canUseTool
  })) {
    // 5. Yield each response message
    yield response
    
    // 6. If response contains tool calls, execute them
    if (response.type === 'assistant') {
      const toolCalls = extractToolCalls(response)
      
      for (const toolCall of toolCalls) {
        // 7. Execute tool
        const toolResult = await executeTool(toolCall)
        
        // 8. Add tool result to messages
        this.mutableMessages.push(toolResult)
        
        // 9. Yield tool result
        yield toolResult
      }
    }
  }
}
```

### Step 6: AI Provider Call

```typescript
// src/query.ts - query()
async function* query(params: QueryParams) {
  const {
    messages,
    systemPrompt,
    tools,
    canUseTool
  } = params
  
  // 1. Normalize messages for API
  const normalizedMessages = normalizeMessagesForAPI(messages, tools)
  
  // 2. Call the AI provider
  const stream = await callModel({
    model: getMainLoopModel(),
    messages: normalizedMessages,
    system: systemPrompt,
    tools: formatToolsForAPI(tools),
    stream: true
  })
  
  // 3. Stream response
  for await (const chunk of stream) {
    // Parse chunk
    const message = parseStreamChunk(chunk)
    
    // Yield message
    yield message
  }
}
```

### Step 7: Tool Execution

```typescript
// src/services/tools/toolOrchestration.ts
export async function executeTool(
  toolCall: ToolUseBlock,
  tools: Tool[],
  canUseTool: CanUseToolFn,
  context: ToolUseContext
): Promise<ToolResultMessage> {
  // 1. Find the tool
  const tool = tools.find(t => t.name === toolCall.name)
  
  if (!tool) {
    return createToolError(toolCall.id, `Tool ${toolCall.name} not found`)
  }
  
  // 2. Check permissions
  const permissionResult = await canUseTool(
    tool,
    toolCall.input,
    context
  )
  
  if (permissionResult.behavior !== 'allow') {
    return createToolError(toolCall.id, 'Permission denied')
  }
  
  // 3. Execute tool
  try {
    const result = await tool.call(toolCall.input, context)
    
    // 4. Return result
    return createToolResult(toolCall.id, result)
  } catch (error) {
    return createToolError(toolCall.id, error.message)
  }
}
```

### Step 8: Output Formatting

```typescript
// For non-interactive mode (-p flag)
function formatMessage(message: Message): string {
  switch (message.type) {
    case 'assistant':
      // Format assistant message
      return message.message.content
        .map(block => {
          if (block.type === 'text') {
            return block.text
          }
          return ''
        })
        .join('')
      
    case 'user':
      // Format user message (tool results)
      return message.message.content
        .map(block => {
          if (block.type === 'tool_result') {
            return formatToolResult(block)
          }
          return ''
        })
        .join('')
      
    default:
      return ''
  }
}
```

## Integration Points Map

### 1. CLI ↔ Main
```
bin/openclaude
    ↓ (dynamic import)
dist/cli.mjs
    ↓ (exports)
main()
```

### 2. Main ↔ Commands
```
main.tsx
    ↓ (creates)
Commander program
    ↓ (registers)
commands.ts
    ↓ (routes to)
individual commands
```

### 3. Commands ↔ Tools
```
Command handler
    ↓ (invokes)
Tool.call()
    ↓ (executes)
Operation
    ↓ (returns)
ToolResult
```

### 4. QueryEngine ↔ Providers
```
QueryEngine.submitMessage()
    ↓ (calls)
query()
    ↓ (calls)
callModel()
    ↓ (calls)
Provider SDK
    ↓ (returns)
Stream of messages
```

### 5. Providers ↔ External APIs
```
Provider SDK
    ↓ (HTTP request)
External API
    ↓ (HTTP response)
Provider SDK
    ↓ (parsed)
Message objects
```

### 6. State ↔ Components
```
AppState
    ↓ (read by)
React components
    ↓ (updated by)
setAppState()
    ↓ (triggers)
Re-render
```

### 7. Everything ↔ Logging
```
Any component
    ↓ (calls)
logEvent()
    ↓ (sends to)
Analytics service
    ↓ (stores)
Analytics database
```

## Data Flow Examples

### Example 1: Simple Text Response

```
User: "What is 2+2?"
    ↓
main.tsx: Parse args
    ↓
Commander: Route to default action
    ↓
handlePrompt: Create QueryEngine
    ↓
QueryEngine.submitMessage("What is 2+2?")
    ↓
query(): Build system prompt
    ↓
callModel(): Call Anthropic API
    ↓
Anthropic: Process and respond
    ↓
Stream: "2 + 2 = 4"
    ↓
QueryEngine: Yield message
    ↓
formatMessage(): Format output
    ↓
stdout: "2 + 2 = 4"
```

### Example 2: Tool Use

```
User: "Read the file README.md"
    ↓
main.tsx: Parse args
    ↓
Commander: Route to default action
    ↓
handlePrompt: Create QueryEngine
    ↓
QueryEngine.submitMessage("Read the file README.md")
    ↓
query(): Call AI with FileReadTool available
    ↓
AI: Decides to use FileReadTool
    ↓
AI response: { tool_use: { name: "FileReadTool", input: { file_path: "README.md" } } }
    ↓
executeTool(): Find FileReadTool
    ↓
FileReadTool.call(): Read file
    ↓
File content: "# OpenClaude..."
    ↓
Tool result: { content: "# OpenClaude..." }
    ↓
query(): Send tool result back to AI
    ↓
AI: Processes file content
    ↓
AI response: "Here's the content of README.md: ..."
    ↓
QueryEngine: Yield messages
    ↓
formatMessage(): Format output
    ↓
stdout: AI's response with file content
```

### Example 3: Multiple Tool Calls

```
User: "List files and read package.json"
    ↓
QueryEngine.submitMessage()
    ↓
query(): Call AI
    ↓
AI: Decides to use GlobTool
    ↓
executeTool(GlobTool): List files
    ↓
Tool result: ["src/", "bin/", "package.json", ...]
    ↓
query(): Send result to AI
    ↓
AI: Decides to use FileReadTool
    ↓
executeTool(FileReadTool): Read package.json
    ↓
Tool result: { name: "openclaude", version: "0.1.7", ... }
    ↓
query(): Send result to AI
    ↓
AI: Combines information
    ↓
AI response: "Here are the files: ... and package.json contains ..."
    ↓
QueryEngine: Yield all messages
    ↓
stdout: Complete response
```

## Error Flow

```
User input
    ↓
Command handler
    ↓
try {
    operation()
} catch (error) {
    logError(error)
    ↓
    Create error message
    ↓
    Yield error message
    ↓
    Format error for output
    ↓
    Display to user
}
```

## State Persistence Flow

```
Session start
    ↓
Load state from disk (~/.claude/)
    ↓
Initialize in-memory state
    ↓
User interaction
    ↓
State changes
    ↓
Persist to disk (async)
    ↓
Session end
    ↓
Final state save
```

## Summary

The complete code flow shows:

1. **Clear separation of concerns** - Each layer has a specific responsibility
2. **Well-defined interfaces** - Layers communicate through typed interfaces
3. **Streaming architecture** - Responses stream from AI to user
4. **Tool orchestration** - AI can use tools, tools can use AI
5. **State management** - Centralized state with persistence
6. **Error handling** - Errors caught and displayed gracefully

The integration points are clean and modular, making it easy to:
- Add new commands
- Add new tools
- Add new providers
- Modify UI components
- Extend functionality
</task_progress>
</write_to_file>