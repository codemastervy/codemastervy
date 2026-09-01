# Repo context

This is a GitHub "profile README" repo: `README.md` here is what renders on
the owner's GitHub profile page. Besides the README itself, the repo
contains one small piece of automation that is not obvious from reading
`README.md` alone.

## Automation: daily repo-list sync

- `.github/workflows/sync-readme.yml` runs on a daily schedule
  (`cron: "0 20 * * *"`, ~8am NZST) and can also be triggered manually via
  `workflow_dispatch`. It runs `scripts/sync_readme.py`.
- The script calls the GitHub API (`GET /users/codemastervy/repos`),
  authenticating with the workflow's `GITHUB_TOKEN` to get a 5,000/hour
  rate limit instead of the 60/hour limit shared by unauthenticated
  callers on GitHub-hosted runner IPs.
- It skips forks, the repos named `codemastervy` and
  `codemastervy.github.io`, and any repo whose name already appears in
  `README.md` as a `github.com/codemastervy/<name>)` link (found by regex,
  anywhere in the file — not only inside the table).
- For each remaining repo, it appends one row (`**[name](url)** | description
  |`) to the "Repositories" table. New rows are inserted right after the
  last existing row that is both a table row (starts with `| **[`) and
  links to `github.com/codemastervy/...`. This is deliberate: a row with no
  working link yet (e.g. a "repo coming soon" placeholder) stays below all
  the auto-inserted rows, at the bottom of the table.
- If there's nothing new to add, the workflow's commit step is a no-op
  (`git diff --quiet`) and nothing is pushed.
- **The automation only ever adds rows.** It never detects or removes a row
  for a repo that is later deleted, renamed, or made private — that has to
  be done by hand. (This happened twice in this repo's history: a
  `workflow-test` repo's row had to be manually removed in a follow-up
  commit after being auto-added during testing.)

## Implications for editing README.md

- Bio, toolbox, and other prose sections are never touched by the
  automation — edit them freely.
- If you manually add or edit a repo row, keep the exact link format
  `**[name](https://github.com/codemastervy/name)**` so the next automated
  run recognizes it as already present and doesn't add a duplicate row.
- If a repo listed in the table is deleted, renamed, or made private, its
  row will not be auto-removed; remove it manually.
- The script can also be run locally: `python3 scripts/sync_readme.py`
  (uses an unauthenticated API call if `GITHUB_TOKEN` isn't set in the
  environment).
