# Installer un agent

L'agent est le capteur posé sur chaque machine à surveiller (serveur,
poste d'administration). Son installation est **pack-style** : un paquet
signé, vérifié avant tout arrêt de service, installé en une fenêtre de
quelques secondes, avec retour arrière automatique.

```bash
sudo janus-ctl install agent --pack janus-agent-pack.tar.gz
```

## Prérequis en échec automatique

L'agent embarque un **bouclier noyau** (eBPF LSM — la couche de
sécurité du noyau Linux) : la machine doit démarrer avec `lsm=bpf`
activé et le noyau le supporter. Sans cela, l'installation **refuse** —
un agent muet ne serait pas un agent, ce serait un mensonge affiché en
vert.

Prérequis annoncés honnêtement : noyau Linux **5.17 minimum** (nos
programmes eBPF utilisent les boucles du noyau), production validée sur
les noyaux **6.x**.

## L'identité : un certificat signé par un humain

À la première installation, l'agent génère sa clé privée **localement**
(elle ne quitte jamais la machine) et produit une **demande de
certificat**. La commande s'arrête là : la signature par l'autorité de
la stack est une **étape d'habilitation humaine**. Un serveur compromis
ne peut pas obtenir d'identité tout seul.

## Ce que garantit l'installation

1. Le pack est **vérifié** (empreintes + signature) **avant** l'arrêt
   de quoi que ce soit.
2. La fenêtre d'arrêt est typiquement de **~3 secondes** ; une unité
   systemd précédente est sauvegardée et restaurée si l'installation
   échoue (retour arrière automatique).
3. Au démarrage, une **barrière** compare le binaire exécuté au
   manifeste signé du pack : un binaire remplacé **refuse de démarrer**.
4. La configuration existante de la machine est **conservée**.

## Mettre à jour (OTA signé)

Les mises à jour suivent la même discipline :

```bash
janus-ctl ota build   # assembler le paquet
janus-ctl ota sign    # signer (la clé ne quitte jamais le serveur central)
janus-ctl ota verify  # vérifier AVANT l'arrêt du service
janus-ctl ota deploy  # fenêtre courte + retour arrière armé
janus-ctl ota rollback  # retour arrière explicite si besoin
```

Une alerte `agent_offline` pendant une fenêtre de maintenance est
normale : elle part dès la fermeture du flux, et le réarmement se fait
à la reconnexion.

## Vérifier

```bash
janus-ctl doctor      # empreinte du binaire vs manifeste, bouclier, métriques
janus-ctl gate contract  # rejoue le contrat de couverture sur CETTE machine
```
