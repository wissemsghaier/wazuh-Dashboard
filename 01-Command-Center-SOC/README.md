# 1. Dashboard Command Center SOC

## Objectif
Obtenir une vue globale de l'état sécurité en 30 secondes.

## Prérequis
- Index pattern : `wazuh-alerts-*`
- Période par défaut : **Last 24 hours**, rafraîchissement automatique 1 min
- Champs utilisés : `@timestamp`, `rule.level`, `rule.id`, `rule.description`, `agent.name`, `agent.id`, `data.srcip`

## Visualisations

| # | Visualisation | Type | Configuration |
|---|---------------|------|---------------|
| 1 | Total alertes 24h | Metric | Count, sans filtre |
| 2 | Alertes critiques 24h | Metric | Count, filtre `rule.level >= 12` (seuil rouge) |
| 3 | Alertes par heure | Line / Area | X : Date Histogram `@timestamp` intervalle 1h ; Y : Count |
| 4 | Alertes par sévérité | Pie / Donut | Split slices : Terms `rule.level`, taille 15 |
| 5 | Top 10 agents bruyants | Horizontal bar | Terms `agent.name`, taille 10, tri par Count desc |
| 6 | Top 10 règles | Horizontal bar | Terms `rule.id`, taille 10 (ajouter `rule.description` en sous-terme ou tooltip) |
| 7 | 20 dernières alertes critiques | Data table / Saved search | Filtre `rule.level >= 12`, tri `@timestamp` desc, 20 lignes |

### Colonnes du tableau (7)
`@timestamp`, `agent.name`, `rule.id`, `rule.description`, `data.srcip`

## Requêtes (DQL)
- Critiques : `rule.level >= 12`
- Hautes et plus : `rule.level >= 7`

## Seuils d'interprétation
| Niveau Wazuh | Sévérité |
|--------------|----------|
| 0-6 | Faible / informatif |
| 7-11 | Moyenne à haute |
| 12-15 | Critique |

## Utilité / Lecture
- **Priorisation** : traiter d'abord le tableau des critiques.
- **Surcharge SOC** : pic anormal sur la courbe horaire ou KPI total très supérieur à la moyenne.
- **Machine bruyante / compromise** : un agent dominant le Top 10 de façon disproportionnée.

## Actions recommandées
- Agent dominant : ouvrir le dashboard Endpoints (n°5) filtré sur cet agent.
- Règle dominante : analyser dans le dashboard Qualité des Règles (n°2).
- `data.srcip` récurrente : voir dashboard Menaces Réseau (n°3).
