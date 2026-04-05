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
  getPromptForCommand?: (args: string, context: any) => Promise<string>
  contentLength?: number
  progressMessage?: string
  
  // For 'local-jsx' type
  render?: (args: string) => React.ReactNode
  
  // Metadata
  loadedFrom?: 'skills' | 'plugin' | 'bundled' | 'commands_DEPRECATED' | 'mcp'
  pluginInfo?: { pluginManifest: { name: string } }
  disableModelInvocation?: boolean
  hasUserSpecifiedDescription?: boolean
  whenToUse?: string
  kind?: 'workflow'
  availability?: ('claude-ai' | 'console')[]
}
```

## Command Registration

### Built-in Commands

All built-in commands are registered in `src/commands.ts`:

```typescript
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
  dream,
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
  thinkback,
  thinkbackPlay,
  permissions,
  plan,
  privacySettings,
  hooks,
  exportCommand,
  sandboxToggle,
  passes,
  tasks,
  // ... and more
])
```

### Internal-Only Commands

Commands that are only available for internal/ant use:

```typescript
export const INTERNAL_ONLY_COMMANDS = [
  backfillSessions,
  breakCache,
  bughunter,
  commit,
  commitPushPr,
  ctx_viz,
  goodClaude,
  issue,
  initVerifiers,
  forceSnip,
  mockLimits,
  bridgeKick,
  version,
  ultraplan,
  subscribePr,
  resetLimits,
  resetLimitsNonInteractive,
  onboarding,
  share,
  summary,
  teleport,
  antTrace,
  perfIssue,
  env,
  oauthRefresh,
  debugToolCall,
  agentsPlatform,
  autofixPr,
].filter(Boolean)
```

### Feature-Gated Commands

Commands that are only available when specific feature flags are enabled:

```typescript
// KAIROS feature (Assistant mode)
const assistantCommand = feature('KAIROS')
  ? require('./commands/assistant/index.js').default
  : null

// KAIROS_BRIEF feature (Brief mode)
const briefCommand = feature('KAIROS') || feature('KAIROS_BRIEF')
  ? require('./commands/brief.js').default
  : null

// PROACTIVE feature
const proactive = feature('PROACTIVE') || feature('KAIROS')
  ? require('./commands/proactive.js').default
  : null

// BRIDGE_MODE feature
const bridge = feature('BRIDGE_MODE')
  ? require('./commands/bridge/index.js').default
  : null

// VOICE_MODE feature
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null

// HISTORY_SNIP feature
const forceSnip = feature('HISTORY_SNIP')
  ? require('./commands/force-snip.js').default
  : null

// WORKFLOW_SCRIPTS feature
const workflowsCmd = feature('WORKFLOW_SCRIPTS')
  ? require('./commands/workflows/index.js').default
  : null

// UDS_INBOX feature
const peersCmd = feature('UDS_INBOX')
  ? require('./commands/peers/index.js').default
  : null

// FORK_SUBAGENT feature
const forkCmd = feature('FORK_SUBAGENT')
  ? require('./commands/fork/index.js').default
  : null
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

## Complete Command List

### Core Commands

| Command | Type | Description |
|---------|------|-------------|
| `/add-dir` | local | Add directory to context |
| `/advisor` | local | Advisor mode |
| `/agents` | local | Agent management |
| `/branch` | local | Git branch operations |
| `/btw` | local | Quick note |
| `/chrome` | local | Chrome integration |
| `/clear` | local | Clear context |
| `/color` | local | Color settings |
| `/compact` | local | Context compaction |
| `/config` | local | Configuration |
| `/context` | local | Context management |
| `/copy` | local | Copy to clipboard |
| `/cost` | local | Cost tracking |
| `/diff` | local | File differences |
| `/doctor` | local | Diagnostics |
| `/effort` | local | Effort tracking |
| `/exit` | local | Exit command |
| `/export` | local | Export data |
| `/fast` | local | Fast mode |
| `/feedback` | local | User feedback |
| `/files` | local | File operations |
| `/help` | local | Help system |
| `/hooks` | local | Hook management |
| `/ide` | local | IDE integration |
| `/init` | local | Initialization |
| `/keybindings` | local | Key binding configuration |
| `/login` | local | Authentication login |
| `/logout` | local | Authentication logout |
| `/mcp` | local | MCP server management |
| `/memory` | local | Memory management |
| `/mobile` | local | Mobile features |
| `/model` | local-jsx | Model selection |
| `/output-style` | local | Output style configuration |
| `/permissions` | local | Permission management |
| `/plan` | local | Planning features |
| `/plugin` | local | Plugin management |
| `/privacy-settings` | local | Privacy settings |
| `/provider` | local | Provider configuration |
| `/reload-plugins` | local | Reload plugins |
| `/rename` | local | Rename sessions |
| `/resume` | local | Session resumption |
| `/rewind` | local | Rewind to previous state |
| `/session` | local | Session management |
| `/skills` | local | Skills management |
| `/status` | local | Status display |
| `/stats` | local | Statistics |
| `/tag` | local | Tag management |
| `/tasks` | local | Task management |
| `/teleport` | local | Teleport sessions |
| `/theme` | local | Theme management |
| `/upgrade` | local | Upgrade handling |
| `/usage` | local | Usage statistics |
| `/vim` | local | Vim mode |

### Advanced Commands

| Command | Type | Description |
|---------|------|-------------|
| `/autofix-pr` | prompt | Auto-fix PR |
| `/brief` | local | Brief mode (KAIROS) |
| `/buddy` | local | Buddy companion |
| `/desktop` | local | Desktop integration |
| `/dream` | prompt | Dream mode |
| `/heapdump` | local | Heap dump |
| `/install-github-app` | local | Install GitHub app |
| `/install-slack-app` | local | Install Slack app |
| `/onboard-github` | local | GitHub onboarding |
| `/passes` | local | Passes management |
| `/pr_comments` | prompt | PR comments |
| `/release-notes` | prompt | Release notes |
| `/remote-env` | local | Remote environment |
| `/review` | prompt | Code review |
| `/sandbox-toggle` | local | Sandbox toggle |
| `/security-review` | prompt | Security review |
| `/share` | local | Share sessions |
| `/stickers` | local | Stickers |
| `/statusline` | local | Status line |
| `/summary` | prompt | Session summary |
| `/terminal-setup` | local | Terminal setup |
| `/thinkback` | local | Thinkback feature |
| `/thinkback-play` | local | Thinkback play |
| `/ultrareview` | prompt | Ultra review |

### Internal Commands (Ant Only)

| Command | Type | Description |
|---------|------|-------------|
| `/agents-platform` | local | Agent platform |
| `/ant-trace` | local | Ant tracing |
| `/assistant` | local | Assistant mode |
| `/backfill-sessions` | local | Backfill sessions |
| `/break-cache` | local | Break cache |
| `/bridge-kick` | local | Kick bridge session |
| `/bughunter` | local | Bug hunter mode |
| `/commit` | local | Commit changes |
| `/commit-push-pr` | local | Commit and push PR |
| `/ctx_viz` | local | Context visualization |
| `/debug-tool-call` | local | Debug tool calls |
| `/env` | local | Environment variables |
| `/good-claude` | local | Good Claude |
| `/init-verifiers` | local | Verifier initialization |
| `/issue` | local | Issue tracking |
| `/mock-limits` | local | Mock limits |
| `/oauth-refresh` | local | OAuth refresh |
| `/onboarding` | local | Onboarding flow |
| `/perf-issue` | local | Performance issue tracking |
| `/reset-limits` | local | Reset limits |
| `/subscribe-pr` | local | Subscribe to PR |
| `/ultraplan` | local | Ultra plan |
| `/version` | local | Version info |

### Feature-Gated Commands

| Command | Feature Flag | Description |
|---------|--------------|-------------|
| `/bridge` | BRIDGE_MODE | Bridge mode |
| `/brief` | KAIROS_BRIEF | Brief mode |
| `/fork` | FORK_SUBAGENT | Fork subagent |
| `/peers` | UDS_INBOX | Peers management |
| `/proactive` | PROACTIVE | Proactive mode |
| `/remote-control-server` | DAEMON + BRIDGE_MODE | Remote control server |
| `/remote-setup` | CCR_REMOTE_SETUP | Remote setup |
| `/torch` | TORCH | Torch feature |
| `/voice` | VOICE_MODE | Voice mode |
| `/workflows` | WORKFLOW_SCRIPTS | Workflow scripts |

### Special Commands

| Command | Type | Description |
|---------|------|-------------|
| `/insights` | prompt | Generate session analysis report |
| `/extra-usage` | local | Extra usage information |
| `/extra-usage-non-interactive` | local | Extra usage (non-interactive) |
| `/rate-limit-options` | local | Rate limit options |

## Command Availability

Commands can be filtered based on availability requirements:

```typescript
export function meetsAvailabilityRequirement(cmd: Command): boolean {
  if (!cmd.availability) return true
  
  for (const a of cmd.availability) {
    switch (a) {
      case 'claude-ai':
        if (isClaudeAISubscriber()) return true
        break
      case 'console':
        if (
          !isClaudeAISubscriber() &&
          !isUsing3PServices() &&
          isFirstPartyAnthropicBaseUrl()
        ) return true
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

### From Workflows

```typescript
// Workflow scripts can be commands
const workflowCommands = await getWorkflowCommands(cwd)
```

## Command Caching

Commands are memoized for performance:

```typescript
const loadAllCommands = memoize(async (cwd: string): Promise<Command[]> => {
  // Load all command sources
})

export const getSkillToolCommands = memoize(
  async (cwd: string): Promise<Command[]> => {
    // Load skill commands
  }
)

export const getSlashCommandToolSkills = memoize(
  async (cwd: string): Promise<Command[]> => {
    // Load skill commands
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

export function filterCommandsForRemoteMode(commands: Command[]): Command[] {
  return commands.filter(cmd => REMOTE_SAFE_COMMANDS.has(cmd))
}
```

## Bridge Mode

Commands safe for bridge (remote control) execution:

```typescript
export const BRIDGE_SAFE_COMMANDS: Set<Command> = new Set([
  compact, clear, cost, summary, releaseNotes, files
].filter((c): c is Command => c !== null))

export function isBridgeSafeCommand(cmd: Command): boolean {
  if (cmd.type === 'local-jsx') return false
  if (cmd.type === 'prompt') return true
  return BRIDGE_SAFE_COMMANDS.has(cmd)
}
```

## Command Filtering

Commands can be filtered by various criteria:

```typescript
// Filter by availability
const available = allCommands.filter(
  _ => meetsAvailabilityRequirement(_) && isCommandEnabled(_)
)

// Filter for tool invocations (skills)
const skillCommands = allCommands.filter(
  cmd => cmd.type === 'prompt' &&
    !cmd.disableModelInvocation &&
    cmd.source !== 'builtin' &&
    (cmd.loadedFrom === 'skills' ||
     cmd.loadedFrom === 'plugin' ||
     cmd.loadedFrom === 'bundled' ||
     cmd.hasUserSpecifiedDescription ||
     cmd.whenToUse)
)
```

## Dynamic Skills

Skills can be discovered dynamically during file operations:

```typescript
// Get dynamic skills discovered during file operations
const dynamicSkills = getDynamicSkills()

// Dedupe and add to command list
const uniqueDynamicSkills = dynamicSkills.filter(
  s => !baseCommandNames.has(s.name) &&
    meetsAvailabilityRequirement(s) &&
    isCommandEnabled(s)
)
```

## Clearing Caches

When commands change, caches need to be cleared:

```typescript
export function clearCommandsCache(): void {
  clearCommandMemoizationCaches()
  clearPluginCommandCache()
  clearPluginSkillsCache()
  clearSkillCaches()
}

export function clearCommandMemoizationCaches(): void {
  loadAllCommands.cache?.clear?.()
  getSkillToolCommands.cache?.clear?.()
  getSlashCommandToolSkills.cache?.clear?.()
  clearSkillIndexCache?.()
}
```

## MCP Skill Commands

MCP-provided skills are filtered separately:

```typescript
export function getMcpSkillCommands(
  mcpCommands: readonly Command[]
): readonly Command[] {
  if (feature('MCP_SKILLS')) {
    return mcpCommands.filter(
      cmd =>
        cmd.type === 'prompt' &&
        cmd.loadedFrom === 'mcp' &&
        !cmd.disableModelInvocation
    )
  }
  return []
}
```

## Source Description Formatting

Commands can have their source annotated for display:

```typescript
export function formatDescriptionWithSource(cmd: Command): string {
  if (cmd.type !== 'prompt') {
    return cmd.description
  }

  if (cmd.kind === 'workflow') {
    return `${cmd.description} (workflow)`
  }

  if (cmd.source === 'plugin') {
    const pluginName = cmd.pluginInfo?.pluginManifest.name
    if (pluginName) {
      return `(${pluginName}) ${cmd.description}`
    }
    return `${cmd.description} (plugin)`
  }

  if (cmd.source === 'bundled') {
    return `${cmd.description} (bundled)`
  }

  return cmd.description
}
```

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
- **Dynamic**: Can load from plugins, skills, MCP servers, workflows
- **Cached**: Performance optimized with memoization
- **Filtered**: Respects availability, permissions, and feature flags
- **Integrated**: Seamlessly works with the TUI
- **Feature-gated**: Many commands are behind feature flags for gradual rollout
- **Remote-aware**: Different command sets for local vs remote mode