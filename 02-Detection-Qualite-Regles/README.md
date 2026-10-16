# 2. Dashboard Détection et Qualité des Règles

## Objectif
Analyser les alertes générées par les règles Wazuh afin d'identifier les règles les plus actives, les événements répétitifs et les éventuels signaux de bruit. Ce dashboard aide à repérer les règles à examiner et à suivre leur activité dans le temps.

## Prérequis
- Index pattern : `wazuh-alerts-*`
- Période par défaut : **Last 7 days**
- Champs : `rule.id`, `rule.groups`, `rule.level`, `rule.description`, `data.srcip`, `agent.name`, `@timestamp`

## Filtres interactifs
Les filtres en haut du dashboard permettent de limiter les visualisations selon :
- l'identifiant de règle (`rule.id`) ;
- le groupe de règle (`rule.groups`) ;
- le niveau de sévérité (`rule.level`) ;
- le nom de l'agent (`agent.name`) ;
- l'adresse IP source (`data.srcip`) ou `sourceAddress`.

## Visualisations

| # | Visualisation | Type | Configuration |
|---|---------------|------|---------------|
| 1 | Top 10 Rule ID | Barres horizontales | 10 identifiants `rule.id` les plus fréquents |
| 2 | Top rule groups | Barres horizontales | Groupes `rule.groups` les plus fréquents |
| 3 | Alertes par rule level | Histogramme (vertical bar) | X : Terms `rule.level` (ordre croissant) ; Y : Count |
| 4 | Évolution temporelle Top 10 Rule ID | Line | X : Date Histogram `@timestamp` par heure ; séries : 10 `rule.id` les plus fréquents |
| 5 | Règles répétitives par source IP | Data table | Regroupement par IP source et `rule.id` ; métrique Count |
| 6 | Règles critiques peu vues récemment | Data table | `rule.id`, nombre d'alertes et date de la dernière alerte ; voir section ci-dessous |
| 7 | KPI ratio bruit | Metric (Vega / TSVB / calcul) | `alertes rule.level <= 3` / alertes totales, en % |

## KPI ratio bruit
- Numérateur : `rule.level <= 3` (low severity ; adapter le seuil à votre politique, ex. `< 7`)
- Dénominateur : total alertes
- Implémentation : TSVB avec deux séries + Math `params.low / params.total * 100`
- Seuil indicatif : > 70 % = trop de bruit, à investiguer.

## Règles critiques peu vues récemment
Les alertes n'existent que si la règle a déclenché ; une règle « absente » se détecte par comparaison :
1. Lister les règles critiques attendues (`rule.level >= 12`) sur **30 jours** (Terms `rule.id`, last_seen = Max `@timestamp`).
2. Afficher la colonne **Max @timestamp** et trier asc : les règles dont la dernière occurrence est > 7 jours sont à vérifier.
3. Compléter avec une liste de référence des règles essentielles (FIM, Sysmon, brute force) vérifiée dans `/var/ossec/ruleset/rules` et `etc/rules`.

## Utilité / Lecture
- **Faux positifs** : règle top avec level bas et même `data.srcip` répété → candidat au tuning (`overwrite` level 0 ou `if_sid` / `noalert`).
- **Règles custom** : suivre leur volume après déploiement (ID 100000+).
- **Couverture** : règle critique silencieuse → vérifier décodeur, agent, log source.

## Actions recommandées
- Ratio bruit élevé : réduire fréquence (`frequency`, `timeframe`, `ignore`) ou baisser le niveau.
- Règle répétitive : ajouter une exception dans `local_rules.xml`.
- Valider les modifications avec `wazuh-logtest` puis redémarrer le manager.
