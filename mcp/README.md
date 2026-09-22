# Calling all three actors through Apify's MCP server

Verified against `docs.apify.com/platform/integrations/mcp`, current as of Sep 2026.

## The server
Apify hosts a single MCP server for the whole platform at:

```
https://mcp.apify.com
```

It exposes generic tools (`search-actors`, `fetch-actor-details`, docs search — these work
**anonymously**, no token needed) plus one callable tool per actor you scope it to. Scope it with
a `tools` query parameter so an agent only sees the three actors that matter here:

```
https://mcp.apify.com?tools=make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector
```

Add `search-actors` to that list if you want the agent able to discover other actors too:
`?tools=search-actors,make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector`

## Authentication
Two options:
- **OAuth (recommended for Claude Desktop and Cursor's UI-driven setup)** — the client opens a
  browser, you sign into Apify, done. No token in any config file.
- **Bearer token** — add an `Authorization: Bearer <APIFY_TOKEN>` header in the client config.
  Needed for CLI/headless setups (Claude Code, VS Code without the connector UI). Get the token
  from Apify Console → Settings → API & Integrations. **Never commit it** — every example below
  uses a placeholder.

## Claude Desktop
Claude Desktop's MCP setup for hosted servers is a UI flow, not a config file you hand-edit:

1. Settings → Connectors → **Add custom connector**.
2. Server URL: `https://mcp.apify.com?tools=make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector`
3. Approve the OAuth prompt (or, if the connector UI exposes a custom-headers field, add
   `Authorization: Bearer <APIFY_TOKEN>` instead of using OAuth).

Alternative: search "Apify" in Claude Desktop's built-in connector directory and install it
directly — same server, zero manual config.

## Cursor
Create or edit `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com?tools=make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector"
    }
  }
}
```

With a bearer token instead of OAuth:

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com?tools=make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector",
      "headers": {
        "Authorization": "Bearer <APIFY_TOKEN>"
      }
    }
  }
}
```

Or use Apify's CLI helper, which writes this file for you:
```bash
apify mcp install cursor --token <APIFY_TOKEN> --tools make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector
```

## VS Code (GitHub Copilot agent mode)
Command Palette → **MCP: Open User Configuration**, same JSON shape as Cursor above. Or:
```bash
apify mcp install vscode --token <APIFY_TOKEN> --tools make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector
```

## Claude Code CLI
Apify's install helper covers Claude Code directly:
```bash
apify mcp install claude-code --token <APIFY_TOKEN> --tools make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector
```
If you'd rather add it by hand with Claude Code's own `claude mcp add`, point it at the same URL
with an `Authorization: Bearer <APIFY_TOKEN>` header — check `claude mcp add --help` for the exact
current flag names for a remote HTTP server on your installed version.

## Local stdio fallback (no hosted server, npx-based)
Only needed for clients that can't do remote HTTP MCP:
```json
{
  "mcpServers": {
    "actors-mcp-server": {
      "command": "npx",
      "args": ["-y", "@apify/actors-mcp-server", "--actors", "make_no_mistakes/us-building-permits-scraper,make_no_mistakes/multi-ats-job-board-api,make_no_mistakes/company-ats-detector"],
      "env": { "APIFY_TOKEN": "YOUR_APIFY_TOKEN" }
    }
  }
}
```

## What the agent can actually call
Once scoped, the MCP server exposes one tool per actor, named after the actor. An agent calling
the permits actor sees the same input shape as the REST API — `cities`, `daysBack`, `maxItems`,
`permitTypes`, `includeAdministrative` — and gets dataset items back directly. Same for the ATS
actor: `companies`, `platforms`, `discoverOnly`, `postedWithinDays`, `titleIncludes`, `remoteOnly`,
`maxItems`. The free detector actor takes the same `companies` input, plus `includeJobCount` and
`maxCompanies`, and is the cheap first call before the paid ATS actor's `discoverOnly` or full job
pull. All three actors' READMEs already carry a "For AI agents and MCP clients" section with the
minimal-input examples an agent should default to (`maxItems` set low, or no cost cap at all on the
detector) so a first exploratory call doesn't run up spend.

## The honest limit on this move
Per the internal research this kit implements (a Sep 2026 SaaS-growth research note, move #5 and
its §4a counter-evidence): MCP and integration directories are **supply-side, not
demand-side** — listing here does not by itself generate agent traffic, any more than an app store
listing generates installs on its own (swyx's rebuttal: "there's significant SUPPLY for what he
built… the hard part is the demand"). Treat this kit as necessary infrastructure and a backlink
source, not a growth channel in its own right. The n8n/Make templates in `../n8n/` and `../make/`
are the sub-moves in #5 with a demonstrated revenue effect (Postiz doubling MRR by getting into
workflow templates) — do those first if you can only do one thing this week.
