# geekbench-actions

Run [Geekbench 6](https://www.geekbench.com/) CPU benchmarks on GitHub-hosted runners, on demand. Every workflow is manual-only (`workflow_dispatch`), so nothing runs on push or schedule. Each run downloads the official Geekbench build from `cdn.geekbench.com`, runs it, and posts the result URL to the run summary. Logs and the result URL are saved as downloadable artifacts.

## Workflows

| Workflow | File | Runner |
|---|---|---|
| Geekbench (Linux) | `geekbench-linux.yml` | `ubuntu-latest` |
| Geekbench (macOS) | `geekbench-macos.yml` | `macos-latest` (Apple Silicon) |
| Geekbench (Windows) | `geekbench-windows.yml` | `windows-latest` |
| Geekbench (all platforms) | `geekbench-all.yml` | matrix of all three |

`_geekbench.yml` is a shared reusable core (`workflow_call`). The four listed workflows call it and are not run directly.

## How to run

1. Go to the **Actions** tab.
2. Pick a workflow, click **Run workflow**.
3. Override inputs if needed, then confirm.

### Inputs

| Input | Default | Description |
|---|---|---|
| `version` | `6.7.1` | Geekbench version to download. |
| `extra_args` | `--cpu` | Args passed verbatim to the `geekbench6` CLI. |

`extra_args` accepts any flag the CLI supports. GPU benchmarks are not available on hosted runners, so stick to `--cpu` or `--cpu --multi-core`.

## Results

After a run completes:

- The **run summary** shows a `https://browser.geekbench.com/...` link.
- The artifact `geekbench-<os>` contains:
  - `geekbench.log`: full CLI output
  - `result-url.txt`: the result URL
  - `result.json`: Pro license only (see below)

Artifacts are kept for 30 days.

## Geekbench Pro license (optional)

Without a license, results are uploaded to the Geekbench Browser but not exported locally. With one, each run also writes `result.json`.

To enable, add two repository secrets under **Settings → Secrets and variables → Actions**:

- `GEEKBENCH_EMAIL`
- `GEEKBENCH_KEY`

When both secrets are present, the workflow authenticates and passes `--export-json result.json` to the CLI automatically.

## Notes

**Version bumps:** set the `version` input at run time. No file edits needed as long as the CDN keeps the `Geekbench-<version>-{Linux.tar.gz,Mac.zip,Windows.zip}` naming pattern.

**No installers:** every platform downloads and extracts a portable build (the official `Geekbench-<version>-Windows.zip` on Windows), so no admin install step is needed and the three jobs share the same flow.
