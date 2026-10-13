# 4. Dashboard Identité et Authentification

## Objectif
Surveiller les attaques sur comptes et accès.

## Prérequis
- Index pattern : `wazuh-alerts-*`
- Période par défaut : **Last 24 hours** (heatmap : 7 à 30 jours)
- Champs :
  - Windows : `data.win.system.eventID` (4624 succès, 4625 échec), `data.win.eventdata.targetUserName`, `data.win.eventdata.ipAddress`, `data.win.eventdata.logonType`
  - Linux : `data.srcuser`, `data.dstuser`, `data.srcip`, `rule.groups` (`authentication_failed`, `authentication_success`, `sshd`)
  - Commun : `rule.groups`, `agent.name`, `@timestamp`

## Visualisations

| # | Visualisation | Type | Configuration |
|---|---------------|------|---------------|
| 1 | Échecs login 24h | Metric | Count ; filtre échecs |
| 2 | Succès login 24h | Metric | Count ; filtre succès |
| 3 | Failed logon par heure | Line / Area | Date Histogram 1h ; filtre échecs |
| 4 | Top comptes en échec | Horizontal bar | Terms `data.win.eventdata.targetUserName` et `data.dstuser` (deux visualisations ou champ unifié) |
| 5 | Top IP sources d'échecs | Horizontal bar | Terms `data.srcip` / `data.win.eventdata.ipAddress` |
| 6 | Détections brute force | Data table | Filtre brute force ; colonnes `@timestamp`, `agent.name`, `rule.id`, `rule.description`, IP, compte |
| 7 | Heatmap logins heure x jour | Heatmap | X : Date Histogram horaire (ou script heure du jour) ; Y : jour de la semaine ; valeur Count |
| 8 | Succès après plusieurs échecs | Data table / Saved search | Voir section ci-dessous |

## Requêtes (DQL)
- Échecs : `rule.groups : "authentication_failed" or data.win.system.eventID : 4625`
- Succès : `rule.groups : "authentication_success" or data.win.system.eventID : 4624`
- Brute force : `rule.groups : "authentication_failures" or rule.id : (5712 or 5551 or 60204 or 5720)`
  (vérifier les IDs selon la version Wazuh ; 60204 = multiples échecs Windows)

## Pattern suspect : succès après plusieurs échecs
- Même compte/IP : table avec Terms `data.srcip` puis Terms compte, métriques Count d'échecs et dernier succès.
- Alternative : règles de corrélation `if_matched_sid` (ex. 5503 + succès SSH) personnalisées.
- Détection password spraying : une IP avec beaucoup de **comptes distincts** (Unique Count du compte par IP).

## Utilité / Lecture
- **Brute force** : un compte, beaucoup d'échecs ; **spraying** : une IP, beaucoup de comptes.
- **Heatmap** : logins hors heures ouvrées ou week-end.
- **Compromission initiale** : succès précédé d'une rafale d'échecs.

## Actions recommandées
- Verrouiller le compte, forcer reset de mot de passe, révoquer les sessions.
- Bloquer l'IP source (active-response).
- Activer MFA, limiter l'exposition SSH/RDP.
