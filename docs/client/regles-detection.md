# Écrire une règle de détection pour Janus

Ce document s'adresse aux **développeurs et intégrateurs** qui veulent ajouter
des détections adaptées à leurs équipements. Il décrit le format des règles,
les champs disponibles pour la détection, et la manière de valider qu'une
règle est bien chargée.

---

## Le principe en trois phrases

1. Les règles de détection de Janus sont au **format Sigma** — le standard
   ouvert du marché (même format que le grand entrepôt public de règles
   [Sigma HQ](https://github.com/SigmaHQ/sigma)) : ce que vous savez déjà
   écrire pour d'autres outils Sigma fonctionne ici.
2. Une règle est un **fichier YAML** déposé dans le répertoire de règles du
   collecteur — aucun code à compiler, aucun redémarrage de l'agent.
3. Une règle **ne déclenche jamais d'action** (blocage, isolation) par
   elle-même : elle produit une alerte ; le passage à une action est une
   configuration séparée, maîtrisée par l'opérateur du SOC.

---

## Où déposer une règle

| Répertoire | Usage | Contenu |
|---|---|---|
| `/etc/janus/sigma_rules/janus/<source>/` | Corpus **produit** Janus | Règles livrées et maintenues par Janus (ex. `fortigate/`) |
| `/etc/janus/sigma_rules/` (hors `janus/`) | Corpus public | Règles Sigma HQ importées (plusieurs milliers) |
| `/etc/janus/sigma_rules.d/` | **Règles intégrateur** — le vôtre | Chargé EN PLUS du corpus produit au démarrage du collector ; configurable (`sigma_rules_extra_path` dans collector.yml ou `JANUS_SIGMA_RULES_EXTRA_PATH`). Absent = ignoré sans erreur |

Fichiers acceptés : `.yml` et `.yaml`, récursivement dans les sous-répertoires.

⚠️ **Sur une machine où l'agent Janus tourne** : le répertoire `/etc/janus` est
protégé par le bouclier de l'agent (même `root` reçoit un refus). La pose de
règles se fait pendant une courte fenêtre de maintenance (arrêt de l'agent,
copie, redémarrage) — voir le runbook de déploiement.

---

## Le contrat de champs — sur quoi une règle peut matcher

Le moteur évalue chaque règle contre une **carte de champs** construite par
événement. Trois familles de champs :

### 1. Champs universels (toujours présents)

| Champ | Contenu | Exemple |
|---|---|---|
| `message` | Le message brut, tel que reçu | `<190>2026-09-03 ... action="deny" ...` |
| `source_ip` | L'adresse IP d'origine normalisée | `198.51.100.23` |
| `severity_label` | La sévérité normalisée | `warning` |

### 2. Champs normalisés du parseur (selon la source déclarée)

| Champ | Contenu |
|---|---|
| `eventtype` | Type d'événement composite (ex. `fortigate_event_admin`) |
| `action` | Action normalisée (ex. `login failed`, `deny`) |
| `user` | Utilisateur reconnu |
| `sourceip` / `targetip` | IPs normalisées (redondant avec `source_ip`, utile dans les filtres) |

### 3. Champs spécifiques au parseur — la partie intéressante pour l'intégrateur

Chaque parseur expose **tous les champs natifs du format** qu'il reconnaît :

| Source déclarée | Champs natifs exposés |
|---|---|
| `fortigate` | **Toutes les paires clé=valeur FortiOS** telles quelles : `type`, `subtype`, `logid`, `level`, `vd`, `action`, `srcip`, `dstip`, `remip`, `user`, `unauthuser`, `method`, `service`, `dstport`, `policyid`, `attack`, `tunneltype`, `msg`, `devname`, `devid`… — tout champ documenté FortiOS est matchable sans code supplémentaire |
| `panos` | Champs CSV PAN-OS aux **noms canoniques documentés** (types TRAFFIC, THREAT et AUTHENTICATION mappés position par position) : `type`, `subtype`, `src`, `dst`, `rule`, `srcuser`, `app`, `action`, `dport`, `proto`, `category`, `threatid`, `severity`, `misc`, `direction`, `user`, `event`, `desc`, `factorno`… — les autres types (CONFIG, SYSTEM…) exposent le préfixe commun (`type`, `subtype`, `serial`) + le message brut |
| `esxi` | `component` (composant émetteur en minuscules : `sshd`, `hostd`, `vpxa`, `vmkernel`, `vobd`…), `hostname`, `pid`/`procid`, `severity` (normalisée depuis le PRI syslog), `message_body`, `structured_data` (RFC 5424) — et les connexions normalisées : `action` (`login_success`/`login_failed`), `user`, `source_ip` (sshd style OpenSSH + hostd/Vpxa « User x@ip logged in ») |
| `nxlog` | **Toutes les clés du JSON NXLog** (événements Windows) telles quelles **ET en minuscules** : `eventid` (chaîne matchable — 4625, 4740…), `eventtype` (`AUDIT_FAILURE`…), `targetusername`, `subjectusername`, `ipaddress`, `servicename`, `ticketencryptiontype`, `status`, `logontype`, `workstationname`, `hostname`, `channel`, `sourcename`, `message`… — plus les normalisations `user`, `source_ip` (IpAddress « - » = local, ignorée) et `action` sémantique (`login_failed`, `account_locked`, `account_created`, `account_deleted`, `password_reset`) |
| `authlog` | `command` (sudo), champ brut dans `message` |
| `journald` | `message`, `unit`, `pid` |
| `ssh`, `nginx`, `wazuh`, `aws` | Voir le code : `janus-collector/pkg/parsing/mappers.go` |
| `generic` (défaut) | `message` brut uniquement — une règle sur mots-clés/regex du message fonctionne quand même |

> **Équipement sans parseur dédié ?** Déclarez la source en `generic` et
> écrivez votre règle sur `message` (mots-clés, expressions régulières) : la
> détection fonctionne. Les champs structurés demandent un parseur — les
> parseurs déclaratifs (décrire un format en YAML, sans compiler) sont à
  l'étude comme évolution.

---

## Anatomie d'une règle (exemple réel du dépôt)

```yaml
title: FortiGate — échec de connexion SSL-VPN          # visible dans les alertes
id: 3f9a1c52-7b8e-4d21-9c46-a1e0f7d83b01              # UUID unique, généré par vous
status: stable
description: >                                        # pour le SOC qui lira l'alerte
  Échec de connexion au VPN SSL FortiGate. Les rafales sont
  corrélationnées par le karma low-and-slow.
references:
  - https://docs.fortinet.com/document/fortigate/8.0.0/administration-guide/250999/log-settings-and-targets
author: Janus Guardian
date: 2026-09-03
tags:
  - attack.t1110                                      # technique MITRE ATT&CK (optionnel)
logsource:
  product: fortios                                    # informatif pour le classement
detection:
  selection:
    type: event                                       # ← les champs de la carte ci-dessus
    subtype: vpn
    action: tunnel-login-failed
  condition: selection
falsepositives:
  - Utilisateur légitime qui se trompe de mot de passe
level: medium                                          # low | medium | high | critical
```

Modificateurs Sigma usuels supportés : `|contains`, `|startswith`,
`|endswith`, `|re` (regex), listes de valeurs, `selection and not filter`.

---

## Score, confiance et seuil d'alerte

- Le `level` de la règle fixe le score de base : `critical` → 100, `high` → 80,
  `medium` → 50, `low` → 20.
- Le score est **pondéré par la confiance de la source** (déclarée dans le
  registre des sources) : `score × confiance / 100`.
- Une alerte Sigma est émise au-dessus du seuil 50. Concrètement : une règle
  `medium` sur une source à confiance 85 alerte ; une règle `low` reste un
  signal d'investigation sans alerter — utile pour nourrir le fil SOC.

## Corrélation et rafales

Une règle Sigma évalue **un événement à la fois** — pas de compteur interne.
Les rafales (brute force, low-and-slow) sont prises en charge par le moteur
de karma du collecteur : des alertes répétées d'une même zone font monter le
score et déclenchent l'alerte « Karma Violation (Low & Slow) » existante.
Écrivez donc la règle unitaire honnêtement, le produit fait la répétition.

## Garde-fous (pour que tout le monde dormant reste possible)

- **Déduplication** : une alerte identique à moins de ~5 minutes est silenciée.
- **Plafond d'ordres automatiques** : 6 actions automatiques / 10 minutes,
  quoi qu'il arrive (jamais appliqué aux actions humaines).
- **Vue précision** : la console SOC `/detections` montre les vrais positifs /
  faux positifs **par règle** — c'est là qu'une règle bruyante se voit
  immédiatement.

## Valider sa règle (retour honnête, sans redémarrage)

**1. Hors ligne, avant tout dépôt** :

```bash
janus-ctl sigma check ma-regle.yml        # un fichier
janus-ctl sigma check /etc/janus/sigma_rules.d/   # ou tout un répertoire
```

Le contrôle utilise le **même parseur que le collector** : un « OK » garantit
que la règle chargera. Il vérifie en plus le format (titre, détection,
condition) et **les références croisées** — une condition qui cite une
sélection inexistante serait une règle morte (l'évaluateur rend silencieusement
faux) : elle est signalée `INVALIDE` avec la raison. Code de sortie 1 si une
règle est invalide — utilisable en CI.

**2. Après dépôt** (fenêtre de maintenance si l'agent tourne) :
redémarrez le collecteur et vérifiez le **rapport de chargement** :

```
🧠 Sigma Engine loaded N rules from [/etc/janus/sigma_rules /etc/janus/sigma_rules.d] (skipped: K)
   ↳ skipped: /etc/janus/sigma_rules.d/ma-regle.yml: <raison>
```

Chaque règle sautée est **nommée avec sa raison** — jamais de silence. La
métrique `janus_sigma_rules_skipped` (port :9092) expose le compteur pour la
surveillance. La garde des conditions orphelines s'applique AUSSI au
chargement : une règle morte est sautée avec warning, pas laissée inerte.

**3. Test en conditions réelles** : envoyez un message réel (ou votre
émetteur) et surveillez la console SOC `/detections` (précision par règle).

## Compatibilité prouvée

- Le collecteur de production Janus charge **plus de 3 100 règles Sigma HQ
  publiques non modifiées**.
- Le vérificateur `janus-ctl sigma check` passe **2 773 règles sur 2 782** du
  corpus public (les 9 restantes sont rejetées par le parseur Sigma lui-même —
  jeton `*` nu — rejet préexistant, sans lien avec Janus).
- Un **test de non-régression** verrouille la compatibilité : une règle
  publique (détection DCSync, événement 5136) doit charger ET matcher un
  événement Windows réaliste à travers le parseur NXLog de Janus.

## Références

- [Spécification du format Sigma](https://github.com/SigmaHQ/sigma/wiki/Specification)
- [Entrepôt public de règles Sigma HQ](https://github.com/SigmaHQ/sigma)
- Corpus produit Janus : `deploy/sigma_rules/` dans le dépôt
- Parseurs et champs exposés : `janus-collector/pkg/parsing/mappers.go`
