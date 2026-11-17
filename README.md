# Scrapier plugin for Claude

![Scrapier](assets/icon.png)

Scrapier is a web-scraping API that returns structured JSON from public pages on
platforms such as Amazon, YouTube, LinkedIn, TikTok, Instagram, Airbnb, Reddit
and Google Maps. This plugin connects Claude to Scrapier's remote MCP server and
adds four skills that teach Claude how to use it well: find the right endpoint,
check its parameters before spending credits, scrape lists of inputs without
losing track of failures, write code against the Scrapier REST API, and treat
scraped page text as untrusted data.

You need a Scrapier account to use it. Scraping calls are billed to that
account (see [Credits](#credits)).

## What the plugin installs

**An MCP server reference.** The `.mcp.json` file points Claude at the remote
Scrapier MCP server, `https://mcp.scrapier.io/mcp`, over HTTP. It contains no
API key or other credential; you sign in through OAuth when you connect. This
is the same server as the Scrapier connector in the Claude directory, so if you
have both the connector and this plugin you see one set of Scrapier tools, not
two. The server provides these tools:

| Tool | What it does | Cost |
|---|---|---|
| `list_platforms` | Lists the platforms in the Scrapier catalog | Free |
| `search_endpoints` | Finds scraping endpoints that match a query | Free |
| `get_endpoint_schema` | Shows an endpoint's parameters, credit cost and sample output | Free |
| `run_scrape` | Runs a scrape and returns the results | Billed per successful call |
| `get_scrape_result` | Collects or pages through the result of a slower scrape | Free |
| `get_account` | Shows the signed-in account and its credit balance | Free |

**Four skills**, plain Markdown files in `skills/`:

| Skill | Claude uses it when |
|---|---|
| `scrapier` | You ask Claude to scrape or collect data from a platform Scrapier covers. It follows search, then schema, then run, and reads only the pages the task needs. |
| `scrapier-batch-scraping` | A scraping task covers many URLs, profiles or search terms. Claude keeps a per-input tally, retries only failed inputs and reports real counts. |
| `scrapier-api` | You ask for a script, app or integration that calls the Scrapier REST API. Claude uses real endpoint names and parameters and leaves the API key to your app's configuration. |
| `scrapier-safe-data-handling` | Scraped results will be summarised, saved, or passed to another tool. Claude treats page text as data, never as instructions, and tells you if a page contains text that tries to instruct it. |

The plugin contains no hooks, scripts, commands, agents or executable code. It
runs nothing on your computer.

## Install

**From the Claude directory.** Once the plugin is listed, find Scrapier in the
plugin directory in Claude or in the Discover tab of `/plugin` in Claude Code,
and install it from there.

**From this repository in Claude Code.** This repository is also a one-plugin
marketplace. In a Claude Code session run:

```
/plugin install scrapier --marketplace scrapierio-ai/scrapier-plugin
```

Or from your shell:

```
claude plugin install scrapier --marketplace scrapierio-ai/scrapier-plugin
```

You can also add the marketplace first with
`/plugin marketplace add scrapierio-ai/scrapier-plugin` and then run
`/plugin install scrapier@scrapier`.

## Sign in

The Scrapier server uses OAuth, so you sign in once before Claude can use its
tools:

- **Claude Code:** run `/mcp`, select the Scrapier server from this plugin and
  choose Authenticate.
- **claude.ai and Cowork:** connect Scrapier from the plugin's Connectors tab.

Your browser opens Scrapier's sign-in page. Sign in to your Scrapier account,
or create one at https://app.scrapier.io, and approve the connection. Each
connection gets its own key on Scrapier's side and its own daily credit limit.
You can disconnect Scrapier in Claude at any time.

## Try it

- "Find the top 10 Amazon results for wireless earbuds and list price and rating."
- "Look up the NASA and SpaceX Instagram profiles and compare their follower counts."
- "Write a Python script that pulls Google Maps places for coffee shops in Austin from the Scrapier API, with retries and a page limit."

## Data and network use

The plugin itself stores nothing and sends nothing on its own. Data moves only
when Claude calls a Scrapier tool or, for the `scrapier-api` skill, reads
Scrapier's public reference pages.

- **To mcp.scrapier.io (Scrapier, Inc.).** Each tool call sends the tool name
  and the inputs Claude chose for it, such as an endpoint name, a URL, a
  username or a search term. Claude writes those inputs from your conversation,
  so they can contain anything you typed for the scrape. Scrapier does not
  receive the rest of your conversation. Scrapier records each request (the
  tool, the inputs, the time, the status, the credits charged and the caller's
  IP address) in your account's request history, as it does for API calls.
- **From mcp.scrapier.io.** Scrapier returns the scraped results to Claude.
  Results are third-party public web data collected at your request. They can
  include personal information about other people, and you are responsible
  for how you use it. Every result is marked as untrusted content.
- **To Scrapier's public reference pages.** When you ask Claude to write code
  for the Scrapier REST API, the `scrapier-api` skill may have Claude read the
  public endpoint catalog and OpenAPI spec at `api.scrapier.io` and the
  documentation at `docs.scrapier.io`. These requests carry no API key and no
  conversation content.
- **Sign-in.** OAuth sign-in happens between Claude and Scrapier. The plugin
  does not see or store your password, tokens or API keys, and it never reads
  credentials from your computer.

Scrapier's [privacy policy](https://scrapier.io/privacy) explains what Scrapier
collects through the MCP server, how long it keeps it, and how to revoke a
connection.

## Credits

Each successful `run_scrape` call costs the endpoint's credit price, currently 1
to 4 credits depending on the endpoint, whatever the number of results,
including a call that returns no results. Each extra page of results is a
separate call. Failed calls are not charged. All other tools are free. Credits
come from the plan on your Scrapier account; `get_account` shows your balance.

## Responsible use

Use Scrapier only for data you are allowed to collect, and follow the
[acceptable use guidelines](https://docs.scrapier.io/responsible-use/) and the
[terms of use](https://scrapier.io/terms).

## Support and links

- Documentation: https://docs.scrapier.io/ai/claude/
- Support: https://scrapier.io/contact
- Privacy policy: https://scrapier.io/privacy
- Terms of use: https://scrapier.io/terms
- Acceptable use: https://docs.scrapier.io/responsible-use/

## License

The plugin files in this repository are released under the [MIT License](LICENSE).
Use of the Scrapier service is governed by the Scrapier terms of use.
