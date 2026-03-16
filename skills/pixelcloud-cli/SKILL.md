---
name: pixelcloud-cli
description: PixelCloud CLI skill for managing multiplayer game servers. Gives your AI agent the ability to create, start, stop, restart, and delete game servers, search images, and use natural language to manage infrastructure.
---

# PixelCloud CLI

You have access to the `pixel` CLI for managing game servers on PixelCloud.

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
pixel create --image-id <id> --plan <id> [--name <n>] [--env K=V...]
pixel start --server-id <id>
pixel stop --server-id <id>
pixel restart --server-id <id>
pixel force-restart --server-id <id>    # delete pod, K8s auto-recreates
pixel delete --server-id <id> [--yes]
pixel search image [keywords]           # search game server images
pixel prompt <message>                  # AI assistant (natural language)
pixel status                            # list all servers
pixel status --server-id <id>           # detailed view of one server
```

## Tips

- Add `--json` to any command for machine-readable output (no colors, no tables).
- Use `pixel prompt` to let the AI assistant handle complex operations in natural language.
- Environment variables: `PIXELCLOUD_TOKEN`, `PIXELCLOUD_API_URL`, `PIXELCLOUD_WEB_URL`.

## Workflow

1. **Search** for a game image: `pixel search image "minecraft" --json`
2. **Create** a server: `pixel create --image-id <id> --plan <id> --json`
3. **Check** status: `pixel status --json`
4. **Manage** lifecycle: `pixel start/stop/restart --server-id <id> --json`
