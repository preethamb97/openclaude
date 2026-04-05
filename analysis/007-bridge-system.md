# 006 - Bridge System (`src/bridge/`)

## Overview

The Bridge system enables **remote session management** and **multi-device control**. It allows you to:
- Control OpenClaude from mobile devices
- Run sessions remotely
- Manage multiple concurrent sessions
- Bridge between different environments

## Key Concepts

### What is a Bridge?

A Bridge is a **remote control system** that:
1. Polls a server for work items
2. Spawns local Claude sessions
3. Manages session lifecycle
4. Provides remote access via web/mobile

### Architecture

```
┌─────────────────┐     ┌─────────────────┐
│  Mobile/Web     │────▶│  Bridge Server  │
│  Client         │     │  (Cloud)        │
└─────────────────┘     └─────────────────┘
                               │
                               ▼
                        ┌─────────────────┐
                        │  Local Bridge   │
                        │  (Your Machine) │
                        └─────────────────┘
                               │
                               ▼
                        ┌─────────────────┐
                        │  Claude Session │
                        │  (Child Process)│
                        └─────────────────┘
```

## Bridge Components

## Bridge Components

The bridge system consists of many files that work together:

### Core Bridge Files
- `bridgeApi.ts`: API client for communicating with bridge server
- `bridgeConfig.ts`: Configuration management
- `bridgeDebug.ts`: Debug utilities
- `bridgeEnabled.ts`: Feature detection
- `bridgeMain.ts`: Main bridge logic
- `bridgeMessaging.ts`: Message handling
- `bridgePermissionCallbacks.ts`: Permission callbacks
- `bridgePointer.ts`: Pointer tracking
- `bridgeStatusUtil.ts`: Status utilities
- `bridgeUI.ts`: User interface

### Session Management
- `capacityWake.ts`: Capacity management
- `codeSessionApi.ts`: Code session API
- `createSession.ts`: Session creation
- `sessionRunner.ts`: Session execution
- `sessionIdCompat.ts`: Session ID compatibility

### Remote Bridge
- `remoteBridgeCore.ts`: Remote bridge core
- `replBridge.ts`: REPL bridge
- `replBridgeHandle.ts`: REPL bridge handle
- `replBridgeTransport.ts`: REPL bridge transport

### Utilities
- `debugUtils.ts`: Debug utilities
- `envLessBridgeConfig.ts`: Environment-less config
- `flushGate.ts`: Flush gate control
- `inboundAttachments.ts`: Inbound attachments
- `inboundMessages.ts`: Inbound message handling
- `initReplBridge.ts`: REPL bridge initialization
- `jwtUtils.ts`: JWT utilities
- `pollConfig.ts`: Polling configuration
- `pollConfigDefaults.ts`: Default poll config
- `trustedDevice.ts`: Trusted device management
- `types.ts`: Type definitions
- `workSecret.ts`: Secret management

### Main Bridge Loop

The heart of the bridge system:

```typescript
export async function runBridgeLoop(
  config: BridgeConfig,
  environmentId: string,
  environmentSecret: string,
  api: BridgeApiClient,
  spawner: SessionSpawner,
  logger: BridgeLogger,
  signal: AbortSignal
): Promise<void> {
  // Main poll loop
  while (!signal.aborted) {
    const work = await api.pollForWork(environmentId, environmentSecret)
    
    if (work) {
      // Spawn session for this work
      const session = spawner.spawn(work.opts, config.dir)
      activeSessions.set(work.id, session)
    }
    
    await sleep(pollInterval)
  }
}
```

### 2. `bridgeApi.ts` - API Client

Communicates with the bridge server:

```typescript
export function createBridgeApiClient(
  config: BridgeConfig
): BridgeApiClient {
  return {
    async pollForWork(environmentId, secret, signal) {
      // Poll for work items
    },
    
    async acknowledgeWork(environmentId, workId, token) {
      // Acknowledge work receipt
    },
    
    async heartbeatWork(environmentId, workId, token) {
      // Keep session alive
    },
    
    async stopWork(environmentId, workId) {
      // Stop a work item
    }
  }
}
```

### 3. `sessionRunner.ts` - Session Spawning

Spawns Claude sessions for work items:

```typescript
export function createSessionSpawner(
  config: BridgeConfig
): SessionSpawner {
  return {
    spawn(opts, dir) {
      const child = spawn(process.execPath, [
        'dist/cli.mjs',
        '--sdk-url', opts.sdkUrl,
        '--model', opts.model
      ], {
        cwd: dir,
        stdio: ['pipe', 'pipe', 'pipe']
      })
      
      return {
        pid: child.pid,
        stdout: child.stdout,
        stderr: child.stderr,
        kill: () => child.kill()
      }
    }
  }
}
```

### 4. `workSecret.ts` - Token Management

Manages authentication tokens:

```typescript
export function decodeWorkSecret(secret: string): WorkSecret {
  // Decode JWT token
  const decoded = jwt.verify(secret, publicKey)
  
  return {
    session_ingress_token: decoded.session_ingress_token,
    api_base_url: decoded.api_base_url,
    use_code_sessions: decoded.use_code_sessions
  }
}
```

### 5. `bridgeUI.ts` - User Interface

Displays bridge status and logs:

```typescript
export function createBridgeLogger(): BridgeLogger {
  return {
    printBanner(config, environmentId) {
      console.log(`Bridge running: ${environmentId}`)
      console.log(`Mode: ${config.spawnMode}`)
      console.log(`Max sessions: ${config.maxSessions}`)
    },
    
    logSessionComplete(sessionId, duration) {
      console.log(`Session ${sessionId} completed in ${duration}ms`)
    }
  }
}
```

## Bridge Modes

### Single-Session Mode
```typescript
{
  spawnMode: 'single-session',
  maxSessions: 1
}
// One session at a time, exits after completion
```

### Multi-Session Mode
```typescript
{
  spawnMode: 'same-dir',
  maxSessions: 32
}
// Multiple sessions in same directory
```

### Worktree Mode
```typescript
{
  spawnMode: 'worktree',
  maxSessions: 8
}
// Each session gets its own git worktree
```

## Bridge Lifecycle

### 1. Initialization
```typescript
// Create API client
const api = createBridgeApiClient(config)

// Create session spawner
const spawner = createSessionSpawner(config)

// Start bridge loop
await runBridgeLoop(config, envId, envSecret, api, spawner, logger, signal)
```

### 2. Work Polling
```typescript
// Poll for work
const work = await api.pollForWork(environmentId, secret, signal)

if (work) {
  // Decode work secret
  const secret = decodeWorkSecret(work.secret)
  
  // Spawn session
  const session = spawner.spawn({
    sdkUrl: secret.api_base_url,
    model: work.data.model
  }, config.dir)
}
```

### 3. Session Management
```typescript
// Track active sessions
const activeSessions = new Map<string, SessionHandle>()

// Update session status
activeSessions.set(sessionId, {
  pid: child.pid,
  currentActivity: { type: 'thinking' },
  activities: []
})
```

### 4. Heartbeat
```typescript
// Send heartbeat to keep work alive
await api.heartbeatWork(environmentId, workId, token)

// If heartbeat fails, re-queue work
await api.reconnectSession(environmentId, sessionId)
```

## CCR v2 (Code Session Remote)

Newer version with improved architecture:

```typescript
// CCR v2 uses different endpoints
const sdkUrl = buildCCRv2SdkUrl(config.apiBaseUrl, sessionId)

// Register as worker
const epoch = await registerWorker(sdkUrl, token)

// Spawn session with v2 protocol
const session = spawner.spawn({
  sdkUrl,
  useCodeSessions: true
}, config.dir)
```

## Error Handling

### Connection Errors
```typescript
// Exponential backoff for connection errors
const connBackoff = Math.min(
  backoffConfig.connInitialMs * Math.pow(2, attempt),
  backoffConfig.connCapMs
)

if (Date.now() - connErrorStart > backoffConfig.connGiveUpMs) {
  throw new BridgeFatalError('Connection timeout')
}
```

### Session Errors
```typescript
// Track session failures
session.errorCount++

// If error rate too high, mark unhealthy
if (session.errorCount / session.requestCount > 0.7) {
  session.healthy = false
  // Schedule re-check after 60s
  setTimeout(() => recheckProvider(session), 60000)
}
```

## Telemetry

Bridge logs events for monitoring:

```typescript
logEvent('tengu_bridge_session_done', {
  status: 'completed',
  duration_ms: duration
})

logEvent('tengu_bridge_reconnected', {
  disconnected_ms: disconnectedTime
})
```

## Configuration

### Environment Variables
```bash
# Bridge configuration
CLAUDE_BRIDGE_ENVIRONMENT_ID=env_123
CLAUDE_BRIDGE_ENVIRONMENT_SECRET=secret_456
CLAUDE_BRIDGE_API_BASE_URL=https://bridge.example.com
CLAUDE_BRIDGE_SESSION_INGRESS_URL=wss://ingress.example.com
```

### Bridge Config Object
```typescript
type BridgeConfig = {
  apiBaseUrl: string
  sessionIngressUrl: string
  dir: string
  spawnMode: 'single-session' | 'same-dir' | 'worktree'
  maxSessions: number
  debugFile?: string
}
```

## Remote Control Commands

### From Mobile/Web
```typescript
// Send command to session
await api.sendCommand(sessionId, {
  type: 'user_message',
  content: 'Explain this code'
})

// Get session status
const status = await api.getSessionStatus(sessionId)

// Stop session
await api.stopSession(sessionId)
```

## Integration with TUI

The TUI shows bridge status:

```typescript
// In REPL.tsx
if (bridgeMode) {
  return (
    <Box>
      <Text>Bridge Mode Active</Text>
      <Text>Sessions: {activeSessions.size}/{maxSessions}</Text>
    </Box>
  )
}
```

## Security

### Authentication
- JWT tokens for session ingress
- Environment secrets for API access
- Trusted device tokens

### Sandboxing
- Sessions run in isolated directories
- Git worktrees prevent conflicts
- Permission checks on all operations

## Use Cases

### 1. Mobile Development
```bash
# Start bridge on your machine
openclaude remote-control

# Control from your phone
# Open web UI, send commands
```

### 2. CI/CD Integration
```bash
# Run bridge in CI
openclaude remote-control --spawn-mode single-session

# Trigger from CI pipeline
curl -X POST https://bridge.example.com/trigger
```

### 3. Multi-Device Workflow
```bash
# Start on desktop
openclaude remote-control

# Continue on laptop
openclaude --resume <session-id>
```

## Summary

The Bridge system:
- **Enables remote control** of Claude sessions
- **Supports multiple modes** (single, multi, worktree)
- **Handles errors gracefully** with retries and backoff
- **Provides telemetry** for monitoring
- **Integrates with TUI** for status display
- **Secures sessions** with JWT and secrets

It's the foundation for mobile access, CI/CD integration, and multi-device workflows.
</task_progress>
</write_to_file>