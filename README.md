# Tado-Automatic

[Français](#français) · [English](#english)

## Français

Workflow n8n pour adapter une zone de chauffage Tado à la température intérieure, à la météo, aux horaires et à la saison, avec alertes Telegram d'humidité et de fenêtre ouverte.

### Fichier

[Tado.json](Tado.json) contient 32 éléments : 30 nœuds de workflow et 2 notes explicatives. L'export est désactivé et anonymisé : aucune donnée épinglée, aucun identifiant de credential, token de session ou état d'exécution n'est inclus.

### Changements

- Diagnostic manuel du compte et des zones Tado, avec vérification du couple `HOME_ID` / `ZONE_ID` et du type `HEATING`.
- Erreurs HTTP et absence de notification expliquées dans la sortie : `tado_error_code`, `tado_error_message` et `humidity_alert_blocked_reason`.
- Session d'authentification commune aux requêtes, y compris après une réautorisation par device code.
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
7. Exécuter **Diagnostic Tado (manuel)**, puis ouvrir **Résultat diagnostic Tado**. Si `HOME_ID` est inconnu, commencer par relever le bon `id` dans `logements_accessibles`, le renseigner et relancer le diagnostic. Choisir ensuite l'`id` de la pièce de type `HEATING` dans `zones_disponibles` et le reporter dans `ZONE_ID`. Ne pas utiliser le rang de la pièce dans la liste.
8. Relancer le diagnostic jusqu'à obtenir `ZONE_CONFIGUREE_TROUVEE`. Ce résultat confirme l'existence et le type de la pièce, pas encore la disponibilité de ses mesures. La branche de diagnostic n'applique aucune consigne et n'envoie aucun message Telegram. Conserver l'expression du champ `DIAGNOSTIC_TADO` : le déclencheur manuel l'active, le planificateur utilise la branche de chauffage.
9. Publier/activer le workflow une fois les réglages vérifiés. Le planificateur fonctionne toutes les 30 minutes et peut appliquer une consigne réelle. Consulter sa première exécution : `tado_api_ok` doit être `true`, `quality.humidity_valid` doit être `true` et les mesures doivent être présentes. Lors d'une migration, désactiver l'ancien workflow pour ne garder qu'un contrôleur de la zone.

Les données statiques conservent les tokens et l'historique dans votre instance lors des exécutions réussies du workflow publié. Les tests manuels ne sauvegardent pas cette mémoire : ils ne permettent pas de confirmer à eux seuls une humidité persistante entre plusieurs exécutions. Après un nouvel import, le parcours device code initialise l'authentification ; une exécution manuelle ne garantit pas que les nouveaux tokens seront conservés pour la prochaine exécution planifiée.

Avec des prévisions fraîches et suffisamment complètes, `debug_previsions.utilisables` doit être `true` ; `quality.weather_is_forecast_estimate` indique si la température extérieure est estimée.

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

[Tado.json](Tado.json) contains 32 elements: 30 workflow nodes and 2 explanatory sticky notes. The export is inactive and anonymized: it includes no pinned data, credential identifiers, session tokens or execution state.

### Changes

- Manual Tado account and zone diagnostics, checking `HOME_ID`, `ZONE_ID` and the `HEATING` zone type.
- Explicit HTTP errors and notification-blocking reasons: `tado_error_code`, `tado_error_message` and `humidity_alert_blocked_reason`.
- A shared authenticated session for API requests, including after device-code reauthorization.
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
7. Run **Diagnostic Tado (manuel)** and open **Résultat diagnostic Tado**. If `HOME_ID` is unknown, first find the correct `id` in `logements_accessibles`, configure it and run the diagnostic again. Then select the `id` of the intended `HEATING` room in `zones_disponibles` and enter it as `ZONE_ID`. Do not use the room's position in the list.
8. Run the diagnostic again until it returns `ZONE_CONFIGUREE_TROUVEE`. This confirms that the room exists and has the correct type, not yet that its measurements are available. The diagnostic branch applies no heating setpoint and sends no Telegram message. Keep the `DIAGNOSTIC_TADO` expression: the manual trigger enables diagnostics, while the scheduler uses the heating branch.
9. Publish/activate the workflow once the settings are verified. The scheduler runs every 30 minutes and can apply a real setpoint. Inspect its first execution: `tado_api_ok` and `quality.humidity_valid` should be `true`, with actual measurements present. When migrating, deactivate the old workflow so only one controller manages the zone.

Static data stores tokens and history in your instance after successful executions of the published workflow. Manual tests do not persist this memory: they cannot by themselves confirm sustained humidity across executions. After a fresh import, the device code flow initializes authentication; a manual execution does not guarantee that newly obtained tokens will be retained for the next scheduled execution.

With fresh forecasts and sufficient time-slot coverage, `debug_previsions.utilisables` should be `true`; `quality.weather_is_forecast_estimate` indicates whether outdoor temperature is estimated.

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

---

## MAJ du 07/10/2026 : Diagnostic des zones Tado et suivi des alertes d'humidité

### Français

Cette mise à jour intègre les corrections vérifiées les 05 et 07 octobre. Une zone devenue introuvable renvoyait une erreur HTTP `404` : la température et l'humidité intérieures restaient absentes, ce qui empêchait les alertes. Renouveler le token ne pouvait pas corriger le couple `HOME_ID` / `ZONE_ID`. Le diagnostic permet désormais de retrouver les identifiants accessibles et de choisir explicitement la pièce à piloter.

**Diagnostic manuel.** Les nœuds **Diagnostic Tado (manuel)**, **Activer diagnostic** et **Mode diagnostic ?** séparent la vérification initiale des contrôles planifiés. **Lire compte Tado** appelle `GET /api/v2/me`, puis **Lister zones Tado** appelle `GET /api/v2/homes/{HOME_ID}/zones`. **Résultat diagnostic Tado** présente les logements accessibles, les zones avec leur nom et leur type, les codes HTTP, le quota restant lorsqu'il est fourni et les instructions de correction. Il signale aussi un type de zone incompatible avec `HEATING`. Ces deux appels de diagnostic ne sont pas ajoutés à chaque contrôle planifié et aucune autre pièce n'est sélectionnée automatiquement.

**Authentification.** Le nouveau nœud **Session Tado** fournit le token aux lectures du compte, des zones et de l'état ainsi qu'à l'application de la consigne. Après une réautorisation réussie, **Code in JavaScript** rejoint **Vérification Token** puis **Session Tado**, sans rafraîchir immédiatement le token qui vient d'être obtenu. Toutes ces requêtes utilisent ainsi la session validée pour cette exécution.

**Erreurs explicites.** Les requêtes d'état et d'overlay conservent le corps, les en-têtes et le code HTTP, même en cas d'erreur HTTP. Les exceptions réseau continuent d'être transmises comme erreurs. **Fusionner données** renseigne `tado_http_status`, `tado_error_code`, `tado_error_message` et `tado_error_detail` ; **Décision chauffage** et le journal rendent la cause visible. Un échec de lecture laisse les mesures inconnues et n'envoie aucune nouvelle consigne.

| Situation lors de la lecture d'état | Diagnostic |
|---|---|
| HTTP `404` | `TADO_ZONE_INTROUVABLE` : vérifier le logement et la zone |
| HTTP `401` | `TADO_AUTHENTIFICATION_REFUSEE` |
| HTTP `403` | `TADO_ACCES_REFUSE` |
| HTTP `429` | `TADO_QUOTA_ATTEINT` : consulter `quality.tado_rate_limit` |
| Autre erreur HTTP ou réseau | `TADO_REQUETE_EN_ECHEC` |
| Appareil hors ligne | `TADO_APPAREIL_HORS_LIGNE` |
| Réponse sans données de capteur | `TADO_MESURES_ABSENTES` |

**Pourquoi une alerte n'est pas encore envoyée.** Le nouveau champ `humidity_alert_blocked_reason` précise si la mesure manque, si l'humidité est dans les seuils, si la confirmation est en attente (`CONFIRMATION_EN_ATTENTE`) ou si le délai entre alertes s'applique (`DELAI_ENTRE_ALERTES`). Le texte `alertMessage` peut être préparé alors que `sendHumAlert` vaut encore `false` : seul ce booléen autorise le passage vers Telegram.

Les seuils existants sont conservés : ≤30 % ou ≥60 %, avec au moins deux mesures distinctes couvrant 30 minutes pour un avertissement. Les seuils critiques sont ≤25 % ou ≥70 %, avec deux mesures couvrant 5 minutes. Les valeurs extrêmes ≤15 % ou ≥85 % peuvent contourner cette confirmation, mais respectent toujours le délai entre alertes. Un changement de catégorie ou de sévérité recommence la confirmation ; l'hystérésis évite les oscillations près des seuils. Les rappels restent à 12 heures pour un avertissement et 6 heures pour une alerte critique persistante, avec les règles existantes de nouvel épisode et d'aggravation. Un contrôle toutes les 30 minutes ne produit pas d'alerte intermédiaire au bout de 5 minutes.

La confirmation utilise les horodatages des mesures du capteur : répéter la même mesure n'augmente pas le compteur. Le workflow doit être publié et exécuté automatiquement pour conserver son historique. L'horodatage d'une alerte reste mémorisé uniquement après confirmation de son envoi par Telegram.

**Export public.** Les identifiants de compte, mot de passe, logement, zone, chat Telegram et token Browserless sont remplacés par des champs `YOUR_*`. Les coordonnées sont neutres, la ville météo est à renseigner, les références de credentials sont retirées et aucun token, état d'exécution ou donnée épinglée n'est publié. Les prévisions sur cinq jours, les règles de confort et les alertes de fenêtre ouverte restent en place.

**Validation.** Les 32 vérifications et simulations locales ont réussi : structure, syntaxe, diagnostics de zone, erreurs HTTP, mesures absentes ou périmées, confirmation d'humidité et rappels. Elles ne constituent pas un test réel de l'envoi Telegram ou d'une commande de chauffage. Le diagnostic de zone et la reprise des mesures ont été confirmés sur l'installation cible ; l'absence d'alerte sur la première mesure anormale correspondait à l'attente de confirmation.

### English — Update 2026-10-07: Tado zone diagnostics and humidity alert status

This update integrates the fixes checked on October 5 and 7. An unavailable zone returned HTTP `404`, leaving indoor temperature and humidity unavailable and preventing alerts. Refreshing a token could not repair an invalid `HOME_ID` / `ZONE_ID` pair. The diagnostic now lists accessible identifiers so the intended room can be selected explicitly.

**Manual diagnostics.** **Diagnostic Tado (manuel)**, **Activer diagnostic** and **Mode diagnostic ?** separate initial verification from scheduled control. **Lire compte Tado** calls `GET /api/v2/me`, then **Lister zones Tado** calls `GET /api/v2/homes/{HOME_ID}/zones`. **Résultat diagnostic Tado** reports accessible homes, zone names and types, HTTP status codes, remaining quota when available and corrective instructions. It also flags zone types incompatible with `HEATING`. These two diagnostic calls are not added to each scheduled check, and the workflow never automatically switches to another room.

**Authentication.** The new **Session Tado** node supplies the token for account, zone and state reads and for applying a setpoint. After successful reauthorization, **Code in JavaScript** connects to **Vérification Token** and then **Session Tado**, without immediately refreshing the newly issued token. All these requests use the session validated for the current execution.

**Explicit errors.** State and overlay requests retain their response body, headers and HTTP status even on HTTP errors. Network exceptions are still forwarded as errors. **Fusionner données** exposes `tado_http_status`, `tado_error_code`, `tado_error_message` and `tado_error_detail`; **Décision chauffage** and the log make the cause visible. Failed state reads leave measurements unknown and do not send a new heating setpoint.

| State-read condition | Diagnostic |
|---|---|
| HTTP `404` | `TADO_ZONE_INTROUVABLE`: check home and zone identifiers |
| HTTP `401` | `TADO_AUTHENTIFICATION_REFUSEE` |
| HTTP `403` | `TADO_ACCES_REFUSE` |
| HTTP `429` | `TADO_QUOTA_ATTEINT`: inspect `quality.tado_rate_limit` |
| Other HTTP or network errors | `TADO_REQUETE_EN_ECHEC` |
| Offline device | `TADO_APPAREIL_HORS_LIGNE` |
| Response without sensor data | `TADO_MESURES_ABSENTES` |

**Why an alert has not been sent yet.** `humidity_alert_blocked_reason` identifies missing measurements, humidity within thresholds, pending confirmation (`CONFIRMATION_EN_ATTENTE`) or an active cooldown (`DELAI_ENTRE_ALERTES`). `alertMessage` may already contain prepared text while `sendHumAlert` is still `false`; only this boolean allows the Telegram branch to run.

Existing thresholds are retained: ≤30% or ≥60%, with at least two distinct measurements spanning 30 minutes for a warning. Critical thresholds are ≤25% or ≥70%, with two measurements spanning 5 minutes. Extreme values ≤15% or ≥85% can bypass confirmation, but still respect alert cooldowns. Changing category or severity restarts confirmation; hysteresis avoids oscillation near thresholds. Reminders remain 12 hours for warnings and 6 hours for persistent critical alerts, with the existing new-episode and escalation rules. A 30-minute polling schedule cannot deliver an intermediate check after just 5 minutes.

Confirmation uses sensor measurement timestamps: reading the same sample repeatedly does not increment the counter. Publish the workflow and let it run automatically to retain its history. Alert timestamps are still recorded only after Telegram confirms delivery.

**Public export.** Account, password, home, zone, Telegram chat and Browserless token values are replaced with `YOUR_*` placeholders. Coordinates are neutral, the weather city must be configured, credential references are removed and no token, execution state or pinned data is published. Five-day forecasts, comfort rules and open-window alerts remain available.

**Validation.** All 32 local checks and simulations passed, covering structure, syntax, zone diagnostics, HTTP errors, missing or stale measurements, humidity confirmation and reminders. They are not live tests of Telegram delivery or heating commands. Zone diagnostics and restored sensor readings were confirmed on the target installation; no alert on the first abnormal sample was the expected confirmation delay.

### Références / References

- [Tado — REST API](https://help.tado.com/en/collections/15124268-rest-api)
- [Tado — Device code authentication and refresh tokens](https://help.tado.com/en/articles/8565472-how-do-i-authenticate-to-access-the-rest-api)
- [n8n — getWorkflowStaticData](https://docs.n8n.io/build/code-in-n8n/cookbook/built-in-methods-and-variables-examples/getworkflowstaticdata)
