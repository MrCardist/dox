# LLM Proxy

A simple, Go-based reverse proxy for accessing multiple LLM providers through a unified gateway.

## What is LLM Proxy?

LLM Proxy is a lightweight gateway that sits between your applications and LLM providers (OpenAI, Anthropic, Google Gemini), providing:

- **Unified API** - Single endpoint for all LLM providers
- **Provider Routing** - Route requests to different providers based on your needs
- **Streaming Support** - Native support for streaming responses
- **Cost Tracking** - Monitor and control LLM spending
- **Rate Limiting** - Prevent abuse and enforce usage limits
- **API Key Management** - Secure handling of provider credentials

## Key Features

- ✅ **Multi-provider support**: OpenAI, Anthropic, Gemini
- ✅ **Streaming responses**: Efficient handling of streamed completions
- ✅ **Rate limiting**: Token-based and request-based limits
- ✅ **Cost tracking**: Real-time usage monitoring and billing
- ✅ **CORS support**: Browser-based application compatibility
- ✅ **Health checks**: Monitor provider status
- ✅ **Configurable**: Simple YAML configuration

## Use Cases

- **Unified LLM Access**: Applications don't need to know which provider to use
- **Cost Control**: Track and limit LLM spending per user/team/project
- **Provider Switching**: Change LLM providers without updating application code
- **Multi-cloud**: Use cloud providers alongside self-hosted models
- **Central Governance**: Manage API keys and access policies in one place

## Quick Navigation

- **[Quick Start](./quick-start.md)** - Get up and running in minutes
- **[API Reference](./api-reference.md)** - Available endpoints and request formats
- **[Configuration](./configuration.md)** - Setup and configuration options
- **[Development](./development.md)** - Building and extending the proxy

## Project Information

- **Repository**: [github.com/MrCardist/llm-proxy](https://github.com/MrCardist/llm-proxy)
- **Language**: Go 1.24.5+
- **License**: See repository for details
