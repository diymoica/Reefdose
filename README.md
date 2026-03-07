# ReefDose — Universal Smart Dosing Controller

![Version](https://img.shields.io/badge/version-0.9.4-blue) ![HA](https://img.shields.io/badge/Home%20Assistant-2023.x%2B-41BDF5) ![License](https://img.shields.io/badge/license-Custom-orange) ![Use case](https://img.shields.io/badge/use%20case-universal-brightgreen)

**[English](#english) | [Français](#français)**

---

## English

### Overview

ReefDose is a universal liquid dosing controller built on Home Assistant and ESP32 (ESPHome). It controls up to 4 peristaltic pumps with automatic dosing, precise calibration, alerts and statistics.

**Designed for any liquid dosing use case :**
reef aquariums, freshwater, hydroponics, plant care, brewing, DIY lab, and more.
Each pump is fully configurable — name, schedule window, frequency, compatibility with other pumps. No logic is imposed : you decide everything.

### Features

- **Automatic dosing** — configurable time window (start/end) and frequency per pump
- **Auto-calculated interval** — based on window and requested frequency
- **Compatibility matrix** — configurable minimum delay between each pump pair
- **Real-time validation** — warning if configuration is inconsistent
- **Manual dose** — triggerable at any time from the dashboard
- **Calibration** — real flow rate measurement (ml/s) with auto-deactivation
- **Tank management** — remaining volume, % level, days remaining, low level alert
- **Statistics** — weekly/monthly/yearly/total volumes, successful/missed doses
- **Graph** — 7-day tank level history (Mini Graph Card)
- **Full reset** — resets everything except calibration, with confirmation popup
- **Notifications** — pump offline, low tank, missed dose
- **Global pause** — stops all pumps instantly
- **Desktop + mobile dashboard** — available in French and English

### Project architecture

Before installing, it helps to understand the role of each component.

#### Package files (`packages/reefdose/rd_*.yaml`)

This is the core of the project — the complete logic. Split into 5 files to stay readable:

| File | Role |
|---|---|
| `rd_config.yaml` | All configurable parameters (names, doses, time windows, delays...) |
| `rd_sensors.yaml` | Calculated sensors (interval, next dose, window validation...) |
| `rd_scripts.yaml` | Triggerable actions (manual dose, calibration, reset...) |
| `rd_stat.yaml` | Counters and statistics (dosed volumes, successful/missed doses...) |
| `rd_automations.yaml` | Automatic logic (triggers doses, sends alerts...) |

> These 5 files are **identical regardless of the chosen language**.
> They don't need to be edited — everything is configured from the dashboard.

#### The `common/` folder (`packages/common/notify.yaml`)

This folder is intentionally separate. It contains the definition of the `notify.reefdose_admin` notification service — the address ReefDose uses to send alerts (pump offline, low tank, missed dose...).

It lives in `common/` because it can be shared with other projects on the same HA instance. This is **the only file you need to edit** to point to your mobile device.

#### DMC theme (`themes/dmc_theme.yaml`)

ReefDose uses CSS variables to colorize each pump. Without the theme, all card colors disappear and the dashboard renders in grey.

The mapping is straightforward:

| Theme variable | Pump in dashboard |
|---|---|
| `p1-color` | Pump 1 — the one you named (e.g. "Kh", "Nutrient A"...) |
| `p2-color` | Pump 2 — the one you named (e.g. "Ca", "pH Up"...) |
| `p3-color` | Pump 3 — the one you named (e.g. "Mg", "Nutrient B"...) |
| `p4-color` | Pump 4 — the one you named (e.g. "Phyto", "Booster"...) |

> To change a pump color: open `dmc_theme.yaml` and edit the corresponding hex value.
> The `p1-bg`, `p2-bg`... lines are transparent variants — **do not edit them**.

> ⚠️ Theme file comments are preserved since it is a plain text file edited directly.
> Dashboard YAML comments disappear when saved in HA — this is expected behaviour.

The DMC theme is separate from the package for two reasons:
- It can be reused by other projects on the same HA instance
- If you already have a custom theme, just paste the variables into it

#### The dashboard (1 file of your choice)

This is the visual interface. It reads the entities created by the package files and displays them. It determines the language. You only copy one to HA — the one matching your language and device (desktop or mobile).

#### Translations (`translations/`) — GitHub only

Reference files for the community. Do not upload to HA.

---
### Use case examples

| Domain | Pump 1 | Pump 2 | Pump 3 | Pump 4 |
|---|---|---|---|---|
| Reef aquarium | Kh | Ca | Mg | Phyto |
| Hydroponics | Nutrient A | Nutrient B | pH Up | pH Down |
| Indoor plants | N fertilizer | P fertilizer | K fertilizer | Booster |
| Brewing | Acid | Base | Additive 1 | Additive 2 |
| DIY lab | Solution 1 | Solution 2 | Solution 3 | Solution 4 |

> Each pump has a free name, a free color, and its own dosing settings.
> No constraints tied to any specific application domain.

### Required Hardware

- ESP32 with ESPHome (4 switches: `switch.reefdose_pompe_1` to `_4`)
- Home Assistant (version 2023.x or higher)

### Required Theme

This project requires the **DMC Theme** :
```
github.com/diymoica/dmc-theme
```
Copy `dmc_theme.yaml` into `config/themes/` and activate it in your user profile.

> **Already using a custom theme?**
> No need to switch themes. Simply paste the variables below
> into your existing theme block :
> ```yaml
> # DMC Theme — pump colors
> # p1 = Pump 1 / p2 = Pump 2 / p3 = Pump 3 / p4 = Pump 4
> # Only edit the 4 main colors below
> p1-color: "#F5C518"   # Pump 1 — yellow (default)
> p2-color: "#E8547A"   # Pump 2 — pink   (default)
> p3-color: "#4A9FD4"   # Pump 3 — blue   (default)
> p4-color: "#2ECC71"   # Pump 4 — green  (default)
> # Do not edit the lines below
> p1-bg: "rgba(245, 197, 24, 0.12)"
> p2-bg: "rgba(232, 84, 122, 0.12)"
> p3-bg: "rgba(74, 159, 212, 0.12)"
> p4-bg: "rgba(46, 204, 113, 0.12)"
> p1-bg-light: "rgba(245, 197, 24, 0.06)"
> p2-bg-light: "rgba(232, 84, 122, 0.06)"
> p3-bg-light: "rgba(74, 159, 212, 0.06)"
> p4-bg-light: "rgba(46, 204, 113, 0.06)"
> ```

### Required HACS Cards

- [card-mod](https://github.com/thomasloven/lovelace-card-mod)
- [bar-card](https://github.com/custom-cards/bar-card)
- [mini-graph-card](https://github.com/kalkih/mini-graph-card)
- [mushroom](https://github.com/piitaya/lovelace-mushroom)

### Installation

**1. Copy the package files**
```
config/
└── packages/
    ├── common/
    │   └── notify.yaml
    └── reefdose/
        ├── rd_config.yaml
        ├── rd_stat.yaml
        ├── rd_sensors.yaml
        ├── rd_scripts.yaml
        └── rd_automations.yaml
```
In `configuration.yaml` make sure packages and themes are enabled:
```yaml
homeassistant:
  packages: !include_dir_named packages
frontend:
  themes: !include_dir_merge_named themes
```

**2. Copy the theme**
```
config/themes/dmc_theme.yaml
```

**3. Choose your language and copy the dashboard**

| File | Language | Format |
|---|---|---|
| `dashboard_reefdose_desktop_fr.yaml` | 🇫🇷 Français | Desktop / PC |
| `dashboard_reefdose_desktop_en.yaml` | 🇬🇧 English | Desktop / PC |
| `dashboard_reefdose_mobile_fr.yaml` | 🇫🇷 Français | Mobile |
| `dashboard_reefdose_mobile_en.yaml` | 🇬🇧 English | Mobile |

> **The dashboard you choose determines the interface language.**
> The package files (`rd_*.yaml`) are identical regardless of the chosen language.
> The `translations/` folder is a GitHub-only resource — do not upload it to HA.

In HA: **Settings → Dashboards → Add → paste YAML content**

**4. Restart Home Assistant**

**5. Enable the theme**
User profile → Theme → DMC

**6. First setup**
In the **Settings** view, enter for each pump:
- Free name (e.g. "Nutrient A", "pH Up", "Kh"...)
- Tank volume (ml)
- Daily dose (ml)
- Frequency (doses/day)
- Time window (start → end)
- Pump compatibility delays
- Alert threshold (ml)

Then run a **calibration** before the first automatic dosing.

### Notifications

The package uses `notify.reefdose_admin`. Define this service in:
```yaml
# packages/common/notify.yaml
notify:
  - platform: group
    name: reefdose_admin
    services:
      - service: mobile_app_your_device
```

### About translations

> ⚠️ The `translations/` folder is a **GitHub-only resource** — do not upload it to Home Assistant.

The dashboard language is determined solely by the dashboard file you choose at installation. Translation files are a reference for contributing new dashboards in other languages.

**What goes on HA:**
```
config/
├── packages/common/notify.yaml
├── packages/reefdose/rd_config.yaml
├── packages/reefdose/rd_sensors.yaml
├── packages/reefdose/rd_scripts.yaml
├── packages/reefdose/rd_stat.yaml
├── packages/reefdose/rd_automations.yaml
├── themes/dmc_theme.yaml
└── (dashboard pasted via HA UI — only one needed)
```

**What stays on GitHub only:**
```
translations/        ← reference for contributing new dashboards
├── fr.yaml   ✅
├── en.yaml   ✅
├── es.yaml   ✅
├── it.yaml   ✅
└── de.yaml   ✅
```

**Contributing a new dashboard**
Copy `translations/en.yaml`, translate the values, create the corresponding dashboard and submit a pull request.

---

*Made with ❤️ for the DIY community — [diymoica.com](https://diymoica.com)*

---

## License

Copyright (c) 2026 DIY Moi Ca

Personal use only — no commercial use, no redistribution without permission.
See [LICENSE](LICENSE) for full terms.
## Français

### Présentation

ReefDose est un contrôleur de dosage liquide universel basé sur Home Assistant et ESP32 (ESPHome). Il pilote jusqu'à 4 pompes péristaltiques avec dosage automatique, calibration précise, alertes et statistiques.

**Conçu pour tout usage de dosage liquide :**
aquariophilie récifale, eau douce, hydroponie, plantes, brassage, laboratoire DIY, et bien plus.
Chaque pompe est entièrement configurable — nom, plage horaire, fréquence, compatibilité avec les autres pompes. Aucune logique n'est imposée : vous décidez de tout.

### Fonctionnalités

- **Dosage automatique** — fenêtre horaire (début/fin) et fréquence configurables par pompe
- **Intervalle calculé automatiquement** — basé sur la fenêtre et la fréquence choisie
- **Matrice de compatibilité** — délai minimum configurable entre chaque paire de pompes
- **Validation en temps réel** — alerte si la configuration est incohérente
- **Dose manuelle** — déclenchable à tout moment depuis le dashboard
- **Calibration** — mesure du débit réel (ml/s) avec désactivation automatique
- **Gestion des réservoirs** — volume restant, % niveau, jours restants, alerte seuil bas
- **Statistiques** — volumes semaine/mois/année/total, doses réussies/manquées
- **Graphique** — historique 7 jours des niveaux réservoirs (Mini Graph Card)
- **Reset complet** — remet tout à zéro sauf la calibration, avec confirmation
- **Notifications** — pompe hors ligne, réservoir bas, dose manquée
- **Pause globale** — arrête toutes les pompes instantanément
- **Dashboard desktop + mobile** — disponible en français et anglais

### Architecture du projet

Avant d'installer, il est utile de comprendre le rôle de chaque composant.

#### Les fichiers package (`packages/reefdose/rd_*.yaml`)

C'est le cœur du projet — la logique complète. Ils sont découpés en 5 fichiers pour rester lisibles :

| Fichier | Rôle |
|---|---|
| `rd_config.yaml` | Tous les paramètres configurables (noms, doses, fenêtres horaires, délais...) |
| `rd_sensors.yaml` | Les capteurs calculés (intervalle, prochaine dose, validation fenêtre...) |
| `rd_scripts.yaml` | Les actions déclenchables (dose manuelle, calibration, reset...) |
| `rd_stat.yaml` | Les compteurs et statistiques (volumes dosés, doses réussies/manquées...) |
| `rd_automations.yaml` | La logique automatique (déclenche les doses, envoie les alertes...) |

> Ces 5 fichiers sont **identiques quelle que soit la langue choisie**.
> Ils n'ont pas besoin d'être modifiés — tout se configure depuis le dashboard.

#### Le dossier `common/` (`packages/common/notify.yaml`)

Ce dossier est séparé du reste exprès. Il contient la définition du service de notification `notify.reefdose_admin` — l'adresse à laquelle ReefDose envoie ses alertes (pompe hors ligne, réservoir bas, dose manquée...).

Il est dans `common/` car il peut être partagé avec d'autres projets sur le même HA. C'est **le seul fichier que tu dois éditer** pour indiquer ton appareil mobile.

#### Le thème DMC (`themes/dmc_theme.yaml`)

ReefDose utilise des variables CSS pour coloriser chaque pompe. Sans le thème, toutes les couleurs des cartes disparaissent et le dashboard s'affiche en gris.

La correspondance est simple :

| Variable dans le thème | Pompe dans le dashboard |
|---|---|
| `p1-color` | Pompe 1 — celle que tu as nommée (ex: "Kh", "Nutrient A"...) |
| `p2-color` | Pompe 2 — celle que tu as nommée (ex: "Ca", "pH Up"...) |
| `p3-color` | Pompe 3 — celle que tu as nommée (ex: "Mg", "Nutrient B"...) |
| `p4-color` | Pompe 4 — celle que tu as nommée (ex: "Phyto", "Booster"...) |

> Pour changer la couleur d'une pompe : ouvrir `dmc_theme.yaml` et modifier la valeur hex correspondante.
> Les lignes `p1-bg`, `p2-bg`... sont des variantes transparentes calculées automatiquement — **ne pas les modifier**.

> ⚠️ Les commentaires du fichier thème sont préservés car c'est un fichier texte édité directement.
> En revanche, les commentaires du dashboard YAML disparaissent à l'enregistrement dans HA — c'est normal.

Le thème DMC est séparé du package pour deux raisons :
- Il peut être réutilisé par d'autres projets sur le même HA
- Si tu as déjà un thème personnalisé, tu colles juste les variables dedans

#### Le dashboard (1 fichier au choix)

C'est l'interface visuelle. Il lit les entités créées par les fichiers package et les affiche. C'est lui qui détermine la langue. Tu n'en copies qu'un seul sur HA — celui qui correspond à ta langue et ton support (desktop ou mobile).

#### Les traductions (`translations/`) — GitHub uniquement

Fichiers de référence pour la communauté. Ne pas uploader sur HA.

---
### Exemples d'utilisation

| Domaine | Pompe 1 | Pompe 2 | Pompe 3 | Pompe 4 |
|---|---|---|---|---|
| Aquariophilie récifale | Kh | Ca | Mg | Phyto |
| Hydroponie | Nutriment A | Nutriment B | pH Up | pH Down |
| Plantes d'intérieur | Engrais N | Engrais P | Engrais K | Stimulateur |
| Brassage | Acide | Base | Additif 1 | Additif 2 |
| Laboratoire | Solution 1 | Solution 2 | Solution 3 | Solution 4 |

> Chaque pompe a un nom libre, une couleur libre, et ses propres réglages de dosage.
> Aucune contrainte liée à un domaine d'application.

### Matériel requis

- ESP32 avec ESPHome (4 switchs : `switch.reefdose_pompe_1` à `_4`)
- Home Assistant (version 2023.x ou supérieure)

### Thème requis

Ce projet nécessite le **Thème DMC** :
```
github.com/diymoica/dmc-theme
```
Copier `dmc_theme.yaml` dans `config/themes/` et l'activer dans votre profil utilisateur.

> **Vous utilisez déjà un thème personnalisé ?**
> Pas besoin de changer de thème. Copiez simplement les variables ci-dessous
> dans votre thème existant :
> ```yaml
> # DMC Theme — couleurs des pompes
> # p1 = Pompe 1 / p2 = Pompe 2 / p3 = Pompe 3 / p4 = Pompe 4
> # Modifier uniquement les 4 couleurs principales ci-dessous
> p1-color: "#F5C518"   # Pompe 1 — jaune  (par défaut)
> p2-color: "#E8547A"   # Pompe 2 — rose   (par défaut)
> p3-color: "#4A9FD4"   # Pompe 3 — bleu   (par défaut)
> p4-color: "#2ECC71"   # Pompe 4 — vert   (par défaut)
> # Ne pas modifier les lignes suivantes
> p1-bg: "rgba(245, 197, 24, 0.12)"
> p2-bg: "rgba(232, 84, 122, 0.12)"
> p3-bg: "rgba(74, 159, 212, 0.12)"
> p4-bg: "rgba(46, 204, 113, 0.12)"
> p1-bg-light: "rgba(245, 197, 24, 0.06)"
> p2-bg-light: "rgba(232, 84, 122, 0.06)"
> p3-bg-light: "rgba(74, 159, 212, 0.06)"
> p4-bg-light: "rgba(46, 204, 113, 0.06)"
> ```

### Cartes HACS requises

- [card-mod](https://github.com/thomasloven/lovelace-card-mod)
- [bar-card](https://github.com/custom-cards/bar-card)
- [mini-graph-card](https://github.com/kalkih/mini-graph-card)
- [mushroom](https://github.com/piitaya/lovelace-mushroom)

### Installation

**1. Copier les fichiers package**
```
config/
└── packages/
    ├── common/
    │   └── notify.yaml
    └── reefdose/
        ├── rd_config.yaml
        ├── rd_stat.yaml
        ├── rd_sensors.yaml
        ├── rd_scripts.yaml
        └── rd_automations.yaml
```
Dans `configuration.yaml` vérifier que les packages sont activés :
```yaml
homeassistant:
  packages: !include_dir_named packages
frontend:
  themes: !include_dir_merge_named themes
```

**2. Copier le thème**
```
config/themes/dmc_theme.yaml
```

**3. Choisir sa langue et copier le dashboard**

| Fichier | Langue | Format |
|---|---|---|
| `dashboard_reefdose_desktop_fr.yaml` | 🇫🇷 Français | Desktop / PC |
| `dashboard_reefdose_desktop_en.yaml` | 🇬🇧 English | Desktop / PC |
| `dashboard_reefdose_mobile_fr.yaml` | 🇫🇷 Français | Mobile |
| `dashboard_reefdose_mobile_en.yaml` | 🇬🇧 English | Mobile |

> **C'est le choix du dashboard qui détermine la langue de l'interface.**
> Les fichiers package (`rd_*.yaml`) sont identiques quelle que soit la langue choisie.
> Le dossier `translations/` est une ressource GitHub uniquement — ne pas l'uploader sur HA.

Dans HA : **Paramètres → Tableaux de bord → Ajouter → coller le contenu YAML**

**4. Redémarrer Home Assistant**

**5. Activer le thème**
Profil utilisateur → Thème → DMC

**6. Premier paramétrage**
Dans la vue **Réglages**, saisir pour chaque pompe :
- Nom libre (ex: "Nutriment A", "pH Up", "Kh"...)
- Volume du réservoir (ml)
- Dose quotidienne (ml)
- Fréquence (doses/jour)
- Fenêtre horaire (début → fin)
- Délais de compatibilité entre pompes
- Seuil d'alerte (ml)

Puis lancer une **calibration** avant le premier dosage automatique.

### Notifications

Le package utilise `notify.reefdose_admin`. Définir ce service dans :
```yaml
# packages/common/notify.yaml
notify:
  - platform: group
    name: reefdose_admin
    services:
      - service: mobile_app_votre_appareil
```

### À propos des traductions

> ⚠️ Le dossier `translations/` est une **ressource GitHub uniquement** — ne pas uploader sur Home Assistant.

La langue de l'interface est déterminée uniquement par le dashboard choisi à l'installation. Les fichiers de traduction servent de référence pour créer de nouveaux dashboards dans d'autres langues.

**Ce qui va sur HA :**
```
config/
├── packages/common/notify.yaml
├── packages/reefdose/rd_config.yaml
├── packages/reefdose/rd_sensors.yaml
├── packages/reefdose/rd_scripts.yaml
├── packages/reefdose/rd_stat.yaml
├── packages/reefdose/rd_automations.yaml
├── themes/dmc_theme.yaml
└── (dashboard collé via l'interface HA — un seul suffit)
```

**Ce qui reste sur GitHub uniquement :**
```
translations/        ← référence pour contribuer de nouveaux dashboards
├── fr.yaml   ✅
├── en.yaml   ✅
├── es.yaml   ✅
├── it.yaml   ✅
└── de.yaml   ✅
```

**Contribuer un nouveau dashboard**
Copier `translations/en.yaml`, traduire les valeurs, créer le dashboard correspondant et soumettre une pull request.

---

