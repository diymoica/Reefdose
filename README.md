# DIY my Dose — Universal Smart Dosing Controller

![Version](https://img.shields.io/badge/version-1.0.7-blue)
![Status](https://img.shields.io/badge/status-stable-green)
![HA](https://img.shields.io/badge/Home%20Assistant-2023.x%2B-41BDF5)
![ESPHome](https://img.shields.io/badge/ESPHome-required-green)
![License](https://img.shields.io/badge/license-Custom-lightgrey)
![Use case](https://img.shields.io/badge/use%20case-universal-brightgreen)

**[English](#english) | [Français](#français)**

---

## English

### Overview

DIY my Dose is a universal liquid dosing controller built on Home Assistant and ESP32 (ESPHome).
It controls up to 4 peristaltic pumps with automatic dosing, precise calibration, safety checks, alerts and statistics.

**Designed for any liquid dosing use case :**
reef aquariums, freshwater tanks, hydroponics, plant care, brewing, DIY lab, and more.
Each pump is fully configurable — name, schedule, frequency, compatibility with other pumps.
No logic is imposed : you decide everything.

> ✅ **Stable v1.0.7** — running on a real reef aquarium. Automatic dosing, hierarchical safety watchdog and safety locks fully operational.
> See the [ROADMAP](ROADMAP.md) for upcoming features.

---

### Features

#### Dosing
- **Automatic dosing** — configurable time window (start/end) per pump, or full 24h mode
- **Clock-aligned scheduling** — full day mode aligns doses to exact clock hours (e.g. 24 doses → every hour at :00)
- **Auto-calculated interval** — derived from window duration and requested frequency
- **Manual dose** — triggerable at any time from the dashboard
- **Cycle reset option** — manual dose can optionally shift the auto schedule

#### Safety
- **Compatibility matrix** — minimum delay between each pump pair (6 configurable pairs), offsets calculated automatically
- **Mechanical delay** — configurable delay between sequential doses (1–30s)
- **Real-time validation** — warning if window is too short for requested frequency
- **Global pause** — stops all pumps instantly
- **3-level HA watchdog** — Soft (pump stop + warning) → Hard (all pumps stop + lock) → Lock (forced reset). Each level triggered at configurable multiples of dose duration (default ×2 / ×4 / ×8)
- **ESP32 hardware watchdog** — pump cut directly by the ESP32 at ×6 dose duration, even if Home Assistant crashes or WiFi is down
- **Safety lock** — system lock activated by hard/ESP32/lock watchdog. Blocks all dosing until manually unlocked from dashboard
- **Per-pump overrides** — each watchdog level can be configured globally or overridden per pump (expert mode)
- **Hierarchy validation** — sensor warns if multipliers are misconfigured (soft ≥ hard, etc.)
- **Dose blocked notification** — push notification when a dose is prevented by the safety lock

#### Hardware
- **Physical manual dose button** — a hardware button on the ESP32 triggers a manual dose, works even without dashboard access *(planned in v1.3.0)*

#### Simulation
- **Simulation mode** — test full timing and sequencing without affecting real data
- **Configurable interval** — simulate doses every 1–10 min
- **Real pump toggle** — optionally fire real pumps during simulation
- **Auto-reload** — automations reload automatically when simulation ends

#### Monitoring
- **Tank management** — remaining volume, % level, days remaining, low level alert per pump
- **Statistics** — weekly / monthly / yearly / total volumes, successful and missed doses
- **7-day graph** — tank level history per pump (Mini Graph Card)
- **Notifications** — pump offline, low tank, missed dose (via HA mobile app)

#### Tools
- **Calibration** — real flow rate measurement (ml/s), auto-deactivates after run
- **Full reset** — clears all stats, preserves calibration, requires confirmation
- **Simulation mode** — test timing and sequencing at configurable intervals (1–10 min)
  - Stats are fully protected (counters and tank volumes unchanged)
  - Optional real pump trigger for hardware verification
  - Red warning banner when active — impossible to forget

#### Interface
- **Desktop + mobile dashboard** — separate files, same features
- **French and English** — dashboard language set by file choice
- **Free pump names** — each pump has a user-defined label displayed throughout the UI
- **Per-pump colors** — via DMC theme CSS variables

---

### Use case examples

| Domain | Pump 1 | Pump 2 | Pump 3 | Pump 4 |
|---|---|---|---|---|
| Reef aquarium | Kh | Ca | Mg | Phyto |
| Hydroponics | Nutrient A | Nutrient B | pH Up | pH Down |
| Indoor plants | N fertilizer | P fertilizer | K fertilizer | Booster |
| Brewing | Acid | Base | Additive 1 | Additive 2 |
| DIY lab | Solution 1 | Solution 2 | Solution 3 | Solution 4 |

> Each pump has a free name, a free color, and its own independent dosing settings.
> No constraints tied to any specific domain.

---

### Project architecture

#### Package files (`packages/dmd/`)

The core logic — split into 5 files for readability:

| File | Role |
|---|---|
| `dmd_config.yaml` | All configurable parameters (names, doses, windows, delays, simulation...) |
| `dmd_sensors.yaml` | Calculated sensors (interval, next dose, window validation, tank levels...) |
| `dmd_scripts.yaml` | Triggerable actions (manual dose, calibration, tank reset, full reset...) |
| `dmd_stat.yaml` | Statistics (volumes, counters, timestamps, missed doses...) |
| `dmd_automations.yaml` | Automatic logic (schedules doses, sends alerts, handles compatibility delays...) |

> These 5 files are **identical regardless of language**. No editing needed — everything is configured from the dashboard.

#### The `common/` folder (`packages/common/notify.yaml`)

Defines the `notify.dmd_admin` service used for all alerts.
Lives in `common/` so it can be shared with other projects on the same HA instance.
**This is the only file you need to edit** — just point it to your mobile device.

#### DMC Theme (`themes/dmc_theme.yaml`)

DIY my Dose uses CSS variables to colorize each pump. Without the theme, all card colors disappear.

| Theme variable | Used for |
|---|---|
| `p1-color` | Pump 1 color (hex) |
| `p2-color` | Pump 2 color (hex) |
| `p3-color` | Pump 3 color (hex) |
| `p4-color` | Pump 4 color (hex) |

> `p1-bg`, `p2-bg`... are auto-generated transparent variants — **do not edit them**.

The theme is separate from the package so it can be reused by other projects.
If you already use a custom theme, just paste the variables into it (see Installation).

#### Dashboards

Visual interface — reads the entities from the package files. Language is set by the file you choose.
Copy only one to HA. Four files available:

| File | Language | Format |
|---|---|---|
| `dashboadmd_dmd_desktop_fr.yaml` | 🇫🇷 Français | Desktop / PC |
| `dashboadmd_dmd_desktop_en.yaml` | 🇬🇧 English | Desktop / PC |
| `dashboadmd_dmd_mobile_fr.yaml` | 🇫🇷 Français | Mobile |
| `dashboadmd_dmd_mobile_en.yaml` | 🇬🇧 English | Mobile |

#### Translations (`translations/`) — GitHub only

Reference files for contributing dashboards in other languages. Do not upload to HA.

---

### Required hardware

- ESP32 with ESPHome — 4 relay switches named `switch.dmd_pompe_1` to `_4`
- Home Assistant 2023.x or higher

### Required HACS cards

- [Mushroom](https://github.com/piitaya/lovelace-mushroom)
- [card-mod](https://github.com/thomasloven/lovelace-card-mod)
- [bar-card](https://github.com/custom-cards/bar-card)
- [mini-graph-card](https://github.com/kalkih/mini-graph-card)

---

### Installation

**1. Enable packages and themes in `configuration.yaml`**
```yaml
homeassistant:
  packages: !include_dir_named packages
frontend:
  themes: !include_dir_merge_named themes
```

**2. Copy the package files**
```
config/
└── packages/
    ├── common/
    │   └── notify.yaml          ← edit this: point to your mobile device
    └── dmd/
        ├── dmd_config.yaml
        ├── dmd_stat.yaml
        ├── dmd_sensors.yaml
        ├── dmd_scripts.yaml
        └── dmd_automations.yaml
```

**3. Copy the theme**
```
config/themes/dmc_theme.yaml
```

> **Already using a custom theme?** No need to switch. Paste these variables into your existing theme:
> ```yaml
> p1-color: "#F5C518"   # Pump 1 — yellow (default)
> p2-color: "#E8547A"   # Pump 2 — pink   (default)
> p3-color: "#4A9FD4"   # Pump 3 — blue   (default)
> p4-color: "#2ECC71"   # Pump 4 — green  (default)
> p1-bg: "rgba(245, 197, 24, 0.12)"
> p2-bg: "rgba(232, 84, 122, 0.12)"
> p3-bg: "rgba(74, 159, 212, 0.12)"
> p4-bg: "rgba(46, 204, 113, 0.12)"
> p1-bg-light: "rgba(245, 197, 24, 0.06)"
> p2-bg-light: "rgba(232, 84, 122, 0.06)"
> p3-bg-light: "rgba(74, 159, 212, 0.06)"
> p4-bg-light: "rgba(46, 204, 113, 0.06)"
> ```

**4. Copy one dashboard**

In HA: **Settings → Dashboards → Add dashboard → paste YAML content**

**5. Restart Home Assistant**

**6. Activate the theme**

User profile → Theme → DMC

**7. First configuration**

In the **Settings** view, configure each pump:
- Name (e.g. "Kh", "Nutrient A", "pH Up"...)
- Tank volume (ml)
- Daily dose (ml)
- Frequency (doses/day)
- Time window (start → end) or enable Full Day mode
- Compatibility delays between pump pairs
- Alert threshold (ml)

Then run a **calibration** before enabling automatic dosing.

---

### Notifications

Edit `packages/common/notify.yaml` to match your mobile device:
```yaml
notify:
  - platform: group
    name: dmd_admin
    services:
      - service: mobile_app_your_device
```

---

### Contributing

Translation files are in `translations/`. To add a new language:
1. Copy `translations/en.yaml`
2. Translate the values
3. Create the corresponding dashboard file
4. Submit a pull request

---

### Credits

The hardware design (peristaltic pump wiring, ESP32 setup) is based on a build
originally published on **[Joy-Reef.com](https://www.joy-reef.com/)**.

DIY my Dose is a complete software rewrite — the original used an Arduino with cloud connectivity.
This project replaces that with a fully local ESP32 + ESPHome solution integrated into Home Assistant, with no cloud dependency.

> If you know of earlier sources that inspired the Joy-Reef hardware design,
> please open an [Issue](../../issues) so we can credit them properly.

---

### License

Copyright (c) 2026 DIY Moi Ca

Personal use only — no commercial use, no redistribution without permission.
See [LICENSE](LICENSE) for full terms.

*Made with ❤️ for the DIY community — [diymoica.com](https://diymoica.com)*

---
---

## Français

### Présentation

DIY my Dose est un contrôleur de dosage liquide universel basé sur Home Assistant et ESP32 (ESPHome).
Il pilote jusqu'à 4 pompes péristaltiques avec dosage automatique, calibration précise, sécurités, alertes et statistiques.

**Conçu pour tout usage de dosage liquide :**
aquariophilie récifale, eau douce, hydroponie, plantes, brassage, laboratoire DIY, et bien plus.
Chaque pompe est entièrement configurable — nom, plage horaire, fréquence, compatibilité avec les autres pompes.
Aucune logique n'est imposée : vous décidez de tout.

> ⚠️ **Pré-release** — testé activement sur un aquarium récifal réel. Les fonctionnalités principales sont opérationnelles.
> Voir la [ROADMAP](ROADMAP.md) pour ce qui arrive avant v1.0.0.

---

### Fonctionnalités

#### Dosage
- **Dosage automatique** — fenêtre horaire (début/fin) configurable par pompe, ou mode 24h complet
- **Planning aligné sur l'horloge** — le mode jour entier aligne les doses aux heures exactes (ex : 24 doses → toutes les heures à :00)
- **Intervalle calculé automatiquement** — dérivé de la durée de fenêtre et de la fréquence demandée
- **Dose manuelle** — déclenchable à tout moment depuis le dashboard
- **Option reset de cycle** — une dose manuelle peut décaler le planning automatique

#### Sécurité
- **Matrice de compatibilité** — délai minimum entre chaque paire de pompes (6 paires configurables), offsets calculés automatiquement
- **Délai mécanique** — délai configurable entre doses séquentielles (1–30s)
- **Validation en temps réel** — alerte si la fenêtre est trop courte pour la fréquence demandée
- **Pause globale** — arrête toutes les pompes instantanément
- **Watchdog HA 3 niveaux** — Soft (arrêt pompe + warning) → Hard (arrêt toutes pompes + verrou) → Verrou (reset forcé). Chaque niveau déclenché à un multiple configurable de la durée de dose (défaut ×2 / ×4 / ×8)
- **Watchdog hardware ESP32** — coupure directe par l'ESP32 à ×6 durée de dose, même si Home Assistant plante ou si le WiFi est coupé
- **Verrou de sécurité** — verrou système activé par le watchdog hard/ESP32/verrou. Bloque tout dosage jusqu'au déverrouillage manuel depuis le dashboard
- **Overrides par pompe** — chaque niveau de watchdog est configurable globalement ou par pompe (mode expert)
- **Validation de hiérarchie** — sensor d'alerte si les multiplicateurs sont mal configurés (soft ≥ hard, etc.)
- **Notification dose bloquée** — notification push quand une dose est empêchée par le verrou de sécurité

#### Matériel
- **Bouton de dose manuelle physique** — un bouton hardware sur l'ESP32 déclenche une dose manuelle, fonctionne même sans accès au dashboard *(prévu en v1.3.0)*

#### Simulation
- **Mode simulation** — testez le timing complet sans affecter les données réelles
- **Intervalle configurable** — simulez les doses toutes les 1–10 min
- **Toggle pompes réelles** — déclenchez optionnellement les vraies pompes en simulation
- **Rechargement auto** — les automations se rechargent automatiquement à la fin de la simulation

#### Surveillance
- **Gestion des réservoirs** — volume restant, % niveau, jours restants, alerte seuil bas par pompe
- **Statistiques** — volumes semaine / mois / année / total, doses réussies et manquées
- **Graphique 7 jours** — historique des niveaux réservoirs par pompe (Mini Graph Card)
- **Notifications** — pompe hors ligne, réservoir bas, dose manquée (via app mobile HA)

#### Outils
- **Calibration** — mesure du débit réel (ml/s), désactivation automatique après la mesure
- **Reset complet** — efface toutes les stats, préserve la calibration, confirmation requise
- **Mode simulation** — testez le timing et l'enchaînement à intervalle configurable (1–10 min)
  - Stats entièrement protégées (compteurs et volumes réservoirs inchangés)
  - Déclenchement réel des pompes optionnel (pour vérification hardware)
  - Bannière rouge visible quand actif — impossible d'oublier

#### Interface
- **Dashboard desktop + mobile** — fichiers séparés, mêmes fonctionnalités
- **Français et anglais** — la langue est définie par le fichier de dashboard choisi
- **Noms libres par pompe** — label personnalisé affiché dans toute l'interface
- **Couleurs par pompe** — via les variables CSS du thème DMC

---

### Exemples d'utilisation

| Domaine | Pompe 1 | Pompe 2 | Pompe 3 | Pompe 4 |
|---|---|---|---|---|
| Aquariophilie récifale | Kh | Ca | Mg | Phyto |
| Hydroponie | Nutriment A | Nutriment B | pH Up | pH Down |
| Plantes d'intérieur | Engrais N | Engrais P | Engrais K | Stimulateur |
| Brassage | Acide | Base | Additif 1 | Additif 2 |
| Laboratoire | Solution 1 | Solution 2 | Solution 3 | Solution 4 |

> Chaque pompe a un nom libre, une couleur libre, et ses propres réglages de dosage indépendants.
> Aucune contrainte liée à un domaine d'application spécifique.

---

### Architecture du projet

#### Les fichiers package (`packages/dmd/`)

Le cœur du projet — découpé en 5 fichiers pour rester lisible :

| Fichier | Rôle |
|---|---|
| `dmd_config.yaml` | Tous les paramètres configurables (noms, doses, fenêtres, délais, simulation...) |
| `dmd_sensors.yaml` | Les capteurs calculés (intervalle, prochaine dose, validation fenêtre, niveaux...) |
| `dmd_scripts.yaml` | Les actions déclenchables (dose manuelle, calibration, reset réservoir, reset total...) |
| `dmd_stat.yaml` | Les statistiques (volumes, compteurs, horodatages, doses manquées...) |
| `dmd_automations.yaml` | La logique automatique (planifie les doses, envoie les alertes, gère les délais...) |

> Ces 5 fichiers sont **identiques quelle que soit la langue choisie**. Aucune modification nécessaire — tout se configure depuis le dashboard.

#### Le dossier `common/` (`packages/common/notify.yaml`)

Définit le service `notify.dmd_admin` utilisé pour toutes les alertes.
Dans `common/` pour pouvoir être partagé avec d'autres projets sur le même HA.
**C'est le seul fichier à modifier** — indiquez simplement votre appareil mobile.

#### Le thème DMC (`themes/dmc_theme.yaml`)

DIY my Dose utilise des variables CSS pour coloriser chaque pompe. Sans le thème, toutes les couleurs disparaissent.

| Variable dans le thème | Utilisation |
|---|---|
| `p1-color` | Couleur de la pompe 1 (hex) |
| `p2-color` | Couleur de la pompe 2 (hex) |
| `p3-color` | Couleur de la pompe 3 (hex) |
| `p4-color` | Couleur de la pompe 4 (hex) |

> `p1-bg`, `p2-bg`... sont des variantes transparentes calculées automatiquement — **ne pas les modifier**.

Le thème est séparé du package pour pouvoir être réutilisé par d'autres projets.
Si vous utilisez déjà un thème personnalisé, copiez simplement les variables dedans (voir Installation).

#### Les dashboards

Interface visuelle — lit les entités créées par les fichiers package. La langue est définie par le fichier choisi.
N'en copier qu'un seul sur HA. Quatre fichiers disponibles :

| Fichier | Langue | Format |
|---|---|---|
| `dashboadmd_dmd_desktop_fr.yaml` | 🇫🇷 Français | Desktop / PC |
| `dashboadmd_dmd_desktop_en.yaml` | 🇬🇧 English | Desktop / PC |
| `dashboadmd_dmd_mobile_fr.yaml` | 🇫🇷 Français | Mobile |
| `dashboadmd_dmd_mobile_en.yaml` | 🇬🇧 English | Mobile |

#### Les traductions (`translations/`) — GitHub uniquement

Fichiers de référence pour contribuer des dashboards dans d'autres langues. Ne pas uploader sur HA.

---

### Matériel requis

- ESP32 avec ESPHome — 4 relais nommés `switch.dmd_pompe_1` à `_4`
- Home Assistant 2023.x ou supérieur

### Cartes HACS requises

- [Mushroom](https://github.com/piitaya/lovelace-mushroom)
- [card-mod](https://github.com/thomasloven/lovelace-card-mod)
- [bar-card](https://github.com/custom-cards/bar-card)
- [mini-graph-card](https://github.com/kalkih/mini-graph-card)

---

### Installation

**1. Activer les packages et thèmes dans `configuration.yaml`**
```yaml
homeassistant:
  packages: !include_dir_named packages
frontend:
  themes: !include_dir_merge_named themes
```

**2. Copier les fichiers package**
```
config/
└── packages/
    ├── common/
    │   └── notify.yaml          ← à modifier : indiquer votre appareil mobile
    └── dmd/
        ├── dmd_config.yaml
        ├── dmd_stat.yaml
        ├── dmd_sensors.yaml
        ├── dmd_scripts.yaml
        └── dmd_automations.yaml
```

**3. Copier le thème**
```
config/themes/dmc_theme.yaml
```

> **Vous utilisez déjà un thème personnalisé ?** Pas besoin d'en changer. Collez ces variables dans votre thème existant :
> ```yaml
> p1-color: "#F5C518"   # Pompe 1 — jaune (défaut)
> p2-color: "#E8547A"   # Pompe 2 — rose  (défaut)
> p3-color: "#4A9FD4"   # Pompe 3 — bleu  (défaut)
> p4-color: "#2ECC71"   # Pompe 4 — vert  (défaut)
> p1-bg: "rgba(245, 197, 24, 0.12)"
> p2-bg: "rgba(232, 84, 122, 0.12)"
> p3-bg: "rgba(74, 159, 212, 0.12)"
> p4-bg: "rgba(46, 204, 113, 0.12)"
> p1-bg-light: "rgba(245, 197, 24, 0.06)"
> p2-bg-light: "rgba(232, 84, 122, 0.06)"
> p3-bg-light: "rgba(74, 159, 212, 0.06)"
> p4-bg-light: "rgba(46, 204, 113, 0.06)"
> ```

**4. Copier un dashboard**

Dans HA : **Paramètres → Tableaux de bord → Ajouter → coller le contenu YAML**

**5. Redémarrer Home Assistant**

**6. Activer le thème**

Profil utilisateur → Thème → DMC

**7. Premier paramétrage**

Dans la vue **Réglages**, configurer chaque pompe :
- Nom (ex : "Kh", "Nutriment A", "pH Up"...)
- Volume du réservoir (ml)
- Dose quotidienne (ml)
- Fréquence (doses/jour)
- Fenêtre horaire (début → fin) ou activer le mode Jour Entier
- Délais de compatibilité entre paires de pompes
- Seuil d'alerte (ml)

Puis lancer une **calibration** avant d'activer le dosage automatique.

---

### Notifications

Modifier `packages/common/notify.yaml` :
```yaml
notify:
  - platform: group
    name: dmd_admin
    services:
      - service: mobile_app_votre_appareil
```

---

### Contribuer

Les fichiers de traduction sont dans `translations/`. Pour ajouter une nouvelle langue :
1. Copier `translations/en.yaml`
2. Traduire les valeurs
3. Créer le dashboard correspondant
4. Soumettre une pull request

---

### Crédits

Le montage électronique (câblage des pompes péristaltiques, configuration ESP32)
est basé sur un montage publié à l'origine sur **[Joy-Reef.com](https://www.joy-reef.com/)**.

DIY my Dose est une réécriture logicielle complète — le montage original utilisait un Arduino avec une connexion cloud.
Ce projet le remplace par une solution entièrement locale ESP32 + ESPHome intégrée à Home Assistant, sans aucune dépendance cloud.

> Si vous connaissez des sources antérieures qui ont inspiré le montage Joy-Reef,
> signalez-le via [Issues](../../issues) pour qu'on puisse les créditer correctement.

---

### Licence

Copyright (c) 2026 DIY Moi Ca

Usage personnel uniquement — pas d'usage commercial, pas de redistribution sans autorisation.
Voir [LICENSE](LICENSE) pour les conditions complètes.

*Fait avec ❤️ pour la communauté DIY — [diymoica.com](https://diymoica.com)*
