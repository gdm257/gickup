# Project Structure

## Organization Philosophy

**Domain-First Separation**: Each version control system gets its own package. Shared utilities live in dedicated directories. This enables independent development of VCS integrations and clear boundaries between concerns.

## Directory Patterns

### VCS Source/Destination Packages
**Location**: `/` (root-level directories)  
**Purpose**: Each VCS platform has a package handling both source (discovery) and destination (push) operations  
**Examples**: `github/`, `gitlab/`, `gitea/`, `gogs/`, `bitbucket/`, `onedev/`, `sourcehut/`

### Storage Backends
**Location**: `/` (root-level directories)  
**Purpose**: Destination handlers for non-VCS storage targets  
**Examples**: `s3/`, `azureblob/`, `local/`

### Utility Modules
**Location**: `/` (root-level directories)  
**Purpose**: Shared functionality used across VCS packages  
**Examples**: `local/` (Git operations), `logger/` (logging setup), `metrics/` (Prometheus, notifications), `zip/` (compression), `gitcmd/` (shell git commands)

### Types & Configuration
**Location**: `/types/`  
**Purpose**: Shared data structures for configuration and domain models  
**Example**: `types/types.go` contains `Conf`, `Repo`, `Source`, `Destination` structs

### Special-Purpose Packages
**Location**: `/`  
**Purpose**: Additional integrations that don't fit VCS or storage categories  
**Examples**: `whatever/` (generic handling)

## Naming Conventions

- **Files**: `lowercase.go` for implementation files, `*_test.go` for tests
- **Packages**: lowercase (e.g., `github`, `gitlab`, `local`)
- **Structs**: `PascalCase` (e.g., `Repo`, `Conf`, `GenRepo`)
- **Variables**: `camelCase` (e.g., `conf`, `repos`, `client`)
- **Constants**: `PascalCase` or `SCREAMING_SNAKE_CASE` for exported, `camelCase` for unexported

## Import Organization

```go
// Standard library first
import (
    "fmt"
    "os"
    "time"
)

// Then external dependencies
import (
    "github.com/cooperspencer/gickup/github"
    "github.com/rs/zerolog"
)

// Internal imports last
import (
    "github.com/cooperspencer/gickup/types"
)
```

## Code Organization Principles

- **Domain packages own their logic**: Each VCS package handles its own API interactions, not spread across files
- **Shared utilities are truly shared**: `local/` handles git clone/push operations used by all VCS destinations
- **Configuration is central**: `types/` defines all config structures; `main.go` handles orchestration
- **Co-located tests**: `*_test.go` files next to implementation, not in separate `tests/` directory

---

_Document patterns, not file trees. New files following patterns shouldn't require updates_
