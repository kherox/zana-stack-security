# Installer le serveur central

L'installation du serveur central (la « stack ») est **une commande** :

```bash
sudo janus-ctl install stack
```

Elle est **native** : des services systemd sur une machine Linux, sans
Docker. Elle est **idempotente** : rejouable sans casse — une
ré-exécution complète ce qui manque et conserve ce qui existe.

## Ce que la commande installe

| Brique | Rôle |
|---|---|
| PostgreSQL 17 + TimescaleDB | Base de référence (alertes, incidents, flotte) |
| ClickHouse | Base de chasse à grande échelle |
| Redis + NATS JetStream | État partagé et file de messages |
| PKI interne | Certificats mTLS (agents, relai de journaux) |
| Collecteur | Ingestion des agents et équipements, corrélations |
| Nexus | API + console web |
| Prometheus + Grafana + Alertmanager | Métriques et alerting d'infrastructure |

## Le déroulé, en trois temps

1. **Prérequis en échec automatique** — la commande refuse de démarrer
   sur une machine non conforme : Linux avec systemd, droits root,
   Ubuntu 24.04 LTS, disque ≥ 50 Go, commandes requises présentes.
   S'il existe déjà un déploiement, elle refuse aussi — sauf
   `--adopt` (adoption conservatrice d'une installation existante).
2. **L'installateur canonique** — la commande enveloppe et exécute
   l'installateur de référence du produit (dix sections : utilisateurs,
   secrets, paquets, binaires, configurations, bases de données,
   services). Les secrets sont **générés localement** et stockés en
   0600 dans `/etc/janus/` — ils ne quittent jamais la machine.
3. **Assertions** — le verdict est mécanique, pas espéré : services
   actifs, santé de l'API en HTTP 200, port gRPC joignable. Tout échec
   est affiché avec la commande de diagnostic à lancer.

## Essayer sans rien changer

Avant une vraie installation, demandez le plan :

```bash
sudo janus-ctl install stack --dry-run
```

Le mode `--dry-run` exécute les prérequis et affiche ce qui serait fait,
**sans écrire un seul fichier** (garanti par la suite de tests du
produit — chaque mutation est sous garde).

## Après l'installation

```bash
janus-ctl doctor        # état honnête de la machine, en une commande
```

Puis installez un agent sur la première machine à surveiller :
voir [Installer un agent](installation-agent.md).

## Repartir de zéro

`janus-ctl install stack` installe ; il ne désinstalle pas. Un
désinstalleur séparé est fourni avec le produit (`deploy/baremetal/
uninstall.sh` du dépôt source).
