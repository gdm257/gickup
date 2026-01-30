# Research Log: WebDAV Destination

## Summary

WebDAV destination for Gickup enables pushing Git repository backups to WebDAV servers. The implementation follows the established storage backend pattern used by S3 and Azure Blob, leveraging Go's standard `net/http` package for HTTP operations. No external WebDAV client library is required; WebDAV servers respond to standard HTTP methods (PUT, MKCOL, PROPFIND, DELETE). The feature requires configuration struct definition, HTTP client with authentication support, directory upload functionality, and integration with main.go orchestration.

## Technology Alignment

### WebDAV Client Approach

**Decision**: Use Go's standard `net/http` package with WebDAV-specific HTTP methods.

**Rationale**:
- WebDAV is an extension of HTTP; servers respond to standard HTTP methods
- No dedicated WebDAV client library is needed for basic operations (PUT, MKCOL, PROPFIND, DELETE)
- Using standard http.Client provides full control over timeouts, TLS, and authentication
- The `golang.org/x/net/webdav` package is for server implementation, not client usage
- Consistent with project philosophy of minimal dependencies

**Key HTTP Methods for WebDAV**:
| Method | Purpose |
|--------|---------|
| PUT | Upload file content |
| MKCOL | Create collection (directory) |
| PROPFIND | Retrieve properties (used for existence checks) |
| DELETE | Remove resources |
| HEAD | Check resource existence without fetching content |

### Authentication Methods

| Method | Implementation | Use Case |
|--------|---------------|----------|
| Basic Auth | `Authorization: Basic <base64>` | Username/password authentication |
| Digest Auth | HTTP 401 challenge-response | Non-cleartext password transmission |

Client certificate authentication removed from scope.

### WebDAV Feature Compatibility

- **Chunked Transfer Encoding**: Supported by default in Go's http.Client
- **HTTP Keep-Alive**: Default behavior for http.Client with Transport configuration
- **Redirects**: http.Client follows redirects (301, 302, 307, 308) by default
- **WebDAV Headers**: Depth, If-Match, etc. set via request headers

## Architecture Pattern Evaluation

### Pattern: Storage Backend (Existing)

The WebDAV destination follows the established storage backend pattern:

```
webdav/
├── webdav.go          # UploadDirToWebDAV, DeleteObjectsNotInRepo, NewWebDAVClient
types/
└── types.go           # WebDAV struct definition added to Destination
main.go                # Orchestration loop (existing pattern)
```

**Existing Patterns to Follow**:
- `UploadDirToS3()` in s3/s3.go
- `UploadDirToBlobStorage()` in azureblob/azureblob.go
- Client initialization pattern in azureblob.NewAzureBlobClient()

**Components to Create/Modify**:

| Component | Type | Purpose |
|-----------|------|---------|
| `types.WebDAV` | Struct | Configuration (URL, credentials, path prefix) |
| `types.Destination.WebDAV` | Field | Configuration integration |
| `webdav/webdav.go` | Package | Upload logic, client creation |
| `main.go` | Modification | Orchestration loop integration |

## Integration Points

### Types Integration

The `WebDAV` struct must be added to:
- `types.Destination` struct (line 24-34)
- `Destination.Count()` method (line 37-47)

### Main.go Integration

Following the pattern at lines 190-292 (S3) and 295-383 (AzureBlob):
1. Iterate `conf.Destination.WebDAV`
2. Create WebDAV client with authentication
3. Call `webdav.UploadDirToWebDAV()`
4. Call `webdav.DeleteObjectsNotInRepo()`
5. Update Prometheus metrics

### Metrics Integration

Add new label value `"webdav"` to:
- `DestinationBackupsComplete` counter
- `RepoTime` gauge
- `RepoSuccess` gauge

## Risks and Mitigations

| Risk | Severity | Mitigation |
|------|----------|------------|
| Server-specific WebDAV quirks | Medium | Implement retry logic with exponential backoff; log warnings on non-critical failures |
| Large file uploads | Medium | Use streaming PUT with chunked transfer encoding |
| Network timeouts | Medium | Configure appropriate http.Client timeouts (30s connect, 5min request) |
| Authentication failures | High | Clear error messages; distinguish between auth and network errors |
| Partial upload failures | Medium | Verify with PROPFIND after PUT; retry up to 3 times |

## Key Design Decisions

1. **No External WebDAV Library**: Use standard net/http with custom WebDAV method handling
2. **Authentication**: Basic Auth and Digest Auth supported; no client certificate
3. **Path Handling**: Use filepath-based paths locally, convert to WebDAV paths (forward slashes) for remote
4. **Retry Strategy**: Exponential backoff (1s, 2s, 4s) with max 3 attempts for transient failures
5. **Structured Logging**: Use zerolog sub-logger with webdav stage identifier
6. **XML Parsing**: Use `encoding/xml` with struct-based unmarshaling for PROPFIND responses
   - Minimal struct definition to extract href and status fields
   - No external XML library needed; standard library sufficient for DAV responses

## Discovery Log

| Topic | Finding | Source | Implication |
|-------|---------|--------|-------------|
| WebDAV client library | No client library needed; HTTP is sufficient | webdav RFC 4918 | Simplifies dependency management |
| Go WebDAV package | golang.org/x/net/webdav is server-side only | pkg.go.dev | Use net/http instead |
| Authentication | Basic, Digest, Client Cert supported | HTTP standards | Implement all three methods |
| Directory operations | MKCOL for directories, PUT for files | WebDAV spec | Separate handling required |
| LFS support | LFS objects are files; same upload path | S3/AzureBlob patterns | No special handling needed |
