# geekbench-actions

Run [Geekbench 6](https://www.geekbench.com/) CPU benchmarks on GitHub-hosted
runners, on demand. Every workflow is manual-only (`workflow_dispatch`) — nothing
runs on push. Each run downloads the official Geekbench build, runs it, uploads
the result to the Geekbench Browser, and surfaces the result URL in the run
summary plus a downloadable log artifact.

## Workflows

| Workflow | Runner | Trigger |
|---|---|---|
| **Geekbench — Linux** | `ubuntu-latest` | manual |
| **Geekbench — macOS** | `macos-latest` (Apple Silicon) | manual |
| **Geekbench — Windows** | `windows-latest` | manual |
| **Geekbench — all platforms** | matrix of all three | manual |
| `_geekbench.yml` | — | reusable core (`workflow_call`), not run directly |

The four named workflows are thin wrappers that call the shared
`_geekbench.yml`, so the download/run/report logic lives in one place.

## Running

1. Open the **Actions** tab.
2. Pick a workflow (e.g. *Geekbench — Linux*) → **Run workflow**.
3. Optionally override the inputs, then confirm.

### Inputs

| Input | Default | Notes |
|---|---|---|
| `version` | `6.7.1` | Geekbench version pulled from `cdn.geekbench.com`. |
| `extra_args` | `--cpu` | Passed verbatim to the `geekbench6` CLI. |

Useful `extra_args` values: `--cpu` (default), `--cpu --multi-core`, or any flag
the Geekbench CLI accepts. GPU compute is omitted because hosted runners have no
dedicated GPU.

## Results

- **Run summary** shows the `https://browser.geekbench.com/...` result URL.
- **Artifacts** (`geekbench-<os>`) contain:
  - `geekbench.log` — full CLI output
  - `result-url.txt` — the captured result URL
  - `result.json` — only when a Pro license is configured (see below)

## Optional: Geekbench Pro license

The free CLI must upload results to the Geekbench Browser. Add a Pro license to
also export results as JSON locally (`result.json` artifact). Set these repo
secrets (Settings → Secrets and variables → Actions):

- `GEEKBENCH_EMAIL`
- `GEEKBENCH_KEY`

When both are present, each run unlocks Pro and adds `--export-json result.json`.

## Notes

- The Windows job installs Geekbench from the official Inno Setup installer
  (`/VERYSILENT`) and locates `geekbench6.exe` under Program Files. If a future
  Geekbench installer changes its silent-install flags, adjust the Windows step
  in `_geekbench.yml`.
- To benchmark a newer release, set the `version` input — no file edits needed,
  as long as the CDN asset names keep the
  `Geekbench-<version>-{Linux.tar.gz,Mac.zip,WindowsSetup.exe}` pattern.
