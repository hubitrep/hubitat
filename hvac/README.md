<!--
Copyright (c) 2026 PJ
SPDX-License-Identifier: MIT
-->

# HVAC Apps

Apps for heating and cooling. Each is an optional component of the HVAC Apps package in Hubitat Package Manager: install the ones you need. HVAC Season Manager is in beta.

- [Switched Heater Thermostat](#switched-heater-thermostat): Turns plain switches feeding resistive heaters into a thermostat: a Virtual Thermostat decides, the app drives the switches, with frost protection, power-draw checks and alerts when a heater or sensor fails.
- [HVAC Season Manager](#hvac-season-manager): Publishes the heating and cooling season (winter, spring, summer, fall) on a device, from dates fitted to the local climate and the outdoor temperature.

## Switched Heater Thermostat

Turns plain switches (smart plugs or relays feeding resistive heaters) into a thermostat, and tells you when it cannot do the job.

The app reads one or more temperature sensors and feeds the combined reading to a Virtual Thermostat. It turns the heater switches on and off as that thermostat's operating state changes. It protects against frost whatever the thermostat mode, and it raises an alert when a heater does not respond or draws the wrong power, when the heaters run without warming the room or past a time limit, or when no sensor reports.

### Safety

A Hubitat hub is not a safety device, and this app is not certified to control heaters. A lost radio message, a stuck relay, a hub restart or a bug can leave a heater on or off. The alerts report some of these faults after they happen; they do not prevent them. You use this app at your own risk; it comes with no warranty (MIT License).

- Use heaters with their own protection: a built-in thermostat or overheat cut-out, and a tip-over switch on portable heaters. The heater must stay safe if its switch stays on.
- Use switches rated for the heater's load, with margin. A plug or relay running near its rating gets hot.
- Follow the heater maker's instructions on what it may be plugged into and on leaving it unattended. Many portable heaters must go straight into a wall outlet; a smart plug sits in between.
- Have a relay on a hardwired heater installed to your local electrical code.

### Files

| File | Type | Name on the hub |
|---|---|---|
| `SwitchedHeaterThermostat.groovy` | App | Switched Heater Thermostat |

### Install

1. Add the app in **Apps Code**, then choose **Add user app** and pick **Switched Heater Thermostat**.
2. Under **Thermostat**, pick an existing Virtual Thermostat (built-in driver) or turn on **Create a Virtual Thermostat for this app**. Another driver is refused. A created thermostat is deleted with the app. If you later pick a different thermostat, a button removes the unused created one. Uninstalling the app turns the current heaters and fault switches off. A heater you removed from the list earlier is left as it was.
3. Pick the heater switches and the temperature sensors.
4. Choose what the heaters do when no sensor reports, and the alert devices.
5. Press **Done**.
6. Set the thermostat's mode to Heat. A created thermostat starts in mode Off.

On every Done the app sets the thermostat's supported modes to `off` and `heat`. Setpoints and mode stay on the thermostat, so dashboards, voice assistants and schedulers control it like any thermostat.

Keep the Virtual Thermostat's **Enable demonstration mode** preference off. With it on, the device changes its own temperature every 2 seconds and the heaters follow that instead of the room.

### How heating is decided

The app writes the combined sensor reading to the thermostat with `setTemperature`. The sensors are combined by average (default) or minimum, rounded to 0.1 degree (°C or °F, following the hub's scale), ignoring sensors that are quiet (see Quiet sensors). The thermostat decides `heating` or `idle` itself, using its hysteresis preference (default 0.5 degree). With heat mode, setpoint 20 and the default hysteresis (checked on firmware 2.5.2.134), heating starts at 19.4, continues through 20.4, stops at 20.6 and stays off down to 19.6. Change the hysteresis on the thermostat to change that band. When the heating setpoint changes, the app writes the last reading again.

The heaters' wanted state, in order of precedence:

1. Frost protection running: on.
2. Thermostat mode not `heat`: off.
3. Resting after the heating time limit: off.
4. No sensor reports: your choice of off, on or a fixed cycle.
5. Otherwise on while the thermostat is `heating`, off when it is anything else.

The app is the only writer of the heater switches. A heater switched by hand, or by another automation, is set back at once. If another automation keeps writing the same switches, the app reports a load fault. The fault switches below are owned the same way.

When the configuration is unusable (no thermostat, a thermostat on another driver, a heater also picked as a fault switch), the app turns the heaters off and does nothing else until the page is fixed.

### Frost protection

Set **Frost protection** (°C or °F) to turn the heaters on when the combined temperature falls below it, whatever the thermostat mode and the heating time limit rest. The heaters stay on until the temperature is 1 degree above the setting. It needs a live reading: with every sensor quiet, frost protection is off and the choice for no sensor applies. A notification goes out when it starts and when it stops, and the status at the top of the page shows it while it runs. It is not a fault, turns on no switch, and does not count toward the heating time limit. Blank turns it off.

### Minimum on and off times

**Keep the heaters on for at least** and **off for at least** (1 to 60 minutes, blank for none) hold the heaters in their state after they last changed. They delay only changes that come from the thermostat. Frost protection, thermostat mode off, the heating time limit and a sensor fault act at once.

### Keep-alive

**Re-send the heaters' state every** (1 to 120 minutes, blank for none) sends the wanted state to every heater at that interval, even when it already reports it. Use it for switches that turn themselves off without reporting it, and for one-way relays whose reported state is only an echo of the last command. Re-sends are not verified.

### Checking the heaters

After a command, the app checks each heater after **Time a heater has to confirm a command** (5 to 300 seconds, default 30). A heater that has not reached the commanded state gets a second command. If it still has not, it is unresponsive and raises the load fault. Every 5-minute check re-applies the wanted state, which retries unresponsive heaters.

#### Power checks

For heaters that report `power`, and while **Check the power draw of heaters that report it** is on (default on), the app compares the draw with the switch state. The line between drawing and not drawing is **A heater that is on draws at least** (W, default 20):

- Switch on and draw under the line: the heater is not heating (unplugged, tripped, its own thermostat or safety cut-out open).
- Switch off and draw at or above the line: the switch does not cut the load (welded relay, or a switch that reports the wrong state).

A mismatch must last 2 minutes before it counts, which covers meters that report some seconds after the switch. A space heater with its own thermostat stops drawing when that thermostat is satisfied, which looks like the first case. Set the heater's own thermostat to its highest setting, or turn the power check off. Heaters without a meter get the switch-state checks only.

#### Command Retry

Hubitat's Command Retry (Settings, Command Retry) makes the hub resend a command that a radio device did not acknowledge, up to 5 times. For heaters that can use it but have it off, the page and the log (a warning on every Done) say: *Command Retry is off for <heaters>. Turning it on (Settings, Command Retry) lets the hub resend a missed on or off before this app reports a load fault.* This is advice only: no notification, no fault, and the app never changes the setting. Virtual, LAN and most cloud devices cannot use Command Retry and get no advice.

### Warming check

**Alert when the heaters have been on this long without warming the room** (10 to 240 minutes, blank for off) notes the temperature when the heaters turn on. If they are still on that long afterwards and the temperature has not risen by **Warming means a rise of at least** (°C or °F, default 0.3), the warming fault is raised. It clears when the temperature has risen that much, or when the heaters turn off. The check suggests a cause (an open window, an undersized heater, a misplaced sensor) and turns nothing off. It is skipped while no sensor reports.

### Heating time limit

**Longest time the heaters may stay on** (10 to 1440 minutes, blank for no limit) caps how long the heaters stay on without a break. At the limit, the app turns them off for **Then keep them off for** (1 to 240 minutes, default 15), raises the limit fault, then lets them follow the thermostat again. The fault clears the next time the heaters turn off for another reason. The on-time survives a hub restart. The limit applies in every case except frost protection, including "keep on" when no sensor reports.

### When no sensor reports

The choice under **When no sensor reports** decides what the heaters do while every sensor is quiet or none has a reading:

| Choice | Heaters |
|---|---|
| Off (default) | off |
| On | on, still subject to the heating time limit |
| Cycle | on for a share of each cycle (default 30 % of every 30 minutes), starting with the on phase when the fault is raised |

Frost protection needs a live reading, so for rooms where frost matters pick on or cycle. Thermostat mode `off` always wins. During this fault the app sets the thermostat's operating state to match the heaters (`heating` or `idle`), since the thermostat has no temperature to decide from.

### Quiet sensors

A sensor is quiet when its last activity (any event, including battery and check-ins) is older than its quiet limit. Until the app has seen 5 gaps between a sensor's activity, the limit is **Count a sensor as quiet after at least** (30 to 1440 minutes, default 120). After that, the limit is twice the second-longest of the last 48 gaps when that is longer than the setting, so a sensor that reports rarely is not counted quiet early, and one that reports in bursts does not learn a limit shorter than the setting. Using the second-longest keeps a single outage from teaching the app that outages are normal.

A sensor that comes back with an unchanged value sends no event, so the app sees it at the next 5-minute check. One quiet sensor among several is left out of the temperature and notifies once (turn off **Also notify when some sensors go quiet** to skip it). It is not a fault.

### Faults and alerts

| Fault | Raised when | Cleared when |
|---|---|---|
| Sensor fault | every sensor is quiet, or none has a reading | any sensor reports again |
| Load fault | a heater stays unresponsive after its second command, or a power mismatch lasts 2 minutes | every heater responds and draws as expected |
| Limit fault | the heaters reach the heating time limit | the heaters next turn off for another reason |
| Warming fault | the heaters run the set time without the temperature rising enough | the temperature rises that much, or the heaters turn off |

Each fault has two outlets, both optional. **Notification devices** get one message when the fault is raised and one when it clears, with no repeats while it stays raised. A **switch per fault** is on while that fault is raised and off otherwise. One switch may serve several faults; it is then on while any of them is raised. Use these switches to start a backup heater, a siren or a Rule Machine rule. The app owns them and re-asserts them at each check, so do not switch them by hand.

Messages (`<name>` is the app label; temperatures show the hub's scale, such as `4.9 °C`):

- `<name>: no temperature, all sensors quiet (<sensors>). <policy>.` where the policy reads `Heat is off`, `Heaters kept on` or `Heaters on <d>% of every <p> minutes`
- `<name>: temperature back (<sensors>).`
- `<name>: lost control of <heaters>.` where each heater reads `<label> (commanded <on|off>, reads <value>)` or `<label> (<on|off>, draws <n> W)`
- `<name>: heaters under control again.`
- `<name>: heaters on for <max> minutes, the limit. Off for <rest> minutes.`
- `<name>: heating time back to normal.`
- `<name>: heaters on for <n> minutes and the temperature rose <rise> (expected <min>).`
- `<name>: warming check back to normal.`
- `<name>: frost protection on (<temperature>).` and `<name>: frost protection off (<temperature>).`
- `<name>: <sensors> quiet, left out of the temperature.`
- `<name>: <sensors> reporting again.`

### Restarts and code updates

On hub start, on every Done and on the first event after a code update, the app samples sensor activity, rewrites the temperature, re-applies the heaters (restarting the limit, rest, cycle, minimum-time, warming and keep-alive timers from its saved state), re-evaluates power and re-asserts the fault switches.

### Not in this version

Time-proportional control with an outdoor temperature, cooling, and per-heater power thresholds.

## HVAC Season Manager

Publishes the heating and cooling season on a device, from dates and the outdoor temperature.

HVAC Season Manager decides which of four heating and cooling seasons it is (`winter`, `spring`, `summer`, `fall`) and publishes it on its own device. Rules, dashboards and other apps read the device; nothing else needs to know how the season was decided. It knows nothing about heating equipment: what each piece of equipment may do in each season belongs to whatever reads the device.

### Files

| File | Type | Name on the hub |
|---|---|---|
| `HvacSeasonManager.groovy` | App | HVAC Season Manager |
| `HvacSeasonManagerSeason.groovy` | Driver | HVAC Season |

### Install

1. Add the app in **Apps Code** and the driver in **Drivers Code**.
2. In **Apps**, choose **Add user app** and pick **HVAC Season Manager**.
3. Pick the outdoor temperature sensor if you have one, then press **Done**. The app sets the starting season from the date, or from Open-Meteo's daily means inside a window where the weather decides. It asks only when neither works.

Saving creates the device **"HVAC Season"**. If you rename the app, the device is named "<app name> Season" instead, so a second instance does not create a second "HVAC Season". The device belongs to the app and is deleted with it.

### How the season changes

The year has two boundaries, and they behave differently.

**The winter boundary changes once a year each way.** Winter becomes spring inside a window of dates, as soon as the 3-day mean outdoor temperature is above a threshold, and on the window's last day at the latest. Fall becomes winter the same way, when the mean is below a threshold. Neither change goes back until the next year. This suits heating with a large thermal mass, such as a hydronic floor, which should not start and stop with each warm or cold spell.

**Or the winter boundary follows the forecast both ways** (*Winter boundary*: *Both ways, on the 3-day forecast*). Between *winter possible from* and *until* (Oct 1 to Apr 30), the season becomes winter when the mean of the forecast daily means for the day and the next two is below *winter below* (3 °C), and leaves winter when that mean is above *leave winter above* (6 °C), as often as the weather requires. Out of winter the season is `fall` from the *fall from* date to Dec 31 and `spring` from Jan 1. No date forces winter, and after *until* the season leaves it. Without a forecast the app decides on the mean of the three previous days. This suits a house with two heat sources, where winter selects the one that costs more in mild weather: the forecast starts it before a cold spell and hands back after, and the band keeps it from switching on every mild day. The winter-boundary windows are not used in this mode.

**The summer boundary follows the weather both ways.** Between *summer possible from* and *summer possible until*, the season enters summer when the 3-day mean is above the entry threshold and leaves it when the mean is below the exit threshold, as often as the weather requires. There is no forced start: a cold June stays in spring. Summer always ends on its *until* date. Outside summer, the season is `spring` before the *fall from* date and `fall` from that date on.

A cold spell after summer starts is the costly mistake, since summer usually means the heat is off. Following the weather in and out of summer avoids it; a band of a few degrees between the two thresholds keeps the season from changing every day.

### Settings

| Setting | Default | Notes |
|---|---|---|
| Use Open-Meteo daily means | on | Needs internet and the hub's location |
| Outdoor temperature sensor | | Optional; used on days Open-Meteo has no mean |
| Winter to spring, from / until | 03-11 / 04-08 | `MM-DD`; *until* is the last possible day |
| Winter to spring threshold | 3 °C | 3-day mean above it |
| Fall to winter, from / until | 10-26 / 11-11 | `MM-DD`; *until* is the last possible day |
| Fall to winter threshold | 3 °C | 3-day mean below it |
| Winter boundary | once a year each way | Or both ways on the 3-day forecast |
| Winter possible from / until | 10-01 / 04-30 | Forecast mode only |
| Winter below / leave winter above | 3 °C / 6 °C | Forecast mode only; the 3-day forecast mean, strict; *leave* must be higher |
| Summer possible from / until | 05-10 / 09-14 | Summer ends on *until* |
| Fall from | 08-15 | Outside summer, spring becomes fall on this day |
| Enter summer threshold | 17 °C | 3-day mean above it |
| Leave summer threshold | 12 °C | 3-day mean below it; must be below the entry threshold |
| Summer end threshold | 17 °C | Used only to compute *summer possible until* and *fall from* from local climate |
| Notification devices | | Optional; a message on every season change, with its reason |
| Hub variable mirror | off | See below |

The default dates suit a cold-winter climate. On install, with Open-Meteo on, the app replaces them with dates computed from the last 21 full years of daily means at the hub's location, and the *Compute dates from local climate* button in *Season rules* does the same later, showing the computed dates beside the current ones before any change. For each year it finds the first day the 3-day mean crosses each threshold: a winter window runs from the 10th to the 90th percentile of those days, *summer possible from* is the 10th percentile of the summer entry crossing, and *summer possible until* and *fall from* are the 90th and 10th percentiles of the summer end crossing. South of the equator the searches start half a year later. Each search starts on a fixed day: Feb 15 for winter to spring, Apr 1 for summer, Aug 1 for the summer end and Sep 15 for fall to winter.

A threshold that does not suit the local climate is moved along with the dates. A threshold suits it when the 3-day mean crosses it in at least two thirds of the years, and was already past it on the search's first day in at most a third: otherwise the window would be that day, not the climate's. Such a threshold moves by 0.5 °C (1 °F) steps to the nearest one that suits, between −10 and 18 °C for the winter rules and between 10 and 30 °C for summer. The leave and end thresholds move with the summer entry, keeping the band. The page says which thresholds moved and why, and shows them beside the current ones. A rule that no threshold within those limits suits keeps its dates and threshold. Thresholds that suit the climate are never changed, so cold-winter sites keep the defaults.

Temperatures use the hub's scale; on a °F hub the defaults are converted. The dates must follow each other through the year in this order: the winter to spring window, *summer possible from*, *fall from*, *summer possible until*, the fall to winter window. The page shows an error and the season stops changing until they do. Each date field opens a month-and-day calendar, and the *Season rules* section draws the windows on a 12-month strip with today marked.

### Fixed behavior

These are not settings:

- Daily means come from the Open-Meteo archive at the hub's location, rounded to 0.1°. This is the data the default thresholds were calibrated on. The app logs a warning once while Open-Meteo is unavailable.
- On a day Open-Meteo has no mean, the sensor's mean is used, corrected by its mean difference from Open-Meteo over the last 7 days that have both. The sensor is read every hour, and a day's mean needs at least 12 readings. Readings are kept for 10 days.
- The season is checked once a day at 05:00, against the mean of the three previous daily means, and in forecast mode against the forecast for the day and the next two. The check runs after the forecast is fetched; if the fetch fails, the app logs a warning once and uses the previous three days.
- A sensor reading older than 24 hours is not counted, and the app logs a warning once until the sensor reports again. Without a 3-day mean, only the date limits apply: a winter boundary change happens on its window's last day, and summer ends on its *until* date.
- The season changes at most once per day.
- If the hub was off for some days, the next daily check evaluates each missed day in order, up to 120 days back.
- On a hub restart nothing is recomputed: the season stays what it was.

### The HVAC Season device

| Attribute | Values |
|---|---|
| `season` | `winter`, `spring`, `summer`, `fall` |

| Command | What it does |
|---|---|
| `setSeason(season, holdDays)` | Sets the season at once and suspends automatic changes for `holdDays` days (default 3, 0 for none). When the hold ends, automatic changes continue from the season you set. |
| `resumeAuto()` | Ends the hold. The next daily check applies the rules. |

The same controls are on the app page, with the current season, the last three daily means and where each came from, the sensor's offset, and the next possible change.

### Outlook

Each morning the app also fetches Open-Meteo's 7-day forecast of daily means and runs the same rules forward from the current season. The app page lists the forecast means and every change the rules would make, such as "Forecast: winter on Oct 27 (3-day mean 1.7 °C)", or says the forecast predicts no change. Changes forced by a date are listed with their reason. The outlook never changes the season. Saving the settings fetches the daily means and the forecast at once.

### Hub variable mirror

For rules that still read a hub variable, the app can copy the season to a String variable, with your own label for each season. It marks that variable as in use and follows a rename. Turn the mirror off once nothing reads it.

### Choosing thresholds

The defaults suit a Montréal-area climate and come from the 2005 to 2025 daily means: each window spans the 10th to 90th percentile of the day its threshold was first crossed. At a colder or higher site, lower the summer entry; where the summer normal peaks around 18 °C, an entry of 15 °C and an exit of 12 °C work. Keep the exit at least 3 °C below the entry. A narrower band makes the season change more often for no benefit.

In a warm climate, computing the dates also moves the thresholds that never get crossed. With 2005 to 2025 data: London keeps 3 °C into winter and leaves it above 6 °C; Phoenix enters winter below 7 °C, leaves it above 15.5 °C and enters summer above 21.5 °C; Sydney uses 10.5 °C and 13.5 °C for winter. Where the 3-day mean never falls far below its winter normal, as in Miami, no threshold gives the four seasons in their order: the page reports the dates as invalid, and the app does not fit that climate.
