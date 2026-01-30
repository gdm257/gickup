# Implementation Tasks: webdav-destination

## 1. Add WebDAV Configuration Structure
- [ ] 1.1 Add WebDAV struct to types/types.go
  - Define URL, Username, Password, Path, Structured, Zip, DateCreateDir fields with YAML tags
  - Add validation for URL format (http:// or https://) at startup
  - Support environment variable resolution for credentials
  - Match configuration pattern used by S3Repo and AzureBlob structs
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 6.1, 6.2, 6.4_

- [ ] 1.2 Add WebDAV field to Destination struct
  - Embed WebDAV struct in the main Destination configuration structure
  - Ensure YAML unmarshalling works with nested destination configurations
  - _Requirements: 1.1_

## 2. Implement WebDAV HTTP Client
- [ ] 2.1 Create webdav/webdav.go package structure
  - Set up package with proper imports (net/http, context, crypto/tls, encoding/xml)
  - Define WebDAVClient interface with Mkcol, Put, Propfind, Delete, Exists, Close methods
  - Create DavResponse struct for PROPFIND XML parsing
  - _Requirements: 2.1, 4.4, 4.5_

- [ ] 2.2 (P) Implement HTTP client connection handling
  - Configure http.Client with timeouts (30s connect, 5min request)
  - Enable HTTP keep-alive for connection efficiency
  - Implement redirect handling (301, 302, 307, 308) with security considerations
  - Support chunked transfer encoding when required by server
  - _Requirements: 4.1, 4.2, 4.3_

- [ ] 2.3 (P) Implement authentication methods
  - Add Basic Auth support with base64-encoded credentials in Authorization header
  - Implement Digest Auth with nonce handling from 401 challenge response
  - Add client certificate authentication with TLS certificate loading
  - Validate credentials and abort with clear error message on authentication failure
  - _Requirements: 2.2, 2.3, 2.4, 2.5_

- [ ] 2.4 Implement WebDAV HTTP methods
  - Implement Mkcol for directory creation with proper error handling
  - Implement Put for file uploads with streaming from disk
  - Implement Propfind with XML response parsing for resource enumeration
  - Implement Delete for resource removal with proper status code handling
  - Implement Exists for checking resource presence
  - Set appropriate HTTP headers (Depth, If-Match) for WebDAV operations
  - _Requirements: 3.1, 3.2, 4.4_

- [ ] 2.5 Implement retry logic with exponential backoff
  - Create retry wrapper function with configurable attempts (default 3)
  - Implement backoff strategy (1s, 2s, 4s) with jitter
  - Distinguish between transient errors (network timeout, 5xx) and permanent errors (401, 403)
  - Log each retry attempt with error details using zerolog
  - _Requirements: 3.5, 5.1, 5.2, 5.3_

## 3. Implement Repository Upload to WebDAV
- [ ] 3.1 Create UploadDirToWebDAV function signature
  - Define function parameters (context, directory path, repo, config, client)
  - Follow signature pattern established by UploadDirToS3 and UploadDirToBlobStorage
  - Set up zerolog sub-logger with stage=webdav for structured logging
  - _Requirements: 3.1, 5.4_

- [ ] 3.2 Implement directory walking and path conversion
  - Walk local directory tree similar to existing storage backends
  - Convert local filepath separators to WebDAV forward slashes
  - Apply path prefix from configuration to remote paths
  - Filter files appropriately (include LFS objects, exclude .git metadata)
  - _Requirements: 3.3, 3.4_

- [ ] 3.3 Implement file upload with verification
  - Create remote directory structure via MKCOL before uploading files
  - Upload files using HTTP PUT with appropriate content type headers
  - Verify successful upload by checking HTTP response status codes (201, 204)
  - Log progress for each file upload with name and size
  - _Requirements: 3.1, 3.2, 3.4_

- [ ] 3.4 Implement optional zip compression
  - Add zip compression before upload when Zip config option is enabled
  - Ensure compressed archive preserves directory structure
  - Set appropriate content type for zip files
  - _Requirements: 1.5_

## 4. Implement Cleanup Logic for WebDAV
- [ ] 4.1 Create DeleteObjectsNotInRepo function
  - Define function parameters matching UploadDirToWebDAV signature
  - Add dry-run mode support (skip actual deletions when enabled)
  - List remote resources via PROPFIND with Depth:1
  - Parse XML response to extract remote file paths
  - Compare remote paths with local directory contents
  - _Requirements: 3.4, 6.5_

- [ ] 4.2 Implement orphaned file deletion
  - Delete remote files that no longer exist in local repository
  - Skip deletion when dry-run mode is active (log instead)
  - Handle missing remote resources gracefully (410 Gone response)
  - Log deleted files for audit trail with Info level
  - Continue processing remaining files if deletion fails for one resource
  - Return error if any deletion fails (fail-fast behavior)
  - _Requirements: 3.4, 5.5_

## 5. Integrate WebDAV Destination into Orchestration
- [ ] 5.1 Add WebDAV destination handling in main.go
  - Import webdav package alongside other destination handlers
  - Add WebDAV case in destination orchestration switch statement
  - Call UploadDirToWebDAV with appropriate parameters from orchestration loop
  - Call DeleteObjectsNotInRepo after successful uploads
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5_

- [ ] 5.2 Add metrics integration for WebDAV operations
  - Add "webdav" label value to existing prometheus metrics
  - Increment counters for upload attempts, successes, and failures
  - Track upload duration for performance monitoring
  - _Requirements: 5.4_

- [ ] 5.3 Implement dry-run validation mode
  - Validate WebDAV configuration at startup without performing operations
  - Test connection to WebDAV server with authentication
  - Report validation results without making destructive changes
  - _Requirements: 6.5_

## 6. Testing for WebDAV Destination
- [ ] 6.1 Write unit tests for WebDAV struct validation
  - Test URL format validation (http:// and https://)
  - Test credential combination validation
  - Test path normalization
  - _Requirements: 6.1, 6.2, 6.3_

- [ ] 6.2 (P) Write unit tests for authentication methods
  - Test Basic Auth header generation
  - Test Digest Auth nonce handling
  - Test client certificate loading and presentation
  - _Requirements: 2.2, 2.3, 2.4_

- [ ] 6.3 (P) Write unit tests for retry logic
  - Test transient error detection and retry
  - Test permanent error immediate failure
  - Test exponential backoff timing
  - _Requirements: 3.5, 5.1, 5.2, 5.3_

- [ ] 6.4 Write integration tests with mock WebDAV server
  - Set up testdav or similar mock WebDAV server for testing
  - Test full upload flow with mock server
  - Test authentication scenarios (success and failure)
  - Test error handling (401, 403, 5xx responses)
  - Test cleanup of orphaned files
  - _Requirements: 2.1, 2.5, 3.1, 3.2, 3.4, 3.5, 5.5_

- [ ] 6.5* Write end-to-end tests with real WebDAV server
  - Configure integration tests with nginx dav or Apache WebDAV
  - Test full backup workflow from source discovery to destination upload
  - Verify metrics and logging output format
  - Test concurrent uploads performance
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 4.1, 4.2, 4.3, 5.4_

> *Optional test coverage task - deferred post-MVP baseline validation
