# Client API — Intégration

Bienvenue sur l'API de lecture Janus Guardian. Cette surface est destinée aux
**tableaux de bord clients** : consultation de vos alertes, événements,
incidents et de la posture de votre parc — en **lecture seule**.

## Authentification

- Toute requête porte un en-tête `X-API-Key` (les sessions navigateur sont
  refusées sur ces routes).
- La clé est créée par votre administrateur tenant (`POST /api/v1/api-keys`),
  scopée par sujet : `read:alerts`, `read:events`, `read:incidents`,
  `read:agents`. Un scope inconnu rend la clé refusée (fail-closed).
- Révocation : effet en ≤ 60 s (cache de validation).

## Routes de lecture

| Route | Contenu |
|---|---|
| `GET /api/v1/external/alerts` | Vos alertes (filtres severity/status/dates) |
| `GET /api/v1/external/events` | Vos événements SIEM (métadonnées) |
| `GET /api/v1/external/incidents` | Vos incidents |
| `GET /api/v1/external/agents` | Posture de votre parc |
| `GET /api/v1/external/summary` | Résumé agrégé 24 h |

## Pagination par curseur

Réponse enveloppe : `{"data": [...], "next_cursor": "...|null",
"has_more": bool}`. Passez `next_cursor` tel quel au paramètre `cursor` pour
la page suivante. Le curseur est signé : le modifier le rend invalide (400) ;
il ne porte aucun droit d'accès (tout est revérifié à chaque page).

## Bornes et quotas

- `limit` ≤ 200 (alertes/événements), ≤ 100 (incidents) ; fenêtre ≤ 7 jours.
- **60 requêtes/minute** par tenant (réponse `429` + en-tête `Retry-After`).
- Requête SQL bornée à 3 secondes (réponse `504` — réessayez avec des bornes
  plus fines).
- **Quota du plan** : au-delà de la limite `max_api_external_read` de votre
  plan (ou sans plan actif), réponse `402` avec détail `quota_exceeded` et
  `upgrade_url`.

## Codes d'erreur

| Code | Signification |
|---|---|
| 401 | Clé d'API absente/invalide (les sessions navigateur ne passent pas) |
| 402 | Quota du plan épuisé ou plan absent |
| 403 | Scope insuffisant pour la route |
| 429 | Trop de requêtes — respectez `Retry-After` |
| 504 | La lecture a dépassé le time-out — resserrez vos bornes |

## Référence complète

- Contrat machine : [`openapi.json`](../api/openapi.json) (source unique,
  garde anti-dérive en CI).
- Interface interactive : page `/api-docs` de votre instance (activée par
  l'opérateur).
