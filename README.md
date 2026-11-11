# MCPScraper CLI

Scraping MCP servers, compared on a published rubric.

`mcpscraper` fetches live web pages as clean Markdown, text or HTML, straight from your terminal or your agent's shell.
It is the command line client for the [MCPScraper API](https://mcpscraper.net).

## Quick start

```bash
npx @mcpscraper-net/cli login                                   # paste your API key (not echoed)
npx @mcpscraper-net/cli fetch https://example.com -o text       # the page as Markdown
```

Get a key at [mcpscraper.net](https://mcpscraper.net). Node.js 18 or later.

## Install

```bash
npm install -g @mcpscraper-net/cli
mcpscraper --help
```

## Use

```bash
mcpscraper fetch https://example.com/a https://example.com/b      # up to 10 URLs; JSON with results[] and statuses[]
mcpscraper fetch https://example.com -o text                      # just the content (Markdown, else text, else HTML)
mcpscraper fetch https://example.com -o html --html               # the page HTML
mcpscraper fetch https://example.com --country-code de --render   # from Germany, rendered in a real browser
mcpscraper fetch --data @request.json                             # the whole request body from a file (- for stdin)
mcpscraper account                                                # plan, credits and rate limit
mcpscraper fetch --help                                           # every option, with the API's own descriptions
```

A fetch that is still running after about 100 seconds is collected automatically with the same
`Idempotency-Key`, so it is charged once.

## MCP server

The MCPScraper API is also a remote MCP server (Streamable HTTP) at `https://mcpscraper.net/mcp`, with the tools
`get_page_contents` and `get_account_status`. It takes the same API key as a bearer token.

```bash
claude mcp add --transport http mcpscraper https://mcpscraper.net/mcp \
  --header "Authorization: Bearer $MCPSCRAPER_API_KEY"
```

Any other client: URL `https://mcpscraper.net/mcp`, header `Authorization: Bearer <key>`. Config for Cursor, VS Code and
Claude Desktop: [MCP docs](https://mcpscraper.net/docs/mcp). The server's entry for the MCP Registry is [server.json](server.json)
(`net.mcpscraper/mcpscraper`).

## Agent plugin and skill

This repo is also a plugin for Claude Code, Grok Build and Gemini CLI, and holds an agent skill. Each one connects the
MCP server above and teaches the agent when to fetch a page with it or with `mcpscraper`.

```bash
npx skills add mcpscraper-net/cli                                    # the skill, for any agent that reads SKILL.md
claude plugin marketplace add mcpscraper-net/cli && claude plugin install mcpscraper@mcpscraper
gemini extensions install https://github.com/mcpscraper-net/cli
```

Network and credentials: the plugin calls `https://mcpscraper.net/mcp` only, and the CLI `https://mcpscraper.net/api` and
`https://mcpscraper.net/openapi.json`. Both need one credential, your MCPScraper API key, read from `MCPSCRAPER_API_KEY`. No hooks, no
scripts run on install.

## The commands follow the live API

`mcpscraper` builds its commands and options from the API's OpenAPI document at
[https://mcpscraper.net/openapi.json](https://mcpscraper.net/openapi.json), cached for an hour. When the API gains a field or an endpoint, the
option or command is there on your next run, without updating the CLI. Offline, it uses the last cached
copy, then the copy bundled with the installed version (`mcpscraper spec` shows which one; `--offline` skips the
download). An hourly job here publishes a new version whenever the API spec changed.

## Configuration

| Setting | Option | Environment | Default |
|---|---|---|---|
| API key | `--api-key` | `MCPSCRAPER_API_KEY` | the key saved by `mcpscraper login` (`~/.config/mcpscraper/config.json`, mode 0600) |
| API base URL | `--base-url` | `MCPSCRAPER_API_BASE_URL` | `https://mcpscraper.net/api` |
| API spec URL | `--spec-url` | `MCPSCRAPER_SPEC_URL` | `https://mcpscraper.net/openapi.json` |
| Output | `-o json\|text\|<field>` | | `json` |
| Update notice | | `MCPSCRAPER_NO_UPDATE_NOTIFIER=1` turns it off | once a day, terminals only |

Requests and your key only ever go to the base URL, never to a server named in the spec.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | OK |
| 1 | The API answered with an error (JSON on stderr, with its `requestId`), every URL failed with `-o text`, or the fetch was still running at `--max-wait` |
| 2 | Usage: unknown command or option, a value the API does not accept |
| 3 | Network: no answer. The error carries the `Idempotency-Key` to collect the result without a new charge |

## Links

- [mcpscraper.net](https://mcpscraper.net)
- [CLI docs](https://mcpscraper.net/docs/cli)
- [API docs](https://mcpscraper.net/docs)
- [API spec](https://mcpscraper.net/openapi.json)

## License

MIT, © mcpscraper.net
