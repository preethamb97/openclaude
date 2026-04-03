# 011 - Entry Points Deep Dive

## Overview

This document provides a deep dive into how OpenClaude boots up and connects all its subsystems together.

## Boot Sequence

### 1. CLI Entry (`bin/openclaude`)

```javascript
#!/usr/bin/env node

import { existsSync } from 'fs'
import { join, dirname } from 'path'
import { fileURLToPath, pathToFileURL } from 'url'

const __dirname = dirname(fileURLToPath(import.meta.url))
const distPath = join(__dirname, '..', 'dist', 'cli.mjs')

if (existsSync(distPath)) {
  await import(pathToFileURL(distPath).href)
} else {
  console.error('dist/cli.mjs not found. Build first: bun run build')
  process.exit(1)
}
```

**What happens:**
1. Script runs with Node.js
2. Checks if `dist/cli.mjs` exists (built version)
3. If yes, dynamically imports it
4. If no, shows error message

### 2. Main Application (`src/main.tsx`)

The `main()` function is the heart of the application:

```typescript
export async function main() {
  // Step 1: Security setup
  process.env.NoDefaultCurrentDirectoryInExePath = '1'
  
  // Step 2: Initialize warning handler
  initializeWarningHandler()
  
  // Step 3: Setup signal handlers
  process.on('exit', () => resetCursor())
  process.on('SIGINT', () => process.exit(0))
  
  // Step 4: Parse early arguments (before full init)
  // - Handle cc:// URLs
  // - Handle deep links
  // - Handle ssh commands
  // - Handle assistant commands
  
  // Step 5: Determine if interactive or non-interactive
  const hasPrintFlag = cliArgs.includes('-p') || cliArgs.includes('--print')
  const isNonInteractive = hasPrintFlag || !process.stdout.isTTY
  
  // Step 6: Set client type
  const clientType = determineClientType()
  setClientType(clientType)
  
  // Step 7: Load settings from flags
  eagerLoadSettings()
  
  // Step 8: Run the main logic
  await run()
}
```

### 3. The `run()` Function

```typescript
async function run(): Promise<CommanderCommand> {
  // Step 1: Create Commander program
  const program = new CommanderCommand()
    .name('claude')
    .description('Claude Code - AI-powered code assistant')
    .argument('[prompt]', 'Your prompt')
    // ... add all options
  
  // Step 2: Setup preAction hook
  program.hook('preAction', async (thisCommand) => {
    // Initialize MDM settings
    await ensureMdmSettingsLoaded()
    
    // Initialize keychain prefetch
    await ensureKeychainPrefetchCompleted()
    
    // Initialize the application
    await init()
    
    // Set process title
    process.title = 'claude'
    
    // Initialize sinks for logging
    initSinks()
    
    // Load inline plugins
    const pluginDir = thisCommand.getOptionValue('pluginDir')
    if (pluginDir) {
      setInlinePlugins(pluginDir)
    }
    
    // Run migrations
    runMigrations()
    
    // Load remote managed settings (async)
    void loadRemoteManagedSettings()
    void loadPolicyLimits()
  })
  
  // Step 3: Add all commands
  // Each command is imported and registered
  
  // Step 4: Parse arguments
  return program.parseAsync(process.argv)
}
```

## Command Execution Flow

### Example: User runs `openclaude -p "explain this code"`

```
1. bin/openclaude
   └─> import dist/cli.mjs

2. dist/cli.mjs (compiled main.tsx)
   └─> main()

3. main()
   ├─> Parse args: ["-p", "explain this code"]
   ├─> Set isNonInteractive = true
   ├─> Set clientType = "sdk-cli"
   └─> run()

4. run()
   ├─> Create Commander program
   ├─> preAction hook:
   │   ├─> init()
   │   ├─> Load settings
   │   └─> Run migrations
   ├─> Parse argv
   └─> Match to default action (prompt)

5. Default action handler
   ├─> Create QueryEngine
   ├─> Submit message: "explain this code"
   ├─> QueryEngine calls AI provider
   ├─> AI processes and responds
   └─> Output formatted response

6. process.exit(0)
```

## Initialization Sequence (`init()`)

```typescript
export async function init(): Promise<void> {
  // 1. Load global config
  const config = getGlobalConfig()
  
  // 2. Initialize model strings
  await ensureModelStringsInitialized()
  
  // 3. Initialize permissions
  initializeToolPermissionContext()
  
  // 4. Initialize plugins
  await initializeVersionedPlugins()
  
  // 5. Initialize skills
  await initBundledSkills()
  
  // 6. Initialize MCP servers
  await initializeMcpServers()
  
  // 7. Initialize LSP
  await initializeLspServerManager()
}
```

## Provider Initialization

```typescript
// In src/services/api/claude.ts
export async function initializeProvider(): Promise<void> {
  const provider = getProvider()
  
  switch (provider) {
    case 'anthropic':
      // Use Anthropic SDK
      break
    case 'openai':
      // Use OpenAI SDK
      break
    case 'ollama':
      // Use Ollama Python provider
      break
    case 'gemini':
      // Use Gemini SDK
      break
    // ... etc
  }
}
```

## State Management

### Global State (`src/bootstrap/state.ts`)

```typescript
// Session state
let sessionId: string
let originalCwd: string
let isInteractive: boolean
let clientType: string

// Model state
let initialMainLoopModel: string
let mainLoopModelOverride: string

// Permission state
let sessionBypassPermissionsMode: string

// MCP state
let inlinePlugins: string[]
let allowedSettingSources: string[]
```

### App State (`src/state/AppState.ts`)

```typescript
interface AppState {
  // Messages
  messages: Message[]
  
  // Tools
  toolPermissionContext: ToolPermissionContext
  
  // MCP
  mcp: {
    clients: MCPServerConnection[]
    tools: Tool[]
    resources: ServerResource[]
  }
  
  // UI
  fastMode: boolean
  effortValue: string
  advisorModel: string
  
  // History
  fileHistory: FileHistoryState
  attribution: AttributionState
}
```

## Component Integration

### How TUI Connects to Backend

```typescript
// 1. TUI component renders
const REPL = () => {
  const [appState, setAppState] = useAppState()
  
  // 2. User types input
  const handleSubmit = async (input: string) => {
    // 3. Create QueryEngine
    const engine = new QueryEngine({
      cwd: getCwd(),
      tools: appState.mcp.tools,
      canUseTool,
      getAppState: () => appState,
      setAppState
    })
    
    // 4. Submit message
    for await (const message of engine.submitMessage(input)) {
      // 5. Update UI with each message
      setAppState(prev => ({
        ...prev,
        messages: [...prev.messages, message]
      }))
    }
  }
  
  return (
    <Box>
      <Messages messages={appState.messages} />
      <Input onSubmit={handleSubmit} />
    </Box>
  )
}
```

## Error Handling Chain

```
User Action
    ↓
Command Handler
    ↓
Tool Execution
    ↓
try {
    operation()
} catch (error) {
    logError(error)
    yield createErrorMessage(error)
}
    ↓
TUI displays error
```

## Key Integration Points

### 1. CLI → Main
- `bin/openclaude` imports `dist/cli.mjs`
- `dist/cli.mjs` exports `main()` function

### 2. Main → Commands
- `main.tsx` creates Commander program
- Each command registers itself
- Commander routes to correct handler

### 3. Commands → Tools
- Commands can invoke tools
- Tools execute operations
- Results returned to commands

### 4. Tools → Providers
- Tools can call AI providers
- Providers return responses
- Tools process responses

### 5. Providers → External APIs
- Anthropic SDK for Claude
- OpenAI SDK for GPT
- httpx for Ollama/Gemini

### 6. Everything → State
- All components read from state
- State changes trigger re-renders
- State persisted to disk

## Configuration Loading

```
1. Environment variables (.env)
   ↓
2. Global config (~/.claude/config.json)
   ↓
3. Project config (.claude/config.json)
   ↓
4. Command-line flags
   ↓
5. Final merged configuration
```

## Summary

The entry points form a clear hierarchy:

1. **`bin/openclaude`** - CLI bootstrap
2. **`src/main.tsx`** - Application initialization
3. **`src/commands.ts`** - Command routing
4. **`src/QueryEngine.ts`** - AI interaction
5. **`src/tools.ts`** - Tool execution
6. **`src/services/api/`** - Provider communication

Each layer is cleanly separated and communicates through well-defined interfaces.
</task_progress>
</write_to_file>