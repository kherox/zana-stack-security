# Opérer la plateforme

Cette section s'adresse aux **exploitants SOC** : faire vivre la
plateforme au quotidien, vérifier qu'elle tient ses promesses, et
savoir ce qu'elle fait — et ne fait pas — quand quelque chose se passe
mal.

- [Exploitation au quotidien](exploitation.md) — diagnostic, mise à jour
  des agents, dépannage courant, états dégradés assumés.
- [Contrat de couverture](contrat-couverture.md) — les lignes de
  détection **garanties**, rejouables sur votre installation avec
  `janus-ctl gate contract`.

## Les trois principes d'exploitation

1. **Preuve avant affirmation.** Une intervention se valide par un test
   rejouable (`janus-ctl gate contract`, `janus-ctl doctor`) — jamais
   par un « ça devrait marcher ». Un verdict vert archivé vaut mieux
   qu'une conviction.
2. **Mise à jour signée, ou rien.** Un agent n'accepte qu'un pack signé ;
   un pack falsifié est refusé avant l'arrêt du service, et un binaire
   non conforme refuse de démarrer. La mise à jour est donc toujours un
   acte sûr, jamais un pari.
3. **Deux paires de mains sur le disruptif.** L'isolation d'une machine
   ou le blocage d'une IP exigent une validation à double verrou et sont
   intégralement journalisés. La plateforme peut se défendre ; elle ne
   se lance pas toute seule.
