# Copilot Instructions for ha_lionel_controller

These instructions help an AI coding agent be productive in this Home Assistant custom integration.

**Repository Overview:**
- **Domain:** `lionel_controller` (custom integration in `custom_components/lionel_controller`).
- **Purpose:** Control Lionel LionChief Bluetooth locomotives via BLE using `bleak` and `bleak-retry-connector`.
- **Entry points:** `__init__.py` (coordinator & setup), `config_flow.py` (config entry), platform files (`button.py`, `switch.py`, `number.py`, `sensor.py`, `binary_sensor.py`).

**High-level architecture & data flow**
- A single `LionelTrainCoordinator` (in `__init__.py`) manages BLE connection state, parses notifications, and exposes async methods to send commands.
- Platforms (entities) access the coordinator via `hass.data[DOMAIN][entry.entry_id]` and call coordinator methods like `async_set_speed`, `async_set_direction`, `async_play_announcement`.
- Config flows create a config entry with `CONF_MAC_ADDRESS`, `CONF_NAME`, and `CONF_SERVICE_UUID` stored in `entry.data` and used to initialize the coordinator.

**Project-specific conventions and patterns**
- Coordinator-first pattern: keep connection, parsing, and command encoding inside `LionelTrainCoordinator` and keep platform code thin (only mapping UI actions to coordinator calls). See `custom_components/lionel_controller/button.py` for examples.
- Unique IDs: entity `unique_id`s use the locomotive MAC address as a prefix (e.g. `{mac}_stop`). Use `coordinator.mac_address` when constructing IDs.
- Device info: platforms set `self._attr_device_info` using `coordinator.device_info` for consistent device registration.
- Bluetooth discovery: config flow supports both user-entered MAC addresses and BLE discovery (`async_step_bluetooth`), and validates MAC format in `config_flow._is_valid_mac_address`.

**Important files to inspect when making changes**
- `custom_components/lionel_controller/__init__.py` — coordinator, setup/unload logic, BLE handling, notification parsing, connection retries.
- `custom_components/lionel_controller/config_flow.py` — validation, discovery, config entry population.
- `custom_components/lionel_controller/const.py` — command codes, UUIDs, defaults, and helper builders like `build_command` / `build_simple_command`.
- Platform files (`button.py`, `switch.py`, `number.py`, `sensor.py`, `binary_sensor.py`) — entity patterns and expected coordinator APIs.
- `manifest.json` — runtime requirements (e.g. `bleak`, `bleak-retry-connector`) and bluetooth discovery metadata.
- `services.yaml` and `strings.json` — service definitions and localized strings; mirror changes here when adding services or translations.

**Diagnostics, logging & debugging**
- Use the integration logger name (module `__name__` in `__init__.py`) to enable debug logging in Home Assistant: set `logger.lionel_controller: debug` in `configuration.yaml` (or via UI). Many debug messages exist around BLE discovery and notification parsing.
- BLE discovery logs all services/characteristics (`_log_ble_characteristics`) — prefer reading these logs when troubleshooting device-specific UUID issues.

**Behavioral rules for edits**
- Preserve the coordinator-centered design: new features that talk to the locomotive should add methods to `LionelTrainCoordinator` (async, use the existing `_lock`), and platforms should call these methods.
- Keep entity availability tied to `coordinator.connected` and call `self._coordinator.add_update_callback` / `remove_update_callback` if an entity needs push updates.
- When modifying commands or protocol parsing, update both `const.py` helpers and the parsing logic in `_notification_handler` so state is in sync.

**Examples of common changes**
- Add a new button that triggers a command: create platform class in `button.py` that calls `await coordinator.async_play_announcement(code)` and follows the existing `LionelTrainAnnouncementButton` pattern.
- Add a new characteristic read at connect time: extend `_read_device_info` in `__init__.py` and add a corresponding entry to `device_info` so platforms can expose it.

**Testing & runtime**
- There are no unit tests in the repo; manual testing is via Home Assistant. To develop locally:
  - Install dependencies from `manifest.json` into your HA environment (or use a devcontainer). Key packages: `bleak`, `bleak-retry-connector`.
  - Add the `custom_components/lionel_controller` folder to Home Assistant `custom_components` and restart HA.
  - Enable debug logs for the domain to observe BLE discovery and commands.

**Quick grep patterns**
- Search for coordinator methods: `async_set_` or `async_play_` to find command entrypoints.
- Look for `CONF_MAC_ADDRESS` and `CONF_SERVICE_UUID` in `config_flow.py` and `__init__.py` to track config entry usage.

If anything above is unclear or you want more detail (examples of tests, CI, or a runbook for manual QA), tell me which area to expand. I can iterate on this file.
