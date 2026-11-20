# Réponses TP 6 : Variables d'environnement, Inputs et Outputs

## Q1. Pourquoi la variable n'est-elle pas disponible dans la step qui l'a définie ?

Le `echo "APP_NAME=my-app" >> $GITHUB_ENV` écrit seulement dans un fichier. Il ne modifie pas l'environnement du processus en cours. Le runner lit ce fichier **à la fin de la step**, puis injecte les variables dans les **steps suivantes**. Dans la step qui la définit, la variable n'existe donc pas encore.

## Q2. Que constate-t-on dans le job `verify-env` ? Quelle est la portée de `GITHUB_ENV` ?

`$APP_NAME` s'affiche **vide** dans `verify-env`. Chaque job s'exécute sur un **runner différent**, et le fichier `GITHUB_ENV` n'existe que sur le runner du job qui l'a écrit.

**Portée de `GITHUB_ENV` :** uniquement les steps suivantes du **même job**.

## Q3. Que se passe-t-il avec `>` au lieu de `>>` dans `GITHUB_OUTPUT` (deux outputs dans la même step) ?

`>` **écrase** le fichier à chaque écriture. Seul le dernier output est conservé, et le premier est perdu : il est vide dans le job suivant. Avec `>>`, chaque ligne est **ajoutée** à la suite, donc les deux outputs sont conservés.

## Q4. Pourquoi `GITHUB_ENV` ne suffit-il pas entre deux jobs, alors que `GITHUB_OUTPUT` fonctionne ?

`GITHUB_ENV` est un fichier local à un runner, détruit à la fin du job. Un autre job tourne sur une autre machine et ne le voit pas.

`GITHUB_OUTPUT` fonctionne parce que les valeurs sont **envoyées au service GitHub Actions** via `outputs:` du job. Les jobs dépendants les récupèrent ensuite avec `needs.<job>.outputs.<nom>`.

## Q5. Que se passe-t-il si on retire `needs: build` du job `deploy` ?

Deux conséquences :

1. `deploy` et `build` démarrent **en parallèle**, donc `deploy` n'attend plus la fin de `build`.
2. `needs.build.outputs.full_version` n'est plus accessible, donc la valeur affichée est **vide**.