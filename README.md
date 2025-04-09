# Guide complet : Modifier les dates des commits Git

## Table des matières
1. [Introduction](#introduction)
2. [Comprendre les dates dans Git](#comprendre-les-dates-dans-git)
3. [Modifier le dernier commit](#modifier-le-dernier-commit)
4. [Modifier un commit spécifique](#modifier-un-commit-spécifique)
5. [Exemple pratique complet](#exemple-pratique-complet)
6. [Formats de date acceptés](#formats-de-date-acceptés)
7. [Troubleshooting](#troubleshooting)
8. [Bonnes pratiques](#bonnes-pratiques)

## Introduction

Git stocke deux dates pour chaque commit : la date d'auteur (quand le code a été écrit) et la date de commit (quand le commit a été créé). Ce guide vous montre comment modifier ces dates pour vos commits, que ce soit pour corriger une erreur ou pour organiser votre historique.

⚠️ **Attention** : Modifier l'historique des commits change leur hash. Si vous avez déjà poussé vers un dépôt distant, vous devrez utiliser `git push --force`.

## Comprendre les dates dans Git

### Les deux types de dates

- **Author Date** (`GIT_AUTHOR_DATE`) : Quand le code a été écrit originalement
- **Commit Date** (`GIT_COMMITTER_DATE`) : Quand le commit a été créé/modifié

### Visualiser les dates

```bash
# Voir les dates d'auteur
git log --pretty=format:"%h %s %ad" --date=format:"%Y-%m-%d %H:%M"

# Voir les deux dates
git log --pretty=format:"%h %s %ad (author) %cd (commit)" --date=format:"%Y-%m-%d %H:%M"

# Format plus détaillé
git log --pretty=format:"%C(yellow)%h%C(reset) %s %C(green)%ad%C(reset) %C(blue)(%an)%C(reset)" --date=format:"%Y-%m-%d %H:%M:%S"
```

## Modifier le dernier commit

### Changer les deux dates

```bash
# Modifier author date ET commit date
git commit --amend --date="2024-06-08 14:30:00"

# Avec des formats plus flexibles
git commit --amend --date="yesterday 15:30"
git commit --amend --date="2 hours ago"
git commit --amend --date="last friday 10:00"
```

### Changer seulement une date

```bash
# Changer seulement la date d'auteur
GIT_AUTHOR_DATE="2024-06-08 14:30:00" git commit --amend --no-edit

# Changer seulement la date de commit
GIT_COMMITTER_DATE="2024-06-08 14:30:00" git commit --amend --no-edit

# Changer les deux séparément
GIT_AUTHOR_DATE="2024-06-08 14:30:00" GIT_COMMITTER_DATE="2024-06-08 14:30:00" git commit --amend --no-edit
```

## Modifier un commit spécifique

### Méthode 1 : Rebase interactif (recommandée)

```bash
# Voir l'historique des commits
git log --oneline

# Exemple de sortie :
# 7d6d71f Ajout liste des tâches
# 7d7e8ff Ajout configuration
# 3e78fae Initial commit: ajout README

# Rebase interactif depuis le commit AVANT celui à modifier
git rebase -i 3e78fae^  # Pour modifier le commit 3e78fae
# ou
git rebase -i HEAD~3    # Pour les 3 derniers commits
```

Dans l'éditeur qui s'ouvre :
```
pick 3e78fae Initial commit: ajout README
pick 7d7e8ff Ajout configuration  
pick 7d6d71f Ajout liste des tâches
```

Changez `pick` en `edit` pour le(s) commit(s) à modifier :
```
edit 3e78fae Initial commit: ajout README
pick 7d7e8ff Ajout configuration  
edit 7d6d71f Ajout liste des tâches
```

Puis pour chaque commit marqué `edit` :
```bash
# Modifier les dates
GIT_COMMITTER_DATE="2024-06-07 18:23:00" git commit --amend --date="2024-06-07 18:23:00" --no-edit

# Continuer le rebase
git rebase --continue
```

### Méthode 2 : Filter-branch (pour des modifications complexes)

```bash
# Modifier un commit spécifique par son hash
git filter-branch --env-filter '
if [ $GIT_COMMIT = 7d7e8ff ]; then
    export GIT_AUTHOR_DATE="2024-06-08 10:00:00"
    export GIT_COMMITTER_DATE="2024-06-08 10:00:00"
fi
' --tag-name-filter cat -- --branches --tags
```

### Méthode 3 : Avec git-redate (outil externe)

```bash
# Installer git-redate
npm install -g git-redate

# Modifier interactivement les dates
git redate
```

## Exemple pratique complet

### Étape 1 : Créer un projet de test

```bash
# Créer le dossier et initialiser git
mkdir test-dates-git
cd test-dates-git
git init

# Créer des fichiers de test
echo "# Projet de Test" > README.md
echo "Ce projet sert à tester les modifications de dates" >> README.md

echo "# Configuration" > config.txt
echo "port=8080" >> config.txt
echo "debug=true" >> config.txt
echo "log_level=info" >> config.txt

echo "# Liste des Tâches" > todo.txt
echo "- [ ] Implémenter l'authentification" >> todo.txt
echo "- [ ] Ajouter les tests unitaires" >> todo.txt
echo "- [ ] Documenter l'API" >> todo.txt

echo "function hello() {" > script.js
echo "  console.log('Hello World!');" >> script.js
echo "}" >> script.js
```

### Étape 2 : Faire des commits avec des dates personnalisées

```bash
# Commit 1 : il y a 5 jours à 9h
git add README.md
git commit -m "docs: ajout du README initial" --date="5 days ago 09:00"

# Commit 2 : il y a 3 jours à 14h30
git add config.txt
git commit -m "config: ajout de la configuration de base" --date="3 days ago 14:30"

# Commit 3 : il y a 2 jours à 16h45
git add todo.txt
git commit -m "docs: ajout de la liste des tâches" --date="2 days ago 16:45"

# Commit 4 : hier à 11h15
git add script.js
git commit -m "feat: ajout du script principal" --date="yesterday 11:15"
```

### Étape 3 : Vérifier les dates

```bash
# Voir l'historique avec les dates
git log --pretty=format:"%C(yellow)%h%C(reset) %C(green)%ad%C(reset) %s" --date=format:"%Y-%m-%d %H:%M"

# Exemple de sortie :
# a1b2c3d 2024-06-09 11:15 feat: ajout du script principal
# b2c3d4e 2024-06-08 16:45 docs: ajout de la liste des tâches
# c3d4e5f 2024-06-07 14:30 config: ajout de la configuration de base
# d4e5f6g 2024-06-05 09:00 docs: ajout du README initial
```

### Étape 4 : Modifier un commit spécifique

Supposons qu'on veuille modifier la date du commit de configuration :

```bash
# Identifier le commit à modifier
git log --oneline

# Rebase interactif
git rebase -i d4e5f6g^  # Hash du commit AVANT celui à modifier

# Dans l'éditeur, changer "pick" en "edit" pour le commit config
# Sauvegarder et fermer

# Modifier les dates
GIT_COMMITTER_DATE="2024-06-07 10:30:00" git commit --amend --date="2024-06-07 10:30:00" --no-edit

# Continuer le rebase
git rebase --continue
```

### Étape 5 : Pousser vers GitHub

```bash
# Ajouter le remote
git remote add origin https://github.com/votre-username/test-dates-git.git

# Créer la branche main et pousser
git branch -M main
git push -u origin main

# Si vous avez modifié l'historique après avoir déjà poussé
git push --force
```

### Étape 6 : Vérifier sur GitHub

Sur GitHub, vous devriez voir vos commits avec les dates personnalisées. Note : GitHub affiche parfois la "commit date" plutôt que l'"author date" dans certaines vues.

## Formats de date acceptés

### Formats absolus

```bash
# Format ISO
git commit --amend --date="2024-06-08T14:30:00"

# Format RFC 2822
git commit --amend --date="Sat, 8 Jun 2024 14:30:00 +0200"

# Format simple
git commit --amend --date="2024-06-08 14:30:00"
git commit --amend --date="June 8 2024 2:30 PM"
```

### Formats relatifs

```bash
# Temps relatif
git commit --amend --date="2 hours ago"
git commit --amend --date="3 days ago"
git commit --amend --date="last week"
git commit --amend --date="yesterday"

# Avec heure spécifique
git commit --amend --date="yesterday 15:30"
git commit --amend --date="last friday 10:00"
git commit --amend --date="2 days ago 14:30"
```

### Formats avec fuseau horaire

```bash
# Avec fuseau horaire
git commit --amend --date="2024-06-08 14:30:00 +0200"
git commit --amend --date="2024-06-08 14:30:00 CET"
git commit --amend --date="2024-06-08 14:30:00 UTC"
```

## Troubleshooting

### Problème : GitHub affiche "X minutes ago" au lieu de ma date

**Solution** : GitHub affiche la "commit date". Vous devez modifier les deux dates :

```bash
GIT_COMMITTER_DATE="2024-06-08 14:30:00" git commit --amend --date="2024-06-08 14:30:00" --no-edit
```

### Problème : Erreur "fatal: bad revision"

**Solution** : Vérifiez que le hash du commit existe :

```bash
git log --oneline  # Voir tous les commits
git show HASH      # Vérifier qu'un commit existe
```

### Problème : Conflit lors du rebase

**Solution** :

```bash
# Résoudre les conflits manuellement, puis :
git add .
git rebase --continue

# Ou abandonner le rebase :
git rebase --abort
```

### Problème : Filter-branch ne fonctionne pas

**Solution** : Nettoyer l'historique filter-branch :

```bash
rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

## Bonnes pratiques

### 1. Sauvegardez avant de modifier

```bash
# Créer une branche de sauvegarde
git branch backup-before-date-change

# Ou créer un tag
git tag backup-$(date +%Y%m%d) HEAD
```

### 2. Testez sur une copie locale

```bash
# Cloner votre repo dans un dossier temporaire
git clone /path/to/your/repo /tmp/test-repo
cd /tmp/test-repo

# Tester vos modifications
# Si tout va bien, appliquer sur le vrai repo
```

### 3. Coordonnez avec votre équipe

Si vous travaillez en équipe :
- Prévenez avant de modifier l'historique
- Assurez-vous que personne n'a de travail en cours sur les commits modifiés
- Documentez les changements

### 4. Utilisez des messages de commit clairs

```bash
# Bon format de message
git commit -m "fix(date): correct commit date for configuration setup"

# Avec plus de détails
git commit -m "fix(date): correct commit date for configuration setup

- Original date was incorrect due to timezone issue
- Should have been committed on 2024-06-07 at 10:30
- Affects configuration setup milestone"
```

### 5. Vérifiez après modification

```bash
# Vérifier que tout est correct
git log --pretty=format:"%h %s %ad %cd" --date=format:"%Y-%m-%d %H:%M"

# Vérifier l'intégrité du repo
git fsck --full

# Vérifier que GitHub affiche correctement
git push --dry-run  # Simuler le push
```

## Scripts utiles

### Script pour modifier plusieurs commits

```bash
#!/bin/bash
# modify-dates.sh

# Usage: ./modify-dates.sh

commits=(
    "a1b2c3d:2024-06-05 09:00:00"
    "b2c3d4e:2024-06-07 14:30:00"
    "c3d4e5f:2024-06-08 16:45:00"
)

git filter-branch --env-filter '
for commit_info in "${commits[@]}"; do
    hash="${commit_info%%:*}"
    date="${commit_info##*:}"
    
    if [ $GIT_COMMIT = $hash ]; then
        export GIT_AUTHOR_DATE="$date"
        export GIT_COMMITTER_DATE="$date"
    fi
done
' --tag-name-filter cat -- --branches --tags
```

### Script de vérification

```bash
#!/bin/bash
# check-dates.sh

echo "=== Vérification des dates des commits ==="
git log --pretty=format:"%C(yellow)%h%C(reset) %C(green)%ad%C(reset) %C(blue)%cd%C(reset) %s" --date=format:"%Y-%m-%d %H:%M:%S" -10

echo -e "\n=== Légende ==="
echo "Jaune: Hash du commit"
echo "Vert: Date d'auteur"
echo "Bleu: Date de commit"
```

---

## Conclusion

Modifier les dates des commits Git est un outil puissant pour organiser votre historique de développement. Utilisez ces techniques avec précaution, surtout sur des projets partagés, et n'oubliez pas de tester vos modifications avant de les appliquer définitivement.

Pour plus d'informations sur Git, consultez la [documentation officielle](https://git-scm.com/docs) ou utilisez `git help <command>` pour obtenir de l'aide sur une commande spécifique.