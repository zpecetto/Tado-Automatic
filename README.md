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

Import [Tado.json](Tado.json) to run the 23-node n8n workflow. It adjusts one Tado zone using indoor measurements, weather, schedule, season, thermal trends and estimated solar gain. Telegram branches report persistent humidity issues and native Tado open-window events.

This update adds freshness checks, stepped setpoints from 6 to 25 °C, humidity confirmation and hysteresis, window eco mode, and alert state recorded only after Telegram acknowledges delivery. The workflow does not issue an OFF command and can respect manual OFF.

Before activation, replace all `YOUR_...` values, configure housing coordinates and comfort settings, select OpenWeatherMap and Telegram credentials, and verify Browserless device-code authentication. Check the decision output before enabling real Tado writes.

The schedule runs every 30 minutes. Shorter confirmation thresholds do not increase polling frequency. A Tado window signal is an inference; solar gain and wall temperature are estimates.

Exports are inactive and contain no credentials, pinned data or runtime state. Local validation covers JSON, connections, JavaScript syntax and simulated decisions. Live services must be checked on the target instance.
