# Janus Guardian — Documentation

Documentation produit de la plateforme **Janus Guardian** (XDR / SIEM /
SOAR on-premise — détection étendue, supervision sécurité, orchestration
de réponse), organisée selon votre parcours :

| Section | Pour qui | Contenu |
|---|---|---|
| [Comprendre](comprendre/index.md) | Tous | Présentation, architecture, capacités |
| [Déployer](deployer/index.md) | Exploitation | Installation serveur central et agents, flux et ports |
| [Opérer](operateur/index.md) | Exploitants SOC | Exploitation quotidienne, contrat de couverture G1-G21 |
| [Intégrer](client/index.md) | Intégrateurs, développeurs | API REST, webhooks signés, MCP, règles Sigma, équipements sans agent |
| [Gouverner](gouverner/index.md) | Sécurité, direction | Durcissement, limites assumées, licence |

Règles de cette documentation :

- **Générée, jamais recopiée** : chaque contrat (API, base de données,
  capacités de détection) est généré depuis sa source de vérité dans le
  code, avec garde anti-dérive en CI.
- **Auto-hébergée et hors ligne** : construction sans aucune ressource
  externe (zéro CDN, zéro police distante) — le site se consulte en
  ligne et s'embarque tel quel dans un réseau isolé.
- **Honnête par construction** : aucun chiffre sans preuve de la maison,
  aucune promesse sans test rejouable.
