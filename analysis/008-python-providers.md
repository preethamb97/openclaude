# 008 - Python Providers

## Overview

OpenClaude includes **Python provider files** that enable integration with local AI models. These providers allow you to run AI models locally without relying on cloud APIs.

## Available Providers

### 1. Ollama Provider (`ollama_provider.py`)

**Purpose**: Provides integration with [Ollama](https://ollama.ai/), a local AI model runner.

**Supported Models**:
- Llama 3 (8B, 70B)
- Mistral
- CodeLlama
- Phi-3
- Qwen2
- DeepSeek Coder
- And many more...

**Key Features**:
- No API key required
- Runs locally on your machine
- OpenAI-compatible API
- Supports streaming responses
- Handles image inputs

**Configuration**:
```bash
# Set in .env file
PREFERRED_PROVIDER=ollama
OLLAMA_BASE_URL=http://localhost:11434
BIG_MODEL=codellama:34b
SMALL_MODEL=llama3:8b
```

**Usage Example**:
```python
from ollama_provider import ollama_chat, ollama_chat_stream

# Non-streaming
result = await ollama_chat(
    model="llama3:8b",
    messages=[{"role": "user", "content": "Hello!"}],
    system="You are helpful."
)

# Streaming
async for chunk in ollama_chat_stream(model="llama3:8b", messages=messages):
    print(chunk, end="")
```

### 2. Atomic Chat Provider (`atomic_chat_provider.py`)

**Purpose**: Provides integration with [Atomic Chat](https://atomic.chat/), an Apple Silicon-optimized local model runner.

**Supported Models**:
- Any model compatible with Atomic Chat
- Optimized for Apple M1/M2/M3 chips

**Key Features**:
- Apple Silicon native
- OpenAI-compatible API
- Fast inference
- No API key required
- Supports streaming

**Configuration**:
```bash
# Set in .env file
PREFERRED_PROVIDER=atomic-chat
ATOMIC_CHAT_BASE_URL=http://127.0.0.1:1337
```

**Usage Example**:
```python
from atomic_chat_provider import atomic_chat, atomic_chat_stream

# Non-streaming
result = await atomic_chat(
    model="llama3",
    messages=[{"role": "user", "content": "Hello!"}],
    system="You are helpful."
)

# Streaming
async for chunk in atomic_chat_stream(model="llama3", messages=messages):
    print(chunk, end="")
```

### 3. Smart Router (`smart_router.py`)

**Purpose**: Intelligently routes requests to the best available provider based on latency, cost, and health.

**Key Features**:
- Automatic provider selection
- Health monitoring
- Latency tracking
- Cost optimization
- Fallback support

**Configuration**:
```bash
# Set in .env file
ROUTER_MODE=smart          # or: fixed
ROUTER_STRATEGY=latency    # or: cost, balanced
ROUTER_FALLBACK=true       # auto-retry on failure
```

**Usage Example**:
```python
from smart_router import SmartRouter

# Initialize router
router = SmartRouter()
await router.initialize()

# Route a request
result = await router.route(
    messages=[{"role": "user", "content": "Hello!"}],
    claude_model="claude-sonnet"
)

# Record result for learning
await router.record_result(
    provider_name=result["provider"],
    success=True,
    duration_ms=150
)
```

## Provider Architecture

### Common Structure

All providers follow a similar pattern:

```python
async def provider_chat(
    model: str,
    messages: list[dict],
    system: str | None = None,
    max_tokens: int = 4096,
    temperature: float = 1.0,
) -> dict:
    """
    Send a chat request to the provider.
    
    Returns:
        dict with keys:
        - id: Message ID
        - type: "message"
        - role: "assistant"
        - content: List of content blocks
        - model: Model name
        - stop_reason: "end_turn"
        - usage: Token usage
    """
    pass

async def provider_chat_stream(
    model: str,
    messages: list[dict],
    system: str | None = None,
    max_tokens: int = 4096,
    temperature: float = 1.0,
) -> AsyncIterator[str]:
    """
    Stream a chat response from the provider.
    
    Yields:
        Server-Sent Events (SSE) formatted strings
    """
    pass
```

### Health Checks

Each provider includes a health check function:

```python
async def check_ollama_running() -> bool:
    """Check if Ollama is running and accessible."""
    try:
        async with httpx.AsyncClient(timeout=3.0) as client:
            resp = await client.get(f"{OLLAMA_BASE_URL}/api/tags")
            return resp.status_code == 200
    except Exception:
        return False
```

### Model Listing

Providers can list available models:

```python
async def list_ollama_models() -> list[str]:
    """List all available Ollama models."""
    try:
        async with httpx.AsyncClient(timeout=5.0) as client:
            resp = await client.get(f"{OLLAMA_BASE_URL}/api/tags")
            resp.raise_for_status()
            data = resp.json()
            return [m["name"] for m in data.get("models", [])]
    except Exception as e:
        logger.warning(f"Could not list Ollama models: {e}")
        return []
```

## Message Format Conversion

### Anthropic to Ollama

```python
def anthropic_to_ollama_messages(messages: list[dict]) -> list[dict]:
    """Convert Anthropic message format to Ollama format."""
    ollama_messages = []
    
    for msg in messages:
        role = msg.get("role", "user")
        content = msg.get("content", "")
        
        if isinstance(content, str):
            ollama_messages.append({"role": role, "content": content})
        elif isinstance(content, list):
            # Handle image blocks
            text_parts = []
            image_parts = []
            
            for block in content:
                if block.get("type") == "text":
                    text_parts.append(block.get("text", ""))
                elif block.get("type") == "image":
                    image_data = extract_image_data(block)
                    if image_data:
                        image_parts.append(image_data)
            
            ollama_message = {"role": role, "content": "\n".join(text_parts)}
            if image_parts:
                ollama_message["images"] = image_parts
            ollama_messages.append(ollama_message)
    
    return ollama_messages
```

## Streaming Response Format

All providers use Server-Sent Events (SSE) format:

```
event: message_start
data: {"type": "message_start", "message": {"id": "msg_123", "role": "assistant", "content": []}}

event: content_block_start
data: {"type": "content_block_start", "index": 0, "content_block": {"type": "text", "text": ""}}

event: content_block_delta
data: {"type": "content_block_delta", "index": 0, "delta": {"type": "text_delta", "text": "Hello"}}

event: content_block_stop
data: {"type": "content_block_stop", "index": 0}

event: message_delta
data: {"type": "message_delta", "delta": {"stop_reason": "end_turn"}, "usage": {"output_tokens": 10}}

event: message_stop
data: {"type": "message_stop"}
```

## Integration with TypeScript

The Python providers are called from TypeScript using child processes:

```typescript
// In src/services/api/provider.ts
import { exec } from 'child_process'

async function callPythonProvider(
  provider: string,
  messages: Message[]
): Promise<string> {
  return new Promise((resolve, reject) => {
    exec(
      `python3 -c "import ${provider}; print(${provider}.chat(${JSON.stringify(messages)}))"`,
      (error, stdout, stderr) => {
        if (error) reject(error)
        else resolve(stdout)
      }
    )
  })
}
```

## Error Handling

Each provider includes robust error handling:

```python
async def ollama_chat(...) -> dict:
    try:
        async with httpx.AsyncClient(timeout=120.0) as client:
            resp = await client.post(
                f"{OLLAMA_BASE_URL}/api/chat",
                json=payload
            )
            resp.raise_for_status()
            data = resp.json()
            
            return {
                "id": f"msg_ollama_{data.get('created_at', 'unknown')}",
                "type": "message",
                "role": "assistant",
                "content": [{"type": "text", "text": assistant_text}],
                "model": model,
                "stop_reason": "end_turn",
                "usage": {
                    "input_tokens": data.get("prompt_eval_count", 0),
                    "output_tokens": data.get("eval_count", 0),
                },
            }
    except httpx.TimeoutException:
        raise TimeoutError("Ollama request timed out")
    except httpx.HTTPStatusError as e:
        raise RuntimeError(f"Ollama API error: {e.response.status_code}")
    except Exception as e:
        raise RuntimeError(f"Ollama error: {str(e)}")
```

## Performance Considerations

### Timeouts

- Health checks: 3 seconds
- Model listing: 5 seconds
- Chat requests: 120 seconds (2 minutes)

### Connection Pooling

```python
# Reuse HTTP connections
async with httpx.AsyncClient(timeout=120.0) as client:
    # Multiple requests can use the same client
    resp1 = await client.post(...)
    resp2 = await client.post(...)
```

### Async Operations

All provider functions are async to avoid blocking:

```python
async def ollama_chat(...):
    # Non-blocking HTTP request
    async with httpx.AsyncClient() as client:
        resp = await client.post(...)
```

## Use Cases

### 1. Offline Development
```bash
# Run Ollama locally
ollama run llama3:8b

# Use with OpenClaude
openclaude --provider ollama
```

### 2. Privacy-Sensitive Work
```bash
# Keep data on your machine
openclaude --provider atomic-chat
```

### 3. Cost Optimization
```bash
# Use free local models
openclaude --provider ollama --model llama3:8b
```

### 4. Custom Models
```bash
# Use fine-tuned models
ollama create mymodel -f Modelfile
openclaude --provider ollama --model mymodel
```

## Configuration Examples

### .env File

```bash
# Provider selection
PREFERRED_PROVIDER=ollama

# Ollama settings
OLLAMA_BASE_URL=http://localhost:11434
BIG_MODEL=codellama:34b
SMALL_MODEL=llama3:8b

# Atomic Chat settings
ATOMIC_CHAT_BASE_URL=http://127.0.0.1:1337

# Router settings
ROUTER_MODE=smart
ROUTER_STRATEGY=balanced
ROUTER_FALLBACK=true
```

### Command Line

```bash
# Use Ollama
openclaude --provider ollama --model llama3:8b

# Use Atomic Chat
openclaude --provider atomic-chat

# Use smart routing
openclaude --provider smart
```

## Troubleshooting

### Ollama Not Running

```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# Start Ollama
ollama serve

# Pull a model
ollama pull llama3:8b
```

### Atomic Chat Not Running

```bash
# Check if Atomic Chat is running
curl http://127.0.0.1:1337/v1/models

# Start Atomic Chat
atomic-chat serve
```

### Connection Errors

```python
# Increase timeout
async with httpx.AsyncClient(timeout=300.0) as client:
    resp = await client.post(...)
```

## Summary

The Python providers:
- **Enable local AI** without cloud APIs
- **Support multiple providers** (Ollama, Atomic Chat)
- **Use smart routing** for optimal performance
- **Handle streaming** for real-time responses
- **Are easy to configure** via .env files
- **Integrate seamlessly** with TypeScript code

They're perfect for offline development, privacy-sensitive work, and cost optimization.
</task_progress>
</write_to_file>