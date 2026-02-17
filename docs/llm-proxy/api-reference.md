# API Reference

Complete reference for all LLM Proxy endpoints and request formats.

## General Endpoints

### Health Check

Check the status of the proxy and all configured providers.

```
GET /health
```

**Response:**
```json
{
  "status": "healthy",
  "proxy_version": "1.0.0",
  "providers": {
    "openai": "healthy",
    "anthropic": "healthy",
    "gemini": "healthy"
  }
}
```

## OpenAI Endpoints

All OpenAI API endpoints are available through the `/openai` prefix.

### Chat Completions

```
POST /openai/v1/chat/completions
```

**Headers:**
- `Authorization: Bearer YOUR_API_KEY` (required)
- `Content-Type: application/json` (required)

**Request:**
```json
{
  "model": "gpt-3.5-turbo",
  "messages": [
    {"role": "user", "content": "Hello!"}
  ],
  "temperature": 0.7,
  "max_tokens": 100,
  "stream": false
}
```

**Streaming:** Add `"stream": true` to receive tokens as they're generated.

**Response:**
```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "gpt-3.5-turbo",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 15,
    "total_tokens": 25
  }
}
```

### Completions

```
POST /openai/v1/completions
```

**Headers:**
- `Authorization: Bearer YOUR_API_KEY` (required)
- `Content-Type: application/json` (required)

**Request:**
```json
{
  "model": "text-davinci-003",
  "prompt": "Translate this to French: Hello",
  "max_tokens": 50,
  "stream": false
}
```

### Other OpenAI Endpoints

All other OpenAI API endpoints are supported:

```
* /openai/v1/*
```

## Anthropic Endpoints

All Anthropic API endpoints are available through the `/anthropic` prefix.

### Messages

```
POST /anthropic/v1/messages
```

**Headers:**
- `x-api-key: YOUR_API_KEY` (required)
- `anthropic-version: 2023-06-01` (required)
- `Content-Type: application/json` (required)

**Request:**
```json
{
  "model": "claude-3-sonnet-20240229",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": "Hello!"
    }
  ],
  "stream": false
}
```

**Streaming:** Add `"stream": true` to receive tokens as they're generated.

**Response:**
```json
{
  "id": "msg_...",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! How can I help you today?"
    }
  ],
  "model": "claude-3-sonnet-20240229",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 10,
    "output_tokens": 15
  }
}
```

### Other Anthropic Endpoints

All other Anthropic API endpoints are supported:

```
* /anthropic/v1/*
```

## Google Gemini Endpoints

All Gemini API endpoints are available through the `/gemini` prefix.

### Generate Content

```
POST /gemini/v1/models/{model}:generateContent?key=YOUR_API_KEY
```

**Query Parameters:**
- `key: YOUR_GEMINI_KEY` (required)

**Request:**
```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "Hello!"
        }
      ]
    }
  ]
}
```

**Response:**
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "Hello! How can I assist you today?"
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP"
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 3,
    "candidatesTokenCount": 10,
    "totalTokenCount": 13
  }
}
```

### Stream Generate Content

For streaming responses:

```
POST /gemini/v1/models/{model}:streamGenerateContent?key=YOUR_API_KEY
```

### Other Gemini Endpoints

All other Gemini API endpoints are supported:

```
* /gemini/v1/*
```

## Rate Limiting Headers

When rate limiting is enabled, responses include:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1234567890
```

If you exceed the rate limit, you'll receive:

```
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Remaining: 0
```

## Error Responses

### Invalid API Key

```json
{
  "error": "Invalid API key: API key not found"
}
```

Status: `401 Unauthorized`

### Rate Limit Exceeded

```json
{
  "error": "Rate limit exceeded"
}
```

Status: `429 Too Many Requests`

### Provider Error

Errors from the LLM provider are passed through with the original status code.

### CORS Support

The proxy includes CORS headers for browser-based requests:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization, x-api-key
```
