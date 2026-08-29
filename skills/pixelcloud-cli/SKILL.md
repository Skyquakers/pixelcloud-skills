---
name: pixelcloud-cli
description: PixelCloud CLI and HTTP API skill for managing multiplayer game servers. Gives your AI agent the ability to create servers with the CLI and inspect, start, stop, or restart selected servers through a scoped API key.
---

# PixelCloud CLI and HTTP API

Manage game servers on PixelCloud through the `pixel` CLI or the versioned HTTP
API.

## Choose an access method

- When `PIXELCLOUD_API_KEY` is available, prefer the HTTP API for listing,
  inspecting, starting, stopping, and restarting existing servers. Read
  [references/http-api.md](references/http-api.md) before making an API request.
- Use the CLI for interactive device login, image and plan discovery, server
  creation, server deletion, and natural-language chat.
- Never print, log, or repeat an API key. Pass it from the environment in the
  `Authorization` header. If no key is configured, direct the user to
  `https://edgerunners.cn/dashboard/api-keys`; do not ask them to paste the key
  into chat.
- Use graceful `restart` first. Call `force-restart` only when the server is
  stuck and the user has asked for that stronger action.

## HTTP API quick start

```bash
curl https://api.edgerunners.cn/v1/servers \
  -H "Authorization: Bearer $PIXELCLOUD_API_KEY"
```

An API key can see and control only the servers selected when the key was
created or edited. Treat `404 server_not_permitted` as unavailable; do not try
to discover servers outside that scope.

## Install

```bash
npm install -g @pixelcloud/cli
```

## Authentication

Before using any command, authenticate:

```bash
pixel auth login
```

This opens a browser for device-code approval and stores a token locally.

## Commands

```
pixel auth login                        # authenticate (opens browser)
pixel auth logout                       # remove stored credentials
pixel auth set-api-url <url>            # override API base URL
pixel auth set-web-url <url>            # override web dashboard URL
pixel create --image-id <id> --plan <nameOrId> [--name <n>] [--env K=V...]
pixel start --server-id <id>
pixel stop --server-id <id>
pixel restart --server-id <id>
pixel force-restart --server-id <id>    # delete pod, K8s auto-recreates
pixel delete --server-id <id> [--yes]
pixel search image [keywords]           # search game server images
pixel search plan [keywords]            # list available hardware plans
pixel prompt <message>                  # AI assistant (natural language)
pixel status                            # list all servers
pixel status --server-id <id>           # detailed view of one server
```

## Tips

- `--plan` accepts human-readable names (e.g. `basic`, `standard-14900k`) or numeric IDs. Matching is case-insensitive and supports partial names.
- Use `pixel search plan` to discover available plan names and IDs before creating a server.
- Add `--json` to any command for machine-readable output (no colors, no tables).
- Use `pixel prompt` to let the AI assistant handle complex operations in natural language.
- Environment variables: `PIXELCLOUD_TOKEN`, `PIXELCLOUD_API_URL`, `PIXELCLOUD_WEB_URL`.

## Workflow

1. **Browse** available plans: `pixel search plan --json`
2. **Search** for a game image: `pixel search image "minecraft" --json`
3. **Create** a server: `pixel create --image-id <id> --plan basic --json`
4. **Check** status: `pixel status --json`
5. **Manage** lifecycle: `pixel start/stop/restart --server-id <id> --json`
