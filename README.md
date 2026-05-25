# Hubitat Apps

Apps for monitoring and inventorying [Hubitat Elevation](https://hubitat.com/) hubs. Each app
installs from a single Groovy file and serves its own web dashboard directly from the hub.

## Apps

### [Hub Diagnostics](HubDiagnostics/README.md)

A comprehensive diagnostic dashboard for a single Hubitat hub — real-time and historical
visibility into devices, apps, network health, performance, and configuration, all in one web UI
served from the hub. It also exposes a read-only audit API that other tools can consume.

### [Multi-Hub Inventory](MultiHubInventory/README.md)

A read-only, cross-hub view that aggregates the audit data from every hub in your fleet into
unified device, firmware-drift, and maintenance reports.

> **Requires Hub Diagnostics.** Multi-Hub Inventory has no data of its own — it reads the audit
> API that Hub Diagnostics exposes. Every hub you want to include must already be running a
> configured Hub Diagnostics instance. See the
> [Multi-Hub Inventory requirements](MultiHubInventory/README.md#requirements) for details.

## Installing

Each app is self-contained: import its `.groovy` file into the Hubitat **Apps Code** editor, and
the app downloads its own web dashboard (the matching `*_ui.html`) into File Manager — no separate
upload needed. See each app's README for step-by-step instructions.
