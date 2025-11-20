### `GET /api/servers/:uuid`

Returns a server object by its UUID.

### Responses

| Code | Description                 |
| ---- | --------------------------- |
| 200  | The request was successful. |
| 404  | The server was not found.   |

<!-- ### Example Object -->

Sources:

- [router/router_server.go#L25](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L25)

### `DELETE /api/servers/:uuid`

Deletes a specified server by its UUID and removes all the files in the server's volume on the node. If configured, this endpoint will also automatically remove all backups associated with the server from the backup directory.

### Responses

| Code | Description               |
| ---- | ------------------------- |
| 204  | The request was accepted. |
| 404  | The server was not found. |

Sources:

- [router/router_server.go#L225](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L225)

### `GET /api/servers/:uuid/logs`

Returns an object containing a list of logs from the server's console.

### Parameters

| Name | Visibility | Description                                     |
| ---- | ---------- | ----------------------------------------------- |
| size | optional   | The number of logs to return (defaults to 100). |

### Responses

| Code | Description                    |
| ---- | ------------------------------ |
| 200  | The request was successful.    |
| 400  | The logs could not be fetched. |
| 404  | The server was not found.      |

Sources:

- [router/router_server.go#L30](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L30)

### `GET /api/servers/:uuid/install-logs`

Returns the installation logs for a server. The response contains only the script output section of the installation log file. If the script output section header is not found, the entire log file content is returned.

### Responses

| Code | Description                              |
| ---- | ---------------------------------------- |
| 200  | The request was successful.              |
| 404  | The server was not found.                |
| 500  | The installation log could not be read.  |

Sources:

- [router/router_server.go#L53](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L53)

### `POST /api/servers/:uuid/power`

Updates the power state of a server. The `action` field must be either "start", "stop", "restart", or "kill", and the `wait_seconds` field must be an integer value between 0 and 300.

### Body

| Field        | Visibility | Type    | Description                                                    |
| ------------ | ---------- | ------- | -------------------------------------------------------------- |
| action       | required   | string  | The power action to set.                                       |
| wait_seconds | optional   | integer | The amount of wait time to allow when setting the power state. |

### Responses

| Code | Description                              |
| ---- | ---------------------------------------- |
| 202  | The request was accepted.                |
| 400  | The request body could not be parsed.    |
| 404  | The server was not found.                |
| 422  | The request body could not be validated. |

Sources:

- [router/router_server.go#L86](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L86)

### `POST /api/servers/:uuid/commands`

Sends one or more commands to the server console.

### Body

| Field    | Visibility | Type            | Description                          |
| -------- | ---------- | --------------- | ------------------------------------ |
| commands | required   | array of string | The commands to the server commands. |

### Responses

| Code | Description                           |
| ---- | ------------------------------------- |
| 204  | The request was successful.           |
| 400  | The request body could not be parsed. |
| 404  | The server was not found.             |
| 502  | The server is not running.            |

Sources:

- [router/router_server.go#L141](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L141)

### `POST /api/servers/:uuid/sync`

Triggers the synchronize process of the server on the panel and the node.

### Responses

| Code | Description                                      |
| ---- | ------------------------------------------------ |
| 204  | The request was successful.                      |
| 400  | An error occurred while synchronizing the server. |
| 404  | The server was not found.                        |

Sources:

- [router/router_server.go#L175](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L175)

### `POST /api/servers/:uuid/install`

Triggers the install process of a server and returns no content. The installation process runs in the background.

### Responses

| Code | Description               |
| ---- | ------------------------- |
| 202  | The request was accepted. |
| 404  | The server was not found. |

Sources:

- [router/router_server.go#L186](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L186)

### `POST /api/servers/:uuid/reinstall`

Triggers the reinstall process for a server. The reinstallation process runs in the background.

### Responses

| Code | Description                                       |
| ---- | ------------------------------------------------- |
| 202  | The request was accepted.                         |
| 404  | The server was not found.                         |
| 409  | Another power action is currently being executed. |

Sources:

- [router/router_server.go#L205](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L205)

### `POST /api/servers/:uuid/ws/deny`

Adds the given JWT IDs (or "jti"s) to the server websocket's deny list, preventing tokens with the JTIs from connecting or interacting with the server websocket.

### Body

| Field | Visibility | Type            | Description             |
| ----- | ---------- | --------------- | ----------------------- |
| jtis  | required   | array of string | A list of JTIs to deny. |

### Responses

| Code | Description                           |
| ---- | ------------------------------------- |
| 204  | The request was successful.           |
| 400  | The request body could not be parsed. |
| 404  | The server was not found.             |

Sources:

- [router/router_server.go#L298](https://github.com/pastanetwork/wings/blob/develop/router/router_server.go#L298)
