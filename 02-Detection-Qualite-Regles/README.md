# 2. Dashboard Détection et Qualité des Règles

## Objectif
Mesurer ce que détectent les règles et améliorer leur qualité.

## Prérequis
- Index pattern : `wazuh-alerts-*`
- Période par défaut : **Last 7 days**
- Champs : `rule.id`, `rule.groups`, `rule.level`, `rule.description`, `data.srcip`, `agent.name`, `@timestamp`

## Visualisations

| # | Visualisation | Type | Configuration |
|---|---------------|------|---------------|
| 1 | Top rule id 7j | Horizontal bar | Terms `rule.id`, taille 20 |
| 2 | Top rule groups 7j | Horizontal bar / Tag cloud | Terms `rule.groups`, taille 20 |
| 3 | Alertes par rule level | Histogramme (vertical bar) | X : Terms `rule.level` (ordre croissant) ; Y : Count |
| 4 | Série temporelle des 10 règles les plus fréquentes | Line | X : Date Histogram 1h ; Split series : Terms `rule.id` taille 10 |
| 5 | Règles répétitives par même IP | Data table | Lignes : Terms `rule.id` puis Terms `data.srcip` ; métrique Count ; tri Count desc ; filtre `data.srcip : *` |
| 6 | Règles critiques jamais vues récemment | Data table | Voir section ci-dessous |
| 7 | KPI ratio bruit | Metric (Vega / TSVB / calcul) | `alertes rule.level <= 3` / alertes totales, en % |

## KPI ratio bruit
- Numérateur : `rule.level <= 3` (low severity ; adapter le seuil à votre politique, ex. `< 7`)
- Dénominateur : total alertes
- Implémentation : TSVB avec deux séries + Math `params.low / params.total * 100`
- Seuil indicatif : > 70 % = trop de bruit, à investiguer.

## Règles critiques jamais vues récemment
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
