# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Home Assistant custom integration (HACS-compatible) for Victor Smart-Kill WiFi mouse/rat traps. It polls the Victor cloud API via the [victor-smart-kill](https://pypi.org/project/victor-smart-kill/) library — there is no local device access. This is a maintained fork of toreamun/victorsmartkill-homeassistant; the hardware wiki still lives upstream.

## Commands

```bash
scripts/setup    # pip install -r requirements.txt (needs Python >= 3.14.2 for current homeassistant)
scripts/lint     # ruff format . && ruff check . --fix
scripts/develop  # run a local HA instance with this integration, debugpy on localhost:5678
```

CI runs the non-mutating equivalents: `ruff check .` and `ruff format --diff --target-version=py314 .` (lint.yml), plus hassfest and HACS validation (validate.yaml) and CodeQL. There is no test suite.

Ruff is configured in `.ruff.toml` with `select = ["ALL"]` and a short ignore list — expect strict linting; prefer a targeted `# noqa: RULE reason` over adding global ignores.

## Architecture

Everything lives in `custom_components/victorsmartkill/`. Modules use absolute imports (`from custom_components.victorsmartkill...`), so anything that imports the package needs the repo root on `sys.path` — scripts/develop achieves this by `cd`-ing to the repo root before launching (`python -m` puts the cwd on `sys.path`).

Data flow, defined in `__init__.py`:

- `VictorSmartKillDataUpdateCoordinator` owns the API client pair (`VictorAsyncClient` + `VictorApi`) and polls `get_traps()` on the configured interval. Coordinator data is `list[victor.Trap]`; entities never talk to the API directly.
- `async_setup_entry` stores an `IntegrationContext` (coordinator + unsubscribe callbacks) on `entry.runtime_data` (typed alias `VictorSmartKillConfigEntry`), then forwards setup to the enabled platforms.
- If the set of trap IDs changes between polls, the coordinator fires the `victorsmartkill_trap_list_changed` event, and a listener reloads the whole config entry to recreate entities. Options changes also reload via an update listener.
- On auth failure the coordinator raises `ConfigEntryAuthFailed`, which triggers the reauth flow in `config_flow.py` (reauth clears the stored password until new credentials validate).

Entity layer:

- `entity.py` defines the abstract `VictorSmartKillEntity` (a `CoordinatorEntity`): looks up its trap by `trap_id` in coordinator data, builds `unique_id` as `victorsmartkill_{trap_id}_{suffix}`, and declares `device_info` with identifier `(DOMAIN, "IDSN:{trap_id}:{serial_number}")`. `_migrate_device_identifiers` in `__init__.py` migrates a legacy 3-tuple identifier format — don't remove it.
- `sensor.py` creates one `VictorSmartKillSensor` per entry in a list of `VictorSmartKillSensorEntityDescription`s (each carries a `value_func` lambda over the Trap dataclass). `binary_sensor.py` has a single occupancy "capture" sensor (`kills_present > 0`).
- Platforms can be individually disabled via options (`config_flow.py` options flow: platform toggles + scan interval).

## Compatibility constraints

- Minimum supported HA is pinned in `hacs.json` (`homeassistant` key); don't use core APIs newer than that floor without raising it. Dependabot deliberately ignores the `homeassistant` pin in requirements.txt for the same reason.
- Version in `manifest.json` uses `YYYY.M.P` format and must match the release tag. Publishing a GitHub release triggers publish.yml, which attaches `victorsmartkill.zip` (required by the README's manual-install instructions).
- User-visible strings live in `translations/` (en, nb, sv); config/options flow changes must keep those keys in sync.
