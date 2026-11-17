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
is the same server as the Scrapier connector for Claude. The server provides
these tools:

| Tool | What it does | Cost |
|---|---|---|
| `list_platforms` | Lists the platforms in the Scrapier catalog | Free |
| `search_endpoints` | Finds scraping endpoints that match a query | Free |
| `get_endpoint_schema` | Shows an endpoint's parameters, credit cost and sample output | Free |
| `run_scrape` | Runs a scrape and returns the results | Billed per successful call |
| `get_scrape_result` | Kept for compatibility: collects a job result if a scrape ever returns a `job_id` (`run_scrape` returns results directly) | Free |
| `get_account` | Shows the signed-in account and its credit balance | Free |

**Four skills**, plain Markdown files in `skills/`:

| Skill | Claude uses it when |
|---|---|
| `scrapier` | You ask Claude to scrape or collect data from a platform Scrapier covers. It follows search, then schema, then run, and reads only the pages the task needs. |
| `scrapier-batch-scraping` | A scraping task covers many URLs, profiles or search terms. Claude keeps a per-input tally, retries only failed inputs and reports real counts. |
| `scrapier-api` | You ask for a script, app or integration that calls the Scrapier REST API. Claude uses real endpoint names and parameters and leaves the API key to your app's configuration. |
| `scrapier-safe-data-handling` | Scraped results will be summarised, saved, or passed to another tool, or a request concerns individual people. Claude treats page text as data, never as instructions, tells you if a page contains text that tries to instruct it, takes only the personal data the task needs, and declines to find or track a specific private person. |

The plugin contains no hooks, scripts, commands, agents or executable code. It
runs nothing on your computer.

## Install

This repository is a one-plugin marketplace for Claude Code. From your shell,
add the marketplace and install the plugin:

```
claude plugin marketplace add scrapierio-ai/scrapier-plugin
claude plugin install scrapier@scrapier
```

Or run `/plugin` in a Claude Code session to add the marketplace and install
the plugin from the plugin manager.

## Sign in

The Scrapier server uses OAuth, so you sign in once before Claude can use its
tools:

- **Claude Code:** run `/mcp`, select the Scrapier server from this plugin and
  choose Authenticate.
- **Other Claude apps:** connect the Scrapier server wherever the app lists the
  plugin's connectors or MCP servers.

Your browser opens Scrapier's sign-in page. Sign in to your Scrapier account,
or create one at https://app.scrapier.io, and approve the connection. Each
connection gets its own key on Scrapier's side and its own daily credit limit.
Signing in shares your Scrapier account email with the connection so it can
show your balance. You can disconnect Scrapier in Claude, or revoke the
connection under Connected apps in the Scrapier dashboard, at any time.

## Try it

- "Find the top 10 Amazon results for wireless earbuds and list price and rating."
- "Look up the NASA and SpaceX Instagram profiles and compare their follower counts."
- "Write a Python script that pulls Google Maps places for coffee shops in Austin from the Scrapier API, with retries and a page limit."

## Limitations

- A few slower endpoints, marked `long_running` in search results (for example
  `instagram-profile-posts`, `google-play-reviews` and `zillow-detail`), are
  refused over the plugin's sign-in connection because they can take longer
  than two minutes. They work through the REST API with an API key.
- Each connection has a daily credit cap (1,000 credits by default). When it is
  reached, scrapes stop until 00:00 UTC; you can change it under Connected apps in the Scrapier dashboard.

## Data and network use

The plugin itself stores nothing and sends nothing on its own. Data moves only
when Claude calls a Scrapier tool or, for the `scrapier-api` skill, reads
Scrapier's public reference pages.

- **To mcp.scrapier.io (Scrapier, Inc.).** Each tool call sends the tool name
  and the inputs Claude chose for it, such as an endpoint name, a URL, a
  username or a search term. Claude writes those inputs from your conversation,
  so they can contain anything you typed for the scrape. Scrapier does not
  receive the rest of your conversation. Scraping calls reach the Scrapier API
  from Scrapier's MCP server, not from your computer, and Scrapier records each
  one (the endpoint, the inputs, the time, the status and the credits charged)
  in your account's request history, as it does for API calls.
- **Session cookies.** A few endpoints accept an optional login cookie for the
  target site. The skills tell Claude never to ask for one. If you supply one
  anyway, it is sent to Scrapier like any other input and stored in your
  request history.
- **From mcp.scrapier.io.** Scrapier returns the scraped results to Claude.
  Results are third-party public web data collected at your request. They can
  include personal information about other people, and you are responsible
  for how you use it. Every result is marked as untrusted content.
- **To Scrapier's public reference pages.** When you ask Claude to write code
  for the Scrapier REST API, the `scrapier-api` skill may have Claude read the
  public endpoint catalog and OpenAPI spec at `api.scrapier.io` and the
  documentation, including its `llms` text files, at `docs.scrapier.io`. These requests carry no API key and no
  conversation content.
- **Sign-in.** OAuth sign-in happens between Claude and Scrapier. The plugin
  does not see or store your password, tokens or API keys, and it never reads
  credentials from your computer.

Scrapier's [privacy policy](https://scrapier.io/privacy) explains what Scrapier
collects through the MCP server, how long it keeps it, and how to revoke a
connection.

## Credits

Each successful `run_scrape` call costs the endpoint's credit price, its
`cost_credit`, which `search_endpoints` and `get_endpoint_schema` show for free.
Most endpoints cost 1 credit per call. The price is the same whatever the
number of results, including a call that returns no results. Each extra page of results is a
separate call. Failed calls are not charged. All other tools are free. Credits
come from the plan on your Scrapier account; `get_account` shows your balance.

## Responsible use

Scrapier returns publicly accessible data at your instruction, and you are
responsible for what you ask it to collect and what you do with the results.
Under Scrapier's [acceptable use rules](https://docs.scrapier.io/responsible-use/),
you may not use it to:

- stalk, harass, surveil or track a person, or find someone who doesn't want to
  be found
- compile profiles of individuals in order to sell them, or to target or
  discriminate against people
- identify people from anonymous activity
- collect data that is private, behind a login you aren't entitled to use, or
  that you have no lawful basis to process
- break the law, or the terms of the site you scrape, where that is your
  responsibility
- send abusive volumes of requests, or evade limits to harm a site

The same rules apply when Claude calls Scrapier for you through this plugin.
Results can contain personal data about other people; collect only what you
need and follow the privacy and data-protection laws that apply to you. See
also the [terms of use](https://scrapier.io/terms).

## Support and links

- Scrapier documentation: https://docs.scrapier.io/
- Claude connector and sign-in: https://docs.scrapier.io/ai/claude/
- Support: https://scrapier.io/contact
- Privacy policy: https://scrapier.io/privacy
- Terms of use: https://scrapier.io/terms
- Acceptable use: https://docs.scrapier.io/responsible-use/

## License

The plugin files in this repository are released under the [MIT License](LICENSE).
Use of the Scrapier service is governed by the Scrapier terms of use.
