# Changelog — ReefDose

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)

---

## [0.9.4]
### Added
- Manual dose cycle reset option per pump (input_boolean.rd_pX_manual_resets_cycle)
  - OFF (default) : manual dose is out-of-cycle — auto schedule continues unchanged
  - ON : manual dose resets the cycle — next auto dose fires 1 interval after manual dose
- Toggle added in Control view next to Auto button, per pump
- All 4 dashboards updated (desktop FR/EN + mobile FR/EN)

### Behaviour details
- OFF : useful for occasional corrections without disrupting the schedule
  - Example : manual dose at 07:45, auto dose still fires at 08:00 as planned
- ON  : useful when a manual dose replaces a missed auto dose
  - Example : manual dose at 07:45, next auto dose fires at 07:45 + interval
- Condition checks last_dose timestamp vs calculated interval
- Graceful handling if last_dose is unknown (allows dosing)

### Changed
- Theme CSS variables renamed for universal clarity
  - `kh-color` / `ca-color` / `mg-color` / `ph-color` → `p1-color` / `p2-color` / `p3-color` / `p4-color`
  - `kh-bg` / `ca-bg` / `mg-bg` / `ph-bg` → `p1-bg` / `p2-bg` / `p3-bg` / `p4-bg`
  - Same for `*-bg-light` variants
  - All 4 dashboards and README updated accordingly
  - No functional impact — visual only

---

## [0.9.3]
### Added
- Free pump name per pump (input_text.rd_pX_name)
  - Displayed dynamically in all dashboard section headers
  - Default : "Pump 1" / "Pump 2" / "Pump 3" / "Pump 4"
  - Configurable in Settings view — first field of each pump section
  - Max 20 characters
- Dashboard section headers now show the user-defined pump name
- Calibration section headers also show the pump name dynamically
- All 4 dashboards updated (desktop FR/EN + mobile FR/EN)

### Notes
- entity_id stays rd_p1/p2/p3/p4 — neutral and universal
- Only the visible label changes — no impact on automations or sensors
- Examples : "Kh", "Ca", "Nutrient A", "pH Up", "Engrais N", "Solution 1"...

---

## [0.9.2]
### Added
- Compatibility matrix — minimum delay between each pump pair
  - 6 configurable delays : p1↔p2, p1↔p3, p1↔p4, p2↔p3, p2↔p4, p3↔p4
  - Default values :
    - p1↔p2 : 30 min / p1↔p3 : 30 min / p2↔p3 : 30 min
    - p1↔p4 : 0 min  / p2↔p4 : 0 min  / p3↔p4 : 0 min
    - (adjust to match your use case — reactive liquids need delay, compatible ones can be 0)
  - Set to 0 to allow compatible pairs to dose immediately after each other
- Each auto dosing automation now checks last dose time of all other pumps
  - Dose is blocked if minimum delay not elapsed since another pump dosed
  - Graceful handling : delay check skipped if other pump has never dosed
- Dashboard : "Pump compatibility" section added in Settings view (global)
  - All 4 dashboards updated (desktop FR/EN + mobile FR/EN)

### Notes
- Delays are bidirectional : rd_delay_p1_p2 applies both p1→p2 and p2→p1
- Compatible with future modular expansion (delays named by pump index, not solution)
- Default delays assume reactive liquids on p1/p2/p3 and a compatible additive on p4
  Adjust to match your actual use case

---

## [0.9.1]
### Added
- End time per pump (input_datetime.rd_pX_end_time, default 23:59)
- Dosing window : start_time → end_time, reset at midnight every day
  - No cycle ever crosses midnight
  - Example : 2 doses 21:00→23:59 = doses at 21:00 and 22:29 only
- Interval sensor per pump (sensor.rd_pX_interval_min)
  - Calculated automatically = (end - start) / frequency
- Window validation sensor per pump (sensor.rd_pX_window_ok)
  - "OK" → configuration valid
  - "Too many doses for window" → interval < minimum delay
  - "End before start" → end time set before start time
- Default start time : 00:00 (dosing starts at midnight if not set)
- Default end time : 23:59 (full day window by default)
- Automations blocked automatically if window_ok != "OK"

### Changed
- next_dose sensor updated : shows "Tomorrow HH:MM" after end time
- next_dose sensor shows window error message if configuration invalid
- All 4 dashboards : end_time + interval + window_ok added to Settings view
- v1.1.0 smart dosing window merged into v0.9.1 (same feature, cleaner)

### Removed
- v1.1.0 from planned roadmap (implemented here)

---

## [0.9.0]
### Added
- Fixed start time per pump (input_datetime.rd_pX_start_time)
- Dosing schedule calculated from start time + frequency
  - Interval = 24h / frequency in minutes
  - Example : start 22:00, 4 doses/day = doses at 22:00 / 04:00 / 10:00 / 16:00
- Next dose sensor per pump (sensor.rd_pX_next_dose)
  - Displays time remaining until next scheduled dose
- Trigger changed from hourly to per-minute for exact scheduling
- Graceful handling if start_time not set (dosing disabled until configured)

### Changed
- Auto dosing automations : time_pattern hours:/1 → minutes:/1
- Offset (rd_pX_offset_min) replaced by start_time logic
- Condition logic rewritten to use start_time + interval calculation

### Removed
- rd_pX_offset_min no longer used in automations (kept in config for compatibility)

---

## [0.8.0]
### Added
- Full English translation of all package files (names, aliases, messages)
- Multi-language support : FR / EN / ES / IT / DE translation files
- GitHub-ready structure (packages/, dashboards/, themes/, translations/)
- Split into 5 modular files : rd_config, rd_stat, rd_sensors, rd_scripts, rd_automations
- DMC Theme (dmc_theme.yaml) with CSS variables for Balling colors
- Mobile dashboard (2-column layout, all features included)
- Desktop dashboard FR + EN, Mobile dashboard FR + EN
- Dashboard naming convention with language suffix (_fr, _en)
- Full reset button with confirmation popup and description label
- Script rd_reset_all : resets all stats except calibration
- Auto-deactivation of calibration mode after pump run
- "Tank refilled!" buttons in Settings view
- README.md bilingual FR + EN with installation guide
- notify.yaml documented in English with multi-device support

### Changed
- All entity_id renamed to English (522 replacements across 3 files)
- Dashboard filenames renamed with language suffix
- Comments and labels fully translated to English in all package files

---

## [0.7.0]
### Added
- Balling color theme via HA theme CSS variables
- Mini Graph Card : 7-day tank level history graph
- panel:true on all views for full-width layout
- vertical-stack wrapper to fix column alignment issues
- YAML anchors for bar-card and mini-graph-card colors
- Titles added on bar-card and glance tank cards
- Mobile dashboard planning (2-column layout)

### Fixed
- Corrupted emojis in titles (removed all emojis)
- Column misalignment (Global card out of grid)
- Columns too narrow (fixed with panel:true)
- Card overlap due to card-mod margins (removed card-mod from Global card)

---

## [0.6.0]
### Added
- ESPHome binary_sensor for connection status
- Dashboard complete restructure with 3 views : Control / Settings / Statistics
- 4-column grid layout (one per pump)
- Samba share setup for file transfer to HA

### Fixed
- Duplicate homeassistant: key issue in package loading
- Package !include_dir directives corrected
- Dashboard rebuild after layout issues

---

## [0.5.0]
### Added
- Single ESPHome device setup (renamed to "reefdose")
- Complete package structure with !include_dir_named
- Notification group (notify.reefdose_admin)
- Dashboard integration with HACS cards
- configuration.yaml with packages + themes directives
- Lovelace dashboard with 4 views (overview, pump1, pump2, statistics)

### Fixed
- Package loading issues with !include_dir directives
- ESPHome device naming conflicts

---

## [0.4.0]
### Added
- Split monolithic YAML into 5 modular files :
  - rd_config.yaml (inputs, booleans)
  - rd_stat.yaml (statistics, counters)
  - rd_sensors.yaml (template sensors)
  - rd_scripts.yaml (scripts)
  - rd_automations.yaml (automations)

---

## [0.3.0]
### Added
- Comprehensive statistics tracking :
  - Total dosed volumes per pump
  - Daily / weekly / monthly / yearly volume sensors
  - Missed dose counter per pump
  - Last dose timestamp per pump
  - Time since last dose sensor
- Daily counter reset automation at midnight
- Missed dose detection when device offline
- Complete file regeneration with all 4 pumps (Kh/Ca/Mg/Phyto)

---

## [0.2.0]
### Added
- Calibration system per pump (flow rate calculation ml/s)
- Calibration mode + start boolean per pump
- Calculated dose duration based on flow rate
- Tank level tracking (remaining volume, %, days remaining)
- Low tank alert notifications per pump
- Alert threshold configurable per pump
- Anti-overlap system (rd_dosing boolean)
- Delay between doses (global setting)
- Offline notification after 2 minutes

### Changed
- Migrated from scattered YAML to packages architecture
- Refactored all automations to use central rd_execute_dose script

### Fixed
- Jinja delay bug in dose execution script

---

## [0.1.0]
### Added
- Initial package structure
- 4 peristaltic pumps via ESP32 / ESPHome (Kh, Ca, Mg, Phyto)
- Auto dosing per pump (daily dose, frequency, offset)
- Manual dose trigger per pump
- Auto mode toggle per pump
- Global pause (all pumps)
- Daily dose counter per pump
- Basic dashboard with Mushroom + Bar Card (HACS)
- Second dosing unit support (ESPHome (IP configured in ESPHome YAML))
