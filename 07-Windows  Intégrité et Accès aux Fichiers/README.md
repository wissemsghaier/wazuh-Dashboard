# 7. Dashboard Windows — Intégrité et accès aux fichiers

## Objectif
Suivre les changements de fichiers et de chemins Windows remontés par Wazuh FIM (File Integrity Monitoring), ainsi que les opérations sur les fichiers auditées. Le dashboard met en évidence les hôtes, les chemins et les opérations à examiner.

## Prérequis
- Index pattern : `wazuh-alerts-*`
- Période affichée : **Last 7 days**
- Champs principaux : `@timestamp`, `agent.id`, `agent.name`, `rule.level`, `syscheck.path`, `syscheck.event`, `syscheck.changed_attributes`, `syscheck.audit.user.name`, `syscheck.audit.process.name`

## Visualisations

| # | Visualisation | Type | Utilité |
|---|---------------|------|---------|
| 1 | Hôtes concernés | KPI | Nombre d'hôtes ayant généré des alertes FIM |
| 2 | Chemins distincts touchés | KPI | Nombre de chemins distincts concernés |
| 3 | Fichiers ajoutés | KPI | Nombre d'événements d'ajout |
| 4 | Fichiers modifiés | KPI | Nombre d'événements de modification |
| 5 | Fichiers supprimés | KPI | Nombre d'événements de suppression |
| 6 | Alertes FIM niveau 10 et plus | KPI | Nombre d'alertes FIM de niveau 10 ou supérieur |
| 7 | Top chemins avec alertes | Barres horizontales | Chemins ayant généré le plus d'alertes |
| 8 | Changements sur chemins critiques | Tableau | Date, hôte, chemin, type de changement et attributs modifiés |
| 9 | Changements par heure et par hôte | Série temporelle | Évolution des changements FIM par hôte |
| 10 | Résumé par hôte | Tableau | Nombre d'événements, niveau maximal et date du dernier changement par hôte |
| 11 | Opérations auditées sur les fichiers | Tableau | Chemin, événement, utilisateur, processus, attribut modifié et nombre d'occurrences |

## Utilité / Lecture
- Une hausse soudaine des ajouts, modifications ou suppressions peut révéler une activité inhabituelle.
- Les chemins critiques et les clés de registre visibles dans le Top chemins doivent être comparés à l'activité attendue sur les hôtes.
- Les opérations auditées permettent de mettre en relation un changement avec l'utilisateur ou le processus concerné lorsque ces données sont disponibles.
- Les KPI sont calculés sur la période sélectionnée ; utilisez les filtres de temps pour comparer différentes périodes.

## Actions recommandées
- Vérifier tout changement inattendu sur les fichiers et chemins sensibles.
- Examiner l'utilisateur et le processus associés aux opérations auditées.
- Confirmer si les changements sont autorisés ; si nécessaire, investiguer les événements de l'agent et les alertes associées dans les autres dashboards SOC.
