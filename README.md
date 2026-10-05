# starter-stack

Run the full Cell agents stack on your laptop: one `docker compose up`
builds every service from its upstream repo and wires them together.
No DNS, no TLS, no servers.

If you need a public deployment, see
[`game.cellagents.dev`](https://github.com/cellagents/game.cellagents.dev)
instead. This repo is the localhost sibling.

## What you get

| Service     | Local URL                      | Role                                             |
|-------------|--------------------------------|--------------------------------------------------|
| game        | <http://127.0.0.1:3000>        | `cells-game` server + default web client         |
| mcp         | <http://127.0.0.1:4000/mcp>    | `cells-mcp` MCP endpoint                         |
| harness     | <http://127.0.0.1:5000/panel>  | Reference harness and student panel              |
| litellm     | <http://127.0.0.1:4001>        | Model gateway (internal; students never hit this) |

A one-shot container builds the thin-client bundle into a shared
volume mounted into the harness; it is not a long-running service.

All ports bind to `127.0.0.1` only, so the stack does not leak to your
LAN.

## Prerequisites

- Docker Engine or Docker Desktop with the Compose plugin
- An Anthropic or OpenAI API key (one is enough)

## Running

```bash
cp .env.example .env
# edit .env: paste at least one of ANTHROPIC_API_KEY / OPENAI_API_KEY
docker compose up --build
```

First run takes a few minutes; it fetches four repos from GitHub and
builds them. Later runs are faster unless you pass `--build`.

When all services are healthy:

- Open <http://127.0.0.1:5000/panel> to drive the agent through the
  harness UI.
- Open <http://127.0.0.1:3000> to play the game yourself with a mouse.
- Point an external MCP client (Claude Desktop, custom script) at
  <http://127.0.0.1:4000/mcp>.

To stop:

```bash
docker compose down
```

To wipe LiteLLM spend-tracking state and rebuilt images:

```bash
docker compose down -v
```

## Updating

The compose contexts pull from each app repo's default branch. To pick
up upstream changes:

```bash
docker compose pull        # pulls the LiteLLM image
docker compose up --build  # rebuilds the app images from latest main
```

## Spending guardrail

LiteLLM is configured with `max_budget: 10` USD across all keys over a
30-day window (see `litellm/config.yaml`). Hit it and the gateway
stops serving. Raise it, lower it or remove it on your own machine as
you please.

## Troubleshooting

- **Harness panel loads but model calls fail.** Check `docker compose
  logs litellm` for an auth error from Anthropic / OpenAI; usually a
  missing or wrong key in `.env`.
- **MCP endpoint returns 502 / 404.** Make sure the `game` service is
  up (`docker compose ps`). MCP depends on it.
- **Port already in use.** Edit the host-side port numbers in
  `docker-compose.yml` under each service's `ports:` block.

## License

MIT.
