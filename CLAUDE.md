# Thunderbird contributor measurements dashboard

A single-page dashboard of Thunderbird community contributions and contributors, published with GitHub Pages. It is updated about once a quarter from the Community Team's **Contributor measurements** Google Doc.

## Files

- `index.html`: the page (layout, wording, charts). It holds **no numbers**; it loads everything from the data file.
- `data/contributors.json`: every number on the page. A routine update changes only this file.
- `CLAUDE.md`: this file.

The page works out which years to talk about from the data. The newest year with `"period": "Full year"` is the headline year, the year before it is the comparison year, and a year with any other period is shown as the year in progress. When a year closes, `index.html` doesn't need editing.

## Updating the numbers

When asked to update the dashboard (for example "update the numbers from the doc"), do the following.

### 1. Read the doc

- Source: the Google Doc titled **Contributor measurements** on the Community Team's Google Drive. Link: `<ADD THE DOC LINK HERE>`
- Read it with the Google Drive connector. If you can't open it, stop and say so. Don't use figures from anywhere else, and don't reuse old values.

### 2. Map the doc's tables to `data/contributors.json`

Only the **Bitergia** section and the **Weblate** table feed the dashboard. Ignore the "Thunderbird Community Metrics Scripts" section entirely; the dashboard doesn't show it.

| Doc table | JSON dataset | Columns |
|---|---|---|
| Bitergia Totals | `bitergia_totals` | Number of Contributions → `contributions`; Number of Contributors → `contributors` |
| SUMO (under "Broken Down by Platform") | `bitergia_platforms`, `"platform": "SUMO"` | `contributions`, `contributors`, `"locales": null` |
| Pontoon | `bitergia_platforms`, `"platform": "Pontoon"` | `contributions`, `contributors`, Number of Locales → `locales` |
| Topicbox | `bitergia_platforms`, `"platform": "Topicbox"` | `contributions`, `contributors`, `"locales": null` |
| GitHub | `bitergia_platforms`, `"platform": "GitHub"` | `contributions`, `contributors`, `"locales": null` |
| Bugzilla | `bugzilla` | Number of tickets opened → `tickets_opened`; Number of comments made → `comments`; Number of Contributors → `contributors` |
| Weblate report (under "Miscellaneous") | `weblate` | `contributions`, `contributors` |

Rules for every row:

- **Year and period.** A plain year such as `2025` becomes `"year": 2025, "period": "Full year"`. A partial year such as `2026 (thru Q3)` becomes `"year": 2026` with a period by quarter:
  - thru Q1 → `"Jan 1 – Mar 31"`
  - thru Q2 → `"Jan 1 – Jun 30"`
  - thru Q3 → `"Jan 1 – Sep 30"`

  Use exactly `"Full year"` for full years, because the page matches that string.
- **Numbers** are plain integers with no commas.
- **Bugzilla** has no `contributions` field. The page adds `tickets_opened` and `comments` together itself.
- Keep **one row per platform per year**, with rows in year order.
- Set `"updated"` to today's date as `YYYY-MM-DD`.
- Leave a value out rather than guess. If a cell is blank or `N/A`, don't invent a number. Leave that row unchanged and mention it in the pull request.

### 3. Check before committing

Don't silently "fix" anything. List each of these in the pull request description if it happens:

- A number for a **past full year** changed.
- A year-in-progress figure went **down** since the last update.
- The doc's **table headings or columns** changed, or a table is missing.
- The deduplicated total in Bitergia Totals is far from the platform contributors added together. A normal total sits a little under that sum, because people who contribute on several platforms count once. It can sit a little over it, because the total counts Bugzilla commenters.
- A **new platform or source** appears in the doc. Don't add it; mention it.

### 4. Preview

The page loads its data with `fetch`, so it won't work when opened straight from disk. Run:

```
python3 -m http.server 8000
```

Then open http://localhost:8000 and check that every number shows and no card says "Failed to load".

### 5. Open a pull request

- Work on a branch named `update-numbers-YYYY-MM-DD`. Don't push to `main`, which publishes the site.
- In the pull request description, list **every changed value as old → new**, grouped by table, followed by anything from step 3.

## Definitions

These match the doc. Don't change them without being asked.

- **SUMO:** contributors are people who answered at least one question (askers aren't counted); a contribution is one answer.
- **Pontoon:** contributors submitted a change to at least one string; a contribution is one submitted change.
- **Topicbox:** contributors posted to any Thunderbird Topicbox list; a contribution is one message.
- **Bugzilla** (Thunderbird, MailNews Core, Calendar, Chat Core): contributors opened an issue or commented on one; a contribution is one opened issue or one comment. Bitergia's 2023 Bugzilla data is known to be incomplete.
- **GitHub:** contributors authored a commit in a tracked repo, excluding the l10n repos; a contribution is one commit.
- **Bitergia Totals:** contributions are from Bitergia's Contributors Growth panel. Contributors are a deduplicated count of the people above, each counted once. Bots are excluded throughout.

## This site is public

Anything committed here is published on GitHub Pages. Copy only the numbers into this repo. Don't copy anything else from the Google Doc (notes, links to internal docs, the ideas section).

## Editing the page itself

- Keep numbers out of `index.html`. Text that names a year uses `<span data-yr="latest|prev|partial|partial-period">`, which the page fills in from the data.
- Elements marked `data-partial` hide themselves when no year is in progress.
- Numbers are drawn by `data-draw="num"` elements. Each one reads its value from `data-source` (a dataset or calculation), `data-field` and `data-where`. In `data-where`, `@latest`, `@prev` and `@partial` stand for those years.
