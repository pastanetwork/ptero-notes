> [!NOTE]
> All of the endpoints listed in this document use custom authorization methods which are specified under each endpoint.

### `HEAD /download/backup`

Returns headers for a server backup file without downloading the content. Supports Range requests via the `Accept-Ranges: bytes` header. The `token` query parameter is required for this endpoint. It is a JWT token comprised of the following claims:

- `server_uuid`: the UUID of the server in the panel
- `backup_uuid`: the UUID of the backup to download
- `unique_id`: a randomly generated string

### Responses

| Code | Description                   |
| ---- | ----------------------------- |
| 200  | The request was successful.   |
| 404  | The backup was not found.     |

Sources:

- [router/router_download.go#L100](https://github.com/pastanetwork/wings/blob/develop/router/router_download.go#L100)
- [tokens/backup.go](https://github.com/pastanetwork/wings/blob/develop/router/tokens/backup.go)

### `GET /download/backup`

Returns the backup file for download. This endpoint now supports HTTP Range requests for resumable downloads and partial content delivery. The `token` query parameter is required for this endpoint. It is a JWT token comprised of the following claims:

- `server_uuid`: the UUID of the server in the panel
- `backup_uuid`: the UUID of the backup to download
- `unique_id`: a randomly generated string

### Headers

| Name  | Visibility | Description                                                                          |
| ----- | ---------- | ------------------------------------------------------------------------------------ |
| Range | optional   | HTTP Range header for partial content requests (e.g., "bytes=0-1023" or "bytes=500-"). |

### Responses

| Code | Description                                   |
| ---- | --------------------------------------------- |
| 200  | The full file was returned successfully.      |
| 206  | Partial content was returned (Range request). |
| 404  | The backup was not found.                     |
| 416  | The requested range is not satisfiable.       |

Sources:

- [router/router_download.go#L157](https://github.com/pastanetwork/wings/blob/develop/router/router_download.go#L157)
- [DownloadLinkService.php](https://github.com/pterodactyl/panel/blob/release/v1.11.3/app/Services/Backups/DownloadLinkService.php)
- [tokens/backup.go](https://github.com/pastanetwork/wings/blob/develop/router/tokens/backup.go)

### `HEAD /download/file`

Returns headers for a server file without downloading the content. Supports Range requests via the `Accept-Ranges: bytes` header. The `token` query parameter is required for this endpoint. It is a JWT token comprised of the following claims:

- `file_path`: the URL-decoded file path
- `server_uuid`: the UUID of the server in the panel
- `unique_id`: a randomly generated string

### Responses

| Code | Description                   |
| ---- | ----------------------------- |
| 200  | The request was successful.   |
| 404  | The file was not found.       |

Sources:

- [router/router_download.go#L260](https://github.com/pastanetwork/wings/blob/develop/router/router_download.go#L260)
- [tokens/file.go](https://github.com/pastanetwork/wings/blob/develop/router/tokens/file.go)

### `GET /download/file`

Returns a file from a server volume. This endpoint now supports HTTP Range requests for resumable downloads and partial content delivery. The `token` query parameter is required for this endpoint. It is a JWT token comprised of the following claims:

- `file_path`: the URL-decoded file path
- `server_uuid`: the UUID of the server in the panel
- `unique_id`: a randomly generated string

### Headers

| Name  | Visibility | Description                                                                          |
| ----- | ---------- | ------------------------------------------------------------------------------------ |
| Range | optional   | HTTP Range header for partial content requests (e.g., "bytes=0-1023" or "bytes=500-"). |

### Responses

| Code | Description                                   |
| ---- | --------------------------------------------- |
| 200  | The full file was returned successfully.      |
| 206  | Partial content was returned (Range request). |
| 404  | The file was not found.                       |
| 416  | The requested range is not satisfiable.       |

Sources:

- [router/router_download.go#L306](https://github.com/pastanetwork/wings/blob/develop/router/router_download.go#L306)
- [FileController.php#L77](https://github.com/pterodactyl/panel/blob/release/v1.11.3/app/Http/Controllers/Api/Client/Servers/FileController.php#L77)
- [tokens/file.go](https://github.com/pastanetwork/wings/blob/develop/router/tokens/file.go)

### `POST /upload/file`

Uploads the given file(s) to a server matching the identifiers set in the required `token` query parameter. It is a JWT token comprised of the following claims:

- `server_uuid`: the UUID of the server in the panel
- `user_uuid`: the UUID of the user in the panel
- `unique_id`: a randomly generated string

### Parameters

| Name      | Visibility | Description                                                    |
| --------- | ---------- | -------------------------------------------------------------- |
| directory | optional   | The directory path to upload the file(s) to (defaults to "/"). |
| token     | required   | The JWT token containing the required claims.                  |

Unlike other panel and Wings endpoints, this endpoint takes a multipart form-data body rather than a JSON string body, and should be used with the appropriate Content-Type header: "multipart/form-data" (plus additional multipart metadata).

The form-data files must be encoded under the "**files**" header, NOT "**file**". Each file must include:

- the file name; and
- the file contents

You can optionally include:

- the file type
- the file size

### Responses

| Code | Description                                                                                         |
| ---- | --------------------------------------------------------------------------------------------------- |
| 200  | The request was successful.                                                                         |
| 400  | The form-data could not be extracted from the request, or there were no files found in the request. |
| 404  | The server was not found.                                                                           |

Sources:

- [router/router_server_files.go#L573](https://github.com/pastanetwork/wings/blob/develop/router/router_server_files.go#L573)
- [FileUploadController.php#L40](https://github.com/pterodactyl/panel/blob/release/v1.11.3/app/Http/Controllers/Api/Client/Servers/FileUploadController.php#L40)
- [tokens/upload.go](https://github.com/pastanetwork/wings/blob/develop/router/tokens/upload.go)

### `GET /api/servers/:uuid/ws`

Initializes a websocket connection to the specified server (see [websocket](/wings/websocket.md) for more information).

### `POST /api/transfers`

> [!CAUTION]
> This endpoint is only meant to be used by other Wings instances, and can be potentially damaging. The documentation for this endpoint is for learning purposes only.

Handles the incoming transfer request from another Wings instance (the source node). In summary, the process goes as follows:

1. The request is authenticated using a bearer token in the `Authorization` header – this is a standard JWT token with no additional claims
2. The file(s) are extracted from the request form-data and written into the destination node's volumes
3. The checksum is computed and compared with the one received from the source node

Sources:

- [router/router_transfer.go#L29](https://github.com/pastanetwork/wings/blob/develop/router/router_transfer.go#L29)
