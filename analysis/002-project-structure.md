# OpenClaude Codebase Analysis - Part 1: Project Structure

## Overview
OpenClaude is a full-stack TypeScript/JavaScript application that provides an AI-powered code assistant with CLI, TUI (Terminal User Interface), and backend services.

## Root Directory Structure
```
openclaude/
├── bin/                    # CLI entry point scripts
├── docs/                   # Documentation
├── python/                 # Python providers for local AI
├── scripts/                # Build and utility scripts
├── src/                    # Main source code
├── analysis/               # This analysis (you are here)
├── vscode-extension/       # VS Code extension
├── package.json            # Dependencies and scripts
├── tsconfig.json           # TypeScript configuration
└── README.md              # Project documentation
```

## Source Code Organization (`src/`)

### Core Entry Points
- **`main.tsx`**: Main application entry point, initializes the app
- **`commands.ts`**: Command registration and routing system
- **`tools.ts`**: Tool registration and management
- **`context.ts`**: Context management for the application
- **`history.ts`**: Command history persistence
- **`ink.ts`**: TUI framework (React/Ink)
- **`setup.ts`**: Session setup and initialization
- **`Task.ts`**: Task management
- **`tasks.ts`**: Task utilities
- **`Tool.ts`**: Tool type definitions
- **`query.ts`**: Query execution
- **`QueryEngine.ts`**: AI conversation engine
- **`cost-tracker.ts`**: Cost tracking
- **`costHook.ts`**: Cost tracking hooks
- **`dialogLaunchers.tsx`**: Dialog launching utilities
- **`interactiveHelpers.tsx`**: Interactive TUI helpers
- **`replLauncher.tsx`**: REPL launcher
- **`projectOnboardingState.ts`**: Onboarding state management

### Major Subsystems

#### 1. **CLI Layer** (`src/cli/`)
- `exit.ts`: Exit handling
- `ndjsonSafeStringify.ts`: NDJSON stringification
- `print.ts`: Output formatting
- `remoteIO.ts`: Remote I/O handling
- `structuredIO.ts`: Structured output
- `update.ts`: Update management
- `handlers/`: CLI request handlers
- `transports/`: CLI transport implementations

#### 2. **Commands** (`src/commands/`)
Each command has its own directory with implementation files. Commands include:

**Core Commands:**
- `add-dir/`: Add directory to context
- `agents/`: Agent management
- `branch/`: Git branch operations
- `btw/`: Quick notes
- `clear/`: Clear context
- `color/`: Color settings
- `compact/`: Context compaction
- `config/`: Configuration
- `context/`: Context management
- `copy/`: Copy to clipboard
- `cost/`: Cost tracking
- `diff/`: File differences
- `doctor/`: Diagnostics
- `effort/`: Effort tracking
- `exit/`: Exit command
- `export/`: Export data
- `fast/`: Fast mode
- `feedback/`: User feedback
- `files/`: File operations
- `help/`: Help system
- `hooks/`: Hook management
- `ide/`: IDE integration
- `init/`: Initialization
- `init-verifiers/`: Verifier initialization (internal)
- `keybindings/`: Key binding configuration
- `login/`: Authentication login
- `logout/`: Authentication logout
- `mcp/`: MCP server management
- `memory/`: Memory management
- `mobile/`: Mobile features
- `model/`: Model selection
- `onboard-github/`: GitHub onboarding
- `output-style/`: Output style configuration
- `permissions/`: Permission management
- `plan/`: Planning features
- `plugin/`: Plugin management
- `privacy-settings/`: Privacy settings
- `provider/`: Provider configuration
- `reload-plugins/`: Reload plugins
- `rename/`: Rename sessions
- `resume/`: Session resumption
- `rewind/`: Rewind to previous state
- `session/`: Session management
- `skills/`: Skills management
- `status/`: Status display
- `stats/`: Statistics
- `tag/`: Tag management
- `tasks/`: Task management
- `teleport/`: Teleport sessions
- `terminal-setup/`: Terminal setup
- `theme/`: Theme management
- `upgrade/`: Upgrade handling
- `usage/`: Usage statistics
- `vim/`: Vim mode

**Advanced/Feature-Gated Commands:**
- `advisor/`: Advisor mode
- `agents-platform/`: Agent platform (internal)
- `ant-trace/`: Ant tracing (internal)
- `assistant/`: Assistant mode (KAIROS feature)
- `autofix-pr/`: Auto-fix PR
- `backfill-sessions/`: Backfill sessions (internal)
- `break-cache/`: Break cache (internal)
- `bridge/`: Bridge mode
- `bridge-kick/`: Kick bridge session
- `brief/`: Brief mode (KAIROS feature)
- `bughunter/`: Bug hunter mode (internal)
- `buddy/`: Buddy companion
- `chrome/`: Chrome integration
- `commit/`: Commit changes (internal)
- `commit-push-pr/`: Commit and push PR (internal)
- `ctx_viz/`: Context visualization (internal)
- `debug-tool-call/`: Debug tool calls (internal)
- `desktop/`: Desktop integration
- `diff/`: File differences
- `dream/`: Dream mode
- `env/`: Environment variables (internal)
- `good-claude/`: Good Claude (internal)
- `heapdump/`: Heap dump
- `issue/`: Issue tracking (internal)
- `install-github-app/`: Install GitHub app
- `install-slack-app/`: Install Slack app
- `mock-limits/`: Mock limits (internal)
- `oauth-refresh/`: OAuth refresh (internal)
- `onboarding/`: Onboarding flow
- `passes/`: Passes management
- `peers/`: Peers management (UDS_INBOX feature)
- `perf-issue/`: Performance issue tracking (internal)
- `pr_comments/`: PR comments
- `release-notes/`: Release notes
- `remote-control-server/`: Remote control server
- `remote-env/`: Remote environment
- `remote-setup/`: Remote setup (CCR_REMOTE_SETUP feature)
- `reset-limits/`: Reset limits (internal)
- `review/`: Code review
- `sandbox-toggle/`: Sandbox toggle
- `security-review/`: Security review
- `share/`: Share sessions
- `stickers/`: Stickers
- `statusline/`: Status line
- `subscribe-pr/`: Subscribe to PR (KAIROS_GITHUB_WEBHOOKS feature)
- `summary/`: Session summary
- `terminalSetup/`: Terminal setup
- `thinkback/`: Thinkback feature
- `thinkback-play/`: Thinkback play
- `torch/`: Torch feature
- `ultraplan/`: Ultra plan (ULTRAPLAN feature)
- `ultrareview/`: Ultra review
- `version/`: Version info (internal)
- `voice/`: Voice mode (VOICE_MODE feature)
- `workflows/`: Workflow scripts (WORKFLOW_SCRIPTS feature)

#### 3. **Components** (`src/components/`)
React/Ink UI components for the TUI:
- Design system components
- Dialog components
- Display components
- Input components
- Message components
- Layout components

#### 4. **Services** (`src/services/`)
- `analytics/`: Analytics tracking (GrowthBook, Statsig)
- `api/`: API client implementations
- `claudeAiLimits.js`: Claude.ai quota management
- `compact/`: Context compaction (snip, snip projection)
- `lsp/`: Language Server Protocol
- `mcp/`: MCP protocol implementation
- `plugins/`: Plugin system
- `policyLimits/`: Policy limit management
- `PromptSuggestion/`: Prompt suggestions
- `remoteManagedSettings/`: Remote settings management
- `remoteSettings/`: Remote settings
- `settingsSync/`: Settings synchronization
- `skillSearch/`: Skill search functionality
- `skills/`: Skills system
- `tips/`: Tip system
- `tools/`: Tool orchestration

#### 5. **Tools** (`src/tools/`)
Each tool has its own directory:
- `AgentTool/`: Agent spawning and management
- `AskUserQuestionTool/`: Ask user for input
- `BashTool/`: Shell command execution
- `BriefTool/`: Brief mode tool (KAIROS feature)
- `ConfigTool/`: Configuration tool (internal)
- `CronCreateTool/`: Create cron triggers (AGENT_TRIGGERS feature)
- `CronDeleteTool/`: Delete cron triggers (AGENT_TRIGGERS feature)
- `CronListTool/`: List cron triggers (AGENT_TRIGGERS feature)
- `CtxInspectTool/`: Context inspection (CONTEXT_COLLAPSE feature)
- `EnterPlanModeTool/`: Enter plan mode
- `EnterWorktreeTool/`: Enter worktree mode
- `ExitPlanModeTool/`: Exit plan mode
- `ExitWorktreeTool/`: Exit worktree mode
- `FileEditTool/`: File editing
- `FileReadTool/`: File reading
- `FileWriteTool/`: File writing
- `GlobTool/`: File pattern matching
- `GrepTool/`: Text search
- `ListMcpResourcesTool/`: List MCP resources
- `ListPeersTool/`: List peers (UDS_INBOX feature)
- `LSPTool/`: Language Server Protocol
- `MonitorTool/`: Monitor tool (MONITOR_TOOL feature)
- `NotebookEditTool/`: Notebook editing
- `OverflowTestTool/`: Overflow test (OVERFLOW_TEST_TOOL feature)
- `PowerShellTool/`: PowerShell execution
- `PushNotificationTool/`: Push notifications (KAIROS feature)
- `ReadMcpResourceTool/`: Read MCP resources
- `RemoteTriggerTool/`: Remote triggers (AGENT_TRIGGERS_REMOTE feature)
- `REPLTool/`: REPL tool (internal)
- `SendUserFileTool/`: Send user files (KAIROS feature)
- `SendMessageTool/`: Send messages between agents
- `SkillTool/`: Execute skills/commands
- `SleepTool/`: Sleep tool (PROACTIVE feature)
- `SnipTool/`: Snip history (HISTORY_SNIP feature)
- `SubscribePRTool/`: Subscribe to PR webhooks
- `SuggestBackgroundPRTool/`: Suggest background PR (internal)
- `SyntheticOutputTool/`: Synthetic output for structured responses
- `TaskCreateTool/`: Create tasks
- `TaskGetTool/`: Get task details
- `TaskListTool/`: List tasks
- `TaskStopTool/`: Stop running tasks
- `TaskUpdateTool/`: Update tasks
- `TaskOutputTool/`: Task output
- `TeamCreateTool/`: Create agent teams (agent swarms)
- `TeamDeleteTool/`: Delete agent teams
- `TerminalCaptureTool/`: Terminal capture (TERMINAL_PANEL feature)
- `TestingPermissionTool/`: Testing permissions (test mode only)
- `TodoWriteTool/`: Manage todo lists
- `ToolSearchTool/`: Search for tools
- `TungstenTool/`: Tungsten tool (internal)
- `VerifyPlanExecutionTool/`: Verify plan execution
- `WebBrowserTool/`: Web browser control (WEB_BROWSER_TOOL feature)
- `WebFetchTool/`: Fetch web pages
- `WebSearchTool/`: Search the web
- `WorkflowTool/`: Workflow execution (WORKFLOW_SCRIPTS feature)

#### 6. **Bridge** (`src/bridge/`)
Remote connection and session management:
- `bridgeApi.ts`: API client
- `bridgeConfig.ts`: Configuration
- `bridgeDebug.ts`: Debug utilities
- `bridgeEnabled.ts`: Feature detection
- `bridgeMain.ts`: Main bridge logic
- `bridgeMessaging.ts`: Message handling
- `bridgePermissionCallbacks.ts`: Permission callbacks
- `bridgePointer.ts`: Pointer tracking
- `bridgeStatusUtil.ts`: Status utilities
- `bridgeUI.ts`: User interface
- `capacityWake.ts`: Capacity management
- `codeSessionApi.ts`: Code session API
- `createSession.ts`: Session creation
- `debugUtils.ts`: Debug utilities
- `envLessBridgeConfig.ts`: Environment-less config
- `flushGate.ts`: Flush gate control
- `inboundAttachments.ts`: Inbound attachments
- `inboundMessages.ts`: Inbound message handling
- `initReplBridge.ts`: REPL bridge initialization
- `jwtUtils.ts`: JWT utilities
- `pollConfig.ts`: Polling configuration
- `pollConfigDefaults.ts`: Default poll config
- `remoteBridgeCore.ts`: Remote bridge core
- `replBridge.ts`: REPL bridge
- `replBridgeHandle.ts`: REPL bridge handle
- `replBridgeTransport.ts`: REPL bridge transport
- `sessionIdCompat.ts`: Session ID compatibility
- `sessionRunner.ts`: Session execution
- `trustedDevice.ts`: Trusted device management
- `types.ts`: Type definitions
- `workSecret.ts`: Secret management

#### 7. **State Management** (`src/state/`)
- `AppState.ts`: Application state type definitions
- `AppStateStore.tsx`: State store with React context
- `onChangeAppState.ts`: State change handlers
- `store.ts`: State store implementation

#### 8. **Utilities** (`src/utils/`)
- `abortController.ts`: Abort controller utilities
- `advisor.ts`: Advisor mode utilities
- `agentSwarmsEnabled.ts`: Agent swarms feature detection
- `api.js`: API utilities
- `array.ts`: Array utilities
- `asciicast.ts`: Asciicast recording
- `auth.ts`: Authentication utilities
- `autoUpdater.ts`: Auto-update functionality
- `betas.ts`: Beta feature management
- `bundledMode.ts`: Bundled mode detection
- `cleanupRegistry.ts`: Cleanup registration
- `claudeInChrome/`: Chrome integration utilities
- `cliArgs.ts`: CLI argument parsing
- `commitAttribution.ts`: Commit attribution
- `concurrentSessions.ts`: Concurrent session management
- `config.ts`: Configuration utilities
- `context.js`: Context utilities
- `conversationRecovery.ts`: Conversation recovery
- `cost-tracker.ts`: Cost tracking
- `cwd.ts`: Current working directory
- `debug.ts`: Debug utilities
- `deepLink/`: Deep link handling
- `diagLogs.ts`: Diagnostic logging
- `earlyInput.ts`: Early input capture
- `effort.ts`: Effort level management
- `embeddedTools.ts`: Embedded tool detection
- `envUtils.ts`: Environment variable utilities
- `errors.ts`: Error handling
- `exampleCommands.ts`: Example command generation
- `fastMode.ts`: Fast mode utilities
- `fileHistory.ts`: File history tracking
- `fileStateCache.ts`: File state caching
- `filesystem.js`: Filesystem utilities
- `fpsTracker.ts`: FPS tracking
- `fsOperations.js`: File system operations
- `git.ts`: Git operations
- `github/`: GitHub integration
- `githubRepoPathMapping.ts`: GitHub repo path mapping
- `gracefulShutdown.ts`: Graceful shutdown
- `headlessProfiler.ts`: Headless profiling
- `hooks/`: Hook utilities
- `json.ts`: JSON utilities
- `log.ts`: Logging utilities
- `managedEnv.ts`: Managed environment variables
- `messages.ts`: Message utilities
- `model/`: Model management utilities
- `permissions/`: Permission utilities
- `platform.ts`: Platform detection
- `plugins/`: Plugin utilities
- `process.js`: Process utilities
- `queryContext.ts`: Query context utilities
- `releaseNotes.ts`: Release notes management
- `renderOptions.ts`: Render options
- `ripgrep.ts`: Ripgrep utilities
- `sandbox/`: Sandbox utilities
- `sessionIngressAuth.ts`: Session ingress authentication
- `sessionRestore.ts`: Session restoration
- `sessionStart.ts`: Session start hooks
- `sessionStorage.ts`: Session storage
- `settings/`: Settings management
- `shell/`: Shell utilities
- `skills/`: Skill utilities
- `slowOperations.ts`: Slow operation utilities
- `startupProfiler.ts`: Startup profiling
- `stringUtils.ts`: String utilities
- `swarm/`: Agent swarm utilities
- `systemPromptType.ts`: System prompt type utilities
- `systemTheme.ts`: System theme detection
- `tasks.ts`: Task utilities
- `teammate.js`: Teammate utilities
- `telemetry/`: Telemetry utilities
- `tempfile.ts`: Temporary file generation
- `terminalSetup.ts`: Terminal setup
- `thinking.ts`: Thinking mode configuration
- `toolPool.ts`: Tool pool management
- `toolSearch.ts`: Tool search utilities
- `updateSystemTheme.ts`: System theme updates
- `user.ts`: User utilities
- `uuid.ts`: UUID utilities
- `warningHandler.ts`: Warning handling
- `worktree.ts`: Worktree utilities
- `worktreeModeEnabled.ts`: Worktree mode detection

#### 9. **Types** (`src/types/`)
TypeScript type definitions:
- `ids.ts`: ID type definitions
- `logs.ts`: Log type definitions
- `message.ts`: Message types
- `permissions.ts`: Permission types
- `textInputTypes.ts`: Text input types
- `textProcessing.ts`: Text processing types
- `tools.ts`: Tool types

#### 10. **Ink (TUI)** (`src/ink/`)
Terminal UI framework:
- `root.tsx`: Root component
- `hooks/`: React hooks
- `components/`: UI components
- `events/`: Event handling
- `termio/`: Terminal I/O

#### 11. **Bootstrap** (`src/bootstrap/`)
Application bootstrap and state:
- `state.ts`: Bootstrap state management

#### 12. **Buddy** (`src/buddy/`)
Companion sprite system:
- `companion.ts`: Companion logic
- `CompanionSprite.tsx`: Sprite component
- `feature.ts`: Feature detection
- `observer.ts`: Observer pattern
- `prompt.ts`: Prompt utilities
- `sprites.ts`: Sprite definitions
- `types.ts`: Type definitions
- `useBuddyNotification.tsx`: Notification hook

#### 13. **Coordinator** (`src/coordinator/`)
Coordinator mode for agent orchestration:
- `coordinatorMode.ts`: Coordinator mode implementation

#### 14. **Memory** (`src/memdir/`)
Persistent memory system:
- `memdir.ts`: Memory directory management
- `paths.ts`: Memory path utilities

#### 15. **Migrations** (`src/migrations/`)
Data migrations for upgrades:
- `migrateAutoUpdatesToSettings.ts`
- `migrateBypassPermissionsAcceptedToSettings.ts`
- `migrateEnableAllProjectMcpServersToSettings.ts`
- `migrateFennecToOpus.ts`
- `migrateLegacyOpusToCurrent.ts`
- `migrateOpusToOpus1m.ts`
- `migrateReplBridgeEnabledToRemoteControlAtStartup.ts`
- `migrateSonnet1mToSonnet45.ts`
- `migrateSonnet45ToSonnet46.ts`
- `resetAutoModeOptInForDefaultOffer.ts`
- `resetProToOpusDefault.ts`

#### 16. **Plugins** (`src/plugins/`)
Plugin system:
- `bundled/`: Built-in plugins
- `builtinPlugins.ts`: Built-in plugin definitions
- `pluginCliCommands.ts`: Plugin CLI commands

#### 17. **Skills** (`src/skills/`)
Skill system:
- `bundled/`: Built-in skills
- `bundledSkills.ts`: Built-in skill definitions
- `loadSkillsDir.ts`: Skill directory loading

#### 18. **Constants** (`src/constants/`)
Application constants:
- `oauth.ts`: OAuth configuration
- `product.ts`: Product configuration
- `tools.ts`: Tool constants
- `xml.ts`: XML constants

#### 19. **Context** (`src/context/`)
Context management:
- `stats.ts`: Statistics context

#### 20. **Entry Points** (`src/entrypoints/`)
Application entry points:
- `agentSdkTypes.ts`: SDK type definitions
- `init.ts`: Initialization
- `initSinks.ts`: Sink initialization

#### 21. **Hooks** (`src/hooks/`)
React hooks:
- `useCanUseTool.ts`: Tool usage hook

#### 22. **Keybindings** (`src/keybindings/`)
Keybinding management:
- Various keybinding definitions

#### 23. **Screens** (`src/screens/`)
UI screens:
- Various screen components

#### 24. **Server** (`src/server/`)
Server functionality:
- `createDirectConnectSession.ts`: Direct connect session creation
- `parseConnectUrl.ts`: Connect URL parsing

#### 25. **Tasks** (`src/tasks/`)
Task management:
- Task-related utilities

#### 26. **Vim** (`src/vim/`)
Vim mode:
- Vim keybinding support

#### 27. **Voice** (`src/voice/`)
Voice mode:
- Voice interaction support

## Build System

### Scripts (`scripts/`)
- `build.ts`: Build script
- `provider-bootstrap.ts`: Provider bootstrap
- `provider-discovery.ts`: Provider discovery
- `provider-launch.ts`: Provider launch
- `provider-recommend.ts`: Provider recommendations
- `system-check.ts`: System diagnostics
- `system-check.test.ts`: System check tests
- `verify-no-phone-home.sh`: Phone home verification
- `verify-no-phone-home.ts`: Phone home verification script
- `no-telemetry-plugin.ts`: Telemetry plugin

### Configuration Files
- `package.json`: NPM configuration
- `tsconfig.json`: TypeScript configuration
- `bun.lock`: Bun lockfile

## Python Providers (`python/`)
- `__init__.py`: Package initialization
- `ollama_provider.py`: Ollama integration
- `atomic_chat_provider.py`: Atomic Chat (Apple Silicon) integration
- `smart_router.py`: Intelligent request routing
- `tests/`: Python provider tests

## Key Architectural Patterns

1. **Command Pattern**: All user actions are commands with consistent interface
2. **Tool Pattern**: AI capabilities are tools with input/output schemas
3. **Provider Pattern**: Multiple AI providers with unified interface
4. **Component Pattern**: React/Ink components for TUI
5. **Service Pattern**: Backend services for API, analytics, etc.
6. **State Management**: Centralized state with change handlers
7. **Feature Flags**: Conditional features via `bun:bundle` feature()
8. **Plugin Architecture**: Extensible via plugins and skills
9. **Agent Swarms**: Multi-agent coordination via teammate mode
10. **Bridge System**: Remote session management

## Data Flow

```
User Input
    ↓
CLI Parser
    ↓
Command Router
    ↓
Command Handler
    ↓
Tool Executor (if needed)
    ↓
AI Provider (if needed)
    ↓
Response Formatter
    ↓
TUI/CLI Output
```

## Feature Flags

The codebase uses feature flags via `bun:bundle` for conditional compilation:

- `COORDINATOR_MODE`: Coordinator mode for agent orchestration
- `KAIROS`: Assistant mode
- `KAIROS_BRIEF`: Brief mode
- `KAIROS_CHANNELS`: MCP channel support
- `KAIROS_GITHUB_WEBHOOKS`: GitHub webhook support
- `KAIROS_PUSH_NOTIFICATION`: Push notifications
- `PROACTIVE`: Proactive mode
- `BRIDGE_MODE`: Bridge/remote control mode
- `DAEMON`: Daemon mode
- `VOICE_MODE`: Voice interaction
- `HISTORY_SNIP`: History snipping
- `WORKFLOW_SCRIPTS`: Workflow script support
- `CCR_REMOTE_SETUP`: Remote setup
- `EXPERIMENTAL_SKILL_SEARCH`: Skill search
- `ULTRAPLAN`: Ultra plan mode
- `TORCH`: Torch feature
- `UDS_INBOX`: Unix domain socket inbox
- `FORK_SUBAGENT`: Fork subagent
- `MONITOR_TOOL`: Monitor tool
- `AGENT_TRIGGERS`: Agent triggers (cron)
- `AGENT_TRIGGERS_REMOTE`: Remote agent triggers
- `CONTEXT_COLLAPSE`: Context collapse
- `TERMINAL_PANEL`: Terminal panel
- `WEB_BROWSER_TOOL`: Web browser tool
- `OVERFLOW_TEST_TOOL`: Overflow test tool
- `CHICAGO_MCP`: Computer use MCP
- `MCP_SKILLS`: MCP skills
- `UPLOAD_USER_SETTINGS`: Settings sync
- `SSH_REMOTE`: SSH remote support
- `DIRECT_CONNECT`: Direct connect
- `LODESTONE`: Deep link handling
- `BG_SESSIONS`: Background sessions
- `TRANSCRIPT_CLASSIFIER`: Transcript classification

## Next Steps
See subsequent parts for detailed analysis of each subsystem.