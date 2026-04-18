[FR](#fr) | [EN](#en)

# n8n - Automatisation intelligente du chauffage Tado
# n8n - Smart Tado Heating Automation

## FR

## Présentation

Ce workflow n8n permet de piloter automatiquement un thermostat connecté Tado en fonction de plusieurs paramètres :

- température intérieure
- humidité intérieure
- météo extérieure
- ensoleillement
- heure de la journée
- saison
- inertie thermique du logement
- logique anti-spam pour éviter les mises à jour inutiles

L’objectif est d’améliorer le confort thermique tout en limitant les changements de consigne inutiles.

---

## Fonctionnalités

- Exécution automatique toutes les 30 minutes
- Lecture de l’état actuel de la zone Tado
- Lecture de la météo extérieure via OpenWeatherMap
- Calcul d’une consigne dynamique de chauffage
- Application d’un overlay Tado uniquement si nécessaire
- Réduction des mises à jour trop fréquentes
- Respect des cas de coupure manuelle du chauffage
- Prise en compte de l’exposition solaire selon l’orientation des fenêtres
- Envoi d’alertes Telegram si l’humidité intérieure est trop basse ou trop élevée
- Gestion du rafraîchissement du token Tado
- Mécanisme de récupération d’authentification si le token n’est plus valide

---

## Logique du workflow

Le workflow suit globalement ces étapes :

1. Déclenchement via un planificateur
2. Chargement de la configuration Tado et logement
3. Rafraîchissement du token Tado
4. Lecture de l’état actuel de la zone Tado
5. Lecture de la météo extérieure
6. Fusion des données intérieures et extérieures
7. Calcul de la décision de chauffage
8. Mise à jour de l’overlay Tado si nécessaire
9. Mémorisation des données utiles pour éviter le spam
10. Envoi éventuel d’une alerte Telegram liée à l’humidité

Si l’authentification échoue, le workflow peut lancer une procédure de récupération via device code.

---

## Modèle de décision

La consigne n’est pas fixe.

Elle s’adapte selon :
- la température intérieure actuelle
- la température extérieure
- la phase de la journée
- semaine ou week-end
- la saison
- l’apport solaire estimé
- l’inertie thermique
- l’évolution récente de la température intérieure
- l’état manuel OFF
- le délai minimum entre deux mises à jour

Cela permet par exemple de :
- réduire le chauffage en cas de fort ensoleillement
- anticiper la baisse thermique avant le coucher du soleil
- maintenir un mode éco si le logement est déjà assez chaud
- éviter les micro-ajustements inutiles

---

## Technologies utilisées

- n8n
- API Tado
- OpenWeatherMap
- Telegram
- Browserless
- JavaScript dans les nœuds Code

---

## Prérequis

Avant d’utiliser ce workflow, vous devez disposer de :

- une instance n8n fonctionnelle
- un compte Tado
- un accès API Tado fonctionnel
- des credentials OpenWeatherMap configurés dans n8n
- des credentials Telegram configurés dans n8n
- une instance Browserless si vous souhaitez utiliser la récupération automatique d’authentification

---

## Configuration

Le workflow repose principalement sur deux blocs de configuration.

### Configuration Tado

À renseigner :
- Client ID Tado
- Home ID
- Zone ID
- Identifiant Telegram / Chat ID

### Configuration logement

Paramètres ajustables :
- latitude / longitude
- azimut des fenêtres
- tolérance d’azimut
- température minimale / maximale du thermostat
- inertie thermique
- seuils anti-spam
- seuil de surchauffe
- limite été
- horaires matin / soir / nuit
- bonus solaire
- poids de tendance thermique
- modes éco / off

---

## Sécurité

Ne publiez pas l’export brut du workflow sur GitHub sans nettoyage préalable.

Avant publication, remplacez toutes les données sensibles par des placeholders :

- adresse email
- mot de passe
- client ID
- home ID
- zone ID
- chat ID Telegram
- tokens
- URLs internes éventuelles

Exemples :
- YOUR_TADO_CLIENT_ID
- YOUR_HOME_ID
- YOUR_ZONE_ID
- YOUR_TELEGRAM_CHAT_ID
- YOUR_TADO_EMAIL
- YOUR_TADO_PASSWORD

---

## Structure conseillée du dépôt

.
├── README.md
├── workflow/
│   └── chauffage.tado.sanitized.json
└── docs/
    └── overview.md

---

## Import dans n8n

1. Ouvrir n8n
2. Importer le fichier JSON nettoyé
3. Reconfigurer les credentials
4. Mettre à jour les nœuds de configuration
5. Tester le workflow manuellement
6. Activer le workflow une fois validé

---

## Cas d’usage

Ce workflow est utile si vous souhaitez :
- automatiser intelligemment votre chauffage
- tenir compte du soleil et de la météo extérieure
- améliorer le confort sans intervenir manuellement
- limiter les changements trop fréquents de consigne
- recevoir des alertes simples sur l’humidité intérieure

---

## Limites

- Dépend de la disponibilité de l’API Tado
- L’apport solaire est estimé, pas mesuré par un capteur réel
- La présence est actuellement basée sur une valeur par défaut si elle n’est pas enrichie par une autre source
- Les réglages doivent être adaptés au logement pour de bons résultats

---

## Améliorations possibles

- Détection réelle de présence
- Gestion avancée des horaires d’occupation
- Mode vacances
- Intégration de prévisions météo plus poussées
- Tableau de bord / historique
- Estimation de consommation énergétique
- Support multi-zones ou multi-pièces

---

## EN

## Overview

This n8n workflow automatically controls a Tado smart thermostat based on several factors:

- indoor temperature
- indoor humidity
- outdoor weather
- solar exposure
- time of day
- season
- home thermal inertia
- anti-spam logic to avoid unnecessary updates

The goal is to improve thermal comfort while reducing unnecessary thermostat changes.

---

## Features

- Automatic execution every 30 minutes
- Reads the current Tado zone state
- Reads outdoor weather data from OpenWeatherMap
- Computes a dynamic heating target
- Applies a Tado overlay only when needed
- Reduces excessive and unnecessary updates
- Respects manual heating shutdown scenarios
- Takes solar exposure into account based on window orientation
- Sends Telegram alerts when indoor humidity is too low or too high
- Handles Tado token refresh
- Includes an authentication recovery flow if the token is no longer valid

---

## Workflow logic

The workflow generally follows these steps:

1. Triggered by a schedule
2. Load Tado and home configuration
3. Refresh Tado token
4. Read the current Tado zone state
5. Read outdoor weather
6. Merge indoor and outdoor data
7. Compute the heating decision
8. Update the Tado overlay if needed
9. Store useful data to prevent spam
10. Optionally send a Telegram humidity alert

If authentication fails, the workflow can start a recovery process using a device code flow.

---

## Decision model

The target temperature is not fixed.

It adapts according to:
- current indoor temperature
- outdoor temperature
- time-of-day phase
- weekday or weekend
- season
- estimated solar gain
- thermal inertia
- recent indoor temperature trend
- manual OFF state
- minimum delay between two updates

This makes it possible to:
- reduce heating when solar gain is strong
- anticipate cooling before sunset
- keep an eco mode when the home is already warm enough
- avoid unnecessary micro-adjustments

---

## Technologies used

- n8n
- Tado API
- OpenWeatherMap
- Telegram
- Browserless
- JavaScript code nodes

---

## Requirements

Before using this workflow, you need:

- a running n8n instance
- a Tado account
- working Tado API access
- OpenWeatherMap credentials configured in n8n
- Telegram credentials configured in n8n
- a Browserless instance if you want to use the automated authentication recovery flow

---

## Configuration

The workflow mainly relies on two configuration blocks.

### Tado configuration

You need to provide:
- Tado Client ID
- Home ID
- Zone ID
- Telegram Chat ID

### Home configuration

Adjustable parameters:
- latitude / longitude
- window azimuth
- azimuth tolerance
- thermostat minimum / maximum values
- thermal inertia
- anti-spam thresholds
- overheating threshold
- summer limit
- morning / evening / night hours
- solar bonus
- temperature trend weight
- eco / off modes

---

## Security

Do not publish the raw workflow export to GitHub without sanitizing it first.

Before publishing, replace all sensitive values with placeholders:

- email address
- password
- client ID
- home ID
- zone ID
- Telegram chat ID
- tokens
- internal URLs if any

Examples:
- YOUR_TADO_CLIENT_ID
- YOUR_HOME_ID
- YOUR_ZONE_ID
- YOUR_TELEGRAM_CHAT_ID
- YOUR_TADO_EMAIL
- YOUR_TADO_PASSWORD

---

## Recommended repository structure

.
├── README.md
├── workflow/
│   └── chauffage.tado.sanitized.json
└── docs/
    └── overview.md

---

## Import into n8n

1. Open n8n
2. Import the sanitized JSON file
3. Reconfigure credentials
4. Update the configuration nodes
5. Test the workflow manually
6. Enable the workflow once validated

---

## Use cases

This workflow is useful if you want to:
- automate your heating intelligently
- account for sunlight and outdoor weather
- improve comfort without manual intervention
- reduce overly frequent target changes
- receive simple indoor humidity alerts

---

## Limitations

- Depends on Tado API availability
- Solar gain is estimated, not measured by a real sensor
- Presence is currently based on a default value unless enriched by another data source
- Settings must be adapted to the home for best results

---

## Possible improvements

- Real presence detection
- Advanced occupancy schedule management
- Holiday mode
- More advanced weather forecast integration
- Dashboard / historical tracking
- Energy consumption estimation
- Multi-zone or multi-room support# Tado-Automatic
