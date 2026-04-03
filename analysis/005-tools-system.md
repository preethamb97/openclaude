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
- **`NotebookEditTool`** - Edit Jupyter notebooks
- **`LSPTool`** - Language Server Protocol integration

### Web Operations
- **`WebFetchTool`** - Fetch web pages
- **`WebSearchTool`** - Search the web
- **`WebBrowserTool`** - Control a web browser

### Agent Operations
- **`AgentTool`** - Spawn sub-agents
- **`SkillTool`** - Execute skills/commands
- **`TaskStopTool`** - Stop running tasks

### Utility Operations
- **`TodoWriteTool`** - Manage todo lists
- **`AskUserQuestionTool`** - Ask user for input
- **`ConfigTool`** - Manage configuration

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
    // ... and many more
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
]
```

## Tool Permissions

Every tool call goes through permission checks:

```typescript
export function filterToolsByDenyRules(
  tools: readonly Tool[],
  permissionContext: ToolPermissionContext
): Tool[] {
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

## Smart Tool Loading

When there are many tools, the **ToolSearchTool** helps the AI find the right one:

```typescript
export const ToolSearchTool = {
  name: 'ToolSearch',
  description: 'Search for tools by keyword',
  // ... implementation
}
```

### Deferred Tools

Some tools can be deferred (loaded on-demand):

```typescript
readonly shouldDefer?: boolean  // Requires ToolSearch first
readonly alwaysLoad?: boolean   // Never defer, always show
```

## MCP Tools

MCP (Model Context Protocol) servers can provide additional tools:

```typescript
export function assembleToolPool(
  permissionContext: ToolPermissionContext,
  mcpTools: Tools
): Tools {
  const builtInTools = getTools(permissionContext)
  const allowedMcpTools = filterToolsByDenyRules(mcpTools, permissionContext)
  
  // Combine and deduplicate
  return uniqBy(
    [...builtInTools, ...allowedMcpTools],
    'name'
  )
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

## Summary

The tools system:
- **Provides capabilities**: File ops, code execution, web search, etc.
- **Is extensible**: Easy to add new tools
- **Is secure**: Permission checks, validation, sandboxing
- **Is efficient**: Deferred loading, progress reporting, result storage
- **Is integrated**: Works seamlessly with the AI and TUI

Tools are the bridge between the AI and the real world, allowing Claude to take actions on your behalf.
</task_progress>
</write_to_file>