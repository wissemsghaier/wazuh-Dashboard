# 6. Dashboard Linux — Intégrité et accès aux fichiers

## Objectif
Surveiller les changements de fichiers détectés par Wazuh FIM (File Integrity Monitoring) sur les hôtes Linux. Ce dashboard aide à repérer les modifications sensibles, identifier les hôtes concernés et examiner les événements associés.

## Prérequis
- Index pattern : `wazuh-alerts-*`
- Période affichée : **Last 7 days**
- Filtre visible dans l'export : agent `UBUNTU-SRV2` et groupes `syscheck` / `syscheck.path`
- Champs principaux : `@timestamp`, `agent.id`, `agent.name`, `rule.level`, `rule.groups`, `syscheck.path`, `syscheck.event`, `syscheck.changed_attributes`

## Visualisations

| # | Visualisation | Type | Utilité |
|---|---------------|------|---------|
| 1 | Total des alertes | KPI | Nombre total d'alertes FIM sur la période |
| 2 | Hôtes concernés | KPI | Nombre d'hôtes Linux ayant généré des alertes FIM |
| 3 | Chemins distincts | KPI | Nombre de chemins de fichiers distincts concernés |
| 4 | Événements Who-data | KPI | Nombre d'événements enrichis avec les informations d'accès Who-data |
| 5 | Alertes niveau 10 et plus | KPI | Nombre d'alertes FIM de niveau 10 ou supérieur |
| 6 | Changements par heure et hôte | Série temporelle | Évolution des alertes FIM, ventilée par hôte |
| 7 | Alertes niveau 10 et plus dans le temps | Série temporelle | Évolution des alertes FIM à sévérité élevée |
| 8 | Top chemins modifiés | Barres horizontales | Chemins ayant généré le plus d'événements |
| 9 | Top hôtes | Barres horizontales | Hôtes ayant généré le plus d'alertes FIM |
| 10 | Changements par périmètre sensible | Barres horizontales | Répartition des alertes pour les chemins surveillés |
| 11 | Top chemins sensibles touchés | Barres horizontales | Principaux chemins sensibles concernés |
| 12 | Types de changement | Diagramme en anneau | Répartition entre fichiers ajoutés, modifiés et supprimés |
| 13 | Attributs modifiés | Tableau | Attributs modifiés, par exemple `mtime`, `md5` et `sha1` |
| 14 | Détail des changements | Tableau | Date, hôte, chemin, type d'événement et attributs modifiés |
| 15 | Changements sur fichiers critiques | Tableau | Détail des changements concernant les chemins critiques |
| 16 | Attribution Who-data | Tableau | Événements enrichis des informations Who-data |
| 17 | Résumé par hôte | Tableau | Volume d'alertes, niveau maximal et dernier changement par hôte |
| 18 | Nombre d'événements Who-data par hôte | Tableau | Volume d'événements Who-data associé à chaque hôte |

## Utilité / Lecture
- Un pic dans les séries temporelles peut signaler une activité de changement inhabituelle.
- Un chemin sensible modifié ou supprimé mérite une vérification dans le tableau de détail.
- Les attributs d'intégrité tels que `md5`, `sha1` et `sha256` aident à qualifier les changements observés.
- Les informations Who-data, lorsqu'elles sont disponibles, aident à attribuer un accès ou un changement à son utilisateur.

## Actions recommandées
- Vérifier les changements inattendus sur les fichiers système et les chemins critiques.
- Confirmer que les changements sont autorisés auprès du propriétaire du système.
- En cas d'activité suspecte, examiner les alertes correspondantes et l'activité de l'agent dans les autres dashboards SOC.
