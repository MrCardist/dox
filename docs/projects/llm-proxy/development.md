# Development

Information for developers who want to build, test, and extend LLM Proxy.

## Development Setup

### Prerequisites

- Go 1.24.5 or later
- Make (included on macOS/Linux; use MinGW on Windows)
- API keys for testing (optional, but recommended)

### Getting Started

```bash
# Clone the repository
git clone https://github.com/MrCardist/llm-proxy.git
cd llm-proxy

# Install dependencies
make install

# Build the binary
make build

# Run in development mode
make dev
```

## Available Make Commands

### Code Quality

```bash
make check      # Run all code quality checks
make fmt        # Format Go code
make vet        # Run go vet analysis
make lint       # Run golint
```

### Building

```bash
make build      # Build the binary (outputs to ./bin/llm-proxy)
make clean      # Clean build artifacts
make install    # Install Go dependencies
make help       # Show all available commands
```

### Running

```bash
make run        # Run the built binary
make dev        # Run in development mode with auto-reload
make stop       # Stop running proxy
```

### Testing

```bash
make test           # Run unit tests only
make test-all       # Run all tests (unit + integration)
make test-openai    # Run OpenAI integration tests
make test-anthropic # Run Anthropic integration tests
make test-gemini    # Run Gemini integration tests
make test-health    # Run health check tests
make env-check      # Verify environment variables
```

## Project Structure

```
llm-proxy/
├── cmd/
│   ├── llm-proxy/
│   │   └── main.go           # Server entry point
│   └── llm-proxy-keys/
│       └── main.go           # Key management CLI tool
├── internal/
│   ├── providers/
│   │   ├── provider.go       # Provider interface
│   │   ├── openai.go         # OpenAI implementation
│   │   ├── anthropic.go      # Anthropic implementation
│   │   └── gemini.go         # Gemini implementation
│   ├── middleware/
│   │   ├── logging.go        # Request logging
│   │   ├── cors.go           # CORS headers
│   │   └── streaming.go      # Streaming detection
│   ├── ratelimit/
│   │   └── memory.go         # Rate limiting logic
│   ├── cost/
│   │   └── tracker.go        # Cost tracking
│   └── config/
│       └── config.go         # Configuration loading
├── configs/
│   ├── base.yml              # Default configuration
│   ├── dev.yml               # Development overrides
│   ├── staging.yml           # Staging overrides
│   └── production.yml        # Production settings
├── tests/
│   ├── openai_test.go        # OpenAI tests
│   ├── anthropic_test.go     # Anthropic tests
│   ├── gemini_test.go        # Gemini tests
│   ├── common_test.go        # Shared tests
│   └── test_helpers.go       # Test utilities
└── Makefile                  # Build and development commands
```

## Setting Up for Testing

### Environment Variables

Export API keys for integration testing:

```bash
export OPENAI_API_KEY="your-openai-key"
export ANTHROPIC_API_KEY="your-anthropic-key"
export GEMINI_API_KEY="your-gemini-key"
```

### Run Tests

```bash
# All tests (requires all API keys)
make test-all

# Specific provider
make test-openai

# Check if environment is set up correctly
make env-check
```

### Test Structure

Each provider has integration tests in `*_test.go`:
- **Streaming tests** - Verify streaming responses work correctly
- **Non-streaming tests** - Verify regular completions work
- **Error handling tests** - Verify error responses are handled
- **Metadata parsing** - Verify token counts and response metadata

## Architecture

### Request Flow

1. **Client Request** → HTTP server receives request
2. **Middleware Stack**
   - CORS headers added
   - Request logging
   - Streaming detection
   - Rate limiting enforcement
   - Token estimation
3. **Provider Router** → Routes to appropriate provider (OpenAI, Anthropic, Gemini)
4. **Provider Handler** → Translates request to provider API format
5. **External API Call** → Calls the LLM provider
6. **Response Processing**
   - Parse response
   - Extract metadata (tokens, costs)
   - Handle streaming if needed
7. **Cost Tracking** → Log usage and costs
8. **Response Returned** → Client receives response

### Provider Interface

Each provider implements:

```go
type Provider interface {
    // Handle processes a request through the provider
    Handle(w http.ResponseWriter, r *http.Request)

    // IsStreamingRequest detects if request asks for streaming
    IsStreamingRequest(body []byte) bool

    // Health checks provider availability
    Health() (bool, string)
}
```

## Adding a New Provider

To add support for a new LLM provider:

### 1. Create Provider Implementation

Create `internal/providers/newprovider.go`:

```go
package providers

import (
    "net/http"
)

type NewProviderImpl struct {
    config *ProviderConfig
}

func (p *NewProviderImpl) Handle(w http.ResponseWriter, r *http.Request) {
    // Translate request
    // Call provider API
    // Handle streaming
    // Write response
}

func (p *NewProviderImpl) IsStreamingRequest(body []byte) bool {
    // Check if request asks for streaming
    return false
}

func (p *NewProviderImpl) Health() (bool, string) {
    // Check provider health
    return true, "healthy"
}
```

### 2. Create Tests

Create `tests/newprovider_test.go` with streaming and non-streaming tests.

### 3. Register Provider

Add to `cmd/llm-proxy/main.go`:

```go
router.HandleFunc("/newprovider/v1/{path:.*}", providers.NewProvider.Handle)
```

### 4. Add Configuration

Add to `configs/base.yml`:

```yaml
providers:
  newprovider:
    enabled: false
    api_endpoint: "https://api.newprovider.com"
    timeout: 30
```

## Debugging

### Enable Debug Logging

Set `logging.level: "debug"` in your config file or:

```bash
LOG_LEVEL=debug make dev
```

### Inspect Requests

The proxy logs all requests with:
- Request path and method
- Headers (sensitive headers are masked)
- Request size
- Streaming detection
- Provider routing

### Common Issues

**Provider returns 401**: Check that the API key is correct and has valid credentials

**Slow responses**: Check network connectivity to provider and provider status page

**Rate limiting too strict**: Adjust `chars_per_token` values based on your content type

## Benchmarking

The repository includes token estimation benchmarks:

```bash
python scripts/token_estimation.py
```

This generates a table of estimated tokens per 1k characters for each provider, helping tune rate limiting settings.

## Release Process

1. Update version in `cmd/llm-proxy/main.go`
2. Commit changes with version bump
3. Tag commit: `git tag v1.2.3`
4. Push: `git push origin main --tags`
5. CircleCI automatically builds and publishes release

## Deployment

### Docker

```bash
# Development
docker-compose up

# Production
docker build -f Dockerfile.prod -t llm-proxy:v1.0.0 .
docker push your-registry/llm-proxy:v1.0.0
```

### Kubernetes

The proxy is stateless and easily deployable to Kubernetes:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

See the repository for example K8s manifests.

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Write tests for your changes
4. Ensure `make check` passes
5. Submit a pull request

## Performance Targets

- **Request Processing**: <10ms overhead per request
- **Token Estimation**: <5% error margin
- **Cost Tracking**: <1% error margin
- **Throughput**: Support thousands of concurrent requests
