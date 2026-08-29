# pixelcloud-skills

A collection of AI agent skills for [PixelCloud](https://pixelcloud.cn) — making game server management accessible to AI coding assistants.

## What's Inside

| Skill                                            | Description                                                                                                               |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| [pixelcloud-cli](skills/pixelcloud-cli/SKILL.md) | Gives your AI agent the ability to manage multiplayer game servers through the `pixel` CLI or the server-scoped HTTP API. |

## HTTP API

PixelCloud also provides a versioned JSON API for server status and lifecycle
automation. Create a server-scoped key in the dashboard, then follow the
[HTTP API reference](skills/pixelcloud-cli/references/http-api.md).

## Usage

```bash
npx skills add skyquakers/pixelcloud-skills
```

## License

Copyright © Skyquakers
