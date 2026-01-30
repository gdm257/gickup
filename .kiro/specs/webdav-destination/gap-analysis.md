# Gap Analysis: webdav-destination

## Analysis Summary

- **Feature Scope**: Add WebDAV as a new destination handler following existing S3/AzureBlob patterns
- **Key Gaps**: No existing WebDAV package, no WebDAV config struct, no integration in orchestration (main.go) or metrics
- **Implementation Approach**: New package creation (Option B) - clean separation matching storage backend pattern
- **Effort & Risk**: Medium effort, Medium risk - uses familiar Go HTTP libraries but requires new authentication and XML parsing logic
- **Research Needed**: WebDAV server compatibility testing (nginx dav, Apache, Nextcloud, ownCloud)

---

## 1. Current State Investigation

### Existing Assets
| Asset | Location | Status | Notes |
|-------|----------|--------|-------|
| Destination struct | types/types.go:24-34 | Missing | No WebDAV field |
| S3Repo struct | types/types.go:548-560 | Reference | Pattern for config fields |
| AzureBlob struct | types/types.go:577-587 | Reference | Pattern for config fields |
| S3 handler | s3/s3.go | Reference | UploadDirToS3, DeleteObjectsNotInRepo |
| AzureBlob handler | azureblob/azureblob.go | Reference | UploadDirToBlobStorage, DeleteObjectsNotInRepo |
| Orchestration | main.go:277-382 | Missing | No WebDAV case in switch |
| Metrics | metrics/prometheus/prometheus.go | Missing | No "webdav" label value |

### Integration Surfaces
- **Configuration**: YAML unmarshalling into WebDAV struct
- **Orchestration**: main.go loop iterating destinations
- **Metrics**: prometheus labels for destination type
- **Logging**: zerolog sub-logger with stage="webdav"

---

## 2. Requirements Feasibility Analysis

### Technical Needs from Requirements
| Requirement | Technical Need | Status | Gap |
|-------------|----------------|--------|-----|
| 1.1-1.5 | WebDAV struct with URL, credentials, path | Missing | Create new struct |
| 2.1-2.5 | HTTP client with auth (Basic, Digest, Cert) | Missing | New implementation |
| 3.1-3.5 | UploadDirToWebDAV with MKCOL, PUT | Missing | New implementation |
| 4.1-4.5 | WebDAV headers, redirects, chunked encoding | Missing | New implementation |
| 5.1-5.5 | Structured logging, retry logic | Partial | Extend zerolog pattern |
| 6.1-6.5 | Configuration validation, dry-run | Missing | New validation logic |

### Complexity Signals
- **Authentication**: Multiple auth methods (Basic, Digest, Cert) - moderate complexity
- **HTTP Operations**: Standard WebDAV methods - straightforward
- **XML Parsing**: PROPFIND response parsing - simple struct unmarshalling
- **Retry Logic**: Exponential backoff - reusable pattern from S3/AzureBlob

---

## 3. Implementation Approach Options

### Option A: Extend Existing Components
**Rationale**: Not applicable - no existing WebDAV components to extend.

### Option B: Create New Components (Recommended)
**Rationale**: Clean separation following established storage backend pattern.

| New Component | Purpose | Lines (est.) |
|---------------|---------|--------------|
| types.WebDAV struct | Configuration | ~15 |
| webdav/webdav.go | HTTP client + upload logic | ~300 |
| webdav/webdav_test.go | Unit tests | ~100 |
| main.go modifications | Integration | ~30 |

**Pros**:
- Clean domain separation
- Matches S3/AzureBlob pattern
- Easy to test in isolation
- No impact on existing functionality

**Cons**:
- More files to navigate
- Requires careful interface design

### Option C: Hybrid Approach
**Rationale**: Not needed - feature is self-contained.

---

## 4. Gap Identification

### Missing Capabilities
| Gap | Impact | Mitigation |
|-----|--------|------------|
| No WebDAV struct in types | Cannot parse config | Create struct following S3Repo pattern |
| No webdav package | No implementation | Create new package |
| No WebDAV integration in main.go | Feature not callable | Add case in destination loop |
| No "webdav" metric label | Metrics gap | Add to prometheus labels |

### Unknowns (Research Needed)
| Item | Research Question | Priority |
|------|-------------------|----------|
| WebDAV server compatibility | Which servers support which features? | High |
| Digest auth implementation | Nonce handling edge cases? | Medium |
| XML parsing edge cases | Namespaces, malformed responses? | Medium |

---

## 5. Effort & Risk Assessment

### Effort: M (3-7 days)
- Configuration struct: 0.5 day
- HTTP client and auth: 2 days
- Upload/cleanup logic: 1.5 days
- Integration and testing: 2 days
- Buffer for unknowns: 1 day

### Risk: Medium
- **Known patterns**: Follows S3/AzureBlob, low architectural risk
- **New tech**: WebDAV HTTP methods, moderate complexity
- **Integration**: Standard Go net/http, well-documented
- **Testing**: Requires WebDAV server for integration tests

---

## 6. Recommendations for Design Phase

### Preferred Approach: Option B (New Components)
- Create webdav/ package with all WebDAV-specific logic
- Extend types.Destination with WebDAV field
- Add WebDAV case in main.go orchestration loop
- Extend prometheus labels for "webdav" destination type

### Key Design Decisions
1. **HTTP client**: Use net/http standard library (no external deps)
2. **XML parsing**: Use encoding/xml with struct-based unmarshalling
3. **Retry logic**: Implement with exponential backoff (3 attempts)
4. **Dry-run support**: Validate config without performing operations

### Research Items to Carry Forward
- WebDAV server compatibility matrix (nginx, Apache, Nextcloud, ownCloud)
- Digest auth nonce handling edge cases
- PROPFIND XML response variations

---

## 7. Requirements-to-Asset Map

| Requirement | Asset | Status |
|-------------|-------|--------|
| 1.1-1.5 | types.WebDAV struct | Missing |
| 2.1-2.5 | webdav.WebDAVClient interface | Missing |
| 3.1-3.5 | webdav.UploadDirToWebDAV | Missing |
| 4.1-4.5 | webdav.HTTP methods (Mkcol, Put, etc.) | Missing |
| 5.1-5.5 | webdav.Retry wrapper, zerolog | Partial |
| 6.1-6.5 | Validation logic in types | Missing |

---

## 8. Tasks Alignment Verification

| Task | Requirements Covered | Validation |
|------|----------------------|------------|
| 1.1 | 1.1, 1.2, 1.3, 1.4, 1.5, 6.1, 6.2, 6.4 | ✅ Complete |
| 1.2 | 1.1 | ✅ Complete |
| 2.1 | 2.1, 4.4, 4.5 | ✅ Complete |
| 2.2 | 4.1, 4.2, 4.3 | ✅ Complete |
| 2.3 | 2.2, 2.3, 2.4, 2.5 | ✅ Complete |
| 2.4 | 3.1, 3.2, 4.4 | ✅ Complete |
| 2.5 | 3.5, 5.1, 5.2, 5.3 | ✅ Complete |
| 3.1 | 3.1, 5.4 | ✅ Complete |
| 3.2 | 3.3, 3.4 | ✅ Complete |
| 3.3 | 3.1, 3.2, 3.4 | ✅ Complete |
| 3.4 | 1.5 | ✅ Complete |
| 4.1 | 3.4, 6.5 | ✅ Complete (updated) |
| 4.2 | 3.4, 5.5 | ✅ Complete (updated) |
| 5.1 | 3.1, 3.2, 3.3, 3.4, 3.5 | ✅ Complete |
| 5.2 | 5.4 | ✅ Complete |
| 5.3 | 6.5 | ✅ Complete |
| 6.1-6.5 | 6.1-6.5, 2.1-2.5, 3.1-3.5, 4.1-4.3, 5.1-5.5 | ✅ Complete |

**Gap Analysis Status**: ✅ All requirements covered by tasks
