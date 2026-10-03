# Tado-Automatic

[Français](#français) · [English](#english)

## Français

Workflow n8n pour adapter une zone de chauffage Tado à la température intérieure, à la météo, aux horaires et à la saison, avec alertes Telegram d'humidité et de fenêtre ouverte.

### Fichier

[Tado.json](Tado.json) contient les 23 nœuds du workflow. L'export est désactivé et anonymisé : aucune donnée épinglée, aucun identifiant de credential, token de session ou état d'exécution n'est inclus.

### Changements

- Décision de chauffage enrichie : présence lorsqu'elle est disponible, tendance sur des mesures distinctes, anticipation horaire, estimation solaire et inertie thermique.
- Prévisions météo sur cinq jours : anticipation limitée des redoux et refroidissements, avec priorité au confort mesuré.
- Contrôle de la fraîcheur des mesures Tado et de la météo, avec cache temporaire des prévisions.
- Consignes de 6 à 25 °C : pas de 1 °C entre 6 et 14 °C, puis de 0,5 °C à partir de 15 °C. Le code demande une consigne en mode ON et respecte l'arrêt manuel selon `RESPECT_MANUAL_OFF`.
- Humidité : seuils normal/critique, hystérésis, confirmation sur plusieurs mesures, estimation du point de rosée et comparaison avec l'air extérieur.
- Fenêtre ouverte : signal natif Tado, passage à la consigne éco, temporisation et alerte Telegram avec rappel configurable.
- Les délais entre alertes ne sont mémorisés qu'après confirmation de l'envoi Telegram.

### Configuration

1. Importer `Tado.json` et laisser le workflow désactivé pendant la configuration.
2. Dans **Configuration Tado**, remplacer `YOUR_TADO_EMAIL`, `YOUR_TADO_PASSWORD`, `YOUR_CLIENT_ID`, `YOUR_HOME_ID`, `YOUR_ZONE_ID` et `YOUR_ID_TELEGRAM`.
3. Dans **Configuration Logement**, renseigner `Lat` et `Lon` (valeurs neutres `0`), l'orientation des fenêtres, le fuseau horaire et les paramètres de confort.
4. Dans **Lire météo**, remplacer `YOUR_CITY,COUNTRY_CODE`, sélectionner les credentials OpenWeatherMap et conserver **5 Day Forecast** avec le format **Metric** (°C).
5. Sélectionner les credentials du bot dans les deux nœuds Telegram.
6. Dans **HTTP Request**, configurer Browserless et remplacer `YOUR_BROWSERLESS_TOKEN`. Ce nœud sert à la récupération de l'authentification Tado par device code.
7. Vérifier l'authentification, l'état de la zone et la sortie de **Décision chauffage** avant d'exécuter **Appliquer overlay Tado**, qui peut modifier la consigne réelle. Avec des prévisions fraîches et suffisamment complètes, `debug_previsions.utilisables` doit être `true` ; `quality.weather_is_forecast_estimate` indique si la température extérieure est estimée.
8. Activer le workflow une fois les réglages vérifiés. Le planificateur fonctionne toutes les 30 minutes.

Les données statiques du workflow conservent les tokens et l'historique de décision dans votre instance. Après un nouvel import, le parcours device code initialise l'authentification.

### Réglages principaux

| Réglage | Valeur | Usage |
|---|---:|---|
| `Min_thérmostat` / `Max_thérmostat` | 6 / 25 | Bornes de consigne |
| `MIN_DELAY_MINUTES` | 25 | Délai entre ajustements, sauf urgence |
| `SENSOR_MAX_AGE_MINUTES` | 90 | Fraîcheur maximale des mesures |
| `WEATHER_MAX_AGE_MINUTES` | 120 | Fraîcheur maximale de la météo observée |
| `FORECAST_ENABLED` | `true` | Activer l'anticipation météo |
| `FORECAST_MAX_AGE_MINUTES` | 180 | Âge maximal du cache depuis sa récupération |
| `FORECAST_HORIZON_HOURS` | 6 | Horizon des décisions proches, réglable de 6 à 12 heures |
| `FORECAST_MAX_REDUCTION_C` / `FORECAST_MAX_BOOST_C` | 0,5 / 0,5 | Corrections maximales par défaut liées aux prévisions |
| `FORECAST_MILD_MIN_C` / `FORECAST_ECO_MIN_C` | 16 / 18 | Seuils de douceur à court terme / sur 24 heures |
| `HUMIDITY_LOW_PERCENT` / `HUMIDITY_HIGH_PERCENT` | 30 / 60 | Seuils d'humidité |
| `HUMIDITY_CRITICAL_LOW_PERCENT` / `HUMIDITY_CRITICAL_HIGH_PERCENT` | 25 / 70 | Seuils critiques |
| `HUMIDITY_CONFIRM_MINUTES` | 30 | Confirmation de l'anomalie |
| `HUMIDITY_REMINDER_HOURS` | 12 | Rappel standard |
| `WINDOW_HOLD_MINUTES` | 15 | Temporisation du signal d'ouverture |
| `WINDOW_REMINDER_MINUTES` | 60 | Rappel fenêtre ouverte |

Les paramètres `FORECAST_*` utilisent ces valeurs par défaut dans le code. Ajouter les champs souhaités dans **Configuration Logement** pour les personnaliser.

La fréquence des alertes dépend du planificateur : une confirmation réglée à 5 minutes ne déclenche pas un contrôle toutes les 5 minutes si le workflow tourne toutes les 30 minutes.

### Limites et validation

Le signal d'ouverture est déduit par Tado ; il ne remplace pas un capteur de contact. L'apport solaire et la température de paroi sont estimés. La présence utilise la configuration si aucune donnée exploitable n'est disponible.

JSON, connexions, syntaxe JavaScript et décisions simulées ont été contrôlés localement. Les services Tado, météo, Browserless et Telegram doivent être vérifiés sur l'installation cible.

## English

n8n workflow that adjusts a Tado heating zone based on indoor temperature, weather, schedules and the season, with Telegram alerts for humidity and open windows.

### File

[Tado.json](Tado.json) contains the workflow's 23 nodes. The export is inactive and anonymized: it includes no pinned data, credential identifiers, session tokens or execution state.

### Changes

- Enhanced heating decisions: occupancy when available, trends based on distinct measurements, schedule anticipation, estimated solar gain and thermal inertia.
- Five-day weather forecasts: limited anticipation of warming and cooling, with priority given to measured indoor comfort.
- Freshness checks for Tado measurements and weather data, with a temporary forecast cache.
- Setpoints from 6 to 25 °C: 1 °C increments between 6 and 14 °C, then 0.5 °C increments from 15 °C. The code requests a setpoint in ON mode and respects manual OFF according to `RESPECT_MANUAL_OFF`.
- Humidity: normal and critical thresholds, hysteresis, confirmation over multiple measurements, estimated dew point and comparison with outdoor air.
- Open windows: native Tado signal, switch to the eco setpoint, a hold period and a Telegram alert with a configurable reminder.
- Alert cooldowns are recorded only after Telegram confirms delivery.

### Configuration

1. Import `Tado.json` and leave the workflow inactive during configuration.
2. In **Configuration Tado**, replace `YOUR_TADO_EMAIL`, `YOUR_TADO_PASSWORD`, `YOUR_CLIENT_ID`, `YOUR_HOME_ID`, `YOUR_ZONE_ID` and `YOUR_ID_TELEGRAM`.
3. In **Configuration Logement**, enter `Lat` and `Lon` (neutral default values: `0`), window orientation, time zone and comfort settings.
4. In **Lire météo**, replace `YOUR_CITY,COUNTRY_CODE`, select your OpenWeatherMap credentials and keep **5 Day Forecast** with the **Metric** format (°C).
5. Select your bot credentials in both Telegram nodes.
6. In **HTTP Request**, configure Browserless and replace `YOUR_BROWSERLESS_TOKEN`. This node handles Tado authentication through the device code flow.
7. Check authentication, the zone's state and the output of **Décision chauffage** before running **Appliquer overlay Tado**, which can change the actual setpoint. With fresh forecasts and sufficient time-slot coverage, `debug_previsions.utilisables` should be `true`; `quality.weather_is_forecast_estimate` indicates whether outdoor temperature is estimated.
8. Activate the workflow once you have checked the settings. The scheduler runs every 30 minutes.

The workflow's static data stores tokens and the decision history in your instance. After a fresh import, the device code flow initializes authentication.

### Main settings

| Setting | Value | Purpose |
|---|---:|---|
| `Min_thérmostat` / `Max_thérmostat` | 6 / 25 | Setpoint limits |
| `MIN_DELAY_MINUTES` | 25 | Delay between adjustments, except in emergencies |
| `SENSOR_MAX_AGE_MINUTES` | 90 | Maximum age of sensor measurements |
| `WEATHER_MAX_AGE_MINUTES` | 120 | Maximum age of observed weather data |
| `FORECAST_ENABLED` | `true` | Enable forecast-based anticipation |
| `FORECAST_MAX_AGE_MINUTES` | 180 | Maximum cache age since retrieval |
| `FORECAST_HORIZON_HOURS` | 6 | Near-term decision horizon, configurable from 6 to 12 hours |
| `FORECAST_MAX_REDUCTION_C` / `FORECAST_MAX_BOOST_C` | 0.5 / 0.5 | Default maximum forecast corrections |
| `FORECAST_MILD_MIN_C` / `FORECAST_ECO_MIN_C` | 16 / 18 | Mild-weather thresholds for near-term / 24-hour decisions |
| `HUMIDITY_LOW_PERCENT` / `HUMIDITY_HIGH_PERCENT` | 30 / 60 | Humidity thresholds |
| `HUMIDITY_CRITICAL_LOW_PERCENT` / `HUMIDITY_CRITICAL_HIGH_PERCENT` | 25 / 70 | Critical thresholds |
| `HUMIDITY_CONFIRM_MINUTES` | 30 | Anomaly confirmation period |
| `HUMIDITY_REMINDER_HOURS` | 12 | Standard reminder interval |
| `WINDOW_HOLD_MINUTES` | 15 | Hold period for the open-window signal |
| `WINDOW_REMINDER_MINUTES` | 60 | Open-window reminder interval |

The code uses these default values for `FORECAST_*` settings. Add the desired fields in **Configuration Logement** to override them.

Alert frequency depends on the scheduler: a confirmation period set to 5 minutes does not trigger a check every 5 minutes when the workflow runs every 30 minutes.

### Limitations and validation

Tado infers whether a window is open; its signal does not replace a contact sensor. Solar gain and wall temperature are estimates. Occupancy falls back to the configuration when no usable data is available.

JSON, connections, JavaScript syntax and simulated decisions have been checked locally. Tado, weather, Browserless and Telegram services must be verified on the target installation.

---

## MAJ du 02/10/2026 : Anticipation du chauffage avec les prévisions météo sur cinq jours

### Français

Cette entrée décrit les évolutions du workflow **Chauffage** fourni le 02/10/2026. Le workflow complet est intégré à `Tado.json`, avec **Lire météo**, **Fusionner données** et **Décision chauffage** compatibles entre eux. L'export reste anonymisé et désactivé à l'import.

- **Prévisions météo sur cinq jours** : **Lire météo** passe à l'opération OpenWeatherMap `5DayForecast`. Le nouveau code de **Décision chauffage** prévoit d'analyser jusqu'à 120 heures, réparties en cinq périodes glissantes de 24 heures, avec une importance décroissante pour les jours les plus éloignés.
- **Anticipation à court terme** : les six prochaines heures pilotent les ajustements, avec un horizon configurable de 6 à 12 heures. Les jours suivants peuvent renforcer un signal proche, mais ne déclenchent pas seuls une modification.
- **Redoux annoncé** : réduction limitée de la consigne (`FORECAST_REDUCED`, jusqu'à 0,5 °C par défaut) si la pièce est suffisamment chaude et ne se refroidit pas trop vite. Un besoin de confort ou un préchauffage horaire empêche cette réduction.
- **Refroidissement annoncé** : hausse limitée (`FORECAST_COLD`) uniquement si la température intérieure ou sa tendance indique un besoin, en présence et hors des phases de nuit. Le cumul des corrections pour le froid actuel et prévu est plafonné à +0,5 °C.
- **Temps doux durable** : passage possible à la consigne éco (`FORECAST_MILD_ECO`) si les prévisions couvrent les prochaines 24 heures, avec un minimum de 18 °C par défaut, et si la pièce est déjà chaude avec une tendance mesurée stable ou montante. Le chauffage reste en mode ON à la consigne minimale configurée ; aucune commande OFF n'est ajoutée. Sinon, les règles habituelles restent applicables et peuvent conserver la consigne.
- **Contrôle des données** : fraîcheur maximale des prévisions de 180 minutes par défaut, filtrage des dates et températures invalides, suppression des doublons et vérification de la continuité des créneaux. Les prévisions absentes, périmées ou insuffisantes ne servent pas à anticiper le chauffage. Les réductions solaire, tendance et prévisions ne sont pas additionnées : seule la plus forte est retenue.
- **Diagnostic enrichi** : ajout de `debug_previsions` avec les températures moyennes/minimales, le résumé des cinq jours, les corrections et le motif de décision. Le message d'humidité peut aussi préciser quand la météo extérieure est estimée à partir des prévisions.

**Complément du 03/10/2026 — Fusion des prévisions intégrée :** **Fusionner données** traite désormais la liste de prévisions OpenWeatherMap et transmet `previsions_meteo` ainsi que `quality.forecast_fetched_at` à **Décision chauffage**. Il accepte aussi les réponses encapsulées dans `body` et les créneaux répartis sur plusieurs items n8n. Les dates sont normalisées en millisecondes, les créneaux triés et les doublons supprimés.

Le cache des prévisions peut être réutilisé pendant une panne météo tant qu'il reste frais, sans prolonger artificiellement sa date de récupération. En l'absence d'observation météo récente, le créneau le plus proche à moins de trois heures fournit une estimation extérieure, signalée par `quality.weather_is_forecast_estimate`. Si aucune donnée exploitable n'est disponible, l'anticipation météo est désactivée et le contrôle intérieur reste utilisable.

**Validation :** 18 simulations locales de la chaîne **Fusionner données → Décision chauffage** ont réussi : formats de réponse, dates UTC, redoux, refroidissement, maintien, mode éco, priorité au confort, météo indisponible, cache frais/périmé, créneaux incomplets, valeurs invalides, arrêt manuel, fenêtre ouverte et capteur périmé. Le JSON, les connexions et la syntaxe des nœuds JavaScript ont également été vérifiés. Ces contrôles n'ont effectué aucun appel aux services ni modifié une consigne réelle.

### English — Update 2026-10-02: Heating anticipation using five-day weather forecasts

This entry describes the changes in the **Chauffage** workflow supplied on 2026-10-02. The complete workflow is integrated into `Tado.json`, with compatible **Lire météo**, **Fusionner données** and **Décision chauffage** nodes. The export remains anonymized and inactive on import.

- **Five-day forecasts**: **Lire météo** switches to OpenWeatherMap's `5DayForecast` operation. The new **Décision chauffage** code is designed to analyse up to 120 hours in five rolling 24-hour periods, with decreasing weight for more distant days.
- **Short-term anticipation**: the next six hours drive adjustments, with a configurable horizon of 6 to 12 hours. Later days can strengthen a near-term signal but cannot trigger a change on their own.
- **Expected warming**: a limited setpoint reduction (`FORECAST_REDUCED`, up to 0.5 °C by default) is possible when the room is warm enough and is not cooling too quickly. A comfort deficit or scheduled preheating prevents this reduction.
- **Expected cooling**: a limited increase (`FORECAST_COLD`) is allowed only when indoor temperature or its trend indicates a need, with occupancy and outside night phases. The combined correction for current and forecast cold weather is capped at +0.5 °C.
- **Sustained mild weather**: the eco setpoint (`FORECAST_MILD_ECO`) can be selected when forecasts cover the next 24 hours with a minimum of 18 °C by default, and the room is already warm with a measured stable or rising trend. Heating stays in ON mode at the configured minimum setpoint; no OFF command is added. Otherwise, the usual rules apply and may keep the existing setpoint.
- **Data checks**: forecasts must be no older than 180 minutes by default. Invalid timestamps and temperatures are filtered, duplicates removed and time-slot coverage checked. Missing, stale or insufficient forecasts do not drive heating anticipation. Solar, trend and forecast reductions are not added together: only the largest is used.
- **Additional diagnostics**: `debug_previsions` reports mean/minimum temperatures, a five-day summary, corrections and the decision reason. Humidity messages can also indicate when outdoor weather is estimated from forecasts.

**Follow-up on 2026-10-03 — Forecast merging integrated:** **Fusionner données** now processes the OpenWeatherMap forecast list and passes `previsions_meteo` and `quality.forecast_fetched_at` to **Décision chauffage**. It also accepts responses wrapped in `body` and time slots returned as separate n8n items. Timestamps are normalized to milliseconds, time slots sorted and duplicates removed.

The forecast cache can be reused during a weather outage while it remains fresh, without artificially extending its retrieval time. When no recent weather observation is available, the nearest slot within three hours provides an outdoor estimate, flagged by `quality.weather_is_forecast_estimate`. If no usable data is available, forecast-based anticipation is disabled while indoor control remains available.

**Validation:** 18 local simulations of **Fusionner données → Décision chauffage** passed, covering response formats, UTC dates, warming, cooling, unchanged setpoints, eco mode, comfort priority, unavailable weather, fresh/expired caches, incomplete time slots, invalid values, manual OFF, open windows and stale sensors. JSON, connections and JavaScript node syntax were also checked. These checks made no service calls and did not change a real heating setpoint.
