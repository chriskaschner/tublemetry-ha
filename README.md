# Tublemetry HA -- Home Assistant Packages

Home Assistant package YAML files for the [Tublemetry](https://github.com/chriskaschner/tublemetry) hot tub automation system. The Git Pull add-on syncs this repo to `/config/`, placing the `packages/` directory where HA expects it.

## Files

All package YAML files live in `packages/`:

| File | Description |
|------|-------------|
| `packages/tou_automation.yaml` | Time-of-Use setpoint schedule (Rg-2A rate windows) |
| `packages/thermal_runaway.yaml` | Graduated thermal runaway protection (warning/moderate/severe) |
| `packages/thermal_runaway_clear.yaml` | Auto-clears runaway flag when temperature returns to setpoint |
| `packages/stale_data.yaml` | ESP32 offline detection -- disables TOU on stale sensor data |
| `packages/drift_detection.yaml` | Detects setpoint drift (physical button press or failed injection) |
| `packages/heater_power.yaml` | Enphase-derived heater power validation binary sensor (stubbed) |
| `packages/sensors.yaml` | SQL sensors: heating rate and cooling rate from recorder history |
| `packages/templates.yaml` | Template sensors: outdoor temp, preheat minutes, preheat ETA |
| `packages/thermal_model.yaml` | Full thermal model: SQL rates, template estimates, input_number snapshots, daily refresh automation |
| `packages/helpers.yaml` | Input boolean: `thermal_runaway_active` flag |

## Setup

One-time bootstrap on your HA instance (RPi4 or other):

### 1. Enable packages in configuration.yaml

Update `/config/configuration.yaml`:

```yaml
# Loads default set of integrations. Do not remove.
default_config:

# Load frontend themes from the themes folder
frontend:
  themes: !include_dir_merge_named themes

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

homeassistant:
  packages: !include_dir_named packages/
```

Remove any existing `template: !include templates.yaml` or `sensor: !include sensors.yaml` lines -- those are now handled by packages.

### 2. Install Git Pull add-on

Settings > Add-ons > Add-on Store > search "Git pull" > Install

### 3. Configure Git Pull add-on

In the add-on Configuration tab:

| Setting | Value |
|---------|-------|
| Repository | `https://github.com/chriskaschner/tublemetry-ha` |
| Branch | `main` |

The add-on syncs the entire repo to `/config/`. Since the YAML files are in the `packages/` subdirectory in this repo, they land at `/config/packages/` where HA expects them.

Enable the repeat interval (300s / 5 minutes) to auto-sync, or leave it off and start the add-on manually after pushing changes.

### 4. Remove duplicate helpers (if any)

If `input_boolean.thermal_runaway_active` was previously created via the HA UI (Settings > Helpers), **delete it** before restarting. YAML-defined and UI-defined helpers cannot share the same entity ID.

### 5. Restart HA

Developer Tools > YAML > Check Configuration, then restart.

### 6. Verify

After restart:
- Settings > Automations should show: Hot Tub TOU Schedule, Thermal Runaway Protection, Thermal Runaway Clear, ESP32 Offline Detection, Setpoint Drift Detection, Refresh Thermal Model
- Settings > Helpers should show: Thermal Runaway Active (input_boolean)
- Developer Tools > States > search "hot_tub" should show the template and SQL sensors

After the initial setup, the Git Pull add-on auto-syncs changes. Push to this repo and HA picks up the changes within 5 minutes.

## Reloadable vs Restart-Required

| Domain | After Change | How |
|--------|-------------|-----|
| automation | Reload | Developer Tools > YAML > Automations (or automatic via Git Pull) |
| template | Reload | Developer Tools > YAML > Template Entities |
| input_boolean | Reload | Developer Tools > YAML > Input Booleans |
| input_number | Reload | Developer Tools > YAML > Input Numbers |
| sql (new sensor) | Restart | Requires HA restart |
| shell_command (new) | Restart | Requires HA restart |
| configuration.yaml | Restart | Requires HA restart |

## Dashboard

Dashboard cards are **not** included in this repo. Lovelace dashboards cannot be deployed via HA packages. The card YAML is maintained in the main [tublemetry](https://github.com/chriskaschner/tublemetry) repo at `ha/dashboard.yaml` as a paste reference for manual dashboard setup.

## Related

- [tublemetry](https://github.com/chriskaschner/tublemetry) -- ESP32 firmware, protocol documentation, and tests
