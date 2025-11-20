# Configuration

Wings uses a YAML configuration file located at `/etc/pterodactyl/config.yml`. This file contains various settings that control Wings behavior.

## File Management Configuration

### Search Recursion

Controls file search behavior and directory traversal limits.

| Field              | Type     | Default                                                          | Description                                              |
| ------------------ | -------- | ---------------------------------------------------------------- | -------------------------------------------------------- |
| blacklisted_dirs   | string[] | `["node_modules", ".git", ".wine", "appcache", "depotcache", "vendor"]` | Directories excluded from recursive search operations.   |
| max_recursion_depth| int      | `8`                                                              | Maximum depth for directory recursion during searches.   |

### Example

```yaml
Search:
  blacklisted_dirs:
    - node_modules
    - .git
    - .wine
    - appcache
    - depotcache
    - vendor
  max_recursion_depth: 8
```

### Chunked Upload

Settings for large file uploads using the chunked upload system.

| Field                    | Type  | Default | Description                                                           |
| ------------------------ | ----- | ------- | --------------------------------------------------------------------- |
| max_file_size            | int64 | `10240` | Maximum file size allowed for chunked uploads (in MB).                |
| max_concurrent_uploads   | int   | `5`     | Maximum number of simultaneous chunked upload sessions per server.    |
| chunk_expiration_hours   | int   | `24`    | Hours before incomplete upload sessions are automatically cleaned up. |

### Example

```yaml
ChunkedUpload:
  max_file_size: 10240
  max_concurrent_uploads: 5
  chunk_expiration_hours: 24
```

### Partial Edit

Configuration for partial file editing features that allow editing large files without loading them entirely.

| Field                 | Type  | Default | Description                                                        |
| --------------------- | ----- | ------- | ------------------------------------------------------------------ |
| max_lines_per_request | int   | `500`   | Maximum number of lines that can be fetched in a single request.   |
| max_line_size         | int64 | `10240` | Maximum size of a single line in bytes.                            |
| enable_line_caching   | bool  | `true`  | Whether to enable caching for line-based operations.               |

### Example

```yaml
PartialEdit:
  max_lines_per_request: 500
  max_line_size: 10240
  enable_line_caching: true
```

## Complete Configuration Example

```yaml
# ... other Wings configuration ...

Search:
  blacklisted_dirs:
    - node_modules
    - .git
    - .wine
    - appcache
    - depotcache
    - vendor
  max_recursion_depth: 8

ChunkedUpload:
  max_file_size: 10240
  max_concurrent_uploads: 5
  chunk_expiration_hours: 24

PartialEdit:
  max_lines_per_request: 500
  max_line_size: 10240
  enable_line_caching: true
```

## Notes

- All file size limits are specified in megabytes (MB) or bytes as indicated.
- The chunked upload system automatically cleans up expired sessions based on `chunk_expiration_hours`.
- Partial edit operations use buffered I/O for optimal performance on high-bandwidth networks.
- Search recursion blacklist is case-insensitive.
