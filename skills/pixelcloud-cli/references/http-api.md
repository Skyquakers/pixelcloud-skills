# PixelCloud HTTP API

Use the PixelCloud HTTP API to inspect and control existing game servers from
scripts, automations, and AI agents.

## Base URL

```text
https://api.edgerunners.cn/v1
```

All request and response bodies use JSON over HTTPS.

## Create an API key

1. Sign in to the [PixelCloud API Keys dashboard](https://edgerunners.cn/dashboard/api-keys).
2. Choose **Create API key**.
3. Give the key a recognizable name and select every server it may access.
4. Copy the secret immediately. PixelCloud shows it only once and stores only
   its SHA-256 hash.

Keys begin with `egrs_sk_`. Keep the secret in a password manager, deployment
secret, or environment variable. Never commit it to a repository or include it
in logs.

## Authentication

Send the key as an HTTP Bearer token:

```http
Authorization: Bearer egrs_sk_...
```

For shell examples, export the key without putting it in command history:

```bash
read -rsp "PixelCloud API key: " PIXELCLOUD_API_KEY
export PIXELCLOUD_API_KEY
```

Then make a request:

```bash
curl https://api.edgerunners.cn/v1/servers \
  -H "Authorization: Bearer $PIXELCLOUD_API_KEY"
```

## Server scope

Each key has an explicit server allowlist. `GET /servers` returns only selected
servers. A request for an unselected, deleted, or other user's server returns
the same `404 server_not_permitted` response, so the API does not reveal whether
that server exists.

Editing a key changes its allowlist without changing the secret. Rotating a key
preserves its allowlist but immediately invalidates the old secret. Deleting a
key immediately rejects future requests made with it.

## Server object

The API intentionally returns a safe operational summary. It does not expose
environment variables, credentials, Kubernetes Pod names, internal agent
ports, or the account owner ID.

```json
{
  "id": 42,
  "name": "Minecraft friends",
  "description": "Survival world",
  "status": "Running",
  "lifecycleAction": null,
  "ports": [{ "number": 25565, "protocol": "tcp" }],
  "game": { "id": 1, "name": "Minecraft" },
  "image": { "id": 10, "name": "Paper 1.21" },
  "hardware": {
    "id": 3,
    "name": "Standard",
    "cpu": 4,
    "memoryGiB": 8,
    "storageGiB": 40
  },
  "createdAt": "2026-08-01T00:00:00.000Z",
  "updatedAt": "2026-08-29T00:00:00.000Z",
  "lastStoppedAt": null,
  "willStopAt": null
}
```

`status` reflects the observed game Pod state. `lifecycleAction` is `start`,
`stop`, or `restart` while an accepted action is still being reconciled.

## Endpoints

### List permitted servers

```http
GET /servers
```

```bash
curl https://api.edgerunners.cn/v1/servers \
  -H "Authorization: Bearer $PIXELCLOUD_API_KEY"
```

Response:

```json
{
  "servers": []
}
```

### Get one server

```http
GET /servers/{serverId}
```

```bash
curl https://api.edgerunners.cn/v1/servers/42 \
  -H "Authorization: Bearer $PIXELCLOUD_API_KEY"
```

The response is one server object.

### Start a server

```http
POST /servers/{serverId}/start
```

```bash
curl -X POST https://api.edgerunners.cn/v1/servers/42/start \
  -H "Authorization: Bearer $PIXELCLOUD_API_KEY"
```

The request fails with `400` when the server is already active or the account
does not have enough balance to start it.

### Stop a server

```http
POST /servers/{serverId}/stop
```

```bash
curl -X POST https://api.edgerunners.cn/v1/servers/42/stop \
  -H "Authorization: Bearer $PIXELCLOUD_API_KEY"
```

### Restart a server

```http
POST /servers/{serverId}/restart
```

```bash
curl -X POST https://api.edgerunners.cn/v1/servers/42/restart \
  -H "Authorization: Bearer $PIXELCLOUD_API_KEY"
```

Use this graceful restart for normal automation.

### Force restart a stuck server

```http
POST /servers/{serverId}/force-restart
```

```bash
curl -X POST https://api.edgerunners.cn/v1/servers/42/force-restart \
  -H "Authorization: Bearer $PIXELCLOUD_API_KEY"
```

Force restart deletes the current game Pod so the platform can recreate it.
Use it only when a graceful restart cannot recover the server. A server with an
accepted lifecycle action returns `409` instead of running both operations.

Lifecycle endpoints return `200` after the action is accepted. Reconciliation
continues asynchronously. Poll `GET /servers/{serverId}` and inspect `status`
and `lifecycleAction` for progress.

## Error responses

Errors contain a human-readable `message`. Public API authentication and scope
errors also include a stable `code`.

```json
{
  "code": "invalid_api_key",
  "message": "A valid PixelCloud API key is required"
}
```

| Status | Code or meaning                          | When it happens                                                                       |
| ------ | ---------------------------------------- | ------------------------------------------------------------------------------------- |
| `400`  | `invalid_server_id` or action validation | The ID or requested lifecycle transition is invalid.                                  |
| `401`  | `invalid_api_key`                        | The key is missing, malformed, rotated, deleted, or otherwise invalid.                |
| `404`  | `server_not_permitted`                   | The server is absent or outside this key's allowlist.                                 |
| `409`  | Lifecycle conflict                       | Another lifecycle action is already in progress.                                      |
| `500`  | Internal error                           | PixelCloud could not complete the request. Retry only idempotent reads automatically. |

## JavaScript example

```js
const baseUrl = "https://api.edgerunners.cn/v1";
const apiKey = process.env.PIXELCLOUD_API_KEY;

if (!apiKey) throw new Error("PIXELCLOUD_API_KEY is required");

const response = await fetch(`${baseUrl}/servers`, {
  headers: { Authorization: `Bearer ${apiKey}` },
});

if (!response.ok) {
  const error = await response.json();
  throw new Error(`PixelCloud ${response.status}: ${error.message}`);
}

const { servers } = await response.json();
console.log(servers.map(({ id, name, status }) => ({ id, name, status })));
```

Do not include the key in query parameters. URLs are more likely than headers
to be retained by browser history, proxies, access logs, and monitoring tools.
