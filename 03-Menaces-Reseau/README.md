# 3. Dashboard Menaces Réseau

## Objectif
Détecter scans, brute force réseau, C2 possible et flux anormaux.

## Prérequis
- Index pattern : `wazuh-alerts-*`
- Sources : firewall (pfSense, Fortinet, iptables…), Suricata/Zeek si intégrés, logs système
- Champs : `data.srcip`, `data.dstip`, `data.srcport`, `data.dstport`, `data.protocol`, `data.action`, `rule.groups`, `GeoLocation.*`
- Géolocalisation : champs `GeoLocation.location` (type geo_point) pour la carte, activée via le pipeline GeoIP de l'indexer

## Visualisations

| # | Visualisation | Type | Configuration |
|---|---------------|------|---------------|
| 1 | Événements réseau par heure | Line | Date Histogram 1h ; filtre `data.srcip : *` |
| 2 | Top source IP externes | Horizontal bar | Terms `data.srcip` taille 10 ; exclure RFC1918 (`NOT data.srcip : "10.0.0.0/8"` etc.) |
| 3 | Top destination IP internes | Horizontal bar | Terms `data.dstip` taille 10 |
| 4 | Top ports ciblés | Horizontal bar | Terms `data.dstport` taille 15 |
| 5 | Répartition protocoles | Pie | Terms `data.protocol` (tcp, udp, icmp) |
| 6 | Alertes scan et flood | Data table | Filtre ci-dessous ; colonnes : `@timestamp`, `data.srcip`, `data.dstip`, `data.dstport`, `rule.description` |
| 7 | Firewall bloqués vs autorisés | Stacked bar / Pie | Terms `data.action` (drop/block/deny vs accept/allow) |
| 8 | Carte des IP sources | Maps (Coordinate map) | Geohash sur `GeoLocation.location` |

## Requêtes (DQL)
- Scan / flood : `rule.groups : ("recon" or "web_scan" or "ids" or "firewall_block" or "attack" or "syn_flood")` ou `rule.description : (*scan* or *flood*)`
- Bloqués : `data.action : (drop or block or deny or reject)`
- Autorisés : `data.action : (accept or allow or pass)`

## Utilité / Lecture
- **Attaquant principal** : IP en tête du Top source, confirmée sur la carte.
- **Large vs ciblée** : beaucoup de `dstip` pour une `srcip` = scan large ; une seule `dstip` / peu de ports = attaque ciblée.
- **C2 possible** : connexions sortantes régulières (beaconing) vers IP rare, ports inhabituels.

## Actions recommandées
- Bloquer l'IP au firewall / active-response (`firewall-drop`).
- Segmenter ou restreindre les ports exposés les plus ciblés.
- Enrichir l'IP (AbuseIPDB, VirusTotal) avant blocage définitif.
