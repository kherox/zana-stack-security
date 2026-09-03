# Connecter des équipements sans agent

Tous les actifs ne peuvent pas recevoir un capteur : pare-feux,
routeurs et commutateurs, hyperviseurs, annuaires d'entreprise. La
plateforme les intègre par **journal** : l'équipement envoie son syslog,
la plateforme le traite comme une source de détection à part entière —
avec une **anti-usurpation** qui garantit qu'une source ne peut pas se
faire passer pour une autre.

## Le chemin d'une ligne de journal

```text
Équipement (pare-feu, annuaire...)
   │  syslog RFC 5424, TLS mutuel, port 6514
   ▼
Relai de journaux (colocalisé au collecteur)
   │  file d'attente disque bornée — survit aux coupures
   ▼
Collecteur : registre de sources → corrélations → alertes
```

## Côté équipement : trois réglages

1. **Format** : syslog **RFC 5424** (le format avec horodatage structuré,
   appelé aussi IETF) sur **TCP** — pas l'ancien format BSD sur UDP.
2. **Transport** : **TLS mutuel** vers le port **6514** du relai. Le
   TLS mutuel signifie que l'équipement présente aussi un **certificat
   client** : c'est lui qui prouve son identité, bien avant toute
   inspection du contenu.
3. **Certificat client** : émis par l'autorité de certification de la
   plateforme, avec un **nom commun (CN)** unique par équipement ou par
   famille. Le relai n'accepte que les CN **explicitement habilités** —
   un équipement inconnu est refusé au handshake.

## Côté plateforme : le registre de sources

Chaque équipement (ou famille) est **déclaré nominativement** dans le
registre des sources : un **nom** (celui qui apparaîtra dans les alertes)
et le **bloc d'adresses (CIDR)** d'où il parle. Ce registre fait deux
choses :

- **Attribution** : chaque ligne de journal est rattachée à sa source
  déclarée — les alertes portent le nom de l'équipement, pas une adresse
  anonyme ;
- **Anti-usurpation** : un journal qui se présente sous une identité
  depuis un endroit inattendu est **alerté** (`source_origin_unexpected`)
  et marqué à confiance réduite — un attaquant qui rejouerait du syslog
  est ainsi vu, au lieu d'être cru.

## Résilience : « coupure → perte 0 »

La file d'attente du relai vit **sur disque** et redémarre où elle
s'était arrêtée. Une coupure du collecteur ne perd pas les journaux
reçus pendant l'arrêt : ils sont rejoués dès le retour du service. Ce
comportement est **prouvé par le contrat de couverture** : une coupure
de 10 minutes avec poursuite de l'émission, puis vérification que les
100 % des messages sont arrivés en base après rétablissement.

## Honnêteté sur le périmètre

- L'analyse fine des journaux est **famille par famille** (pare-feu,
  annuaire...) : les formats constructeurs sont ajoutés progressivement.
  À défaut de parseur dédié, la source est traitée en générique — les
  journaux arrivent en base et alimentent le registre, mais sans
  traduction constructeur.
- Les équipements sans agent sont une source de **détection**, pas de
  **blocage** : il n'y a pas de capteur pour y appliquer une isolation.
  Les directives de blocage en boucle SOAR concernent les adresses IP
  vues attaquer — appliquées là où un contrôle existe.
