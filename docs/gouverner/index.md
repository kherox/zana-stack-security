# Gouverner — sécurité, conformité, limites

Cette section décrit les garanties de sécurité du produit, les contrôles de
conformité, et — tout aussi important — **les limites assumées**, énoncées
honnêtement.

## Sécurité de la plateforme elle-même

- **Chiffrement mutuel (mTLS) partout** : agent, collecteur, hub et outils
  d'administration s'authentifient réciproquement par certificats. Le mode
  non chiffré est refusé en production.
- **Chaîne de signature des packs** : chaque pack de capteurs est signé
  (Ed25519) ; un pack falsifié est refusé au staging, et s'il passait, la
  barrière de démarrage refuse le binaire non conforme à son manifeste.
- **Bouclier de l'agent** : un eBPF LSM protège l'agent contre l'injection
  mémoire, le remplacement de son binaire et l'exécution sans fichier — la
  bannière de démarrage est honnête (elle refuse d'afficher « actif » si un
  hook critique manque).
- **Ordres à règle des deux** : les opérations perturbantes (isolation d'une
  machine, blocage d'IP) passent par une confirmation et sont journalisées
  (audit).
- **Quotas API** : l'API externe est en lecture seule, limitée (60
  requêtes/min) et refuse fail-closed sans plan associé (402).

## Conformité (SCA)

Quatre référentiels embarqués, évalués par machine avec score honnête et
empreinte SHA-256 des versions de règles : **CIS** (36 règles), **ANSSI**
(15), **PCI-DSS** (11), **ISO 27001** (10). Les versions de règles sont
immuables ; une règle falsifiée est refusée et listée. Gestion des
vulnérabilités (TVM) : bundles signés alimentés par les catalogues KEV /
EPSS / paquets de la distribution.

## Isolation multi-tenant — formulation officielle

> L'isolation entre locataires est **applicative** : chaque requête porte une
> identité vérifiée (JWT ou clé d'API à périmètre), et le filtre locataire
> est appliqué par les dépendances d'authentification et revérifié module par
> module, avec des tests d'isolation par module (lecture croisée refusée en
> 404 indistinguable) et une matrice d'isolation automatisée sur l'API
> externe. Le Row-Level Security PostgreSQL est planifié en **seconde couche
> de défense** au passage multi-tenant (SaaS), avec un rôle dédié documenté
> pour l'ingestion.

Cette formulation est volontairement falsifiable : elle décrit ce qui est
vérifié aujourd'hui, pas une promesse d'exhaustivité.

## Limites assumées (extraits du contrat de couverture)

| Limite | Détail |
|---|---|
| Linux uniquement | Windows/macOS exclus par choix d'architecture |
| Suppression via chemins relatifs | Un `rm -rf` opérant en chemins relatifs (dirfd) n'est pas attribué au chemin surveillé — les suppressions par chemin absolu sont couvertes |
| Écritures page-cache (Dirty Pipe) | L'écriture par corruption du cache page échappe au FIM (ouverture en lecture seule) — détection par effets aval |
| Isolation multi-tenant complète | Reportée au passage SaaS (RLS seconde couche) — voir ci-dessus |
| Un seul store sous RLS | PostgreSQL seulement ; la couche application reste le point d'enforcement commun (relationnel, séries temporelles, ClickHouse) |

La liste exhaustive et tenue à jour des exclusions vit dans le contrat de
couverture (section Opérateur) — une exclusion n'y entre qu'avec une décision
documentée.
