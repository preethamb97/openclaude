# 004 - Commands System (`src/commands.ts`)

## Overview

The commands system is the backbone of OpenClaude's interactive features. It manages **all slash commands** (like `/help`, `/model`, `/clear`) and routes them to the appropriate handlers.

## Command Architecture

### Command Types

There are three types of commands:

1. **`'local'`** - Simple commands that execute locally
   ```typescript
   { type: 'local', name: 'help', description: 'Show help' }
   ```

2. **`'prompt'`** - Commands that send prompts to the AI
   ```typescript
   { type: 'prompt', name: 'explain', content: 'Explain this code' }
   ```

3. **`'local-jsx'`** - Commands that render React components
   ```typescript
   { type: 'local-jsx', name: 'model', render: () => <ModelPicker /> }
   ```

### Command Structure

```typescript
interface Command {
  name: string              // Command name (e.g., 'help')
  aliases?: string[]        // Alternative names (e.g., ['h', '?'])
  description: string       // What the command does
  type: 'local' | 'prompt' | 'local-jsx'
  source: 'builtin' | 'plugin' | 'skills' | 'bundled' | 'mcp'
  
  // For 'local' type
  action?: (args: string) => Promise<void>
  
  // For 'prompt' type
  content?: string | ((args: string) => string)
  
  // For 'local-jsx' type
  render?: (args: string) => React.ReactNode
  
  // Metadata
  aliases?: string[]
  loadedFrom?: string
  pluginInfo?: { pluginManifest: { name: string } }
  disableModelInvocation?: boolean
  hasUserSpecifiedDescription?: boolean
  whenToUse?: string
  kind?: 'workflow'
}
```

## Command Registration

### Built-in Commands

```typescript
// All built-in commands are defined in this file
const COMMANDS = memoize((): Command[] => [
  addDir,
  advisor,
  agents,
  branch,
  btw,
  chrome,
  clear,
  color,
  compact,
  config,
  copy,
  desktop,
  context,
  contextNonInteractive,
  cost,
  diff,
  doctor,
  effort,
  exit,
  fast,
  files,
  heapDump,
  help,
  ide,
  init,
  keybindings,
  installGitHubApp,
  installSlackApp,
  mcp,
  memory,
  mobile,
  model,
  onboardGithub,
  outputStyle,
  remoteEnv,
  plugin,
  provider,
  pr_comments,
  releaseNotes,
  reloadPlugins,
  rename,
  resume,
  session,
  skills,
  stats,
  status,
  statusline,
  stickers,
  tag,
  theme,
  feedback,
  review,
  ultrareview,
  rewind,
  securityReview,
  terminalSetup,
  upgrade,
  extraUsage,
  extraUsageNonInteractive,
  rateLimitOptions,
  usage,
  usageReport,
  vim,
  // ... and many more
])
```

### Dynamic Commands

Commands can also be loaded from:

1. **Plugins** - Installed plugins can register commands
2. **Skills** - User-created skill files
3. **MCP Servers** - Model Context Protocol servers
4. **Workflows** - Automated workflow scripts

```typescript
async function getSkills(cwd: string): Promise<{
  skillDirCommands: Command[]
  pluginSkills: Command[]
  bundledSkills: Command[]
  builtinPluginSkills: Command[]
}> {
  // Load commands from various sources
}
```

## Command Routing

### How Commands Are Found

```typescript
export function findCommand(
  commandName: string,
  commands: Command[]
): Command | undefined {
  return commands.find(
    _ =>
      _.name === commandName ||
      getCommandName(_) === commandName ||
      _.aliases?.includes(commandName)
  )
}
```

### Command Execution Flow

1. **User types**: `/help`
2. **Parser extracts**: Command name = 'help'
3. **Finder searches**: `findCommand('help', allCommands)`
4. **Handler executes**: `command.action()` or `command.render()`
5. **Result displayed**: In the TUI

## Example Commands

### 1. Simple Local Command (`/clear`)
```typescript
{
  name: 'clear',
  description: 'Clear the conversation',
  type: 'local',
  action: async () => {
    // Clear messages
    messages.length = 0
  }
}
```

### 2. Prompt Command (`/explain`)
```typescript
{
  name: 'explain',
  description: 'Explain the selected code',
  type: 'prompt',
  content: 'Explain this code in detail',
  disableModelInvocation: false
}
```

### 3. JSX Command (`/model`)
```typescript
{
  name: 'model',
  description: 'Change the AI model',
  type: 'local-jsx',
  render: (args) => <ModelPicker onSelect={handleModelSelect} />
}
```

## Command Availability

Commands can be filtered based on:

```typescript
export function meetsAvailabilityRequirement(cmd: Command): boolean {
  if (!cmd.availability) return true
  
  for (const a of cmd.availability) {
    switch (a) {
      case 'claude-ai':
        if (isClaudeAISubscriber()) return true
        break
      case 'console':
        if (!isClaudeAISubscriber() && !isUsing3PServices()) return true
        break
    }
  }
  return false
}
```

## Command Loading

### From Skills Directory

```typescript
// Commands can be loaded from .claude/skills/
// Each .md file becomes a command
const skillCommands = await getSkillDirCommands(cwd)
```

### From Plugins

```typescript
// Plugins can register commands
const pluginCommands = await getPluginCommands()
```

### From MCP Servers

```typescript
// MCP servers expose tools as commands
const mcpTools = await getMcpToolsCommandsAndResources()
```

## Command Caching

Commands are memoized for performance:

```typescript
const COMMANDS = memoize((): Command[] => [
  // ... all commands
])

export const getSkillToolCommands = memoize(
  async (cwd: string): Promise<Command[]> => {
    // ... load skill commands
  }
)
```

## Remote Mode

Some commands are safe for remote mode (mobile/web clients):

```typescript
export const REMOTE_SAFE_COMMANDS: Set<Command> = new Set([
  session, exit, clear, help, theme, color, vim, cost, usage,
  copy, btw, feedback, plan, keybindings, statusline, stickers, mobile
])
```

## Command Filtering

Commands can be filtered by various criteria:

```typescript
// Filter by availability
const available = allCommands.filter(
  _ => meetsAvailabilityRequirement(_) && isCommandEnabled(_)
)

// Filter for tool invocations
const toolCommands = allCommands.filter(
  cmd => cmd.type === 'prompt' && !cmd.disableModelInvocation
)
```

## Integration with TUI

The TUI (Terminal User Interface) uses commands for:

1. **Autocomplete**: When user types `/`, shows available commands
2. **Execution**: When user presses Enter, executes the command
3. **Help**: `/help` shows all available commands
4. **History**: Commands are saved in history

## Adding New Commands

To add a new command:

1. **Create command definition**
   ```typescript
   const myCommand: Command = {
     name: 'mycommand',
     description: 'My custom command',
     type: 'local',
     action: async (args) => {
       console.log('Hello from my command!')
     }
   }
   ```

2. **Register in COMMANDS array**
   ```typescript
   const COMMANDS = memoize((): Command[] => [
     // ... existing commands
     myCommand
   ])
   ```

3. **Test in TUI**
   ```bash
   openclaude
   /mycommand
   ```

## Summary

The commands system is:
- **Extensible**: Easy to add new commands
- **Flexible**: Supports local, prompt, and JSX commands
- **Dynamic**: Can load from plugins, skills, MCP servers
- **Cached**: Performance optimized with memoization
- **Filtered**: Respects availability and permissions
- **Integrated**: Seamlessly works with the TUI
</task_progress>
</write_to_file>