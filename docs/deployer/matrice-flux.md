# Matrice des flux et ports

Tout ce qui doit être ouvert — et surtout **ce qui n'a pas besoin de
l'être**. Règle d'architecture : seuls le port des agents, le port web
et le port des journaux d'équipements sont exposés ; **tout le reste
écoute sur le bouclage local uniquement**.

## Flux entrants sur le serveur central

| Port | Service | Exposition | Chiffrement | Qui parle |
|---|---|---|---|---|
| 50051 | Collecteur (gRPC) | réseau | mTLS mutuel | les agents |
| 8000 | Nexus (API + console) | réseau (à restreindre) | HTTPS/TLS conseillé | opérateurs, intégrateurs |
| 6514 | Relai de journaux | réseau | TLS mutuel (RFC 5424) | pare-feux, annuaires, équipements |

## Flux locaux au serveur central (bouclage — rien à ouvrir)

| Port | Service | Consommateur |
|---|---|---|
| 5432 | PostgreSQL | Nexus, collecteur |
| 6379 | Redis | collecteur |
| 4222 | NATS | collecteur, Nexus |
| 9000 / 8123 | ClickHouse (natif / HTTP) | collecteur, chasse |
| 8083 | Ingestion de journaux du collecteur | relai local uniquement |
| 9092 | Métriques du collecteur | Prometheus |
| 9090 / 3000 / 9093 | Prometheus / Grafana / Alertmanager | opérateurs |

## Flux depuis un agent

Un agent n'a besoin que du serveur central : **gRPC 50051** (télémétrie
et directives), sortant uniquement. Aucun port entrant n'est requis sur
la machine surveillée — sa compromission ne crée aucun accès vers elle.

## Flux depuis un équipement sans agent

L'équipement (pare-feu, annuaire) n'a besoin que de **6514/TCP** vers le
relai, en TLS mutuel — voir
[Connecter des équipements sans agent](../client/sources-agentless.md).

## Exemples d'adresses

Cette documentation utilise exclusivement, en exemple, les plages
réservées à la documentation (RFC 5737 : `192.0.2.x`, `198.51.100.x`,
`203.0.113.x`) et des noms génériques (`collector.exemple.lan`). Le
visualiseur d'API et les captures référencent `127.0.0.1` pour l'accès
local.

## Prérequis noyau (honnêtes)

- **Agents** : noyau Linux **5.17 minimum** — les programmes eBPF du
  capteur utilisent les boucles du noyau introduites là ; **6.x
  recommandé** (production validée sur noyaux 6.x Azure/Ubuntu 24.04).
- **Bouclier de l'agent** : démarrage avec `lsm=bpf` (paramètre noyau) —
  sans lui, l'installation refuse par conception.
- **Serveur central** : aucun prérequis noyau particulier ; systemd est
  la seule brique d'orchestration.
