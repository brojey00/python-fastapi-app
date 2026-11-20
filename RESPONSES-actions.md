# Réponses – TP 7 : Actions tierces et actions personnalisées

## Partie 1 – Actions tierces (`uses` et `with`)

### Q1. Pourquoi `actions/checkout` n'a-t-il pas besoin de `with` alors que `setup-node` en a besoin ?

`actions/checkout` fonctionne avec des valeurs par défaut suffisantes : il récupère le
dépôt courant, à la référence qui a déclenché le workflow, avec le token par défaut.
Tous ses paramètres (`ref`, `path`, `fetch-depth`, etc.) sont optionnels.

`actions/setup-node`, lui, doit savoir quelle version de Node.js installer. Sans
`node-version`, aucune version précise n'est demandée et le résultat dépend de ce qui est
déjà présent sur le runner. Le mot-clé `with` sert donc à passer les paramètres d'entrée
(inputs) de l'action, obligatoires ou non selon l'action.

### Q2. Pourquoi le job `verify` ne retrouve-t-il pas `report.txt` sans l'artefact, alors que les deux jobs partagent le même dépôt ?

Chaque job s'exécute sur un runner distinct et neuf (machine virtuelle propre), avec son
propre système de fichiers. Le dépôt n'est pas « partagé » : chaque job doit le récupérer
lui-même (checkout). Or `report.txt` est un fichier généré pendant le job `build` : il
n'est pas commité dans le dépôt, donc il n'existe que sur le runner de `build`, qui est
détruit à la fin du job. Pour transmettre un fichier d'un job à l'autre, il faut passer par
un artefact (`upload-artifact` puis `download-artifact`).

## Partie 3 – Action composite locale

### Q6. Pourquoi `actions/checkout` doit-il s'exécuter avant l'appel d'une action locale ?

Une action locale (`uses: ./.github/actions/build-info`) est un dossier qui se trouve dans
le dépôt. Sur un runner neuf, le dépôt n'est pas encore présent : sans checkout, le chemin
`./.github/actions/build-info` n'existe pas et le workflow échoue (`action.yml`
introuvable). Le checkout télécharge le code, ce qui rend l'action locale disponible.

### Q7. Pourquoi `shell: bash` est-il obligatoire sur les steps `run` d'une action composite ?

Dans un workflow classique, le shell par défaut est défini par le runner. Une action
composite est conçue pour être réutilisée par n'importe quel workflow, sur n'importe quel
runner, et ne connaît pas à l'avance l'environnement qui l'appellera. GitHub impose donc de
déclarer explicitement le shell sur chaque step `run`, afin que son comportement soit
identique et prévisible partout.
