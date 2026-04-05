# 002 - CLI Entry Point and Build System

## CLI Entry Point: `bin/openclaude`

The CLI is a simple Node.js script that:
1. Checks if the built version exists (`dist/cli.mjs`)
2. If yes, imports and runs it
3. If no, shows an error with build instructions

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
  console.error(`
  openclaude: dist/cli.mjs not found.
  
  Build first:
    bun run build
  
  Or run directly with Bun:
    bun run dev
  
  See README.md for setup instructions.
`)
  process.exit(1)
}
```

## Build System: Bun

The project uses **Bun** as its build tool (not webpack or esbuild).

### Build Scripts

```json
{
  "scripts": {
    "build": "bun run scripts/build.ts",
    "dev": "bun run build && node dist/cli.mjs",
    "start": "node dist/cli.mjs"
  }
}
```

### TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": ".",
    "paths": {
      "src/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Build Process

1. **TypeScript Compilation**: Converts `.tsx` and `.ts` files to JavaScript
2. **Bundling**: Bun bundles all dependencies into a single file
3. **Output**: Creates `dist/cli.mjs` (ES module)
4. **Dead Code Elimination**: Removes unused code via feature flags

### Feature Flags

The build system uses **feature flags** for conditional compilation:

```typescript
import { feature } from 'bun:bundle'

// Conditional imports based on features
const coordinatorModeModule = feature('COORDINATOR_MODE') 
  ? require('./coordinator/coordinatorMode.js')
  : null
```

This allows different builds for different environments (ant-only features, experimental features, etc.)

## Entry Points in Detail

### 1. CLI Entry (`bin/openclaude`)
- Loads the built application
- Minimal code - just a bootstrap

### 2. Main Application (`src/main.tsx`)
- The heart of the application
- Handles command-line arguments
- Initializes the REPL
- Manages sessions

### 3. Commands (`src/commands.ts`)
- Registers all slash commands
- Handles command routing
- Manages command metadata

## How Commands Work

1. User types `/help` in TUI
2. Command is parsed from input
3. `commands.ts` finds the matching command
4. Command handler is executed
5. Result is displayed

## Development Mode

```bash
# Run with hot reload
bun run dev

# Build and run
bun run build && node dist/cli.mjs

# Or just run the dev server
bun run dev:profile
```

## Production Build

```bash
# Full build
bun run build

# Type check only
bun run typecheck

# Run production
npm start
```

## Key Files

- `package.json` - Dependencies and scripts
- `tsconfig.json` - TypeScript configuration
- `scripts/build.ts` - Build script
- `bin/openclaude` - CLI entry
- `dist/cli.mjs` - Built output (generated)
</task_progress>
</write_to_file>