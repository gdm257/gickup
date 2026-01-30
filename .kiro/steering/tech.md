# Technology Stack

## Architecture

CLI application with domain-driven separation of concerns. Each VCS platform has dedicated source/destination modules. Single-entry point (`main.go`) orchestrates discovery, backup, and metrics collection.

## Core Technologies

- **Language**: Go 1.24+
- **CLI Framework**: [kong](https://github.com/alecthomas/kong) - struct-based CLI parsing
- **Git Operations**: [go-git/v5](https://github.com/go-git/go-git) - pure Go Git implementation
- **Logging**: [zerolog](https://github.com/rs/zerolog) - structured, zero-allocation logging

## Key Libraries

- **YAML Config**: [goccy/go-yaml](https://github.com/goccy/go-yaml) - config file parsing
- **Scheduling**: [robfig/cron/v3](https://github.com/robfig/cron) - cron expression parsing
- **Metrics**: [prometheus/client_golang](https://github.com/prometheus/client_golang) - Prometheus instrumentation
- **Cloud Storage**: Azure SDK for Go, MinIO client for S3

## Development Standards

### Type Safety

- Go's static typing with struct-based configuration
- Custom `types/` package for shared data structures

### Code Quality

- Structured logging with `zerolog` throughout
- Config-driven behavior, no hardcoded business logic in CLI
- Consistent error handling with contextual logging

### Testing

- `_test.go` files co-located with implementation
- `go test ./...` for running tests

## Development Environment

### Required Tools

- Go 1.24+
- Docker (for containerized builds)

### Common Commands

```bash
# Development build
go build .

# Run with config
./gickup path-to-conf.yml

# Run tests
go test ./...

# Docker build
docker-compose build
```

## Key Technical Decisions

- **YAML Configuration**: Chosen for readability and widespread tooling support
- **Structured Logging**: Zerolog for machine-parseable logs with human-readable console output
- **Domain Separation**: Each VCS platform in its own package for independent maintenance
- **No Database**: Stateless operation; config file is the source of truth
- **Dry Run Support**: `-dryrun` flag for safe testing without mutations

---

_Document standards and patterns, not every dependency_
