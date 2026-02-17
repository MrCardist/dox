# Configuration

Configure LLM Proxy to suit your deployment environment and requirements.

## Configuration Files

The proxy uses YAML configuration files located in the `configs/` directory:

- `base.yml` - Default configuration
- `dev.yml` - Development environment overrides
- `staging.yml` - Staging environment overrides
- `production.yml` - Production environment overrides

The appropriate config file is selected based on the `ENVIRONMENT` variable (default: `dev`).

## Basic Configuration

### Server Settings

```yaml
server:
  port: 9002  # Port to listen on
```

### Environment Variables

Override configuration with environment variables:

```bash
# Port
export PORT=9002

# Providers
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export GEMINI_API_KEY="..."

# Environment selection
export ENVIRONMENT=production
```

## Features

### Rate Limiting

Enable token-based and request-based rate limiting:

```yaml
features:
  rate_limiting:
    enabled: true
    backend: "memory"  # Only 'memory' is available for single instances
    estimation:
      max_sample_bytes: 20000
      bytes_per_token: 4
      chars_per_token: 4
      provider_chars_per_token:
        openai: 5        # ~185-190 tokens per 1k chars
        anthropic: 3     # ~290-315 tokens per 1k chars
        gemini: 4        # Estimated
    limits:
      requests_per_minute: 100
      tokens_per_minute: 100000
```

**How it works:**
- Limits are per API key
- Token counts are estimated from request size
- Actual token usage is reconciled from provider responses
- First request in a window is always allowed even if it would exceed limits

### API Key Management

Secure management of provider API keys with proxy-specific keys:

```yaml
features:
  api_key_management:
    enabled: true
    table_name: "llm-proxy-api-keys"
    region: "us-west-2"
```

**Benefits:**
- Generate proxy-specific keys prefixed with `iw:`
- Store actual provider keys securely in DynamoDB
- Enable/disable keys without deleting them
- Set cost limits per key (future implementation)
- Track usage and metadata

See the API Key Management guide in the repository for detailed setup.

## Cost Tracking

Monitor and track LLM usage and costs:

```yaml
features:
  cost_tracking:
    enabled: true
    backend: "dynamodb"  # Options: dynamodb, datadog, file
```

### Available Backends

- **file**: Store costs locally (development only)
- **dynamodb**: Store costs in AWS DynamoDB
- **datadog**: Send costs to Datadog for monitoring

## Provider Configuration

### OpenAI

```yaml
providers:
  openai:
    enabled: true
    api_endpoint: "https://api.openai.com"
    timeout: 30  # seconds
```

### Anthropic

```yaml
providers:
  anthropic:
    enabled: true
    api_endpoint: "https://api.anthropic.com"
    timeout: 30  # seconds
```

### Google Gemini

```yaml
providers:
  gemini:
    enabled: true
    api_endpoint: "https://generativelanguage.googleapis.com"
    timeout: 30  # seconds
```

## Logging

Configure request/response logging:

```yaml
logging:
  level: "info"  # Options: debug, info, warn, error
  format: "json"  # Options: json, text
```

## Development Configuration Example

Full `dev.yml` example:

```yaml
server:
  port: 9002

providers:
  openai:
    enabled: true
  anthropic:
    enabled: true
  gemini:
    enabled: true

features:
  rate_limiting:
    enabled: false  # Disabled in dev
  api_key_management:
    enabled: false  # Disabled in dev

logging:
  level: "debug"
  format: "text"
```

## Production Configuration Example

Secure `production.yml` example:

```yaml
server:
  port: 9002

providers:
  openai:
    enabled: true
    timeout: 30
  anthropic:
    enabled: true
    timeout: 30
  gemini:
    enabled: true
    timeout: 30

features:
  rate_limiting:
    enabled: true
    backend: "memory"
    limits:
      requests_per_minute: 100
      tokens_per_minute: 500000
  api_key_management:
    enabled: true
    table_name: "llm-proxy-api-keys"
    region: "us-west-2"
  cost_tracking:
    enabled: true
    backend: "dynamodb"

logging:
  level: "info"
  format: "json"
```

## Deployment with Docker

The proxy includes Docker support for easy deployment:

```bash
# Build the image
docker build -f Dockerfile -t llm-proxy:latest .

# Run the container
docker run -p 9002:9002 \
  -e ENVIRONMENT=production \
  -e OPENAI_API_KEY="sk-..." \
  -e ANTHROPIC_API_KEY="sk-ant-..." \
  -e GEMINI_API_KEY="..." \
  llm-proxy:latest
```

### Docker Compose

For development with multiple services:

```bash
docker-compose up
```

See the repository for complete `docker-compose.yml` examples.

## Security Best Practices

1. **Use HTTPS** - Always use HTTPS in production
2. **Secure Credentials** - Store API keys in environment variables or secrets management systems
3. **API Key Rotation** - Regularly rotate both provider and proxy API keys
4. **Rate Limiting** - Enable rate limiting to prevent abuse
5. **Access Control** - Use AWS IAM to restrict DynamoDB table access
6. **Monitoring** - Monitor logs and set up alerts for failures

## Troubleshooting

### Health Check Failing

```bash
curl http://localhost:9002/health
```

If a provider shows unhealthy, check that the API key is set correctly and has valid credentials.

### Rate Limiting Too Strict

Adjust token estimation values in `chars_per_token` to match your usage patterns, or increase rate limit values.

### High Latency

The proxy overhead should be <10ms. If you see higher latency:
- Check network connectivity to providers
- Verify provider API isn't rate limiting you
- Check server CPU/memory availability
