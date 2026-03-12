# Changelog — DIY my Dose

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and [Semantic Versioning](https://semver.org/)

**Versioning rules :**
- **MAJOR** — complete rewrite or breaking architecture change
- **MINOR** — significant new feature (new capability, new section, new mode)
- **PATCH** — bug fix, minor improvement, text/label/icon change, translation update

---

## [0.9.9]

### Changed
- Project renamed from **ReefDose** to **DIY my Dose (DmD)** — avoid confusion with Red Sea's trademarked ReefDose product
- All HA entities renamed from `rd_` prefix to `dmd_` prefix
- All files renamed from `rd_*.yaml` to `dmd_*.yaml`
- ESPHome device references updated from `reefdose` to `dmd`

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

# Journal des modifications — DIY my Dose

Toutes les modifications notables sont documentées ici.
Format basé sur [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) et [Semantic Versioning](https://semver.org/)

**Règles de versioning :**
- **MAJEUR** — réécriture complète ou changement d'architecture cassant
- **MINEUR** — nouvelle fonctionnalité significative (nouvelle capacité, nouvelle section, nouveau mode)
- **PATCH** — correction de bug, amélioration mineure, changement de texte/label/icône, traduction

---

## [0.9.9]

### Modifié
- Projet renommé de **ReefDose** à **DIY my Dose (DmD)** — éviter la confusion avec le produit ReefDose de Red Sea (marque déposée)
- Toutes les entités HA renommées du préfixe `rd_` vers `dmd_`
- Tous les fichiers renommés de `rd_*.yaml` vers `dmd_*.yaml`
- Références ESPHome mises à jour de `reefdose` vers `dmd`

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
