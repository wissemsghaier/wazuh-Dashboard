# 5. Dashboard Endpoints et Santé des Agents

## Objectif
Garantir la visibilité SOC et repérer tôt les signaux de compromission endpoint.

## Prérequis
- Index patterns : `wazuh-alerts-*` et `wazuh-monitoring-*` (statut des agents, selon version) ou API Wazuh `/agents`
- Modules actifs : FIM (syscheck), Sysmon (Windows), logcollector journald/syslog
- Champs : `agent.name`, `agent.status`, `syscheck.path`, `syscheck.event`, `data.win.eventdata.image`, `data.win.eventdata.commandLine`, `rule.groups`, `rule.level`

## Visualisations

| # | Visualisation | Type | Configuration |
|---|---------------|------|---------------|
| 1 | Agents connectés vs déconnectés | Pie / Metric | Terms `status` (active / disconnected / never_connected) ; source : monitoring ou API |
| 2 | Agents inactifs > X minutes | Data table | Filtre `status : disconnected` ; colonnes nom, IP, `lastKeepAlive`, groupe ; X par défaut = 15 min |
| 3 | Changements FIM par hôte | Line | Date Histogram ; Split series : Terms `agent.name` ; filtre `rule.groups : "syscheck"` |
| 4 | Alertes Sysmon suspectes | Line | Filtre `rule.groups : "sysmon"` et `rule.level >= 7` |
| 5 | Top commandes/processus à risque | Data table / Bar | Terms `data.win.eventdata.commandLine` / `image` ; filtre ci-dessous |
| 6 | Incidents système Linux | Data table | Filtre ci-dessous ; colonnes `@timestamp`, `agent.name`, `rule.description`, `full_log` |
| 7 | Résumé par host | Data table | Lignes : Terms `agent.name` ; métriques : Max `rule.level`, Count, Max `@timestamp` |

## Requêtes (DQL)
- Processus à risque :
  `data.win.eventdata.commandLine : (*-enc* or *-EncodedCommand* or *DownloadString* or *IEX*) or data.win.eventdata.image : (*mshta.exe or *certutil.exe or *regsvr32.exe or *rundll32.exe or *bitsadmin.exe or *wmic.exe)`
- Incidents Linux : `rule.groups : "syslog" and (full_log : (*"Failed with result"* or *"Out of memory"* or *oom-killer* or *segfault*) or rule.level >= 8)`
- FIM critiques : `rule.groups : "syscheck" and syscheck.path : (*/etc/* or *\\System32\\* or *\\Windows\\*)`

## Utilité / Lecture
- **Visibilité** : un agent déconnecté = angle mort (panne, arrêt volontaire, ou sabotage de l'agent).
- **Compromission** : pics FIM sur chemins sensibles, PowerShell encodé, LOLBins.
- **Résumé host** : trier par niveau max puis dernier événement pour prioriser les machines.

## Actions recommandées
- Agent inactif : vérifier service `wazuh-agent`, réseau (1514/1515), puis investiguer si inattendu.
- Processus suspect : isoler l'hôte, collecter mémoire/logs, vérifier l'arbre de processus Sysmon.
- Changement FIM non planifié : comparer avec la baseline et la gestion des changements.
