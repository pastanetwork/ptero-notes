# Wings API

The Wings API is used for communicating to the panel and other Wings instances on the panel.

> [!IMPORTANT]
> The Wings API does not have any control over the literal server instances in the panel, only the Docker containers for said servers and the server volumes. Use the [application API](/pterodactyl/application/README.md) for server control.

## Documentation

- [Configuration](./configuration.md) - Wings configuration file settings
- [Server Management](./server.md) - Server lifecycle and control endpoints
- [Server Files](./server_files.md) - File management, uploads, and editing
- [Server Backups](./server_backups.md) - Backup creation and restoration
- [Server Transfer](./server_transfer.md) - Server transfers between nodes
- [System](./system.md) - System information and node management
- [Public Endpoints](./public.md) - Publicly accessible download/upload endpoints
- [Websockets](./websocket.md) - Real-time server event streaming
- [JWT Specifications](./jwt.md) - JWT authentication details

## Access

The `Authorization` header must be present in the requests and must be prefixed with "Bearer " followed by the node's token (also known as the daemon token or master daemon key). The token can be obtained in the configuration page of a node in the admin panel, or from the [node configuration endpoint](/pterodactyl/application/nodes.md#get-nodesidconfiguration) in the application API. The `Accept` header must be set to "application/json" for requests, and the `Content-Type` must be set appropriately when making `POST`, `PATCH` or `PUT` requests.
