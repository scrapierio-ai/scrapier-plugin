---
name: scrapier-batch-scraping
description: "Scrape many URLs, profiles or search queries with Scrapier without silently dropping items or paying twice. Use when a Scrapier scraping task covers multiple inputs, such as a list of product URLs, several competitor profiles or a set of search terms."
---

# Batch scraping with Scrapier

Scrapier's endpoints take **one input per call** and return **one page of
results per call**. A batch is therefore a loop you run: one `run_scrape` per
input, plus extra calls for any further pages you need. Nothing on the server
tracks the batch for you, so you have to track it yourself.

## Before you start, say what it will take

Tell the user how many inputs you are about to scrape and that every
successful call is billed. Fifty URLs is fifty billed calls, and more if you
follow `next_cursor` for extra pages. Ask before running a very large batch.

## One call per input

```
run_scrape(endpoint: "instagram-profile", params: { usernames: "nasa" })
run_scrape(endpoint: "instagram-profile", params: { usernames: "spacex" })
```

Do not pack several inputs into one call. `get_endpoint_schema` types a param
as `["string", "array"]` only when the endpoint really accepts a list (today
only `contact-extractor.urls`). Everywhere else the param is a plain `"string"`,
and it expects a single value, not a list. The `Aliases:` in a description
lists other names for the same single value. It does not mean the param takes
several.

## What a result looks like

A normal result comes back inline:

```json
{
  "status": "completed",
  "credits_charged": 1,
  "next_cursor": "eyJwIjoy…",
  "has_more": true,
  "results": { "query": "laptop", "page": 1, "page_size": 20, "products": [ … ] }
}
```

- `results` echoes the input it ran for, so you can match every result to its
  input even if you ran them out of order.
- The rows sit under an endpoint-specific key (`products`, `posts`, `listings`,
  `comments`…). Read the schema's `sample_output` to know which.
- **There is no `progress` or `errors` block.** Each call succeeds or fails on
  its own, so the batch report has to come from your own tally.

## Keep a tally, and report it honestly

For every input, record one of:

- **ok**: rows came back
- **empty**: the call succeeded with zero rows, which is a real answer ("no
  results for this query"). It is billed like any other call, so it counts as
  done, not as something to retry
- **failed**: `isError` was set, or `status` is `"failed"`

Then report the real counts: *"47 of 50 scraped, 2 returned nothing, 1 failed
(@example: auth wall)."* Never write "scraped all 50" unless the tally says so.

## Paging: decide per input how deep to go

If `next_cursor` is set, more results exist for **that** input. Fetch the next
page by passing the cursor alone:

```
run_scrape(endpoint: "amazon-product", params: { cursor: "eyJwIjoy…" })
```

A cursor belongs to one input. Never reuse it for another input. Every page is
billed, so set a page limit up front (for example "first page only" or "up to 3
pages each") and apply it to every input the same way. When you stop early,
tell the user the data is a sample. When `next_cursor` is null, that input is
complete.

## Retry only what failed

A failed call is not billed, so retrying it only costs time. Re-running an
input that already succeeded, including one that came back empty, is billed
again. Retry the **failed** inputs from your tally, not the whole list.

## Slower endpoints return a job

Some scrapes return a `job_id` instead of a result. Collect it with
`get_scrape_result` (polling is free). A finished job can carry:

- `errors`: which inputs failed inside the job (`chunk_inputs`), and why
- `inputs_truncated: true`: fewer inputs were accepted than you sent, so later
  ones were never attempted
- `data_truncated: true`: the result was capped, so what you have is incomplete

`status: "completed"` does not mean every input succeeded. Check these fields
and count them into your tally before reporting.

## Cost

Every successful call is billed its `cost_credit` (1 to 4 credits, depending on the
endpoint; see `get_endpoint_schema`) however many results it returns, and every page
counts as a call. An empty result is still a successful call. Failed calls are
not billed. Each result's `credits_charged` shows what that call actually
cost, so add them up and give the user the total. What makes a batch expensive
is the number of calls: one per input, times the pages you follow. Follow
`next_cursor` only as far as the task needs.
