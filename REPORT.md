# Rapport TP3 : CI/CD with GitHub Actions and Render

## 1. Le pipeline en quelques mots
Le pipeline mis en place dans ce TP suit un flux classique de CI/CD (Intégration et Déploiement Continus). 
Tout commence sur ma machine locale (le "Developer"). Je travaille sur une branche de fonctionnalité (feature branch) et je lance `npm run verify` pour m'assurer que le code est propre (typecheck, tests, build). 
Une fois poussé sur GitHub, je crée une Pull Request (PR) vers `main`. C'est là que GitHub Actions entre en jeu : le workflow `ci.yml` se déclenche et exécute le job "Verify" sur une machine virtuelle Ubuntu. Il installe Node 24, récupère les dépendances avec `npm ci` et lance `npm run verify`. 
Grâce au Ruleset "Protect main", la fusion de la PR est bloquée tant que le check "Verify" n'est pas vert. Une fois la PR fusionnée, Render entre en action : il attend que le check de la branche `main` soit vert (grâce à l'option "After CI Checks Pass") pour déclencher le déploiement de l'application en production.

## 2. Comparaison avec le TP2 (Coolify vs GitHub Actions + Render)
Dans le TP2, nous avions utilisé **Coolify** avec un webhook Forgejo. Le déploiement était déclenché directement par le `git push`, sans réelle phase de validation automatisée avant la mise en production (sauf si on configurait manuellement des tests dans Coolify, ce qui restait limité).
Avec **GitHub Actions + Render**, la philosophie est différente et beaucoup plus robuste :
- **GitHub Actions** apporte une vraie couche de CI (Intégration Continue). On peut exécuter des tests, du linting, du build, et bloquer la fusion si quelque chose casse.
- **Render** est uniquement la couche de CD (Déploiement Continu). Il ne déploie que ce qui a été validé par la CI.
- Le système de **Pull Request Previews** de Render est un énorme plus par rapport à Coolify : il permet de tester une version isolée de l'application avant de la fusionner, avec sa propre URL temporaire.

## 3. Difficultés rencontrées
- **Installation de Node.js 24** : Mon environnement local était en Node 20. J'ai dû utiliser `winget install OpenJS.NodeJS.LTS` pour passer à la version 24, et surtout rouvrir le terminal pour que le PATH soit mis à jour.
- **Commandes Linux vs Windows** : L'énoncé fournissait des commandes Bash (`rm -rf`, `mkdir -p`). J'ai dû les adapter pour PowerShell (`Remove-Item -Recurse -Force`, `New-Item -ItemType Directory -Force`).
- **Configuration de Render** : Render a détecté automatiquement le projet en Node, mais il a rempli des commandes par défaut incorrectes (`npm install` au lieu de `npm ci && npm run build`). Il a fallu corriger manuellement les champs Build Command et Start Command.
- **Preuves d'échec** : Il a fallu faire attention à bien laisser la PR ouverte lors du test du check "Verify" rouge (pour la capture `pr-failing-verify.png`) avant de corriger le code, pour prouver que le pipeline bloque bien les erreurs.