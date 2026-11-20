# Security Notes – TP 7

## Relevés sur le dépôt `actions/setup-node`

Dépôt consulté : https://github.com/actions/setup-node

| Élément relevé | Valeur |
|---|---|
| Badge de créateur vérifié | Oui : l'action est publiée par l'organisation officielle `actions` (GitHub), badge « Verified creator » visible sur sa page du Marketplace |
| Nombre d'étoiles | <5k ⭐ Star> |
| Dossier contenant le code source | `src/` (le code compilé réellement exécuté par le runner est dans `dist/`) |

## Partie 2 – Questions

### Q3. Pourquoi faut-il épingler une version (`@v4`) plutôt que `@main` ?

`@main` pointe vers la dernière version de la branche principale, qui change à chaque
commit de l'auteur. Le workflow n'est alors plus reproductible : un changement (ou un bug)
peut casser la CI du jour au lendemain, sans qu'on ait modifié notre code.

C'est aussi un risque de sécurité (supply chain) : si le dépôt de l'action est compromis,
du code malveillant poussé sur `main` s'exécuterait automatiquement dans nos pipelines, avec
accès à nos secrets. Une version épinglée (`@v4`, et mieux, un SHA de commit complet)
garantit qu'on exécute un code précis et déjà connu.

### Q4. Le badge « créateur vérifié » garantit-il qu'une action est sûre ?

Non. Le badge indique seulement que l'identité de l'organisation éditrice a été vérifiée
par GitHub. Il ne dit rien sur la qualité ni sur la sécurité du code : une action d'un
créateur vérifié peut contenir une vulnérabilité ou un bug, ou être compromise (vol de
compte, dépendance malveillante). Il faut donc quand même relire le code, épingler la
version et limiter les permissions.

### Q5. Deux vérifications avant d'utiliser, en production, une action d'un fournisseur inconnu

1. **Auditer le code source** : lire l'action (`action.yml`, scripts, Dockerfile) et ses
   dépendances pour vérifier qu'elle ne fait pas d'appels réseau suspects et n'exfiltre pas
   de secrets, puis épingler un SHA de commit précis.
2. **Évaluer la fiabilité du projet** : ancienneté, nombre d'étoiles, activité du dépôt,
   mainteneurs, issues et pull requests ouvertes, licence. On limite aussi ses permissions
   (`permissions:` minimales sur le `GITHUB_TOKEN`) et on ne lui donne que les secrets
   strictement nécessaires.
