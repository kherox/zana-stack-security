# Webhooks signés

Un **webhook** pousse chaque nouvelle alerte vers votre système en temps
réel : SIEM externe, billetterie, messagerie, automatisation maison. À la
création d'une alerte, la plateforme envoie un `POST` JSON vers chaque
webhook actif dont le filtre correspond — avec une **signature** qui permet
à votre récepteur de prouver que la requête vient bien de la plateforme.

## Créer un webhook

Via l'API (session administrateur) :

- `POST /api/v1/security/webhooks` — enregistre une URL, un secret partagé et une
  liste de types d'événements ;
- `GET /api/v1/security/webhooks` — liste ; `PATCH /api/v1/security/webhooks/{webhook_id}` — mise à
  jour (activation, filtre) ; `DELETE /api/v1/security/webhooks/{webhook_id}` — suppression.

Le secret partagé est stocké **chiffré** côté plateforme et n'est plus
jamais retourné en clair après la création.

## Filtrage par type d'événement

Chaque webhook porte une liste de types d'événements (`events`) :

- le webhook ne reçoit que les alertes dont le `event_type` figure dans sa
  liste (ex. `brute_force`, `fim_violation`, `exfiltration`) ;
- le type générique **`alert.created`** agit comme attrape-tout : un
  webhook qui le liste reçoit **toutes** les alertes.

## Le payload

Le corps du `POST` est l'alerte elle-même, en JSON :

```json
{
  "event_type": "brute_force",
  "severity": "high",
  "alert_id": "...",
  "tenant_id": "default",
  "..."
}
```

## Vérifier la signature

Chaque requête porte l'en-tête **`X-Janus-Signature`** : le code
d'authentification **HMAC-SHA256** (en hexadécimal) du corps brut de la
requête, calculé avec votre secret partagé.

Vérification côté récepteur (exemple Python) :

```python
import hmac, hashlib

def verifier(raw_body: bytes, header_signature: str, secret: str) -> bool:
    attendu = hmac.new(secret.encode(), raw_body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(attendu, header_signature)
```

Deux règles importantes :

1. calculez la signature sur le **corps brut** (les octets exacts reçus,
   avant tout re-encodage JSON) ;
2. comparez toujours à **temps constant** (`compare_digest`) — jamais `==`.

Une requête sans signature vérifiable doit être rejetée : c'est le seul
moyen de distinguer la plateforme d'un émetteur qui se ferait passer pour
elle.

## Fiabilité et limites honnêtes

- **3 tentatives** avec espacement progressif ; un code `2xx` clôt
  l'envoi.
- **Best-effort** : un webhook en panne ne ralentit jamais la détection —
  l'alerte reste en base quoi qu'il arrive.
- **Coupe-circuit par URL** : après une série d'échecs consécutifs, la
  plateforme cesse d'appeler votre URL pendant un délai de recul (une
  sonde repart ensuite ; un succès referme le circuit). C'est ce qui
  protège la plateforme d'un récepteur mort sans la priver des autres.
- Les webhooks sont un **complément**, pas un journal : pour la chaîne de
  conservation et la chasse, lisez l'API — ne reconstruisez pas votre
  historique à partir des seuls webhooks.
