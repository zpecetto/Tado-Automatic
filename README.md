# Tado-Automatic

[Français](#français) · [English](#english)

## Français

Workflow n8n pour adapter une zone de chauffage Tado à la température intérieure, à la météo, aux horaires et à la saison, avec alertes Telegram d'humidité et de fenêtre ouverte.

### Fichier

[Tado.json](Tado.json) contient les 23 nœuds du workflow. L'export est désactivé et anonymisé : aucune donnée épinglée, aucun identifiant de credential, token de session ou état d'exécution n'est inclus.

### Changements

- Décision de chauffage enrichie : présence lorsqu'elle est disponible, tendance sur des mesures distinctes, anticipation horaire, estimation solaire et inertie thermique.
- Contrôle de la fraîcheur des mesures Tado et de la météo.
- Consignes de 6 à 25 °C : pas de 1 °C entre 6 et 14 °C, puis de 0,5 °C à partir de 15 °C. Le code demande une consigne en mode ON et respecte l'arrêt manuel selon `RESPECT_MANUAL_OFF`.
- Humidité : seuils normal/critique, hystérésis, confirmation sur plusieurs mesures, estimation du point de rosée et comparaison avec l'air extérieur.
- Fenêtre ouverte : signal natif Tado, passage à la consigne éco, temporisation et alerte Telegram avec rappel configurable.
- Les délais entre alertes ne sont mémorisés qu'après confirmation de l'envoi Telegram.

### Configuration

1. Importer `Tado.json` et laisser le workflow désactivé pendant la configuration.
2. Dans **Configuration Tado**, remplacer `YOUR_TADO_EMAIL`, `YOUR_TADO_PASSWORD`, `YOUR_CLIENT_ID`, `YOUR_HOME_ID`, `YOUR_ZONE_ID` et `YOUR_ID_TELEGRAM`.
3. Dans **Configuration Logement**, renseigner `Lat` et `Lon` (valeurs neutres `0`), l'orientation des fenêtres, le fuseau horaire et les paramètres de confort.
4. Dans **Lire météo**, remplacer `YOUR_CITY,COUNTRY_CODE` et sélectionner les credentials OpenWeatherMap.
5. Sélectionner les credentials du bot dans les deux nœuds Telegram.
6. Dans **HTTP Request**, configurer Browserless et remplacer `YOUR_BROWSERLESS_TOKEN`. Ce nœud sert à la récupération de l'authentification Tado par device code.
7. Vérifier l'authentification, l'état de la zone et la sortie de **Décision chauffage** avant d'exécuter **Appliquer overlay Tado**, qui peut modifier la consigne réelle.
8. Activer le workflow une fois les réglages vérifiés. Le planificateur fonctionne toutes les 30 minutes.

Les données statiques du workflow conservent les tokens et l'historique de décision dans votre instance. Après un nouvel import, le parcours device code initialise l'authentification.

### Réglages principaux

| Réglage | Valeur | Usage |
|---|---:|---|
| `Min_thérmostat` / `Max_thérmostat` | 6 / 25 | Bornes de consigne |
| `MIN_DELAY_MINUTES` | 25 | Délai entre ajustements, sauf urgence |
| `SENSOR_MAX_AGE_MINUTES` | 90 | Fraîcheur maximale des mesures |
| `WEATHER_MAX_AGE_MINUTES` | 120 | Fraîcheur maximale de la météo |
| `HUMIDITY_LOW_PERCENT` / `HUMIDITY_HIGH_PERCENT` | 30 / 60 | Seuils d'humidité |
| `HUMIDITY_CRITICAL_LOW_PERCENT` / `HUMIDITY_CRITICAL_HIGH_PERCENT` | 25 / 70 | Seuils critiques |
| `HUMIDITY_CONFIRM_MINUTES` | 30 | Confirmation de l'anomalie |
| `HUMIDITY_REMINDER_HOURS` | 12 | Rappel standard |
| `WINDOW_HOLD_MINUTES` | 15 | Temporisation du signal d'ouverture |
| `WINDOW_REMINDER_MINUTES` | 60 | Rappel fenêtre ouverte |

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
- Freshness checks for Tado measurements and weather data.
- Setpoints from 6 to 25 °C: 1 °C increments between 6 and 14 °C, then 0.5 °C increments from 15 °C. The code requests a setpoint in ON mode and respects manual OFF according to `RESPECT_MANUAL_OFF`.
- Humidity: normal and critical thresholds, hysteresis, confirmation over multiple measurements, estimated dew point and comparison with outdoor air.
- Open windows: native Tado signal, switch to the eco setpoint, a hold period and a Telegram alert with a configurable reminder.
- Alert cooldowns are recorded only after Telegram confirms delivery.

### Configuration

1. Import `Tado.json` and leave the workflow inactive during configuration.
2. In **Configuration Tado**, replace `YOUR_TADO_EMAIL`, `YOUR_TADO_PASSWORD`, `YOUR_CLIENT_ID`, `YOUR_HOME_ID`, `YOUR_ZONE_ID` and `YOUR_ID_TELEGRAM`.
3. In **Configuration Logement**, enter `Lat` and `Lon` (neutral default values: `0`), window orientation, time zone and comfort settings.
4. In **Lire météo**, replace `YOUR_CITY,COUNTRY_CODE` and select your OpenWeatherMap credentials.
5. Select your bot credentials in both Telegram nodes.
6. In **HTTP Request**, configure Browserless and replace `YOUR_BROWSERLESS_TOKEN`. This node handles Tado authentication through the device code flow.
7. Check authentication, the zone's state and the output of **Décision chauffage** before running **Appliquer overlay Tado**, which can change the actual setpoint.
8. Activate the workflow once you have checked the settings. The scheduler runs every 30 minutes.

The workflow's static data stores tokens and the decision history in your instance. After a fresh import, the device code flow initializes authentication.

### Main settings

| Setting | Value | Purpose |
|---|---:|---|
| `Min_thérmostat` / `Max_thérmostat` | 6 / 25 | Setpoint limits |
| `MIN_DELAY_MINUTES` | 25 | Delay between adjustments, except in emergencies |
| `SENSOR_MAX_AGE_MINUTES` | 90 | Maximum age of sensor measurements |
| `WEATHER_MAX_AGE_MINUTES` | 120 | Maximum age of weather data |
| `HUMIDITY_LOW_PERCENT` / `HUMIDITY_HIGH_PERCENT` | 30 / 60 | Humidity thresholds |
| `HUMIDITY_CRITICAL_LOW_PERCENT` / `HUMIDITY_CRITICAL_HIGH_PERCENT` | 25 / 70 | Critical thresholds |
| `HUMIDITY_CONFIRM_MINUTES` | 30 | Anomaly confirmation period |
| `HUMIDITY_REMINDER_HOURS` | 12 | Standard reminder interval |
| `WINDOW_HOLD_MINUTES` | 15 | Hold period for the open-window signal |
| `WINDOW_REMINDER_MINUTES` | 60 | Open-window reminder interval |

Alert frequency depends on the scheduler: a confirmation period set to 5 minutes does not trigger a check every 5 minutes when the workflow runs every 30 minutes.

### Limitations and validation

Tado infers whether a window is open; its signal does not replace a contact sensor. Solar gain and wall temperature are estimates. Occupancy falls back to the configuration when no usable data is available.

JSON, connections, JavaScript syntax and simulated decisions have been checked locally. Tado, weather, Browserless and Telegram services must be verified on the target installation.
