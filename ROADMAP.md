# ReefDose — Roadmap

**[English](#english) | [Français](#français)**

---

## English

### Current status : v0.9.8 — Pre-release testing

The project is actively tested on a real reef aquarium.
Core dosing features are functional and validated. The goal of v1.0.0 is to confirm 48h stability before the first public stable release.

---

### ✅ Already available (v0.9.8)

- Automatic dosing — 4 pumps, configurable time window per pump
- Full day mode — clock-aligned dosing (e.g. 24 doses → every hour at :00)
- Compatibility matrix — minimum delay between each pump pair, offsets calculated automatically
- Manual dose — triggerable at any time
- Cycle reset option — manual dose can optionally shift the auto schedule
- Real-time validation — warning when configuration is inconsistent
- Calibration — real flow rate measurement (ml/s)
- Tank management — volume, %, days remaining, low level alert
- Statistics — weekly / monthly / yearly / total volumes, successful and missed doses
- 7-day tank level graph
- Push notifications — pump offline, low tank, missed dose
- Simulation mode — test timing and sequencing without affecting real data
- Automatic reload after simulation ends
- Global pause
- Full reset with confirmation
- Desktop + mobile dashboard (FR / EN)
- Free pump names and colors

---

### 🔜 v1.0.0 — First stable release

- Hardware safety timeout (ESP32 globals, configured via tutorial)
- HA watchdog (soft + hard levels)
- Responsive single-file dashboard (desktop + mobile merged)
- Interactive setup tutorial (7 steps)
- Complete real-world validation (48h continuous run)
- Final documentation

---

### 🔮 v1.1.0 — Planned

#### Recalculate remaining doses on config change
When a pump's schedule is changed mid-day, the system currently ignores doses already delivered and creates a full new plan — causing overdosing.

Fix : on any config change, read `counter.rd_pX_daily_count`, calculate remaining doses = new frequency - already delivered, and spread only what's left across the remaining window. If already at or above the new frequency → block until midnight with a warning.

**Complexity : medium (2–3h). Workaround until then : avoid changing pump mode mid-day.**

#### Precision improvement — sub-second dose duration
Dose durations are currently calculated as integers (whole seconds).
Fix : use float durations (0.1s resolution) for 10× better precision on small doses.

#### Configurable volume limits (tank / remaining / alert threshold)
Prevent the user from setting a remaining volume or alert threshold above the tank capacity.
Fix : dynamically update the `max` of `input_number` entities when tank volume changes.

#### Emergency Bluetooth mode
When WiFi is unavailable, the ESP32 can be controlled via Bluetooth as an emergency fallback.
Minimal interface : trigger a manual dose per pump. No stats, no schedule — just keep the tank alive.
Requires a lightweight BLE interface or companion app.

#### ESP32 offline fallback
When Home Assistant is unreachable, the ESP32 continues dosing independently using a local schedule stored directly in ESPHome. Statistics are synchronized when HA reconnects.

Requires an external RTC module (DS3231 recommended, ~2-5€) for accurate timekeeping without WiFi.

#### Modular pump expansion
Support for more than 4 pumps via additional ESP32 modules (`reefdose_2`, `reefdose_3`...).
All modules managed from a single dashboard.

#### Per-pump color customization
Choose pump colors directly from the dashboard, without editing theme files.

---

### 💡 Ideas under consideration

- Dose confirmation via push notification
- Weekly dose summary push report (every Monday)
- Holiday / pause mode (suspend dosing for X days)
- Multi-system support (2 independent dosing setups on the same HA instance)

---

### Not planned

- Cloud connectivity — ReefDose is and will remain fully local
- Mobile app — the HA mobile app covers notifications
- Support for more than 4 pumps on a single ESP32 — hardware limitation

---

## Français

### État actuel : v0.9.8 — Tests pré-release

Le projet est testé activement sur un aquarium récifal réel.
Les fonctionnalités de dosage sont opérationnelles et validées. L'objectif de la v1.0.0 est de confirmer la stabilité sur 48h avant la première release publique stable.

---

### ✅ Déjà disponible (v0.9.8)

- Dosage automatique — 4 pompes, fenêtre horaire configurable par pompe
- Mode jour entier — dosage aligné sur l'horloge (ex : 24 doses → toutes les heures à :00)
- Matrice de compatibilité — délai minimum entre chaque paire de pompes, offsets calculés automatiquement
- Dose manuelle — déclenchable à tout moment
- Option reset de cycle — une dose manuelle peut décaler le planning automatique
- Validation en temps réel — alerte si la configuration est incohérente
- Calibration — mesure du débit réel (ml/s)
- Gestion des réservoirs — volume, %, jours restants, alerte seuil bas
- Statistiques — volumes semaine / mois / année / total, doses réussies et manquées
- Graphique historique 7 jours des niveaux réservoirs
- Notifications push — pompe hors ligne, réservoir bas, dose manquée
- Mode simulation — testez le timing et l'enchaînement sans affecter les vraies données
- Rechargement automatique des automations après fin de simulation
- Pause globale
- Reset complet avec confirmation
- Dashboard desktop + mobile (FR / EN)
- Noms et couleurs libres par pompe

---

### 🔜 v1.0.0 — Première release stable

- Sécurité hardware timeout (globals ESP32, configuré via tutoriel)
- Watchdog HA (niveaux doux + dur)
- Dashboard responsive fichier unique (desktop + mobile fusionnés)
- Tutoriel d'installation interactif (7 étapes)
- Validation complète en conditions réelles (run continu 48h)
- Documentation finale

---

### 🔮 v1.1.0 — Prévu

#### Recalcul des doses restantes lors d'un changement de config
Quand la configuration d'une pompe change en cours de journée, le système ignore les doses déjà livrées et crée un nouveau planning complet — causant un surdosage.

Correction : à chaque changement de config, lire `counter.rd_pX_daily_count`, calculer les doses restantes = nouvelle fréquence - doses déjà livrées, et n'étaler que ce qui reste sur la fenêtre restante. Si le compteur atteint déjà la nouvelle fréquence → blocage jusqu'à minuit avec warning.

**Complexité : moyenne (2–3h). Contournement en attendant : éviter de changer le mode d'une pompe en cours de journée.**

#### Précision — durées de dose en dixièmes de seconde
Les durées sont actuellement calculées en secondes entières.
Correction : utiliser des valeurs float (résolution 0.1s) pour une précision 10× supérieure sur les petites doses.

#### Limites de volume cohérentes (réservoir / restant / seuil alerte)
Empêcher l'utilisateur de saisir un volume restant ou un seuil d'alerte supérieur au volume du réservoir.
Correction : mettre à jour dynamiquement le `max` des `input_number` quand le volume du réservoir change.

#### Mode urgence Bluetooth
En cas de perte WiFi, l'ESP32 peut être contrôlé via Bluetooth en mode urgence.
Interface minimale : déclencher une dose manuelle par pompe. Pas de stats, pas de planning — juste garder le bac en vie.

#### Dosage offline ESP32
Quand Home Assistant est inaccessible, l'ESP32 continue à doser de manière autonome grâce à un planning stocké directement dans ESPHome. Les statistiques sont synchronisées au retour de la connexion.

Nécessite un module RTC externe (DS3231 recommandé, ~2-5€) pour maintenir l'heure sans WiFi.

#### Expansion modulaire des pompes
Support de plus de 4 pompes via des modules ESP32 supplémentaires (`reefdose_2`, `reefdose_3`...).
Tous les modules gérés depuis un seul dashboard.

#### Personnalisation des couleurs par pompe
Choisir les couleurs des pompes directement depuis le dashboard, sans modifier les fichiers de thème.

---

### 💡 Idées en réflexion

- Confirmation de dose par notification push
- Rapport hebdomadaire par push (chaque lundi)
- Mode vacances / pause (suspendre le dosage pendant X jours)
- Support multi-système (2 systèmes de dosage indépendants sur le même HA)

---

### Hors périmètre

- Connexion cloud — ReefDose est et restera entièrement local
- Application mobile — l'app HA couvre les notifications
- Plus de 4 pompes sur un seul ESP32 — limitation matérielle
