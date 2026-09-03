# Déployer Janus Guardian

Cette section permet à une équipe d'exploitation d'installer la plateforme
dans son environnement, **sans l'équipe de développement**.

## Le parcours d'installation

1. **[Installer le serveur central](installation-serveur.md)** — une
   commande (`janus-ctl install stack`) : prérequis en échec automatique,
   installation native systemd sans Docker, assertions de vérification.
   Commencez par son mode `--dry-run` : il planifie l'installation sans
   rien écrire.
2. **[Installer un agent](installation-agent.md)** — un paquet signé par
   machine surveillée (`janus-ctl install agent`) : identité signée par un
   humain, bouclier noyau, fenêtre d'arrêt de ~3 secondes, retour arrière
   automatique.
3. **[Vérifier les flux](matrice-flux.md)** — les seuls ports à ouvrir,
   ce qui reste en bouclage local, et les prérequis noyau annoncés
   honnêtement (Linux 5.17+ pour les agents, production validée sur 6.x).

## Après l'installation

- `janus-ctl doctor` — état honnête en une commande : service, empreinte
  du binaire comparée au manifeste signé, bouclier (N hooks attendus),
  métriques.
- `janus-ctl gate contract` — rejoue le contrat de couverture complet ;
  vert = les garanties de détection tiennent sur VOTRE installation.

## Configurer au quotidien

- Configuration agent : `/etc/janus/agent.yml` (capteurs actifs, chemins
  surveillés par le FIM, pont de journaux fluent-bit — actif par défaut).
- Configuration collecteur : `/etc/janus/collector.yml` (seuils de
  corrélation, registre des sources agentless, ingeste ClickHouse).
- Surcharge à distance par agent : `PUT /api/v1/agents/{agent_id}/config`
  (les sections sont remplacées entières — toujours renvoyer la section
  complète).
- Ingestion d'équipements sans agent (pare-feu, annuaire) : voir
  [Équipements sans agent](../client/sources-agentless.md).
