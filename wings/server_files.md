### `GET /api/servers/:uuid/files/contents`

Returns the file contents of a specified file in the server's volume. The `Content-Length` header will be set to the size of the file according to its stat information, and the `X-Mime-Type` header will reflect the file's MIME type. If the `download` query parameter is included (e.g. `?download=true`), the `Content-Disposition` header will be set, and the `Content-Type` header will be set to "application/octet-stream".

### Parameters

| Name     | Visibility | Description                                                       |
| -------- | ---------- | ----------------------------------------------------------------- |
| download | optional   | Whether the file content should be returned as a download stream. |
| file     | required   | The URL-encoded file path.                                        |

### Responses

| Code | Description                        |
| ---- | ---------------------------------- |
| 200  | The request was successful.        |
| 400  | The file type could not be opened. |
| 404  | The file was not found.            |

Sources:

- [router/router_server_files.go#L33](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L33)

### `GET /api/servers/:uuid/files/list-directory`

Returns a list of file objects in the directory of the server's volume.

### Parameters

| Name      | Visibility | Description                     |
| --------- | ---------- | ------------------------------- |
| directory | required   | The URL-encoded directory path. |

### Responses

| Code | Description                      |
| ---- | -------------------------------- |
| 200  | The request was successful.      |
| 400  | The directory could not be read. |
| 404  | The directory was not found.     |

Sources:

- [router/router_server_files.go#L92](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L92)

### `PUT /api/servers/:uuid/files/rename`

Renames one or more files in the server's volume.

### Body

| Name  | Visibility | Type   | Description                                  |
| ----- | ---------- | ------ | -------------------------------------------- |
| files | required   | object | An object containing the from-to name pairs. |
| root  | required   | string | The root directory of the files.             |

### Example Body

```json
{
  "root": "/",
  "files": [
    {
      "from": "bungee.jar",
      "to": "bungeecord.jar"
    }
  ]
}
```

### Responses

| Code | Description                                                                      |
| ---- | -------------------------------------------------------------------------------- |
| 204  | The request was accepted.                                                        |
| 400  | The request body could not be parsed, or one of the destinations already exists. |
| 422  | The from-to name pairs in the request was empty.                                 |

Sources:

- [router/router_server_files.go#L116](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L116)

### `POST /api/servers/:uuid/files/copy`

Creates a copy of a file or directory at the given location in the server volume. For directories, performs a recursive copy of all contents. Symlinks are skipped for security.

### Body

| Field    | Visibility | Type   | Description                           |
| -------- | ---------- | ------ | ------------------------------------- |
| location | required   | string | The file or directory path (not encoded). |

### Responses

| Code | Description                                                                                               |
| ---- | --------------------------------------------------------------------------------------------------------- |
| 204  | The request was accepted.                                                                                 |
| 400  | The request body could not be parsed, the file is ignored by the system, or the file could not be copied. |

Sources:

- [router/router_server_files.go#L184](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L184)

### `POST /api/servers/:uuid/files/write`

Writes the given file content into a file in the server's volume. If the file does not exist, it is created instead. Unlike other panel and Wings endpoints, this endpoint takes a plain text body rather than a JSON string body, and should be used with the appropriate `Content-Type` header: "text/plain". No additional metadata is used by this endpoint, so if you want the file's MIME type and other attributes to be used, you should request the [upload file](./public.md#get-uploadfile) endpoint with the full file.

### Parameters

| Name | Visibility | Description                |
| ---- | ---------- | -------------------------- |
| file | required   | The URL-encoded file path. |

### Responses

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 204  | The request was accepted.                                                                           |
| 400  | The file path is ignored, or the file path conflicts with an existing directory with the same name. |

Sources:

- [router/router_server_files.go#L253](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L253)

### `POST /api/servers/:uuid/files/create-directory`

Creates a directory at a given path in the server's volume.

### Body

| Field | Visibility | Type   | Description                                            |
| ----- | ---------- | ------ | ------------------------------------------------------ |
| name  | required   | string | The name of the directory.                             |
| path  | required   | string | The root path that the directory should be created in. |

### Responses

| Code | Description                                                        |
| ---- | ------------------------------------------------------------------ |
| 204  | The request was accepted.                                          |
| 400  | The request body could not be parsed, or the root path is invalid. |

Sources:

- [router/router_server_files.go#L393](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L393)

### `POST /api/servers/:uuid/files/delete`

Deletes one or more files from the server's volume. Note that if one of the files fail to be deleted, the request is aborted.

### Body

| Field | Visibility | Type            | Description                      |
| ----- | ---------- | --------------- | -------------------------------- |
| files | required   | array of string | A list of file names.            |
| root  | required   | string          | The root directory of the files. |

### Responses

| Code | Description                                   |
| ---- | --------------------------------------------- |
| 204  | The request was accepted.                     |
| 400  | The request body could not be parsed.         |
| 422  | The files list in the request body was empty. |

Sources:

- [router/router_server_files.go#L208](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L208)

### `POST /api/servers/:uuid/files/compress`

Compresses one or more files into a single archive in the server's volume. By default runs synchronously (foreground=true). When foreground=false, returns immediately with a task identifier and the compression runs in the background. Listen to WebSocket events for completion.

### Body

| Field      | Visibility | Type            | Description                                              |
| ---------- | ---------- | --------------- | -------------------------------------------------------- |
| files      | required   | array of string | A list of file names.                                    |
| root       | required   | string          | The root directory of the files.                         |
| foreground | optional   | boolean         | If true (default), wait for completion. If false, async. |

### Responses

| Code | Description                                        |
| ---- | -------------------------------------------------- |
| 200  | The request was successful (foreground mode).      |
| 202  | Task accepted, returns identifier (async mode).    |
| 400  | The request body could not be parsed.              |
| 422  | The files list in the request body was empty.      |

### Example Response (foreground=false)

```json
{
  "identifier": "550e8400-e29b-41d4-a716-446655440000"
}
```

### WebSocket Events (async mode)

- `archive started` - When compression begins
- `archive completed` - When compression finishes successfully (includes result file info)
- `archive failed` - If compression fails (includes error message)

Sources:

- [router/router_server_files.go#L420](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L420)
- [router/archiver/archiver.go](https://github.com/pastanetwork/wings/blob/develop/router/archiver/archiver.go)

### `POST /api/servers/:uuid/files/decompress`

Decompresses an archive file into the server's volume. By default runs synchronously (foreground=true). When foreground=false, returns immediately with a task identifier and the decompression runs in the background. Listen to WebSocket events for completion.

### Body

| Field      | Visibility | Type    | Description                                              |
| ---------- | ---------- | ------- | -------------------------------------------------------- |
| file       | required   | string  | The name of the archive file.                            |
| root       | required   | string  | The root directory of the file.                          |
| foreground | optional   | boolean | If true (default), wait for completion. If false, async. |

### Responses

| Code | Description                                                                                                                   |
| ---- | ----------------------------------------------------------------------------------------------------------------------------- |
| 204  | The request was accepted (foreground mode).                                                                                   |
| 202  | Task accepted, returns identifier (async mode).                                                                               |
| 400  | The request body could not be parsed, the archive could not be processed, or the file is currently in use by another process. |

### Example Response (foreground=false)

```json
{
  "identifier": "550e8400-e29b-41d4-a716-446655440000"
}
```

### WebSocket Events (async mode)

- `archive started` - When decompression begins
- `archive completed` - When decompression finishes successfully
- `archive failed` - If decompression fails (includes error message)

Sources:

- [router/router_server_files.go#L461](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L461)
- [router/archiver/archiver.go](https://github.com/pastanetwork/wings/blob/develop/router/archiver/archiver.go)

### `GET /api/servers/:uuid/files/archive-tasks`

Returns a list of all archive tasks (compress/decompress) for the server. Tasks are automatically cleaned up 5 minutes after completion.

### Responses

| Code | Description                 |
| ---- | --------------------------- |
| 200  | The request was successful. |

### Example Response

```json
[
  {
    "identifier": "550e8400-e29b-41d4-a716-446655440000",
    "server_id": "abc123-def456",
    "type": "compress",
    "status": "processing",
    "root_path": "/",
    "files": ["logs", "config"],
    "started_at": "2025-01-20T10:30:00Z"
  },
  {
    "identifier": "660e8400-e29b-41d4-a716-446655440001",
    "server_id": "abc123-def456",
    "type": "decompress",
    "status": "completed",
    "root_path": "/mods",
    "file": "modpack.tar.gz",
    "started_at": "2025-01-20T10:25:00Z",
    "finished_at": "2025-01-20T10:28:00Z"
  }
]
```

Sources:

- [router/router_server_files.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go)
- [router/archiver/archiver.go](https://github.com/pastanetwork/wings/blob/develop/router/archiver/archiver.go)

### `GET /api/servers/:uuid/files/archive-status/:identifier`

Returns the status of a specific archive task by its identifier.

### Parameters

| Name       | Visibility | Description                                               |
| ---------- | ---------- | --------------------------------------------------------- |
| identifier | required   | The task identifier returned by compress/decompress.      |

### Responses

| Code | Description                 |
| ---- | --------------------------- |
| 200  | The request was successful. |
| 404  | The task was not found.     |

### Example Response

```json
{
  "identifier": "550e8400-e29b-41d4-a716-446655440000",
  "server_id": "abc123-def456",
  "type": "compress",
  "status": "completed",
  "root_path": "/",
  "files": ["logs", "config"],
  "result_file": "archive-2025-01-20T103000.tar.gz",
  "started_at": "2025-01-20T10:30:00Z",
  "finished_at": "2025-01-20T10:32:00Z"
}
```

### Task Status Values

| Status     | Description                          |
| ---------- | ------------------------------------ |
| pending    | Task created but not yet started.    |
| processing | Task is currently running.           |
| completed  | Task finished successfully.          |
| failed     | Task failed (check `error` field).   |

Sources:

- [router/router_server_files.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go)
- [router/archiver/archiver.go](https://github.com/pastanetwork/wings/blob/develop/router/archiver/archiver.go)

### `POST /api/servers/:uuid/files/chmod`

Changes the permissions of a file in the server's volume.

### Body

| Field | Visibility | Type   | Description                      |
| ----- | ---------- | ------ | -------------------------------- |
| files | required   | array  | A list of file-mode pairs.       |
| root  | required   | string | The root directory of the files. |

### Example Body

```json
{
  "root": "/",
  "files": [
    {
      "file": "bungeecord.jar",
      "mode": 493
    }
  ]
}
```

### Responses

| Code | Description                                                         |
| ---- | ------------------------------------------------------------------- |
| 204  | The request was accepted.                                           |
| 400  | The request body could not be parsed, or a file's mode was invalid. |
| 422  | The file-mode pairs in the request body was empty.                  |

Sources:

- [router/router_server_files.go#L509](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L509)

### `GET /api/servers/:uuid/files/pull`

Returns an object containing a list of download objects for in-progress downloads.

Sources:

- [router/router_server_files.go#L291](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L291)

### `POST /api/servers/:uuid/files/pull`

Pulls a file from a remote location and downloads it into the server's volume. All servers are limited to 3 simultaneous downloads at a time, if a server reaches this limit, the request is cancelled immediately. Note that synchronous downloads (ones using the `foreground` field in the request) will keep the request connection open until the download is complete or fails. If the `foreground` field is not specified, the endpoint will return the download's identifier and will continue the download in the background.

### Body

> [!WARNING]
> The `directory` field in the request body is deprecated, use the `root` field instead.

| Field      | Visibility | Type    | Description                                             |
| ---------- | ---------- | ------- | ------------------------------------------------------- |
| url        | required   | string  | The URL of the file to download.                        |
| directory  | optional   | string  | The root directory to download the file into.           |
| file_name  | optional   | string  | The name to save the file as.                           |
| foreground | optional   | boolean | Whether the request should be downloaded synchronously. |
| root       | required   | string  | The root path to download the file to.                  |
| use_header | optional   | boolean | Use the remote file header for file information.        |

### Responses

| Code | Description                                                                                                         |
| ---- | ------------------------------------------------------------------------------------------------------------------- |
| 200  | The request was successful.                                                                                         |
| 204  | The request was accepted.                                                                                           |
| 400  | The request body could not be parsed, the remote URL could not be parsed, or the server reached the download limit. |

Sources:

- [router/router_server_files.go#L299](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L299)
- [downloader/downloader.go](https://github.com/pastanetwork/wings/blob/develop/router/downloader/downloader.go)

### `DELETE /api/servers/:uuid/files/pull/:id`

Deletes an in-progress remote file download.

Sources:

- [router/router_server_files.go#L384](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L384)

### `GET /api/servers/:uuid/files/search`

Performs a recursive search for files matching a given pattern in the server's volume. Supports wildcards, substring matching, and extension-based searches.

### Parameters

| Name      | Visibility | Description                                                    |
| --------- | ---------- | -------------------------------------------------------------- |
| directory | optional   | The URL-encoded directory path to search in (defaults to "/"). |
| pattern   | required   | The search pattern (minimum 3 characters).                     |

### Responses

| Code | Description                                                           |
| ---- | --------------------------------------------------------------------- |
| 200  | The request was successful.                                           |
| 400  | The pattern is too short (less than 3 characters) or directory error. |

### Example Response

```json
[
  {
    "name": "config/settings.json",
    "created": "2025-01-15T10:30:00Z",
    "modified": "2025-01-18T14:22:00Z",
    "mode": "-rw-r--r--",
    "mode_bits": "0644",
    "size": 2048,
    "directory": false,
    "file": true,
    "symlink": false,
    "mime": "application/json"
  }
]
```

Sources:

- [router/router_server_files_search.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_search.go)

### `GET /api/servers/:uuid/files/stat`

Returns file metadata without reading the file contents. Useful for getting file information quickly without the overhead of downloading content.

### Parameters

| Name | Visibility | Description                |
| ---- | ---------- | -------------------------- |
| file | required   | The URL-encoded file path. |

### Responses

| Code | Description                 |
| ---- | --------------------------- |
| 200  | The request was successful. |
| 404  | The file was not found.     |

### Example Response

```json
{
  "name": "server.jar",
  "created": "2025-01-15T10:00:00Z",
  "modified": "2025-01-18T14:30:00Z",
  "mode": "-rw-r--r--",
  "mode_bits": "0644",
  "size": 52428800,
  "directory": false,
  "file": true,
  "symlink": false,
  "mime": "application/java-archive"
}
```

Sources:

- [router/router_server_files.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go)

### `POST /api/servers/:uuid/files/touch`

Creates an empty file at the specified path in the server's volume.

### Body

| Field | Visibility | Type   | Description              |
| ----- | ---------- | ------ | ------------------------ |
| file  | required   | string | The file path to create. |

### Example Body

```json
{
  "file": "/logs/new.log"
}
```

### Responses

| Code | Description                                                        |
| ---- | ------------------------------------------------------------------ |
| 204  | The request was accepted.                                          |
| 400  | The request body could not be parsed, or the file path is invalid. |

Sources:

- [router/router_server_files.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go)

### `GET /api/servers/:uuid/files/disk-usage`

Returns disk usage and quota information for the server. Results are cached for 30 seconds to improve performance.

### Responses

| Code | Description                 |
| ---- | --------------------------- |
| 200  | The request was successful. |

### Example Response

```json
{
  "current_bytes": 1073741824,
  "limit_bytes": 10737418240
}
```

Sources:

- [router/router_server_files.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go)

### `GET /api/servers/:uuid/files/lines`

Returns specific lines from a file without loading the entire file into memory. Useful for viewing portions of large log files.

### Parameters

| Name  | Visibility | Description                              |
| ----- | ---------- | ---------------------------------------- |
| file  | required   | The URL-encoded file path.               |
| start | required   | The starting line number (1-indexed).    |
| end   | required   | The ending line number (inclusive).      |

### Responses

| Code | Description                                                                      |
| ---- | -------------------------------------------------------------------------------- |
| 200  | The request was successful.                                                      |
| 400  | The file path is missing, or too many lines requested (maximum 500 lines).       |
| 404  | The file was not found.                                                          |

### Example Response

```json
{
  "file": "/logs/server.log",
  "start_line": 1000,
  "end_line": 1100,
  "total_lines": 50000,
  "lines": [
    {"number": 1000, "content": "[INFO] Server started"},
    {"number": 1001, "content": "[WARN] Memory usage high"}
  ]
}
```

Sources:

- [router/router_server_files_partial.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_partial.go)

### `GET /api/servers/:uuid/files/metadata`

Returns detailed metadata about a file including encoding, line endings, and line count.

### Parameters

| Name | Visibility | Description                |
| ---- | ---------- | -------------------------- |
| file | required   | The URL-encoded file path. |

### Responses

| Code | Description                 |
| ---- | --------------------------- |
| 200  | The request was successful. |
| 404  | The file was not found.     |

### Example Response

```json
{
  "file": "/logs/big.log",
  "size": 524288000,
  "total_lines": 5000000,
  "encoding": "utf-8",
  "line_ending": "LF",
  "created": "2025-01-15T10:00:00Z",
  "modified": "2025-01-19T14:30:00Z"
}
```

Sources:

- [router/router_server_files_partial.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_partial.go)

### `POST /api/servers/:uuid/files/replace-line`

Replaces a specific line in a file with new content.

### Body

| Field       | Visibility | Description                         |
| ----------- | ---------- | ----------------------------------- |
| file        | required   | The file path (not encoded).        |
| line_number | required   | The line number to replace (1-indexed). |
| content     | required   | The new content for the line.       |

### Example Body

```json
{
  "file": "/config/server.properties",
  "line_number": 42,
  "content": "max-players=100"
}
```

### Responses

| Code | Description                                                  |
| ---- | ------------------------------------------------------------ |
| 204  | The request was accepted.                                    |
| 400  | The request body could not be parsed, or line number is out of range. |
| 404  | The file was not found.                                      |

Sources:

- [router/router_server_files_partial.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_partial.go)

### `POST /api/servers/:uuid/files/insert-lines`

Inserts one or more lines at a specific position in a file.

### Body

| Field       | Visibility | Description                                   |
| ----------- | ---------- | --------------------------------------------- |
| file        | required   | The file path (not encoded).                  |
| line_number | required   | The line number to insert before (1-indexed). |
| lines       | required   | An array of lines to insert.                  |

### Example Body

```json
{
  "file": "/config/whitelist.txt",
  "line_number": 10,
  "lines": ["player1", "player2", "player3"]
}
```

### Responses

| Code | Description                                                  |
| ---- | ------------------------------------------------------------ |
| 204  | The request was accepted.                                    |
| 400  | The request body could not be parsed, or line number is out of range. |
| 404  | The file was not found.                                      |

Sources:

- [router/router_server_files_partial.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_partial.go)

### `POST /api/servers/:uuid/files/delete-lines`

Deletes a range of lines from a file.

### Body

| Field      | Visibility | Description                         |
| ---------- | ---------- | ----------------------------------- |
| file       | required   | The file path (not encoded).        |
| start_line | required   | The starting line number (1-indexed). |
| end_line   | required   | The ending line number (inclusive). |

### Example Body

```json
{
  "file": "/logs/old.log",
  "start_line": 1,
  "end_line": 1000
}
```

### Responses

| Code | Description                                                  |
| ---- | ------------------------------------------------------------ |
| 204  | The request was accepted.                                    |
| 400  | The request body could not be parsed, or line range is invalid. |
| 404  | The file was not found.                                      |

Sources:

- [router/router_server_files_partial.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_partial.go)

### `POST /api/servers/:uuid/files/upload/init`

Initializes a chunked upload session for large files. Returns an upload ID that should be used for subsequent chunk uploads.

### Body

| Field      | Visibility | Description                                |
| ---------- | ---------- | ------------------------------------------ |
| path       | required   | The destination file path (not encoded).   |
| file_size  | required   | The total size of the file in bytes.       |
| chunk_size | required   | The size of each chunk in bytes.           |

### Example Body

```json
{
  "path": "/mods/big-modpack.zip",
  "file_size": 524288000,
  "chunk_size": 10485760
}
```

### Responses

| Code | Description                                                                                     |
| ---- | ----------------------------------------------------------------------------------------------- |
| 200  | The request was successful.                                                                     |
| 400  | The request body could not be parsed, or file size exceeds maximum allowed size.                |
| 409  | Maximum concurrent uploads reached for this server.                                             |

### Example Response

```json
{
  "upload_id": "550e8400-e29b-41d4-a716-446655440000",
  "total_chunks": 50,
  "chunk_size": 10485760
}
```

Sources:

- [router/router_server_files_upload.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_upload.go)

### `POST /api/servers/:uuid/files/upload/chunk`

Uploads a single chunk of data for an active upload session. Chunks can be uploaded in any order.

### Headers

| Name            | Visibility | Description                           |
| --------------- | ---------- | ------------------------------------- |
| Content-Type    | required   | Must be "application/octet-stream".   |
| X-Upload-Id     | required   | The upload session ID.                |
| X-Chunk-Index   | required   | The chunk index (0-based).            |
| X-Total-Chunks  | required   | The total number of chunks.           |

### Body

Binary data of the chunk.

### Responses

| Code | Description                                                           |
| ---- | --------------------------------------------------------------------- |
| 200  | The request was successful.                                           |
| 400  | Missing required headers or invalid chunk index.                      |
| 404  | Upload session not found.                                             |
| 403  | Upload session does not belong to this server.                        |

### Example Response

```json
{
  "chunk_index": 0,
  "status": "received",
  "bytes_received": 10485760,
  "total_bytes": 524288000,
  "progress": 2.0
}
```

Sources:

- [router/router_server_files_upload.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_upload.go)

### `POST /api/servers/:uuid/files/upload/finalize`

Finalizes an upload session by assembling all received chunks into the final file. Verifies all chunks are present and computes a SHA256 checksum.

### Body

| Field     | Visibility | Description            |
| --------- | ---------- | ---------------------- |
| upload_id | required   | The upload session ID. |

### Example Body

```json
{
  "upload_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Responses

| Code | Description                                                  |
| ---- | ------------------------------------------------------------ |
| 200  | The request was successful.                                  |
| 400  | Not all chunks have been received.                           |
| 404  | Upload session not found.                                    |
| 403  | Upload session does not belong to this server.               |

### Example Response

```json
{
  "status": "completed",
  "file_path": "/mods/big-modpack.zip",
  "file_size": 524288000,
  "checksum": "sha256:a3b5c7d9e1f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8f0a2b4"
}
```

Sources:

- [router/router_server_files_upload.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_upload.go)

### `DELETE /api/servers/:uuid/files/upload/:upload_id`

Cancels an active upload session and removes all stored chunks.

### Responses

| Code | Description                                        |
| ---- | -------------------------------------------------- |
| 204  | The request was accepted.                          |
| 403  | Upload session does not belong to this server.     |

Sources:

- [router/router_server_files_upload.go](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files_upload.go)
