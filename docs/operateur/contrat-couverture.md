# Contrat de couverture — lignes garanties

La plateforme garantit **21 lignes de détection**.
Chaque ligne est un scénario d'attaque **rejouable** dont le
verdict automatique est archivé et horodaté : une régression d'une
ligne garantie est un échec **bloquant**. Les lignes G7b, G15b sont des variantes cumulées ou bloquées de leur ligne mère : elles comptent avec elle.

Vérifiez le contrat sur VOTRE installation après déploiement :

```bash
janus-ctl gate contract
```

Un verdict vert signifie que les garanties ci-dessous tiennent sur
votre déploiement, avec vos équipements et vos configurations.

## Lignes garanties

| Ligne | Garantie | Rejouable via |
|---|---|---|
| G1 | **FIM écriture** — modification d'un fichier surveillé (openat écriture dans `watch_paths`) → alerte `fim_violation` haute ≤ 60 s | `01-fim-ecriture` |
| G2 | **Chaîne détection→alerte** — tout event de détection prioritaire agent → alerte Nexus | `—` |
| G3 | **Brute force SSH en rafale** — ≥ 6 échecs en < 10 s même IP (fenêtre rafale DÉPLOYÉE ~10 s) → alerte « Brute Force » haute | `02-bruteforce-rafale` |
| G4 | **Brute force SSH low-and-slow** — 10 échecs cumulés/10 min (jamais 6/min) → alerte « Brute Force » haute | `03-bruteforce-lowslow` |
| G5 | **Exécution fileless memfd BLOQUÉE** — execveat/fexecve sur `/memfd:` → EPERM (hook bprm, shield 5/5) + blocage journalisé | `04-fileless-blocage` |
| G6 | **Ptrace sur l'agent BLOQUÉ** — PTRACE_ATTACH sur le process agent → EPERM + alerte `agent_tampering_attempt` haute émise (non spécifiable) | `05-tamper-ptrace` |
| G7 | **Exfiltration volume+ratio** — ≥ 50 Mio envoyés par un process ET ratio envoyé/lu > 1,1 (renvois répétés du même contenu) → event + alerte `exfiltration` haute | `06-exfil-seuil` |
| G8 | **Honeyfile — DÉTECTION** — accès (lecture/écriture/suppression) sous une zone-appât configurée → détection agent immédiate (journal `HONEYPOT`/`HONEYFILE`) + tentative d'isolation du process | `07-honeyfile-detection` |
| G9 | **Update OTA trojanisée rejetée** — pack trafqué → rejet fail-closed (checksum + Ed25519 2 clés) + sensors-verify refuse au boot + binaire intact + agent vivant | `08-ota-trojan` |
| G10 | **Remplacement du binaire agent refusé au boot** — binaire trojan sur le chemin du pack → le service REFUSE de démarrer (invariant sensors-verify « chemin exécuté dans le pack ») | `10-tamper-remplacement` |
| G11 | **Alerte agent_offline** — agent arrêté/tué → alerte Nexus `agent_offline` haute (debounce 5 min, réarmée à la reconnexion) | `10-agent-offline` |
| G12 | **Boucle SOAR BLOCK_IP sans suicide** — directive BLOCK_IP émise et reçue après alerte; l'IP du collecteur est allowlistée → la télémétrie survit à son propre blocage | `—` |
| G7b | **Exfiltration petits volumes répétée** — envois sous le seuil volume mais répétés dans une fenêtre → 2e signal cumul → event + alerte `exfiltration` confiance 75 | `06b-exfil-cumul` |
| G13 | **Exécution fileless dédiée** — tentative `/memfd:` → event DÉDIÉ `fileless_execution` confiance 100 + alerte critique — le pivot bprm voit TOUTES les tentatives, y compris bloquées (First Seen ne voit jamais l'exec bloqué) | `04-fileless-blocage` |
| G14 | **FIM suppressions** — destruction d'un fichier surveillé — `unlinkat` (coreutils), `sys_unlink` (libc/Python), `renameat2`-départ (`mv`) → alerte `fim_violation` haute ≤ 60 s | `11-fim-suppression` |
| G15 | **Chargement eBPF par un tiers** — `BPF_PROG_LOAD` par un process ≠ agent (outil offensif post-élévation) → alerte `bpf_prog_load` haute ≤ 60 s | `12-bpf-chargement` |
| G15b | **Chargement eBPF tiers BLOQUÉ** — tentative de chargement par un tiers → EPERM au syscall (hook LSM `lsm/bpf`, filtre cmd=PROG_LOAD + exemption PID agent) + violation shield | `12-bpf-chargement` |
| G16 | **Volumétrie DNS** — ≥ 50 requêtes DNS vers une destination EXTERNE non exclue en 60 s → alerte `dns_volume_anomaly` medium | `13-dns-volumetrie` |
| G17 | **Exécution + filiation process** — l'execve d'un enfant porte son guid ET le parent_pid du parent (fork vu) — l'arbre de causalité complet en télémétrie DB | `14-filiation-process` |
| G18 | **Empreintes TLS** — ClientHello → event `tls_handshake` avec JA3 peuplé (et SNI si présent) | `15-tls-empreintes` |
| G19 | **Ingestion sous saturation** — tempête 600 execve (~1800 events), l'alerte critique FIM passe ≤ 60 s | `16-saturation-priorisee` |
| G20 | **Remplacement des binaires/config protégés** — truncate / unlink / rename-source **bloqués** (LSM EPERM); rename **vers** un nom protégé **détecté et alerté avec le chemin** (userspace sur le tracepoint renameat2 — le blocage destination est exclu et documenté : budget vérificateur; filet = refus du faux binaire au boot G10) | `02-rename-onto-protected` |
| G21 | **Fausse OTA rejetée ET alertée** — directive UPDATE_AGENT (artefact checksum-valide, signature ABSENTE) → installation REFUSÉE fail-closed (motif `missing_signature`) + alerte `ota_update_rejected` high portant l'URL | `03-ota-rejected` |

« — » dans la colonne de rejeu = garantie transverse : elle est
vérifiée par l'ensemble de la campagne plutôt que par une phase
unique.
## Ce que le contrat n'est pas

- Ce n'est **pas une liste exhaustive de toutes les détections** :
  c'est l'ensemble que la maison s'engage à maintenir prouvé, en
  campagne complète, à chaque re-jeu.
- Ce n'est **pas un argument marketing** : chaque ligne porte une
  preuve de laboratoire archivée, et le test est rejouable chez
  vous.
- Une ligne garantie qui régresse bloque la publication — elle ne
  peut être retirée que par une décision explicite de l'éditeur,
  documentée.

<!-- GÉNÉRÉ par scripts/export_contrat.py — NE PAS ÉDITER À LA MAIN.
     Garde anti-dérive : scripts/export_contrat.py --check -->
