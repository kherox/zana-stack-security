# Intégrer Janus Guardian

Cette section s'adresse aux **intégrateurs et développeurs** qui
connectent la plateforme à leur système d'information : tableaux de bord
clients, SIEM externes, billetteries, copilotes IA, équipements réseau.

- [Guide d'intégration API](integration.md) — authentification par clé,
  routes de lecture, pagination par curseur, bornes, quotas et codes
  d'erreur.
- [Référence REST interactive](reference-api.md) — le contrat complet de
  l'API, généré depuis le code, explorable dans la page.
- [Webhooks signés](webhooks.md) — recevoir chaque alerte en temps réel,
  avec vérification de signature.
- [Écrire une règle de détection](regles-detection.md) — le format Sigma,
  vos règles séparées du corpus produit.
- [Connecter des équipements sans agent](sources-agentless.md) — pare-feux,
  annuaires, hyperviseurs : syslog RFC 5424 en TLS mutuel, registre de
  sources anti-usurpation.
- [Piloter par MCP](mcp.md) — brancher un copilote IA sur la défense avec
  les 10 outils du protocole MCP.

## La règle d'or de la surface client

L'API intégrateur est en **lecture seule** : alertes, événements,
incidents, posture du parc et résumé agrégé. **Aucune action de défense**
(blocage, isolation) n'est exposée aux clés d'API — les actions
disruptives restent dans la console des opérateurs, avec leurs propres
habilitations et garde-fous.
