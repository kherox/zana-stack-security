# Comprendre Janus Guardian

Janus Guardian est une plateforme **XDR / SIEM / SOAR** pour parcs
**Linux** : elle détecte les comportements malveillants sur vos machines,
les met en corrélation, ouvre des alertes et des incidents, et peut agir
seule pour contenir une attaque (blocage d'adresse IP, gel de processus,
capture mémoire).

## Les quatre composants

| Composant | Rôle | Technologie |
|---|---|---|
| **janus-agent** | Capteur sur chaque machine : télémétrie processus/réseau/fichiers, bouclier eBPF LSM qui bloque les attaques contre l'agent lui-même, réponses autonomes (blocage IP, gel de processus) | Go + eBPF |
| **janus-collector** | Pivot central : reçoit la télémétrie (gRPC chiffré), fait les corrélations (force brute, exfiltration, enchaînements d'attaque), renvoie des ordres aux agents | Go |
| **janus-nexus** | Hub : console web, API REST, corrélation avancée, gestion de la flotte, conformité (SCA), gestion des vulnérabilités (TVM), facturation d'usage | Python / FastAPI |
| **janus-ctl** | CLI d'exploitation : installation, mises à jour signées (OTA), diagnostic, portail des gates | Go |

## Ce que la détection garantit

La détection n'est pas une promesse : un **contrat de couverture** liste les
comportements d'attaque qui sont *garantis détectés*, ceux *en construction*,
et les *exclusions assumées*. Chaque ligne garantie est rejouée automatiquement
à chaque mise à jour (gate `contrat`) — une régression devient un échec
bloquant.

Points couverts (extraits) : écriture/suppression de fichiers surveillés,
exécution sans fichier (memfd), exfiltration de données (volume et cumul),
force brute (rafale et lente), ransomware (leurre + gel), tentatives de
manipulation de l'agent, chargement d'eBPF offensif, tunnel DNS, empreintes
TLS. Le détail ligne à ligne vit dans la section Opérateur (contrat de
couverture).

## Ce que la plateforme peut faire seule (SOAR)

- **Bloquer** une adresse IP attaquante (nftables, avec liste blanche
  anti-auto-blocage du collecteur).
- **Geler** un processus de chiffrement (cgroup v2) — prouvé sur un
  ransomware réel du lab (chiffrement arrêté en cours de course).
- **Capturer** la mémoire d'un processus suspect (forensique) — plafonnée,
  jamais en boucle.
- Chaque ordre automatique est plafonné (6/10 min par agent et par type) :
  la plateforme ne s'emballe pas, et les ordres humains ne sont jamais
  plafonnés.

## Ce que la plateforme n'est PAS (assumé)

- **Linux uniquement** : Windows et macOS sont exclus par choix
  d'architecture.
- **Single-tenant on-premise** en cette version : l'isolation multi-tenant
  complète (RLS en seconde couche) est planifiée au passage SaaS — voir
  Gouverner.
- **Pas un WAF** : le capteur web (WAF-lite) détecte les patterns d'attaque
  dans les journaux et bloque les IP ; il n'inspecte pas les charges utiles.
