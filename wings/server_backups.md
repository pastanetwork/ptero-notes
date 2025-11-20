### `POST /api/servers/:uuid/backup`

Creates a backup of the server's volume in the node to the specified backup store and returns no content. The backup process runs in the background.

### Body

| Field   | Visibility | Type   | Description                                   |
| ------- | ---------- | ------ | --------------------------------------------- |
| adapter | required   | string | The name of the backup adapter to use.        |
| uuid    | required   | string | The UUID of the backup in the panel.          |
| ignore  | required   | string | A string containing the ignored files format. |

### Responses

| Code | Description               |
| ---- | ------------------------- |
| 202  | The request was accepted. |
| 404  | The server was not found. |

Sources:

- [router/router_server_backup.go#L20](https://github.com/pastanetwork/wings/blob/develop/router/router_server_backup.go#L20)

### `POST /api/servers/:uuid/backup/:backup/restore`

Restores a backup on the server. The restoration process runs in the background. This endpoint returns immediately after validating the request and starting the restore operation.

### Body

| Field              | Visibility             | Type    | Description                                                                           |
| ------------------ | ---------------------- | ------- | ------------------------------------------------------------------------------------- |
| adapter            | required               | string  | The name of the backup adapter ("wings" or "s3").                                     |
| truncate_directory | required               | boolean | Whether the existing server volume should be truncated before the backup is restored. |
| download_url       | only with "s3" adapter | string  | The download URL to the S3 storage bucket.                                            |

### Responses

| Code | Description                                                                               |
| ---- | ----------------------------------------------------------------------------------------- |
| 202  | The request was accepted.                                                                 |
| 400  | The request body could not be parsed, the S3 download URL was not present or was invalid. |
| 404  | The server or backup was not found.                                                       |

Sources:

- [router/router_server_backup.go#L77](https://github.com/pastanetwork/wings/blob/develop/router/router_server_backup.go#L77)

### `DELETE /api/servers/:uuid/backup/:backup`

Deletes a specified backup from a server.

### Responses

| Code | Description                         |
| ---- | ----------------------------------- |
| 204  | The request was accepted.           |
| 404  | The server or backup was not found. |

Sources:

- [router/router_server_backup.go#L189](https://github.com/pastanetwork/wings/blob/develop/router/router_server_backup.go#L189)

### `DELETE /api/servers/:uuid/backup/delete-all`

Deletes all backups for a specified server. This removes all backup files from the server's backup directory.

### Responses

| Code | Description                 |
| ---- | --------------------------- |
| 204  | The request was accepted.   |
| 404  | The server was not found.   |
| 500  | Failed to delete backups.   |

Sources:

- [router/router_server.go#L314](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L314)
