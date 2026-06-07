# Hubitat Apps & Drivers

Apps and drivers for [Hubitat Elevation](https://hubitat.com/) hubs. Apps generally install from
a single Groovy file and, where applicable, serve a web dashboard directly from the hub; drivers
install from a single Groovy file in the **Drivers Code** editor.

<!-- AUTO:packages -->
## Apps

- [Hub Diagnostics](HubDiagnostics/) — Diagnostic dashboard for a Hubitat hub: real-time and historical visibility into devices, apps, network, performance, and configuration, served as a web UI from the hub. Also exposes a read-only audit API.
- [Multi-Hub Inventory](MultiHubInventory/) — Read-only cross-hub aggregator that consumes the Hub Diagnostics audit API from every hub in your fleet for unified device, firmware-drift, and maintenance reports.
- [Humidity Fan Controller](HumidityFanController/) — Bathroom-fan automation that runs while humidity stays above a reference baseline and stops once it returns, with debounced state transitions and multi-sensor median input.
- [Switch Monitor](SwitchMonitor/) — Watches groups of switches that must stay on (or off): auto-corrects deviations after a grace period, retries to a configurable limit, and notifies on under-watt drops for power-metered loads.
- [Log Monitor](LogMonitor/) — Hub log aggregator: WebSocket bridges to one or more hubs with independent filter configs that route matched lines to notifications, files, or HTTP endpoints.

## Drivers

- [Aqara WSDCGQ11LM](Aqara_WSDCGQ11LM/) — Zigbee driver for the Xiaomi Aqara temperature, humidity, and pressure sensor (WSDCGQ11LM).
- [Xfinity Contact Sensor](XfinityContactSensor/) — Zigbee driver for the Xfinity / Visonic door/window contact sensor with battery reporting.
- [IKEA Window Blinds](IKEA-Blinds/) — Zigbee driver for IKEA window blind (FYRTUR / KADRILJ family).
- [Sinopé Switch + Dimmer](sinope/) — Zigbee drivers for the Sinopé SW2500ZB switch and DM2500ZB dimmer.
<!-- /AUTO -->

## Installing

For apps: import the `.groovy` file into the Hubitat **Apps Code** editor — bundled apps that
ship a `*_ui.html` download it into File Manager on first save, so no separate upload is needed.
For drivers: import the `.groovy` file into the **Drivers Code** editor. See each package's README
where present, or the file's header comments, for setup specifics.
