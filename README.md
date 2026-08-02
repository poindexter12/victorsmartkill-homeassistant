# Victor Smart-Kill

Home Assistant integration for Victor Smart-Kill WI-FI electronic mouse and rat traps from [VictorPest.com]. This integration is open source and not made by VictorPest.com.

<img src="https://play-lh.googleusercontent.com/tfSd-O7Qwc8p8kYzTbJlDlq-nZzUyRHCMEvM87155kTtwEpVP7iNMgNzg2gWujjZ0jmq=s360-rw" width="128" alt="Victor logo">

[![GitHub Release][releases-shield]][releases]
[![License][license-shield]](LICENSE)
[![Lint][lint-shield]][lint]
[![Validate][validate-shield]][validate]
[![CodeQL][codeql-shield]][codeql]

[![hacs][hacsbadge]][hacs-custom]
![Project Maintenance][maintenance-shield]

![image-m1-mouse-trap](https://user-images.githubusercontent.com/12134766/154821889-45f78843-bcd3-4d67-844a-5767fb0709b4.png)
![image-m2-rat-trap](https://user-images.githubusercontent.com/12134766/154821879-3a3fffeb-964d-4a06-9b65-866c8e4cfb6a.png)

![example][exampleimg]

## Download the integration

### HACS download (alternative 1)

This fork is not in the HACS default store, so add it as a custom repository:

1. In HACS, open the three-dot menu (top right) and select **Custom repositories**.
2. Add `https://github.com/poindexter12/victorsmartkill-homeassistant` with type **Integration**.
3. Search for **Victor Smart-Kill** in HACS and download it.
4. Restart Home Assistant (Settings -> System -> Restart).

### Script download (alternative 2)

You need some kind of terminal to use this method. You can use one of the SSH add-ons from the [Add-on Store](https://my.home-assistant.io/redirect/_change/?redirect=supervisor_store%2) if you run HassOS.

1. Open a terminal. Change directory to your Home Assistant configuration directory (where you find `configuration.yaml`) if you are not using HassOS.
2. Run this script

```
wget -O - https://raw.githubusercontent.com/poindexter12/victorsmartkill-homeassistant/master/get | bash -
```

3. Restart Home Assistant (Settings -> System -> Restart)
4. Before the integration can show up in the list of integrations, you need to clear your browser cache or perform a hard refresh.

### Manual download (alternative 3)

You need some kind of terminal to use this method. You can use one of the SSH add-ons from the Add-on Store if you run HassOS.

1. Open a terminal and change to the directory for your HA configuration (where you find `configuration.yaml`).
2. If you do not have a `custom_components` directory there, you need to create it.
3. Change directory to `custom_components`.
4. Download the latest version

```
wget https://github.com/poindexter12/victorsmartkill-homeassistant/releases/latest/download/victorsmartkill.zip
```

5. Unzip victorsmartkill.zip into folder victorsmartkill

```
unzip victorsmartkill.zip -d victorsmartkill
```

6. Delete victorsmartkill.zip

```
rm victorsmartkill.zip
```

7. Restart Home Assistant (Settings -> System -> Restart)
8. Before the integration can show up in the list of integrations, you need to clear your browser cache or perform a hard refresh.

## Install the integration

Once the integration has been downloaded and Home Assistant has been restarted, go to Settings -> Devices & services (http://homeassistant.local:8123/config/integrations) and click **Add integration**, then select **Victor Smart-Kill** and sign in with your VictorPest account. **You may have to clear your browser cache or perform a hard refresh if the integration is missing from the list.** Check the [log](http://homeassistant.local:8123/config/logs) if you still have problems.

# Entities

Each trap on your account becomes a device with the entities below. The trap is polled through the Victor cloud API (default every 10 minutes — the traps themselves only report to the cloud periodically to save battery, so this is not a real-time feed).

| Entity | Type | Description |
| ------ | ---- | ----------- |
| `binary_sensor.<trap>_capture` | Occupancy | **On** when the trap currently holds a kill (`kills present > 0`). This is the entity to automate on. |
| `sensor.<trap>_kills_present` | Diagnostic | Number of kills currently in the trap |
| `sensor.<trap>_total_kills` | Diagnostic | Lifetime kill count |
| `sensor.<trap>_total_escapes` | Diagnostic | Lifetime escape count |
| `sensor.<trap>_total_retreats` | Diagnostic | Lifetime retreat count (disabled by default; enable it from the entity settings) |
| `sensor.<trap>_battery_level` | Diagnostic | Battery level in % |
| `sensor.<trap>_wireless_network_rssi` | Diagnostic | WiFi signal strength in dBm |
| `sensor.<trap>_temperature` | Diagnostic | Temperature in °C (only enabled if your trap model reports it) |
| `sensor.<trap>_last_kill_date` | Diagnostic | Timestamp of the most recent kill |
| `sensor.<trap>_last_report_date` | Diagnostic | Timestamp of the trap's last report to the cloud |

The capture binary sensor carries extra state attributes: battery level, SSID, last report date, and the trap's latitude/longitude if set. The sensors carry a trimmed subset (each excludes attributes that would duplicate its own state or don't apply).

## Example automation

```yaml
automation:
  - alias: "Notify on mouse trap capture"
    triggers:
      - trigger: state
        entity_id: binary_sensor.garage_trap_capture
        to: "on"
    actions:
      - action: notify.mobile_app_your_phone
        data:
          title: "Mouse trap"
          message: "The garage trap has a capture."
```

# Options

Open Settings -> Devices & services -> Victor Smart-Kill -> **Configure** to change:

- **Scan interval in minutes** — how often the cloud API is polled (default 10).
- **Binary sensor / Sensor enabled** — turn either platform off entirely.

Changing options reloads the integration automatically. New traps added to your account are picked up automatically as well (the integration fires a `victorsmartkill_trap_list_changed` event and reloads itself when the trap list changes).

# Troubleshooting

- **Re-authentication**: if your VictorPest password changes, the integration marks itself as needing reauthentication — open the reauth prompt on the integration card and sign in again.
- **Debug logging**: add this to `configuration.yaml` (or use the integration's *Enable debug logging* button) and restart:

  ```yaml
  logger:
    logs:
      custom_components.victorsmartkill: debug
  ```

- **Diagnostics**: the integration supports diagnostics downloads (integration page -> three-dot menu -> *Download diagnostics*). Sensitive fields (credentials, serial numbers, coordinates, WiFi SSID) are redacted automatically — safe to attach to an issue.
- **System health**: Settings -> System -> Repairs -> three-dot menu -> *System information* shows whether the Victor API endpoint is reachable.

# Development

See [CONTRIBUTING.md](CONTRIBUTING.md). Short version: Python >= 3.14.2, `scripts/setup` to install dependencies, `scripts/develop` to run a local Home Assistant with the integration loaded, `scripts/lint` before pushing. A VS Code devcontainer is included.

# Trap models and versions

This integration should work with traps that have been provisioned with the VictorPest app. Please create an [issue](https://github.com/poindexter12/victorsmartkill-homeassistant/issues/new/choose) if you have trouble with your trap. Please see the upstream wiki if you are interested in details about the [hardware](https://github.com/toreamun/victorsmartkill-homeassistant/wiki/Hardware).

It is very helpful if you can check the [list of models](https://github.com/toreamun/victorsmartkill-homeassistant/wiki/Hardware#list-of-known-traps-versions) and update the list if you have an unlisted trap or version.

# Credits

This is a maintained fork of [toreamun/victorsmartkill-homeassistant](https://github.com/toreamun/victorsmartkill-homeassistant), originally created by [Tore Amundsen](https://github.com/toreamun) ([buy him a coffee](https://www.buymeacoffee.com/toreamun)). It also builds on his [victor-smart-kill](https://pypi.org/project/victor-smart-kill/) Python library. This fork updates the integration for current Home Assistant releases.

[victorpest.com]: https://www.victorpest.com/
[hacsbadge]: https://img.shields.io/badge/HACS-Custom-41BDF5.svg
[hacs-custom]: https://hacs.xyz/docs/faq/custom_repositories/
[license-shield]: https://img.shields.io/github/license/poindexter12/victorsmartkill-homeassistant
[maintenance-shield]: https://img.shields.io/badge/maintainer-%40poindexter12-blue.svg
[releases-shield]: https://img.shields.io/github/v/release/poindexter12/victorsmartkill-homeassistant
[releases]: https://github.com/poindexter12/victorsmartkill-homeassistant/releases
[lint-shield]: https://github.com/poindexter12/victorsmartkill-homeassistant/actions/workflows/lint.yml/badge.svg
[lint]: https://github.com/poindexter12/victorsmartkill-homeassistant/actions/workflows/lint.yml
[validate-shield]: https://github.com/poindexter12/victorsmartkill-homeassistant/actions/workflows/validate.yaml/badge.svg
[validate]: https://github.com/poindexter12/victorsmartkill-homeassistant/actions/workflows/validate.yaml
[codeql-shield]: https://github.com/poindexter12/victorsmartkill-homeassistant/actions/workflows/codeql.yml/badge.svg
[codeql]: https://github.com/poindexter12/victorsmartkill-homeassistant/actions/workflows/codeql.yml
[exampleimg]: https://raw.githubusercontent.com/poindexter12/victorsmartkill-homeassistant/master/example.png
