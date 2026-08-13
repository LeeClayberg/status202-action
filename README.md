<p align="center"><img src="https://status202.dev/icons/Icon-192.png" width="96" height="96" alt="status202 logo"></p>

# Status 202 — GitHub Action

Report progress from a GitHub workflow to a [Status 202](https://status202.dev) tracker.

It's a composite action: it just runs the published [`status202`](https://www.npmjs.com/package/status202)
npm CLI, so there's no bundled `dist/` to keep in sync and the CLI stays the single source of truth.

## Setup

1. Create a tracker at [status202.dev](https://status202.dev).
2. Copy its URL token.
3. Add it to your repo as a secret, e.g. `STATUS202_TOKEN`.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `token` | yes | — | Tracker URL token. Always pass this from a secret. |
| `percent` | one of | — | Percent complete, 0–100. For percent-mode trackers. |
| `value` | one of | — | Raw value. For range-mode trackers. |
| `command` | one of | — | Shell command whose stdout is the payload. Overrides the two above. |
| `api-key` | no | — | Account API key. Only for `apiKey`-mode trackers. |
| `base-url` | no | `https://api.status202.dev/v1` | API base URL. |
| `cli-version` | no | `latest` | Version of the npm package to run. Pin it for reproducible builds. |
| `node-version` | no | `20` | Node version used to run the CLI. |

Exactly one of `percent`, `value`, or `command` is required.

## Recipes

### Mark a job's start and finish

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: LeeClayberg/status202-action@v1
        with:
          token: ${{ secrets.STATUS202_TOKEN }}
          percent: 0

      - run: ./build.sh

      - uses: LeeClayberg/status202-action@v1
        if: success()
        with:
          token: ${{ secrets.STATUS202_TOKEN }}
          percent: 100
```

### Report a computed number

Anything you can print on stdout works — here, test coverage:

```yaml
- uses: LeeClayberg/status202-action@v1
  with:
    token: ${{ secrets.COVERAGE_TRACKER_TOKEN }}
    command: "jq -r '.total.lines.pct' coverage/coverage-summary.json"
```

### Announce completion in Slack

Pair it with a [202 Connect](https://status202.dev/connect) integration: the action drives the
tracker to 100%, and Connect posts to Slack, Discord, Teams, Jira, or your own webhook when it lands.
No extra workflow steps — the notification is configured once, in the app.

```yaml
- uses: LeeClayberg/status202-action@v1
  with:
    token: ${{ secrets.STATUS202_TOKEN }}
    percent: 100
# → tracker completes → 202 Connect fires your Slack webhook
```

## Notes

- All inputs reach the script through `env:` rather than inline `${{ }}` expansion in `run:`,
  so a value containing shell metacharacters can't break out into the runner's shell.
- The action reports once and exits (`--push --once`); it doesn't poll or hold the job open.
- A bad token fails the step loudly rather than passing silently.
