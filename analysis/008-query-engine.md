# 007 - Query Engine (`src/QueryEngine.ts`)

## Overview

The Query Engine is the **core AI interaction layer**. It manages the conversation loop, handles tool execution, manages context, and coordinates between the user, AI, and tools.

## Key Responsibilities

### 1. Conversation Management
- Maintains message history
- Handles multi-turn conversations
- Manages session state

### 2. Tool Orchestration
- Executes tools requested by AI
- Manages tool results
- Handles tool errors

### 3. Context Management
- Manages system prompts
- Handles user context
- Controls token limits

### 4. API Integration
- Sends requests to AI providers
- Handles streaming responses
- Manages retries and fallbacks

## Query Engine Class

```typescript
export class QueryEngine {
  private config: QueryEngineConfig
  private mutableMessages: Message[]
  private abortController: AbortController
  private permissionDenials: SDKPermissionDenial[]
  private totalUsage: NonNullableUsage
  
  constructor(config: QueryEngineConfig) {
    this.config = config
    this.mutableMessages = config.initialMessages ?? []
    this.abortController = config.abortController ?? createAbortController()
    this.permissionDenials = []
    this.totalUsage = EMPTY_USAGE
  }
  
  async *submitMessage(
    prompt: string | ContentBlockParam[],
    options?: { uuid?: string; isMeta?: boolean }
  ): AsyncGenerator<SDKMessage, void, unknown> {
    // Main query loop
  }
}
```

## Query Configuration

```typescript
export type QueryEngineConfig = {
  cwd: string
  tools: Tools
  commands: Command[]
  mcpClients: MCPServerConnection[]
  agents: AgentDefinition[]
  canUseTool: CanUseToolFn
  getAppState: () => AppState
  setAppState: (f: (prev: AppState) => AppState) => void
  initialMessages?: Message[]
  readFileCache: FileStateCache
  customSystemPrompt?: string
  appendSystemPrompt?: string
  userSpecifiedModel?: string
  fallbackModel?: string
  thinkingConfig?: ThinkingConfig
  maxTurns?: number
  maxBudgetUsd?: number
  taskBudget?: { total: number }
  jsonSchema?: Record<string, unknown>
  verbose?: boolean
  replayUserMessages?: boolean
  handleElicitation?: ToolUseContext['handleElicitation']
  includePartialMessages?: boolean
  setSDKStatus?: (status: SDKStatus) => void
  abortController?: AbortController
  orphanedPermission?: OrphanedPermission
  snipReplay?: (yieldedSystemMsg: Message, store: Message[]) => 
    { messages: Message[]; executed: boolean } | undefined
}
```

## Query Flow

### 1. User Input Processing

```typescript
async *submitMessage(prompt: string | ContentBlockParam[], options) {
  // 1. Process user input
  const { messages: messagesFromUserInput, shouldQuery, allowedTools } = 
    await processUserInput({
      input: prompt,
      mode: 'prompt',
      context: processUserInputContext,
      messages: this.mutableMessages
    })
  
  // 2. Push new messages
  this.mutableMessages.push(...messagesFromUserInput)
  
  // 3. Record transcript
  await recordTranscript(this.mutableMessages)
}
```

### 2. System Prompt Building

```typescript
// Build system prompt
const { defaultSystemPrompt, userContext, systemContext } = 
  await fetchSystemPromptParts({
    tools,
    mainLoopModel,
    additionalWorkingDirectories,
    mcpClients,
    customSystemPrompt
  })

// Combine prompts
const systemPrompt = asSystemPrompt([
  ...defaultSystemPrompt,
  ...(memoryMechanicsPrompt ? [memoryMechanicsPrompt] : []),
  ...(appendSystemPrompt ? [appendSystemPrompt] : [])
])
```

### 3. API Call Loop

```typescript
// Main query loop
for await (const message of query({
  messages,
  systemPrompt,
  userContext,
  systemContext,
  canUseTool: wrappedCanUseTool,
  toolUseContext: processUserInputContext,
  fallbackModel,
  querySource: 'sdk',
  maxTurns,
  taskBudget
})) {
  // Handle different message types
  switch (message.type) {
    case 'assistant':
      this.mutableMessages.push(message)
      yield* normalizeMessage(message)
      break
      
    case 'user':
      this.mutableMessages.push(message)
      yield* normalizeMessage(message)
      break
      
    case 'stream_event':
      // Handle streaming events
      break
  }
}
```

### 4. Tool Execution

```typescript
// When AI requests tool use
const toolUseBlocks = message.message.content.filter(
  content => content.type === 'tool_use'
) as ToolUseBlock[]

for (const toolBlock of toolUseBlocks) {
  // Execute tool
  const result = await executeTool(
    toolBlock,
    tools,
    canUseTool,
    toolUseContext
  )
  
  // Add result to messages
  toolResults.push(result)
}
```

## Context Management

### System Context

```typescript
export const getSystemContext = memoize(async (): Promise<{ [k: string]: string }> => {
  const gitStatus = await getGitStatus()
  
  return {
    ...(gitStatus && { gitStatus }),
    cacheBreaker: `[CACHE_BREAKER: ${injection}]`
  }
})
```

### User Context

```typescript
export const getUserContext = memoize(async (): Promise<{ [k: string]: string }> => {
  const claudeMd = await getClaudeMds(getMemoryFiles())
  
  return {
    ...(claudeMd && { claudeMd }),
    currentDate: `Today's date is ${getLocalISODate()}.`
  }
})
```

## Token Management

### Token Counting

```typescript
const tokenCount = tokenCountWithEstimation(messagesForQuery)

// Check if at blocking limit
const { isAtBlockingLimit } = calculateTokenWarningState(
  tokenCount - snipTokensFreed,
  mainLoopModel
)

if (isAtBlockingLimit) {
  yield createAssistantAPIErrorMessage({
    content: PROMPT_TOO_LONG_ERROR_MESSAGE
  })
  return { reason: 'blocking_limit' }
}
```

### Auto-Compact

```typescript
// Check if auto-compact is needed
const { compactionResult } = await deps.autocompact(
  messagesForQuery,
  toolUseContext,
  {
    systemPrompt,
    userContext,
    systemContext,
    toolUseContext,
    forkContextMessages: messagesForQuery
  },
  querySource,
  tracking,
  snipTokensFreed
)

if (compactionResult) {
  // Replace messages with compacted version
  messagesForQuery = buildPostCompactMessages(compactionResult)
}
```

## Tool Result Handling

### Large Results

```typescript
// Apply tool result budget
const toolResultBudgetResult = await applyToolResultBudget(
  messagesForQuery,
  toolUseContext.contentReplacementState,
  persistReplacements ? records => recordContentReplacement(records) : undefined,
  new Set(tools.filter(t => !Number.isFinite(t.maxResultSizeChars)).map(t => t.name))
)

messagesForQuery = toolResultBudgetResult.messages
```

### Streaming Execution

```typescript
// Streaming tool execution
const streamingToolExecutor = useStreamingToolExecution
  ? new StreamingToolExecutor(tools, canUseTool, toolUseContext)
  : null

for (const toolBlock of msgToolUseBlocks) {
  streamingToolExecutor?.addTool(toolBlock, message)
}

for (const result of streamingToolExecutor.getCompletedResults()) {
  yield result.message
}
```

## Error Handling

### API Errors

```typescript
try {
  for await (const message of deps.callModel(...)) {
    // Process messages
  }
} catch (error) {
  if (error instanceof FallbackTriggeredError && fallbackModel) {
    // Switch to fallback model
    currentModel = fallbackModel
    attemptWithFallback = true
    continue
  }
  throw error
}
```

### Tool Errors

```typescript
// Yield missing tool result blocks
yield* yieldMissingToolResultBlocks(
  assistantMessages,
  errorMessage
)
```

## Recovery Paths

### Max Output Tokens

```typescript
if (isWithheldMaxOutputTokens(message)) {
  withheld = true
}

if (!withheld) {
  yield yieldMessage
}
```

### Prompt Too Long

```typescript
if (reactiveCompact?.isWithheldPromptTooLong(message)) {
  withheld = true
}

if (contextCollapse?.isWithheldPromptTooLong(message)) {
  withheld = true
}
```

## Snip (History Snipping)

```typescript
if (feature('HISTORY_SNIP')) {
  const snipResult = snipModule!.snipCompactIfNeeded(messagesForQuery)
  messagesForQuery = snipResult.messages
  snipTokensFreed = snipResult.tokensFreed
  
  if (snipResult.boundaryMessage) {
    yield snipResult.boundaryMessage
  }
}
```

## Micro-Compact

```typescript
const microcompactResult = await deps.microcompact(
  messagesForQuery,
  toolUseContext,
  querySource
)
messagesForQuery = microcompactResult.messages
```

## Context Collapse

```typescript
if (feature('CONTEXT_COLLAPSE') && contextCollapse) {
  const collapseResult = await contextCollapse.applyCollapsesIfNeeded(
    messagesForQuery,
    toolUseContext,
    querySource
  )
  messagesForQuery = collapseResult.messages
}
```

## Budget Tracking

```typescript
const budgetTracker = feature('TOKEN_BUDGET') ? createBudgetTracker() : null

if (budgetTracker) {
  const { shouldContinue } = checkTokenBudget(budgetTracker)
  if (!shouldContinue) {
    yield createAssistantAPIErrorMessage({
      content: 'Token budget exceeded'
    })
    return { reason: 'budget_exceeded' }
  }
}
```

## Key Features

### 1. Streaming Support
- Real-time message streaming
- Partial message handling
- Progress updates

### 2. Multi-Model Support
- Primary and fallback models
- Model switching on errors
- Thinking mode support

### 3. Tool Orchestration
- Parallel tool execution
- Streaming tool results
- Error recovery

### 4. Context Management
- Auto-compact on limit
- History snipping
- Context collapse

### 5. Error Recovery
- Fallback models
- Retry logic
- Graceful degradation

## Integration Points

### With TUI
```typescript
// TUI calls submitMessage
const messages = queryEngine.submitMessage(userInput)

for await (const message of messages) {
  // Update UI with message
}
```

### With Tools
```typescript
// Query engine executes tools
const result = await tool.call(input, context)
// Tool result returned to AI
```

### With Providers
```typescript
// Query engine calls provider
const response = await provider.chat({
  messages,
  tools,
  systemPrompt
})
```

## Summary

The Query Engine:
- **Manages conversations** across multiple turns
- **Orchestrates tools** and handles results
- **Controls context** with compaction and snipping
- **Handles errors** with fallbacks and retries
- **Integrates seamlessly** with TUI, tools, and providers

It's the intelligent core that makes OpenClaude work.
</task_progress>
</write_to_file>