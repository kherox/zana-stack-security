# Piloter la plateforme par MCP

Le **protocole MCP** (Model Context Protocol — le standard ouvert qui
relie un assistant IA à vos outils) est intégré à la plateforme : un
copilote connecté peut interroger et piloter la défense avec les **mêmes
droits** que l'utilisateur auquel il est branché, sans aucun développement
spécifique.

## Le principe en trois phrases

1. La plateforme expose **10 outils** MCP : le copilote les découvre et
   les appelle tout seul, en langage naturel.
2. Chaque appel s'exécute avec **l'identité et l'isolation du tenant** de
   l'utilisateur de la session — un copilote ne voit jamais le parc d'un
   autre client.
3. Aucune route publique dédiée : MCP passe par les sessions de la
   plateforme (WebSocket `/mcp/ws`, SSE `/mcp/sse` + `POST /mcp/messages`).

## Les 10 outils

| Outil exposé | Rôle |
|---|---|
| Analyze Security Events | Interroger et analyser les événements en langage naturel (« échecs de connexion des dernières 24 h ») |
| Manage Security Alerts | Créer, mettre à jour, escalader ou clore des alertes |
| Threat Intelligence Lookup | Enrichir un indicateur (IP, domaine, hash, URL) avec le renseignement de menaces |
| Agent Health Monitoring | État des capteurs et métriques système, par agent ou sur tout le parc |
| Security Policy Management | Appliquer ou ajuster des politiques de sécurité sur les agents |
| Digital Forensics Investigation | Recherche avancée et reconstitution de chronologie |
| Security Reporting | Produire des rapports de sécurité |
| User Access Management | Gérer les accès utilisateurs |
| Automated Incident Response | Déclencher les réponses à incident automatisées |
| Compliance Audit Check | Vérifier la conformité aux référentiels |

Les paramètres exacts de chaque outil sont décrits dans leur schéma MCP
(découverte automatique par le client) — aucune documentation parallèle à
maintenir.

## Ce qu'un copilote ne peut pas faire

- **Dépasser les droits de son utilisateur** : l'habilitation est celle de
  la session, pas une élévation.
- **Voir au-delà de son tenant** : l'isolation s'applique à chaque appel
  d'outil.
- **Contourner les garde-fous humains** : les actions disruptives qui
  exigent une double validation côté plateforme l'exigent aussi via MCP.
