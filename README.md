# Log Reader

A single-file, client-side web app for analyzing Verifone payment terminal and device diagnostic logs — no server, no build step, no install. Open the HTML file in a browser and start dropping in logs.

Built for troubleshooting:
- **Verifone FIPAYEPS** (Electronic Payment Server) logs used with Oracle Simphony POS
- **Android / embedded OS diagnostic bundles** pulled from payment terminals

Everything runs locally in your browser. Logs never leave your machine.

## Getting started

1. Download [ ] 
2. Open it in a modern browser (Chrome, Edge, or Firefox)
3. Drop in a log file/archive, or click **Browse files**

That's it — no dependencies to install, no dev server to run.

## Supported inputs

| Type | Formats |
|---|---|
| FIPAYEPS payment logs | `.log`, `.zip` exports containing `fipayeps.log` and `system.day*.log` |
| Device diagnostics | `.tgz` / `.tar.gz` bundles (including nested archives, e.g. `Combined_AndroidLogs.tgz` containing further `.tgz` files) |

You can load both kinds side by side — the app keeps them in separate workspaces (**EPS Payment Logs** / **Device Diagnostics**) switchable from a tab bar.

## Features

### EPS Payment Logs
- **Transaction flow** — log lines are grouped by invoice/receipt number and shown as a step-by-step timeline (POS → EPS → Pinpad → Host → POS), with failed steps highlighted and embedded receipt text rendered distinctly
- **Errors** — all critical/comm-failure lines, filterable by type (TLS/comm, pinpad comm, timeout, blocked response, declined)
- **Raw logs** — full untouched log text, per-file selector, search with highlighting, table/plain-text toggle, copy and download

### Device Diagnostics
Recognizes and parses six log formats out of the box:

| Format | Source |
|---|---|
| Android logcat | `logcat.log`, `data/misc/logd/logcat*` |
| Linux syslog | `messages`, `syslog_self` |
| App log (CAM) | `app.log` |
| SCA transaction log | `scaapp*.log` |
| Installer daemon log | `installerd.log` and related |
| Kernel log (dmesg) | `dmesg`, both boot-seconds and wall-clock timestamp variants |

Anything else found in a bundle still loads as searchable plain text — it just won't get structured columns or level detection.

- **Sessions** — transaction/session reconstruction for the two structured formats that have a real lifecycle:
  - **App log (CAM)**: sessions bounded by `STARTUP` markers, steps tagged by `<`/`>` direction (Device ↔ App)
  - **SCA transaction log**: transactions bounded by `Start Session` / `Session Finish` commands, steps tagged by direction (POS ↔ SCA ↔ Host)
- **Errors** — combines real severity levels (Android `E`/`F`, syslog `err`/`crit`, SCA `ERROR`/`FAILURE`, installer `error`/`fatal`) with keyword detection (`Exception`, `panic`, `avc: denied`, `ANR in`, etc.), filterable by type
- **Raw logs** — same plain-text/table toggle, per-file selector, search, copy/download as the EPS workspace
- **Device files** — everything in the bundle that isn't a recognized log (config files, `.db`/`.realm` databases, tombstones) is kept in a separate browsable section, not auto-loaded — pick a file to preview as text or download it

## How it works

Everything happens in the browser:
- `.zip` files are read with [JSZip](https://stuk.github.io/jszip/)
- `.tgz` / `.tar.gz` files are gunzipped with [pako](https://github.com/nodeca/pako) and unpacked with a small hand-written USTAR/GNU-tar reader (handles nested archives up to a few levels deep)
- Large archives show a picker so you can choose which files to actually load, with search to help you find a specific file in bundles with hundreds of entries
- Multi-line records (e.g. embedded XML payloads in transaction logs) are automatically merged back into their parent log entry instead of being split into orphaned lines

No data is sent anywhere — there's no backend at all.

## Known limitations

- **Year inference**: Android logcat and Linux syslog timestamps don't include a year. It's inferred from the uploaded archive's filename (e.g. `..._20260916_...`). If a bundle's real capture date doesn't match its filename, timestamps for that file will be off by a year.
- **Heuristic grouping**: Transaction/session/invoice grouping is based on marker patterns in the log text (e.g. `Inv#`, `Eps#`, `STARTUP`, `Start Session Command`). It's a strong heuristic, not a guaranteed boundary — occasionally a line right at a transition point may land in the wrong group.
- **Generic fallback**: log files that don't match one of the recognized formats are still fully viewable and searchable, but won't get timestamps, levels, or session/flow treatment.

## Tech stack

Plain HTML/CSS/JavaScript — no framework, no build tooling. External libraries (JSZip, pako) are loaded from CDN.

## License

No license has been chosen yet — add one (e.g. MIT) if you plan to share this outside your organization.
