# Quick Start

Get LLM Proxy up and running in a few minutes.

## Prerequisites

- Go 1.24.5 or later
- API keys from your chosen providers (OpenAI, Anthropic, and/or Gemini)

## Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/MrCardist/llm-proxy.git
cd llm-proxy
```

### 2. Install Dependencies

```bash
# Using the Makefile
make install

# Or manually
go mod download
```

### 3. Set Up API Keys

Export your LLM provider API keys as environment variables:

```bash
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export GEMINI_API_KEY="..."
```

### 4. Start the Proxy

```bash
# Production build
make install build
make run

# Or development mode with auto-reload
make dev
```

The proxy starts on `http://localhost:9002`

## Making Your First Request

### Health Check

Verify the proxy is running and check provider status:

```bash
curl http://localhost:9002/health
```

### OpenAI Chat Completion

```bash
curl -X POST http://localhost:9002/openai/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_OPENAI_KEY" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [{"role": "user", "content": "Hello, world!"}],
    "max_tokens": 50
  }'
```

### Anthropic Messages

```bash
curl -X POST http://localhost:9002/anthropic/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_ANTHROPIC_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-3-sonnet-20240229",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### Google Gemini

```bash
curl -X POST http://localhost:9002/gemini/v1/models/gemini-pro:generateContent?key=YOUR_GEMINI_KEY \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "Hello!"}]}]
  }'
```

## Streaming Responses

Add `"stream": true` to your request to get streamed responses:

```bash
curl -X POST http://localhost:9002/openai/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_OPENAI_KEY" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": true
  }'
```

## Running Tests

Test that everything is configured correctly:

```bash
# All tests
make test-all

# Provider-specific tests
make test-openai
make test-anthropic
make test-gemini

# Health check tests only
make test-health
```

## Next Steps

- Read the [Configuration](./configuration.md) guide to customize the proxy
- Check the [API Reference](./api-reference.md) for all available endpoints
- See [Development](./development.md) for building custom providers or features
