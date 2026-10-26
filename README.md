# geekbench-actions

Run [Geekbench 7](https://www.geekbench.com/) CPU benchmarks on GitHub-hosted runners, on demand. Every workflow is manual-only (`workflow_dispatch`), so nothing runs on push or schedule. Each run downloads the official Geekbench build from `cdn.geekbench.com`, runs it, and posts the result URL to the run summary. Logs and the result URL are saved as downloadable artifacts.

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
| `version` | `7.0.0` | Geekbench version to download. |
| `extra_args` | `--cpu` | Args passed to the Geekbench CLI, split into words; quoting does not survive. |

`extra_args` accepts any flag the CLI supports; it is split into words rather than parsed as shell input, and on Linux and macOS globs expand. GPU benchmarks are not available on hosted runners, so stick to `--cpu` or `--cpu --multi-core`.

## Results

Two runs of each workflow on 2026-10-26, all Geekbench 7.0.0. Which CPU a runner label hands out does not follow from the label: `macos-latest` gave an Apple M1 with 3 cores and, 22 minutes later, an M2 Pro with 5; `windows-latest` an EPYC 9V45 and then an EPYC 7763. Both `ubuntu-latest` runs drew the same EPYC 7763 and agreed within 2%. Read each row as the CPU that run was handed, not as a score the label owns.

| Workflow | Runner | CPU the runner handed out | Single-core | Multi-core | Result |
|---|---|---|---|---|---|
| Geekbench (Linux) | `ubuntu-latest`, Ubuntu 24.04.5 LTS | AMD EPYC 7763, 2 cores / 4 threads | 1477 | 3343 | [`729863`](https://browser.geekbench.com/v7/cpu/729863) |
| Geekbench (Linux) | `ubuntu-latest`, Ubuntu 24.04.5 LTS | AMD EPYC 7763, 2 cores / 4 threads | 1503 | 3397 | [`730025`](https://browser.geekbench.com/v7/cpu/730025) |
| Geekbench (macOS) | `macos-latest`, macOS 26.6.2 | Apple M1 (virtual), 3 cores | 1435 | 3438 | [`729866`](https://browser.geekbench.com/v7/cpu/729866) |
| Geekbench (macOS) | `macos-latest`, macOS 26.6.2 | Apple M2 Pro (virtual), 5 cores | 1952 | 7712 | [`730010`](https://browser.geekbench.com/v7/cpu/730010) |
| Geekbench (Windows) | `windows-latest`, Windows Server 2025 Datacenter | AMD EPYC 9V45, 2 cores / 4 threads | 2153 | 5299 | [`729849`](https://browser.geekbench.com/v7/cpu/729849) |
| Geekbench (Windows) | `windows-latest`, Windows Server 2025 Datacenter | AMD EPYC 7763, 2 cores / 4 threads | 1435 | 3106 | [`730030`](https://browser.geekbench.com/v7/cpu/730030) |

The scores live only on that Geekbench Browser page; `geekbench.log` carries the system and CPU block and the upload lines, not the numbers.

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

When both secrets are present, the workflow authenticates and passes `--export-json result.json` to the CLI automatically. Setting only one of the two warns and skips the JSON export.

## Notes

**Version bumps:** set the `version` input at run time. No file edits are needed as long as the CDN keeps the `Geekbench-<version>-{Linux.tar.gz,Mac.zip,Windows.zip}` naming pattern and the launcher inside the archive stays `geekbench<major>` — as it is for 6 and 7.

**Requirements:** Geekbench 7 asks for Ubuntu 22.04+, macOS 15+ or Windows 11+ ([download page](https://www.geekbench.com/download/)).

**No installers:** every platform downloads and extracts a portable build (the official `Geekbench-<version>-Windows.zip` on Windows), so no admin install step is needed and the three jobs share the same flow.
