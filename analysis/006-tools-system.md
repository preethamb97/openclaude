# 005 - Tools System (`src/tools.ts`)

## Overview

The tools system provides **AI capabilities** to OpenClaude. Tools are actions the AI can perform: reading files, writing code, executing bash commands, searching the web, and more.

## What Are Tools?

Tools are **functions the AI can call** to interact with the world. When you ask Claude to "read a file" or "run a command", it uses tools to accomplish these tasks.

## Built-in Tools

OpenClaude includes many built-in tools:

### File Operations
- **`FileReadTool`** - Read file contents
- **`FileWriteTool`** - Write/create files
- **`FileEditTool`** - Edit existing files
- **`GlobTool`** - Find files by pattern
- **`GrepTool`** - Search file contents

### Code Operations
- **`BashTool`** - Execute shell commands
- **`PowerShellTool`** - Execute PowerShell commands (Windows)
- **`NotebookEditTool`** - Edit Jupyter notebooks
- **`LSPTool`** - Language Server Protocol integration

### Web Operations
- **`WebFetchTool`** - Fetch web pages
- **`WebSearchTool`** - Search the web
- **`WebBrowserTool`** - Control a web browser (WEB_BROWSER_TOOL feature)

### Agent Operations
- **`AgentTool`** - Spawn sub-agents
- **`SkillTool`** - Execute skills/commands
- **`TaskStopTool`** - Stop running tasks
- **`TeamCreateTool`** - Create agent teams (agent swarms)
- **`TeamDeleteTool`** - Delete agent teams
- **`SendMessageTool`** - Send messages between agents

### Utility Operations
- **`TodoWriteTool`** - Manage todo lists
- **`AskUserQuestionTool`** - Ask user for input
- **`ConfigTool`** - Manage configuration (internal)
- **`TaskOutputTool`** - Task output management

### Plan Mode Operations
- **`EnterPlanModeTool`** - Enter plan mode
- **`ExitPlanModeV2Tool`** - Exit plan mode

### Worktree Operations
- **`EnterWorktreeTool`** - Enter worktree mode
- **`ExitWorktreeTool`** - Exit worktree mode

### Task Operations
- **`TaskCreateTool`** - Create tasks
- **`TaskGetTool`** - Get task details
- **`TaskUpdateTool`** - Update tasks
- **`TaskListTool`** - List tasks

### MCP Operations
- **`ListMcpResourcesTool`** - List MCP resources
- **`ReadMcpResourceTool`** - Read MCP resources

### Search Operations
- **`ToolSearchTool`** - Search for tools

### Feature-Gated Tools
- **`BriefTool`** - Brief mode (KAIROS feature)
- **`SleepTool`** - Sleep tool (PROACTIVE feature)
- **`SnipTool`** - Snip history (HISTORY_SNIP feature)
- **`PushNotificationTool`** - Push notifications (KAIROS feature)
- **`SendUserFileTool`** - Send user files (KAIROS feature)
- **`SubscribePRTool`** - Subscribe to PR webhooks (KAIROS_GITHUB_WEBHOOKS feature)
- **`CronCreateTool`** - Create cron triggers (AGENT_TRIGGERS feature)
- **`CronDeleteTool`** - Delete cron triggers (AGENT_TRIGGERS feature)
- **`CronListTool`** - List cron triggers (AGENT_TRIGGERS feature)
- **`RemoteTriggerTool`** - Remote triggers (AGENT_TRIGGERS_REMOTE feature)
- **`MonitorTool`** - Monitor tool (MONITOR_TOOL feature)
- **`TerminalCaptureTool`** - Terminal capture (TERMINAL_PANEL feature)
- **`CtxInspectTool`** - Context inspection (CONTEXT_COLLAPSE feature)
- **`OverflowTestTool`** - Overflow test (OVERFLOW_TEST_TOOL feature)
- **`ListPeersTool`** - List peers (UDS_INBOX feature)
- **`WorkflowTool`** - Workflow execution (WORKFLOW_SCRIPTS feature)

### Internal Tools
- **`REPLTool`** - REPL tool (internal only)
- **`ConfigTool`** - Configuration (internal only)
- **`TungstenTool`** - Tungsten tool (internal only)
- **`SuggestBackgroundPRTool`** - Suggest background PR (internal only)
- **`VerifyPlanExecutionTool`** - Verify plan execution (internal)
- **`TestingPermissionTool`** - Testing permissions (test mode only)

### Synthetic Tools
- **`SyntheticOutputTool`** - Synthetic output for structured responses

## Tool Structure

```typescript
interface Tool<Input, Output> {
  name: string
  description: string
  inputSchema: Input  // Zod schema for validation
  outputSchema?: Output
  
  // Core methods
  call(args: Input, context: ToolUseContext): Promise<ToolResult<Output>>
  checkPermissions(input: Input, context: ToolUseContext): Promise<PermissionResult>
  validateInput(input: Input, context: ToolUseContext): Promise<ValidationResult>
  
  // UI methods
  renderToolUseMessage(input: Partial<Input>): React.ReactNode
  renderToolResultMessage(output: Output): React.ReactNode
  renderToolUseProgressMessage(progress: ProgressMessage[]): React.ReactNode
  
  // Metadata
  isEnabled(): boolean
  isReadOnly(input: Input): boolean
  isConcurrencySafe(input: Input): boolean
  isDestructive(input: Input): boolean
}
```

## Tool Registration

### All Base Tools

```typescript
export function getAllBaseTools(): Tools {
  return [
    AgentTool,
    TaskOutputTool,
    BashTool,
    ...(hasEmbeddedSearchTools() ? [] : [GlobTool, GrepTool]),
    ExitPlanModeV2Tool,
    FileReadTool,
    FileEditTool,
    FileWriteTool,
    NotebookEditTool,
    WebFetchTool,
    TodoWriteTool,
    WebSearchTool,
    TaskStopTool,
    AskUserQuestionTool,
    SkillTool,
    EnterPlanModeTool,
    ...(process.env.USER_TYPE === 'ant' ? [ConfigTool] : []),
    ...(process.env.USER_TYPE === 'ant' ? [TungstenTool] : []),
    ...(SuggestBackgroundPRTool ? [SuggestBackgroundPRTool] : []),
    ...(WebBrowserTool ? [WebBrowserTool] : []),
    ...(isTodoV2Enabled()
      ? [TaskCreateTool, TaskGetTool, TaskUpdateTool, TaskListTool]
      : []),
    ...(OverflowTestTool ? [OverflowTestTool] : []),
    ...(CtxInspectTool ? [CtxInspectTool] : []),
    ...(TerminalCaptureTool ? [TerminalCaptureTool] : []),
    ...(isEnvTruthy(process.env.ENABLE_LSP_TOOL) ? [LSPTool] : []),
    ...(isWorktreeModeEnabled() ? [EnterWorktreeTool, ExitWorktreeTool] : []),
    getSendMessageTool(),
    ...(ListPeersTool ? [ListPeersTool] : []),
    ...(isAgentSwarmsEnabled()
      ? [getTeamCreateTool(), getTeamDeleteTool()]
      : []),
    ...(VerifyPlanExecutionTool ? [VerifyPlanExecutionTool] : []),
    ...(process.env.USER_TYPE === 'ant' && REPLTool ? [REPLTool] : []),
    ...(WorkflowTool ? [WorkflowTool] : []),
    ...(SleepTool ? [SleepTool] : []),
    ...cronTools,
    ...(RemoteTriggerTool ? [RemoteTriggerTool] : []),
    ...(MonitorTool ? [MonitorTool] : []),
    BriefTool,
    ...(SendUserFileTool ? [SendUserFileTool] : []),
    ...(PushNotificationTool ? [PushNotificationTool] : []),
    ...(SubscribePRTool ? [SubscribePRTool] : []),
    ...(getPowerShellTool() ? [getPowerShellTool()] : []),
    ...(SnipTool ? [SnipTool] : []),
    ...(process.env.NODE_ENV === 'test' ? [TestingPermissionTool] : []),
    ListMcpResourcesTool,
    ReadMcpResourceTool,
    ...(isToolSearchEnabledOptimistic() ? [ToolSearchTool] : []),
  ]
}
```

### Conditional Tools

Some tools are only available in certain modes:

```typescript
const tools = [
  // Always available
  BashTool,
  FileReadTool,
  
  // Only if feature enabled
  ...(isToolSearchEnabledOptimistic() ? [ToolSearchTool] : []),
  
  // Only if MCP enabled
  ListMcpResourcesTool,
  ReadMcpResourceTool,
  
  // Only if worktree mode enabled
  ...(isWorktreeModeEnabled() ? [EnterWorktreeTool, ExitWorktreeTool] : []),
  
  // Only if agent swarms enabled
  ...(isAgentSwarmsEnabled() ? [getTeamCreateTool(), getTeamDeleteTool()] : []),
]
```

## Tool Presets

Tool presets provide predefined tool configurations:

```typescript
export const TOOL_PRESETS = ['default'] as const

export function getToolsForDefaultPreset(): string[] {
  const tools = getAllBaseTools()
  const isEnabled = tools.map(tool => tool.isEnabled())
  return tools.filter((_, i) => isEnabled[i]).map(tool => tool.name)
}
```

## Tool Permissions

Every tool call goes through permission checks:

```typescript
export function filterToolsByDenyRules<
  T extends {
    name: string
    mcpInfo?: { serverName: string; toolName: string }
  },
>(tools: readonly T[], permissionContext: ToolPermissionContext): T[] {
  return tools.filter(tool => !getDenyRuleForTool(permissionContext, tool))
}
```

### Permission Modes

```typescript
type PermissionMode = 
  | 'default'           // Normal permissions
  | 'bypassPermissions' // Skip all checks
  | 'plan'              // Plan mode (read-only)
  | 'auto'              // Auto-approve safe operations
```

## Tool Execution Flow

1. **AI requests tool use**
   ```json
   {
     "tool": "BashTool",
     "input": { "command": "ls -la" }
   }
   ```

2. **Permission check**
   ```typescript
   const result = await tool.checkPermissions(input, context)
   if (result.behavior !== 'allow') {
     // Show permission prompt
   }
   ```

3. **Input validation**
   ```typescript
   const validation = await tool.validateInput(input, context)
   if (!validation.result) {
     // Return error to AI
   }
   ```

4. **Tool execution**
   ```typescript
   const output = await tool.call(input, context)
   ```

5. **Result returned to AI**
   ```json
   {
     "tool_use_id": "tool_123",
     "content": "Output of the command..."
   }
   ```

## Simple Mode

In simple mode (`--bare` or `CLAUDE_CODE_SIMPLE=1`), only essential tools are available:

```typescript
export function getTools(permissionContext: ToolPermissionContext): Tools {
  if (isEnvTruthy(process.env.CLAUDE_CODE_SIMPLE)) {
    // Simple mode: only Bash, Read, and Edit
    const simpleTools: Tool[] = [BashTool, FileReadTool, FileEditTool]
    
    // When coordinator mode is also active, include AgentTool and TaskStopTool
    if (feature('COORDINATOR_MODE') && coordinatorModeModule?.isCoordinatorMode()) {
      simpleTools.push(AgentTool, TaskStopTool, getSendMessageTool())
    }
    
    return filterToolsByDenyRules(simpleTools, permissionContext)
  }
  
  // Full tool set
  // ...
}
```

## REPL Mode

When REPL mode is enabled, primitive tools are hidden:

```typescript
if (isReplModeEnabled()) {
  const replEnabled = allowedTools.some(tool =>
    toolMatchesName(tool, REPL_TOOL_NAME)
  )
  if (replEnabled) {
    allowedTools = allowedTools.filter(
      tool => !REPL_ONLY_TOOLS.has(tool.name)
    )
  }
}
```

## Tool Pool Assembly

Combine built-in tools with MCP tools:

```typescript
export function assembleToolPool(
  permissionContext: ToolPermissionContext,
  mcpTools: Tools,
): Tools {
  const builtInTools = getTools(permissionContext)
  const allowedMcpTools = filterToolsByDenyRules(mcpTools, permissionContext)
  
  // Sort for prompt-cache stability
  const byName = (a: Tool, b: Tool) => a.name.localeCompare(b.name)
  return uniqBy(
    [...builtInTools].sort(byName).concat(allowedMcpTools.sort(byName)),
    'name'
  )
}
```

## Merged Tools

Get all tools including MCP tools:

```typescript
export function getMergedTools(
  permissionContext: ToolPermissionContext,
  mcpTools: Tools,
): Tools {
  const builtInTools = getTools(permissionContext)
  return [...builtInTools, ...mcpTools]
}
```

## Tool Examples

### BashTool Example

```typescript
const BashTool = {
  name: 'Bash',
  description: 'Execute shell commands',
  inputSchema: z.object({
    command: z.string(),
    timeout: z.number().optional()
  }),
  
  async call(args, context) {
    const { command, timeout = 30000 } = args
    
    return new Promise((resolve, reject) => {
      exec(command, { timeout }, (error, stdout, stderr) => {
        if (error) {
          resolve({ error: stderr })
        } else {
          resolve({ output: stdout })
        }
      })
    })
  }
}
```

### FileReadTool Example

```typescript
const FileReadTool = {
  name: 'Read',
  description: 'Read file contents',
  inputSchema: z.object({
    file_path: z.string(),
    offset: z.number().optional(),
    limit: z.number().optional()
  }),
  
  async call(args, context) {
    const { file_path, offset = 0, limit } = args
    const content = await readFile(file_path, 'utf8')
    const lines = content.split('\n')
    
    return {
      content: lines.slice(offset, limit ? offset + limit : undefined).join('\n')
    }
  }
}
```

## Tool Progress

Tools can report progress during execution:

```typescript
type ToolCallProgress<P extends ToolProgressData> = (
  progress: ToolProgress<P>
) => void

// Example usage
const onProgress: ToolCallProgress<ToolProgressData> = (progress) => {
  console.log(`Progress: ${progress.data.percent}%`)
}
```

## Tool Result Storage

Large tool results are stored to disk to save memory:

```typescript
readonly maxResultSizeChars: number  // Max chars before persisting

// When result exceeds max, it's saved to file
// AI receives a preview + file path instead of full content
```

## Tool Search

The ToolSearchTool helps when there are many tools:

```typescript
// AI can search for tools
const results = await ToolSearchTool.call({
  query: 'read file'
}, context)

// Returns matching tools
// [
//   { name: 'Read', description: 'Read file contents' },
//   { name: 'Glob', description: 'Find files by pattern' }
// ]
```

## Adding Custom Tools

To add a custom tool:

1. **Define the tool**
   ```typescript
   const MyTool = {
     name: 'MyTool',
     description: 'Does something cool',
     inputSchema: z.object({
       param1: z.string(),
       param2: z.number()
     }),
     
     async call(args, context) {
       // Tool implementation
       return { result: 'success' }
     }
   }
   ```

2. **Register in tools array**
   ```typescript
   export function getAllBaseTools(): Tools {
     return [
       // ... existing tools
       MyTool
     ]
   }
   ```

3. **Test in TUI**
   ```bash
   openclaude
   # AI will see MyTool in its tool list
   ```

## Tool Security

Tools implement several security features:

### Permission Checks
```typescript
async checkPermissions(input, context) {
  // Check if user allowed this operation
  if (input.command.includes('rm -rf')) {
    return { behavior: 'deny', message: 'Dangerous command' }
  }
  return { behavior: 'allow', updatedInput: input }
}
```

### Read-Only Detection
```typescript
isReadOnly(input) {
  // FileReadTool is read-only
  // BashTool with 'cat' is read-only
  // BashTool with 'rm' is not
  return input.command.startsWith('cat')
}
```

### Destructive Detection
```typescript
isDestructive(input) {
  // Destructive operations can't be undone
  return input.command.includes('rm') || 
         input.command.includes('delete')
}
```

## Tool Orchestration

Tools can be orchestrated in parallel:

```typescript
// Multiple tool calls can run concurrently
const results = await Promise.all([
  BashTool.call({ command: 'ls' }),
  FileReadTool.call({ file_path: 'README.md' })
])
```

## Complete Tool List

### Always Available
| Tool | Description |
|------|-------------|
| `Agent` | Spawn and manage sub-agents |
| `TaskOutput` | Manage task output |
| `Bash` | Execute shell commands |
| `Read` | Read file contents |
| `Edit` | Edit file contents |
| `Write` | Write/create files |
| `Glob` | Find files by pattern |
| `Grep` | Search file contents |
| `NotebookEdit` | Edit Jupyter notebooks |
| `WebFetch` | Fetch web pages |
| `TodoWrite` | Manage todo lists |
| `WebSearch` | Search the web |
| `TaskStop` | Stop running tasks |
| `AskUserQuestion` | Ask user for input |
| `Skill` | Execute skills/commands |
| `EnterPlanMode` | Enter plan mode |
| `ExitPlanModeV2` | Exit plan mode |

### Conditional Tools
| Tool | Condition |
|------|-----------|
| `Config` | Internal only (USER_TYPE=ant) |
| `Tungsten` | Internal only (USER_TYPE=ant) |
| `SuggestBackgroundPR` | Internal only (USER_TYPE=ant) |
| `WebBrowser` | WEB_BROWSER_TOOL feature |
| `TaskCreate`, `TaskGet`, `TaskUpdate`, `TaskList` | TodoV2 enabled |
| `OverflowTest` | OVERFLOW_TEST_TOOL feature |
| `CtxInspect` | CONTEXT_COLLAPSE feature |
| `TerminalCapture` | TERMINAL_PANEL feature |
| `LSP` | ENABLE_LSP_TOOL env var |
| `EnterWorktree`, `ExitWorktree` | Worktree mode enabled |
| `SendMessage` | Always (lazy loaded) |
| `ListPeers` | UDS_INBOX feature |
| `TeamCreate`, `TeamDelete` | Agent swarms enabled |
| `VerifyPlanExecution` | CLAUDE_CODE_VERIFY_PLAN env var |
| `REPL` | Internal only (USER_TYPE=ant) |
| `Workflow` | WORKFLOW_SCRIPTS feature |
| `Sleep` | PROACTIVE or KAIROS feature |
| `CronCreate`, `CronDelete`, `CronList` | AGENT_TRIGGERS feature |
| `RemoteTrigger` | AGENT_TRIGGERS_REMOTE feature |
| `Monitor` | MONITOR_TOOL feature |
| `Brief` | KAIROS or KAIROS_BRIEF feature |
| `SendUserFile` | KAIROS feature |
| `PushNotification` | KAIROS or KAIROS_PUSH_NOTIFICATION feature |
| `SubscribePR` | KAIROS_GITHUB_WEBHOOKS feature |
| `PowerShell` | PowerShell enabled |
| `Snip` | HISTORY_SNIP feature |
| `TestingPermission` | Test mode only |
| `ListMcpResources` | Always (MCP) |
| `ReadMcpResource` | Always (MCP) |
| `ToolSearch` | Tool search enabled |
| `SyntheticOutput` | Structured output enabled |

## Summary

The tools system:
- **Provides capabilities**: File ops, code execution, web search, agents, etc.
- **Is extensible**: Easy to add new tools
- **Is secure**: Permission checks, validation, sandboxing
- **Is efficient**: Deferred loading, progress reporting, result storage
- **Is integrated**: Works seamlessly with the AI and TUI
- **Is feature-gated**: Many tools are behind feature flags
- **Supports MCP**: Can integrate with MCP servers
- **Supports agents**: Can spawn and manage sub-agents
- **Supports workflows**: Can execute workflow scripts

Tools are the bridge between the AI and the real world, allowing Claude to take actions on your behalf.