# Changelog — DIY my Dose

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and [Semantic Versioning](https://semver.org/)

**Versioning rules :**
- **MAJOR** — complete rewrite or breaking architecture change
- **MINOR** — significant new feature (new capability, new section, new mode)
- **PATCH** — bug fix, minor improvement, text/label/icon change, translation update

---

## [1.0.7]

### Added
- **Hierarchical watchdog based on dose count** — all timeouts are now calculated as `dose_duration × multiplier` instead of fixed seconds. Hierarchy : Soft → Hard → ESP32 → Lock. Configurable multipliers with guaranteed ordering (soft < hard < esp32 < lock).
- **Global and per-pump multipliers** — 4 global multipliers (`dmd_safety_soft/hard/esp32/lock_mult`) with per-pump override (value 0 = use global). Defaults : ×2 / ×4 / ×6 / ×8.
- **User safety lock** — `input_boolean.dmd_safety_locked` activated by hard/ESP32/lock watchdog. Blocks all dosing until manually unlocked from dashboard (Settings → Safety).
- **Per-pump soft lock** — `input_boolean.dmd_pX_safety_locked` activated by soft watchdog per pump.
- **Watchdog hierarchy sensor** — `sensor.dmd_safety_config_ok` validates that soft < hard < esp32 < lock. Displays error if misconfigured.
- **Calculated timeout sensors** — `sensor.dmd_pX_soft/hard/lock_timeout` (×4 pumps × 3 levels = 12 sensors). Displayed in dashboard for transparency.
- **Notification when dose blocked** — `execute_dose` now sends `🔒 DmD — Dose impossible` notification if system is locked, instead of silently failing.
- **Soft watchdog deducts overdosed counter** — when soft watchdog fires, 1 dose is added to daily counter to account for the extra pump runtime.
- **Unlock button in dashboard** — conditional card visible only when `dmd_safety_locked = on`. Requires confirmation before re-enabling dosing.
- **Safety lock banner on Control page** — red banner displayed when system is locked.

### Changed
- **Dose lock watchdog now uses `lock_mult`** — replaces fixed `input_number.dmd_dosing_timeout`. Timeout is now calculated as `dose_duration × lock_mult` (min 10s).
- **ESP32 timeout supports per-pump override** — `sensor.dmd_pX_esp32_timeout` now checks `dmd_pX_safety_esp32_mult` override before falling back to global multiplier.
- **Auto dosing blocked when system locked** — all 4 auto dosing automations check `dmd_safety_locked = off` before proceeding.
- **Watchdog hard / lock multiplier range** — soft/hard/esp32 multipliers now range 1–20 (step 1) instead of previous fixed ranges.

### Fixed
- **Watchdog hard never fired** — `| max 30` in timeout calculation forced a 30s minimum, preventing hard watchdog from firing at 11s. Fixed to `| max 5`.
- **Watchdog soft notification** — message now displays correct sensor-based timeout value.

---

## [1.0.6]

### Fixed
- **Silent dose block when tank is empty** — `execute_dose` now checks if remaining volume ≥ dose volume before running the pump. If insufficient, the dose is cancelled and a notification is sent : `🚫 DmD — [pump] dose blocked`.
- **Midnight counter reset during an active dose** — `Reset counters at midnight` now waits for `dmd_dosing = off` before resetting (timeout 2 min). Prevents daily counter going to 0 mid-dose.
- **Pump potentially ON after HA restart** — on startup, all 4 pumps are force-cut before releasing the dose lock. Eliminates the vulnerability window where a pump could stay ON after a crash.

### Added
- **Dose lock watchdog** — new `input_number.dmd_dosing_timeout` (30–300s, default 120s). If the dose lock stays ON longer than this value, all pumps are cut, the lock is reset, and an urgent notification is sent. Configurable in Settings → Safety.

---

## [1.0.5]

### Added
- **Pump health — drift detection** — optional per-pump flow drift measurement. At each tank refill, user can optionally enter remaining volume before refill and volume added. The system calculates real daily dose vs configured dose and displays drift %. Status : OK / Vérifier / Recalibrer / Pas de données. Alert if drift exceeds configurable threshold (5–50%, default 15%).
  - New entities : `input_number.dmd_pX_remaining_before_refill`, `input_number.dmd_pX_added_volume`, `input_number.dmd_pX_drift_threshold`, `input_number.dmd_pX_volume_consumed_last_period`
  - New entities : `input_datetime.dmd_pX_last_refill`, `input_datetime.dmd_pX_prev_refill`
  - New sensors : `sensor.dmd_pX_days_between_refills`, `sensor.dmd_pX_real_daily_dose`, `sensor.dmd_pX_drift_pct`, `sensor.dmd_pX_health_status`
  - Scripts `dmd_tank_full_pX` enriched : auto-save refill dates + calculate consumed volume if optional fields are filled

### Changed
- **"Next dose" and "Last dose" labels** — translated to French in all sensors (Demain, dans Xh Ymin, Maintenant)
- **Pump names (`input_text`)** — `initial:` removed from config, HA now restores last saved value after restart instead of resetting to "Pump 1" etc.

---

## [1.0.4]

### Added
- **Per-pump critical flag** — `input_boolean.dmd_pX_critical`. When ON, hard watchdog stops ALL pumps. When OFF, hard watchdog stops only that pump.

---

## [1.0.3]

### Added
- **Safety section in all 4 dashboards** — 6-step security checklist, watchdog status display, expert mode toggle

---

## [1.0.2]

### Added
- **Expert mode** — `input_boolean.dmd_safety_expert_mode`. Hides safety multiplier sliders by default. Must be enabled to modify multipliers.
- **3 dynamic safety multipliers** — `input_number.dmd_safety_esp32_mult` (×1.5–3), `input_number.dmd_safety_soft_mult` (×2–6), `input_number.dmd_safety_hard_mult` (×3–10)

---

## [1.0.1]

### Added
- **Dynamic safety multipliers** — watchdog timeouts are now calculated as `dose_duration × multiplier` instead of fixed `max_volume ÷ flow_rate`. Simpler and more reliable.

---

## [1.0.0]

### Added
- **Hardware timeout per pump (ESP32-side)** — each pump has a configurable max duration stored on the ESP32 itself. If the switch stays ON beyond this limit, the ESP32 cuts the pump autonomously — even if Home Assistant is unreachable, WiFi is down, or HA crashes. Default: 30s. Value is automatically calculated and pushed to the ESP32 by HA.
- **HA watchdog — soft level** — if a pump stays ON for more than 2× its expected dose duration, HA forces it OFF and sends a warning notification. Uses `mode: restart` — timer resets if a new dose starts.
- **HA watchdog — hard level** — if a pump stays ON for more than 5× its expected dose duration, HA stops ALL pumps, activates global pause, and sends an urgent notification.

### Changed
- **ESP32 offline → dosing paused automatically** — `dmd_notify_offline` now also activates `input_boolean.dmd_pause` after 2 minutes without connection. User must manually re-enable dosing once the device is back online.

---

## [0.9.9]

### Fixed
- **Critical : `seconds: "00"` in time_pattern corrupted HA automation cache after simulation**
  - Symptom : automation fires but pump never doses — error `Cannot read properties of undefined`
  - Fix : `seconds` parameter removed from all 4 auto dosing automations
- **Compatibility matrix bypassed at H:00**
  - Root cause : all pumps triggered simultaneously at top of hour
  - Fix : matrix removed from automation conditions — offset system handles sequencing exclusively
- **Offset not applied to auto dosing timing**
  - `dmd_pX_offset_min` was wired to "Missed dose" automations only, not "Auto dosing"
  - Fix : offset now integrated into timing condition of all 4 auto dosing automations
- **Global dose delay causing unpredictable sequencing**
  - Fixed delay in minutes caused drift and conflicts with matrix
  - Fix : replaced by a short configurable mechanical delay in seconds (default 5s)
- **Dose lock `dmd_dosing` stuck ON after HA restart**
  - Fix : `Reset dose lock on startup` automation added
- **Calibration duration wrong for Pump 3 and Pump 4**
  - Fix : calibration script corrected to 60s for all pumps
- **Simulation sub-options still visible when simulation is OFF**
  - Fix : conditional visibility added — sub-options hidden when sim_mode = OFF
- **Simulation options not reset when simulation turns OFF**
  - Fix : automation added to reset all sim options when sim_mode goes OFF
- **Overdosing when switching from full-day to time-window mode mid-day**
  - Root cause : automations had no daily dose cap — switching to a shorter window re-triggered doses already delivered
  - Fix : condition `daily_count < frequency` added to all 4 auto dosing automations — blocks extra doses once daily target is reached
  - Pumps wait until midnight reset before dosing again

### Added
- Automatic offset calculation from compatibility matrix (`dmd_auto_calculate_offsets`)
- Automatic `automation.reload` when simulation mode turns OFF — clears time_pattern cache transparently
- Mechanical delay between doses now configurable (1–30s, default 5s) — `input_number.dmd_dose_delay`
- Notification test button in Settings → Global
- Startup checklist card in Settings tab
- Push notifications configured and tested (mobile companion app)

### Changed
- Project renamed from **ReefDose** to **DIY my Dose (DmD)** — avoid confusion with Red Sea's trademarked ReefDose product
- All HA entities renamed from `rd_` prefix to `dmd_` prefix
- All files renamed from `rd_*.yaml` to `dmd_*.yaml`
- ESPHome device renamed from `reefdose` to `dmd` (`friendly_name: DIY my Dose`)

---

## [0.9.8]

### Fixed
- **Critical : trigger cache corruption after simulation**
  - Root cause : `seconds: "00"` parameter in `time_pattern` triggers corrupted HA automation cache
  - Symptom : automation fires but pump never doses — error `Cannot read properties of undefined`
  - Fix : `seconds` parameter removed from all 4 auto dosing automations
- **Critical : compatibility matrix bypassed at H:00**
  - Root cause : all pumps triggered simultaneously — matrix checked elapsed time since previous cycle
  - Fix : matrix removed from automation conditions — offset system handles sequencing
- **Offset not applied to auto dosing timing**
  - `dmd_pX_offset_min` was only wired to "Missed dose" automations, not "Auto dosing"
  - Fix : offset now integrated into timing condition of all 4 auto dosing automations
- **Global dose delay causing unpredictable sequencing**
  - Fixed delay in minutes caused drift and made the matrix unreliable
  - Fix : replaced by a short configurable mechanical delay in seconds (default 5s)
- **Header toggle in Global settings accidentally toggling simulation mode**
  - Fix : `show_header_toggle: false` added to Global card

### Added
- Automatic offset calculation from compatibility matrix (`dmd_auto_calculate_offsets`)
- Automatic `automation.reload` when simulation mode turns OFF
- Mechanical delay between doses now configurable (1–30s, default 5s) — `input_number.dmd_dose_delay`
- Notification test button in Settings → Global
- Startup checklist card in Settings tab
- Simulation sub-options auto-hide when simulation is OFF
- Simulation options reset when simulation turns OFF

---

## [0.9.7]

### Added
- Simulation mode (`input_boolean.dmd_sim_mode`)
  - Configurable interval (1–10 min)
  - Real pump toggle (`input_boolean.dmd_sim_real_pumps`)
  - Stats fully protected — counters, tank volumes and totals not modified
  - `last_dose` timestamp still updates for timing verification
  - Red warning banner on Control dashboard when active

### Fixed
- Auto dosing automations blocked each other at H:00
  - Fix : dose lock condition removed — queued script handles sequencing natively

---

## [0.9.6]

### Fixed
- Automatic dosing completely non-functional
  - Root cause : empty message in Pump 4 alert caused YAML parser to discard 4 automations
- Timezone error when comparing last dose timestamp
  - `strptime()` replaced with `as_datetime().replace(tzinfo=now().tzinfo)` (16 occurrences)
- Duplicate entity definitions between `dmd_config.yaml` and `dmd_stat.yaml`
- Status icon always red regardless of pump state
- "Dosing in progress" indicator not tappable — no way to reset stuck dose lock

---

## [0.9.5]

### Fixed
- Dosing window rejected when end time is midnight (0h00)
  - Fix : 0h00 now treated as 1440 minutes (end of day)

---

## [0.9.4]

### Added
- Full day mode per pump (`input_boolean.dmd_pX_full_day`)
  - ON : doses spread across 24h, aligned to clock hours
  - OFF : custom window with start/end time
- Tank refilled scripts (were missing, caused errors)
- Explanatory text above compatibility matrix

### Fixed
- Pump names displayed as raw Jinja template in bar-card and glance cards
- "Next dose" showed "Tomorrow" immediately after enabling full day mode
- Calibration mode did not deactivate after pump run

---

## [0.9.3]

### Added
- Free pump name per pump (`input_text.dmd_pX_name`)
- Manual dose cycle reset option (`input_boolean.dmd_pX_manual_resets_cycle`)

### Changed
- Theme CSS variables renamed to universal `p1/p2/p3/p4` scheme

---

## [0.9.2]

### Added
- Compatibility matrix — 6 configurable minimum delays between pump pairs

---

## [0.9.1]

### Added
- Dosing window per pump (start/end time)
- Interval sensor, window validation sensor
- "Next dose" countdown sensor per pump

---

## [0.9.0]

### Added
- Fixed start time per pump
- Per-minute trigger for exact scheduling (was hourly)

---

## [0.8.0]

### Added
- Full English translation, multi-language support (FR / EN / ES / IT / DE)
- GitHub-ready structure, 5 modular package files
- DMC Theme with CSS variables, desktop + mobile dashboards (FR + EN)
- Full reset button with confirmation

---

## [0.7.0]

### Added
- DMC color theme via CSS variables
- Mini Graph Card — 7-day tank level history
- Mobile dashboard (2-column layout)

---

## [0.6.0]

### Added
- ESPHome binary sensor for connection status
- 3-view dashboard : Control / Settings / Statistics

---

## [0.5.0]

### Added
- Single ESPHome device setup, package structure

---

## [0.4.0]

### Added
- Split monolithic YAML into 5 modular files

---

## [0.3.0]

### Added
- Complete statistics : daily / weekly / monthly / yearly volumes, missed doses

---

## [0.2.0]

### Added
- Calibration system, tank level tracking, alerts, anti-overlap

---

## [0.1.0]

### Added
- Initial release — 4 pumps via ESP32 / ESPHome
- Automatic dosing, manual dose, auto mode toggle, global pause

---
---

---

# Journal des modifications — DIY my Dose

Toutes les modifications notables sont documentées ici.
Format basé sur [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) et [Semantic Versioning](https://semver.org/)

**Règles de versioning :**
- **MAJEUR** — réécriture complète ou changement d'architecture cassant
- **MINEUR** — nouvelle fonctionnalité significative (nouvelle capacité, nouvelle section, nouveau mode)
- **PATCH** — correction de bug, amélioration mineure, changement de texte/label/icône, traduction

---

## [1.0.7]

### Added
- **Hierarchical watchdog based on dose count** — all timeouts are now calculated as `dose_duration × multiplier` instead of fixed seconds. Hierarchy : Soft → Hard → ESP32 → Lock. Configurable multipliers with guaranteed ordering (soft < hard < esp32 < lock).
- **Global and per-pump multipliers** — 4 global multipliers (`dmd_safety_soft/hard/esp32/lock_mult`) with per-pump override (value 0 = use global). Defaults : ×2 / ×4 / ×6 / ×8.
- **User safety lock** — `input_boolean.dmd_safety_locked` activated by hard/ESP32/lock watchdog. Blocks all dosing until manually unlocked from dashboard (Settings → Safety).
- **Per-pump soft lock** — `input_boolean.dmd_pX_safety_locked` activated by soft watchdog per pump.
- **Watchdog hierarchy sensor** — `sensor.dmd_safety_config_ok` validates that soft < hard < esp32 < lock. Displays error if misconfigured.
- **Calculated timeout sensors** — `sensor.dmd_pX_soft/hard/lock_timeout` (×4 pumps × 3 levels = 12 sensors). Displayed in dashboard for transparency.
- **Notification when dose blocked** — `execute_dose` now sends `🔒 DmD — Dose impossible` notification if system is locked, instead of silently failing.
- **Soft watchdog deducts overdosed counter** — when soft watchdog fires, 1 dose is added to daily counter to account for the extra pump runtime.
- **Unlock button in dashboard** — conditional card visible only when `dmd_safety_locked = on`. Requires confirmation before re-enabling dosing.
- **Safety lock banner on Control page** — red banner displayed when system is locked.

### Changed
- **Dose lock watchdog now uses `lock_mult`** — replaces fixed `input_number.dmd_dosing_timeout`. Timeout is now calculated as `dose_duration × lock_mult` (min 10s).
- **ESP32 timeout supports per-pump override** — `sensor.dmd_pX_esp32_timeout` now checks `dmd_pX_safety_esp32_mult` override before falling back to global multiplier.
- **Auto dosing blocked when system locked** — all 4 auto dosing automations check `dmd_safety_locked = off` before proceeding.
- **Watchdog hard / lock multiplier range** — soft/hard/esp32 multipliers now range 1–20 (step 1) instead of previous fixed ranges.

### Fixed
- **Watchdog hard never fired** — `| max 30` in timeout calculation forced a 30s minimum, preventing hard watchdog from firing at 11s. Fixed to `| max 5`.
- **Watchdog soft notification** — message now displays correct sensor-based timeout value.

---

## [1.0.7]

### Ajouté
- **Watchdog de sécurité hiérarchique basé sur le nombre de doses** — tous les timeouts sont calculés en `durée_dose × multiplicateur` au lieu de secondes fixes. Hiérarchie : Soft → Hard → ESP32 → Verrou. Multiplicateurs configurables avec validation de l'ordre (soft < hard < esp32 < lock).
- **Multiplicateurs globaux et par pompe** — 4 multiplicateurs globaux (`dmd_safety_soft/hard/esp32/lock_mult`) avec override par pompe (valeur 0 = utilise le global). Défauts : ×2 / ×4 / ×6 / ×8.
- **Verrou de sécurité système** — `input_boolean.dmd_safety_locked` activé par watchdog hard/ESP32/verrou. Bloque tout dosage jusqu'au déverrouillage manuel depuis Réglages → Sécurité.
- **Verrou par pompe** — `input_boolean.dmd_pX_safety_locked` activé par le watchdog soft par pompe.
- **Sensor de validation de hiérarchie** — `sensor.dmd_safety_config_ok` vérifie que soft < hard < esp32 < lock. Affiche une erreur si mal configuré.
- **Sensors de timeout calculés** — `sensor.dmd_pX_soft/hard/lock_timeout` (×4 pompes × 3 niveaux = 12 sensors). Affichés dans le dashboard.
- **Notification dose impossible** — `execute_dose` envoie `🔒 DmD — Dose impossible` si le système est verrouillé, au lieu d'échouer silencieusement.
- **Watchdog soft déduit 1 dose du compteur** — quand le watchdog soft se déclenche, 1 dose est ajoutée au compteur journalier pour comptabiliser le surdosage.
- **Bouton de déverrouillage dans le dashboard** — carte conditionnelle visible uniquement si `dmd_safety_locked = on`. Confirmation requise avant de réactiver le dosage.
- **Bannière rouge sur la page Controle** — affichée quand le système est verrouillé.

### Modifié
- **Watchdog verrou dosage utilise `lock_mult`** — remplace le `input_number.dmd_dosing_timeout` fixe. Le timeout est maintenant calculé en `durée_dose × lock_mult` (min 10s).
- **Timeout ESP32 supporte les overrides par pompe** — `sensor.dmd_pX_esp32_timeout` vérifie d'abord `dmd_pX_safety_esp32_mult` avant de revenir au multiplicateur global.
- **Dosage auto bloqué si système verrouillé** — les 4 automations de dosage auto vérifient `dmd_safety_locked = off` avant de procéder.
- **Plage des multiplicateurs** — soft/hard/esp32 sur 1–20 (pas 1) au lieu des plages fixes précédentes.

### Corrigé
- **Watchdog hard ne se déclenchait jamais** — `| max 30` dans le calcul du timeout forçait un minimum de 30s, empêchant le hard de tirer à 11s. Corrigé en `| max 5`.
- **Notification watchdog soft** — le message affiche maintenant la valeur correcte basée sur le sensor de timeout.

---

## [1.0.6]

### Corrigé
- **Dose bloquée silencieusement quand le bidon est vide** — `execute_dose` vérifie maintenant que le volume restant ≥ volume de la dose avant d'activer la pompe. Si insuffisant, la dose est annulée et une notification est envoyée : `🚫 DmD — [pompe] dose bloquée`.
- **Reset des compteurs à minuit pendant une dose active** — `Reset counters at midnight` attend maintenant que `dmd_dosing = off` avant de reset (timeout 2 min). Empêche le compteur journalier de passer à 0 en pleine dose.
- **Pompe potentiellement ON après redémarrage HA** — au démarrage, les 4 pompes sont forcées à OFF avant la libération du verrou. Élimine la fenêtre de vulnérabilité où une pompe pouvait rester ON après un crash.

### Ajouté
- **Watchdog verrou dosage** — nouveau `input_number.dmd_dosing_timeout` (30–300s, défaut 120s). Si le verrou dosage reste ON plus longtemps que cette valeur, toutes les pompes sont coupées, le verrou est réinitialisé et une notification urgence est envoyée. Configurable dans Réglages → Sécurité.

---

## [1.0.5]

### Ajouté
- **Santé des pompes — détection de dérive de débit** — mesure optionnelle de dérive par pompe. À chaque remplissage, l'utilisateur peut saisir le volume restant avant remplissage et le volume ajouté. Le système calcule la dose réelle/jour vs configurée et affiche la dérive %. Statut : OK / Vérifier / Recalibrer / Pas de données. Alerte si dérive dépasse le seuil configurable (5–50%, défaut 15%).
  - Nouvelles entités : `input_number.dmd_pX_remaining_before_refill`, `input_number.dmd_pX_added_volume`, `input_number.dmd_pX_drift_threshold`, `input_number.dmd_pX_volume_consumed_last_period`
  - Nouvelles entités : `input_datetime.dmd_pX_last_refill`, `input_datetime.dmd_pX_prev_refill`
  - Nouveaux sensors : `sensor.dmd_pX_days_between_refills`, `sensor.dmd_pX_real_daily_dose`, `sensor.dmd_pX_drift_pct`, `sensor.dmd_pX_health_status`
  - Scripts `dmd_tank_full_pX` enrichis : sauvegarde automatique des dates de remplissage + calcul du volume consommé si les champs optionnels sont renseignés

### Modifié
- **Labels "Prochaine dose" et "Dernière dose"** — traduits en français dans tous les sensors (Demain, dans Xh Ymin, Maintenant)
- **Noms des pompes (`input_text`)** — `initial:` retiré de la config, HA restaure maintenant la dernière valeur sauvegardée après redémarrage au lieu de réinitialiser à "Pump 1" etc.

---

## [1.0.4]

### Ajouté
- **Flag critique par pompe** — `input_boolean.dmd_pX_critical`. Si ON, le watchdog dur coupe TOUTES les pompes. Si OFF, le watchdog dur coupe uniquement cette pompe.

---

## [1.0.3]

### Ajouté
- **Section Sécurité dans les 4 dashboards** — checklist sécurité 6 étapes, affichage statut watchdog, toggle mode expert

---

## [1.0.2]

### Ajouté
- **Mode expert** — `input_boolean.dmd_safety_expert_mode`. Masque les sliders multiplicateurs par défaut. Doit être activé pour modifier les multiplicateurs.
- **3 multiplicateurs de sécurité dynamiques** — `input_number.dmd_safety_esp32_mult` (×1.5–3), `input_number.dmd_safety_soft_mult` (×2–6), `input_number.dmd_safety_hard_mult` (×3–10)

---

## [1.0.1]

### Ajouté
- **Multiplicateurs de sécurité dynamiques** — les timeouts watchdog sont maintenant calculés comme `durée_dose × multiplicateur` au lieu de `volume_max ÷ débit`. Plus simple et plus fiable.

---

## [1.0.0]

### Ajouté
- **Timeout hardware par pompe (côté ESP32)** — chaque pompe a une durée maximale configurable, stockée sur l'ESP32 lui-même. Si le switch reste ON au-delà de cette limite, l'ESP32 coupe la pompe de manière autonome — même si Home Assistant est inaccessible, si le WiFi est coupé ou si HA plante. Défaut : 30s. La valeur est calculée automatiquement et envoyée à l'ESP32 par HA.
- **Watchdog HA — niveau doux** — si une pompe reste ON plus de 2× la durée attendue de sa dose, HA la force à OFF et envoie une notification warning. `mode: restart` — le timer se réinitialise si une nouvelle dose démarre.
- **Watchdog HA — niveau dur** — dernier recours : si une pompe reste ON plus de 5× la durée attendue, HA arrête TOUTES les pompes, active la pause globale et envoie une notification urgence.

### Modifié
- **ESP32 hors ligne → dosage mis en pause automatiquement** — `dmd_notify_offline` active désormais aussi `input_boolean.dmd_pause` après 2 minutes sans connexion. L'utilisateur doit réactiver le dosage manuellement une fois l'appareil de retour en ligne.

---

## [0.9.9]

### Corrigé
- **Critique : `seconds: "00"` dans time_pattern corrompait le cache HA après simulation**
  - Symptôme : l'automation se déclenche mais la pompe ne dose pas — erreur `Cannot read properties of undefined`
  - Correction : paramètre `seconds` retiré des 4 automations de dosage automatique
- **Matrice de compatibilité contournée à H:00**
  - Cause : toutes les pompes se déclenchaient simultanément en début d'heure
  - Correction : matrice retirée des conditions — le système d'offsets gère exclusivement le séquençage
- **Offset non appliqué au timing du dosage automatique**
  - `dmd_pX_offset_min` était câblé aux automations "Dose manquée" uniquement, pas "Auto dosing"
  - Correction : offset intégré dans la condition de timing des 4 automations de dosage automatique
- **Délai global causant un séquençage imprévisible**
  - Le délai fixe en minutes causait des dérives et conflits avec la matrice
  - Correction : remplacé par un court délai mécanique configurable en secondes (défaut 5s)
- **Verrou de dose `dmd_dosing` bloqué ON après redémarrage HA**
  - Correction : automation `Reset dose lock on startup` ajoutée
- **Durée de calibration incorrecte pour les pompes 3 et 4**
  - Correction : script de calibration corrigé à 60s pour toutes les pompes
- **Sous-options simulation visibles alors que la simulation est OFF**
  - Correction : visibilité conditionnelle ajoutée — sous-options masquées quand sim_mode = OFF
- **Options simulation non réinitialisées quand la simulation passe à OFF**
  - Correction : automation ajoutée pour réinitialiser toutes les options sim quand sim_mode = OFF
- **Surdosage lors du passage de "journée entière" à "plage horaire" en cours de journée**
  - Cause : les automations n'avaient pas de plafond sur les doses journalières
  - Correction : condition `daily_count < frequency` ajoutée sur les 4 pompes — bloque toute dose supplémentaire une fois le quota atteint
  - Les pompes attendent le reset de minuit avant de redoser

### Ajouté
- Calcul automatique des offsets depuis la matrice de compatibilité (`dmd_auto_calculate_offsets`)
- `automation.reload` automatique quand le mode simulation se désactive — efface le cache time_pattern de manière transparente
- Délai mécanique entre doses configurable (1–30s, défaut 5s) — `input_number.dmd_dose_delay`
- Bouton de test notification dans Réglages → Global
- Checklist de démarrage dans l'onglet Réglages
- Notifications push configurées et testées (app compagnon mobile)

### Modifié
- Projet renommé de **ReefDose** à **DIY my Dose (DmD)** — éviter la confusion avec le produit ReefDose de Red Sea (marque déposée)
- Toutes les entités HA renommées du préfixe `rd_` vers `dmd_`
- Tous les fichiers renommés de `rd_*.yaml` vers `dmd_*.yaml`
- ESPHome renommé de `reefdose` vers `dmd` (`friendly_name: DIY my Dose`)

---

## [0.9.8]

### Corrigé
- **Critique : corruption du cache des triggers après simulation**
  - Cause : `seconds: "00"` dans les triggers `time_pattern` corrompait le cache HA
  - Symptôme : l'automation se déclenche mais la pompe ne dose jamais
  - Correction : paramètre `seconds` retiré des 4 automations de dosage automatique
- **Critique : matrice de compatibilité contournée à H:00**
  - Cause : toutes les pompes se déclenchaient simultanément
  - Correction : matrice retirée des conditions — le système d'offsets gère le séquençage
- **Offset non appliqué au timing du dosage automatique**
  - `dmd_pX_offset_min` n'était branché qu'aux automations "Dose manquée"
  - Correction : offset intégré dans la condition de timing des 4 automations
- **Délai global causant un séquençage imprévisible**
  - Correction : remplacé par un délai mécanique court configurable en secondes (défaut 5s)
- **Toggle d'en-tête de la carte Global déclenchait accidentellement le mode simulation**
  - Correction : `show_header_toggle: false` ajouté

### Ajouté
- Calcul automatique des offsets depuis la matrice (`dmd_auto_calculate_offsets`)
- `automation.reload` automatique à la désactivation du mode simulation
- Délai mécanique entre doses configurable (1–30s, défaut 5s)
- Bouton test notifications dans Réglages → Global
- Checklist de démarrage dans l'onglet Réglages
- Sous-options simulation masquées quand simulation = OFF
- Reset des options simulation à la désactivation

---

## [0.9.7]

### Ajouté
- Mode simulation (`input_boolean.dmd_sim_mode`)
  - Intervalle configurable (1–10 min)
  - Toggle pompes réelles (`input_boolean.dmd_sim_real_pumps`)
  - Stats entièrement protégées
  - Bannière rouge sur le dashboard Contrôle

### Corrigé
- Les automations de dosage se bloquaient mutuellement à H:00

---

## [0.9.6]

### Corrigé
- Dosage automatique complètement non-fonctionnel (champ message vide dans alerte P4)
- Erreur de fuseau horaire lors de la comparaison des timestamps (16 occurrences)
- Définitions d'entités en double entre `dmd_config.yaml` et `dmd_stat.yaml`
- Icône statut toujours rouge peu importe l'état
- Indicateur "Dose en cours" non cliquable

---

## [0.9.5]

### Corrigé
- Fenêtre de dosage rejetée quand l'heure de fin est minuit (0h00)

---

## [0.9.4]

### Ajouté
- Mode journée entière par pompe (`input_boolean.dmd_pX_full_day`)
- Scripts "Bidon rempli"
- Texte explicatif au-dessus de la matrice de compatibilité

### Corrigé
- Noms des pompes affichés comme template Jinja brut
- "Prochaine dose" affichait "Demain" juste après l'activation du mode journée entière
- Mode calibration ne se désactivait pas après l'exécution

---

## [0.9.3]

### Ajouté
- Nom libre par pompe (`input_text.dmd_pX_name`)
- Option reset de cycle par dose manuelle (`input_boolean.dmd_pX_manual_resets_cycle`)

### Modifié
- Variables CSS du thème renommées en schéma universel `p1/p2/p3/p4`

---

## [0.9.2]

### Ajouté
- Matrice de compatibilité — 6 délais minimaux configurables entre paires de pompes

---

## [0.9.1]

### Ajouté
- Fenêtre de dosage par pompe (heure début/fin)
- Capteur d'intervalle, capteur de validation de fenêtre
- Capteur compte à rebours "Prochaine dose"

---

## [0.9.0]

### Ajouté
- Heure de démarrage fixe par pompe
- Trigger à la minute pour un planning précis

---

## [0.8.0]

### Ajouté
- Traduction anglaise complète, support multilingue (FR / EN / ES / IT / DE)
- Structure GitHub, 5 fichiers de packages modulaires
- Thème DMC, dashboards desktop + mobile FR + EN
- Bouton reset complet avec confirmation

---

## [0.7.0]

### Ajouté
- Thème DMC via variables CSS
- Mini Graph Card — historique 7 jours des niveaux
- Dashboard mobile

---

## [0.6.0]

### Ajouté
- Capteur binaire ESPHome pour le statut de connexion
- Dashboard 3 vues : Contrôle / Réglages / Statistiques

---

## [0.5.0]

### Ajouté
- Appareil ESPHome unique, structure de packages

---

## [0.4.0]

### Ajouté
- Découpage du YAML monolithique en 5 fichiers modulaires

---

## [0.3.0]

### Ajouté
- Statistiques complètes : volumes journalier / hebdo / mensuel / annuel, doses manquées

---

## [0.2.0]

### Ajouté
- Calibration, suivi des niveaux, alertes, anti-chevauchement

---

## [0.1.0]

### Ajouté
- Version initiale — 4 pompes via ESP32 / ESPHome
- Dosage automatique, dose manuelle, toggle auto, pause globale
