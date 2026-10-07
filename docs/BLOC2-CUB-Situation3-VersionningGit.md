# CONTEXTE CUB — BLOC 2 – Situation 3 - Administration des systèmes

*Crys 901 Boisseau*

## Table des matières

- [Partie 1 – Installation et configuration de Git](#partie-1--installation-et-configuration-de-git)
- [Partie 2 – Création d'un premier dépôt Git](#partie-2--création-dun-premier-dépôt-git)
- [Partie 3 – Création des premières versions du script](#partie-3--création-des-premières-versions-du-script)
- [Partie 4 – Modification et suivi d'un fichier](#partie-4--modification-et-suivi-dun-fichier)
- [Partie 5 – Comprendre le fonctionnement de Git](#partie-5--comprendre-le-fonctionnement-de-git)
- [Partie 6 – Annulation d'une modification non validée](#partie-6--annulation-dune-modification-non-validée)
- [Partie 7 – Découverte de GitHub](#partie-7--découverte-de-github)
- [Partie 8 – Relier le dépôt local au dépôt GitHub](#partie-8--relier-le-dépôt-local-au-dépôt-github)
- [Partie 9 – Modification et synchronisation avec GitHub](#partie-9--modification-et-synchronisation-avec-github)
- [Partie 10 – Récupération et mise à jour d'un dépôt](#partie-10--récupération-et-mise-à-jour-dun-dépôt)
- [Partie 11 – Extension : travail avec des branches](#partie-11--extension--travail-avec-des-branches)
- [Partie 12 – Modifier le projet dans une branche](#partie-12--modifier-le-projet-dans-une-branche)
- [Partie 13 – Fusion d'une branche](#partie-13--fusion-dune-branche)
- [Partie 14 – Publier une branche et utiliser une Pull Request](#partie-14--publier-une-branche-et-utiliser-une-pull-request)
- [Partie 15 – Extension : comprendre un conflit de fusion](#partie-15--extension--comprendre-un-conflit-de-fusion)
- [Partie 16 – Synthèse](#partie-16--synthèse)

---

## Partie 1 – Installation et configuration de Git :

### 1. Installer la version Windows de Git sur votre poste ou serveur d'administration :

- Vérifier que Git Bash, Git GUI et Git CMD sont installés et disponibles.
- Ouvrir Git Bash puis vérifier la version installée : `git --version`

**Question 1 : Relever la version de Git installée.**

```
UCRT64:/c/Users/Administrateur

ADM-SRV-WAC@ServeurWAC2 UCRT64 ~
$ git --version
git version 2.56.0.windows.1
```

**Question 2 : Quel est le rôle d'un logiciel de gestion de versions tel que Git ?**

Un logiciel de gestion de versions comme Git sert à enregistrer chaque modification apportée à un ensemble de fichiers au fil du temps pour retrouver facilement une version antérieure.

### 2. Vérifier puis configurer l'identité utilisée par Git :

```bash
git config --global user.name
git config --global user.email
```

Si nécessaire, configurer votre identité :

```bash
git config --global user.name "Prenom Nom"
git config --global user.email "adresse@exemple.fr"
git config --global --list
```

```
UCRT64:/c/Users/Administrateur

ADM-SRV-WAC@ServeurWAC2 UCRT64 ~
$ git config --global --list
user.name=Crys Boisseau
user.email=crys.boisseau@gmail.com
```

**Question 3 : Pourquoi Git associe-t-il un nom et une adresse électronique aux commits ?**

Afin d'identifier de façon claire l'auteur des modifications et permettre le suivi du code au sein d'une équipe.

---

## Partie 2 – Création d'un premier dépôt Git :

### 3. Créer le dossier de travail C:\CUB\Scripts :

Création du dossier `Scripts` dans `C:\CUB\`.

### 4. Créer dans ce dossier un fichier « diagnostic-reseau.ps1 » contenant :

```powershell
Write-host "Diagnostic réseau CUB"
hostname
Get-Date
Get-NetIPConfiguration
```

### 5. Depuis Git Bash, se placer dans le dossier puis initialiser le dépôt :

```bash
cd /c/CUB/Scripts
git init
git status
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 ~
$ git config --global --list
user.name=Crys Boisseau
user.email=crys.boisseau@gmail.com

ADM-SRV-WAC@ServeurWAC2 UCRT64 ~
$ cd /c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts
$ git init
Initialized empty Git repository in C:/CUB/Scripts/.git/

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        diagnostic-reseau.ps1

nothing added to commit but untracked files present (use "git add" to track)
```

**Question 4 : Que signifie l'initialisation d'un dépôt Git ?**

Transformer un dossier ordinaire sur un ordinateur en un projet suivi par Git pour enregistrer l'historique des modifications.

**Question 5 : Quel est le rôle de la commande « git status » ?**

Permet d'afficher l'état actuel du répertoire de travail et de la zone de préparation.

---

## Partie 3 – Création des premières versions du script :

### 6. Ajouter le fichier au suivi de Git puis vérifier son état :

```bash
git add diagnostic-reseau.ps1
git status
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git add diagnostic-reseau.ps1

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   diagnostic-reseau.ps1
```

**Question 6 : Quelle différence observez-vous dans « git status » avant et après la commande « git add » ?**

Avant le `add`, le status nous prévient des fichiers qui n'étaient pas traqués.

Désormais, grâce au `add`, la commande status détecte bien que le fichier est traqué et prêt à être commité.

### 7. Créer le premier commit et afficher l'historique :

```bash
git commit -m "Création du script de diagnostic réseau"
git log
git log --oneline
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git commit -m "Création du script de diagnostic réseau"
[master (root-commit) 8bdf457] Création du script de diagnostic réseau
 1 file changed, 4 insertions(+)
 create mode 100644 diagnostic-reseau.ps1

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git log
commit 8bdf45735d9c3959fbf1162f9d7801bb7254eba9 (HEAD -> master)
Author: Crys Boisseau <crys.boisseau@gmail.com>
Date:   Wed Sep 30 11:21:59 2026 +0200

    Création du script de diagnostic réseau

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git log --oneline
8bdf457 (HEAD -> master) Création du script de diagnostic réseau
```

**Question 7 : Qu'est-ce qu'un commit ?**

Un commit est un instantané de l'état d'un projet à un moment précis. Il fonctionne comme un point de sauvegarde permanent dans l'historique de votre travail.

**Question 8 : Pourquoi faut-il utiliser un message de commit précis et explicite ?**

Afin de maintenir un projet sain, collaboratif et facile à maintenir. Un historique de commit clair sert de documentation.

---

## Partie 4 – Modification et suivi d'un fichier :

### 8. Modifier « diagnostic-reseau.ps1 » en ajoutant :

```powershell
Write-Host "Test de la pile TCP/IP"
Ping 127.0.0.1
```

### 9. Vérifier les modifications :

```bash
git status
git diff
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   diagnostic-reseau.ps1

no changes added to commit (use "git add" and/or "git commit -a")

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git diff
diff --git a/diagnostic-reseau.ps1 b/diagnostic-reseau.ps1
index 2978dc6..378040e 100644
--- a/diagnostic-reseau.ps1
+++ b/diagnostic-reseau.ps1
@@ -1,4 +1,7 @@
 Write-host "Diagnostic réseau CUB"
 hostname
 Get-Date
-Get-NetIPConfiguration
\ No newline at end of file
+Get-NetIPConfiguration
+
+Write-Host "Test de la pile TCP/IP"
+Ping 127.0.0.1
\ No newline at end of file
```

**Question 9 : Que permet d'afficher la commande « git diff » ?**

Permet d'afficher les différences entre différents états d'un projet dans un dépôt.

### 10. Enregistrer cette nouvelle version :

```bash
git add diagnostic-reseau.ps1
git commit -m "Ajout du test TCP IP Local"
git log --oneline
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git add diagnostic-reseau.ps1

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git commit -m "Ajout du test TCP IP Local"
[master 7433ae4] Ajout du test TCP IP Local
 1 file changed, 4 insertions(+), 1 deletion(-)

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git log --oneline
7433ae4 (HEAD -> master) Ajout du test TCP IP Local
8bdf457 Création du script de diagnostic réseau
```

**Question 10 : Combien de versions du projet sont maintenant présentes dans l'historique ?**

Il y a désormais 2 versions du projet présentes dans l'historique.

### 11. Réaliser une troisième modification en ajoutant :

```powershell
Write-Host "Affichage des serveurs DNS"
Get-DnsClientServerAddress
```

```bash
git add diagnostic-reseau.ps1
git commit -m "Ajout du diagnostic DNS"
git log --oneline
```

```
ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git add diagnostic-reseau.ps1

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git commit -m "Ajout Affichage des serveurs DNS"
[master 06a0ebd] Ajout Affichage des serveurs DNS
 1 file changed, 7 insertions(+), 1 deletion(-)

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git log --oneline
06a0ebd (HEAD -> master) Ajout Affichage des serveurs DNS
7433ae4 Ajout du test TCP IP Local
8bdf457 Création du script de diagnostic réseau
```

---

## Partie 5 – Comprendre le fonctionnement de Git :

Le cycle de versionnement utilisé peut être résumé ainsi :

**Fichier modifié → `git add` → zone d'indexation → `git commit` → dépôt Git local**

**Question 11 : Indiquer le rôle de chacune des commandes : git status, git diff, git add, git commit et git log :**

| Commandes | Rôles |
|---|---|
| `git status` | Affiche l'état actuel de la copie de travail. |
| `git diff` | Montre le détail ligne par ligne des modifications apportées aux fichiers qui ne sont pas encore indexés. |
| `git add` | Ajoute les modifications du répertoire de travail à la zone de transit. |
| `git commit` | Enregistre de manière définitive les modifications présentes dans la zone de transit dans l'historique du projet, en y associant un message explicatif. |
| `git log` | Affiche la liste chronologique des commits effectués, avec leur identifiant unique, leur auteur, la date et le message associé. |

---

## Partie 6 – Annulation d'une modification non validée :

### 12. Ajouter volontairement la ligne suivante dans le script sans créer de commit :

```powershell
Write-Host "Test INCORRECT"
```

### 13. Vérifier la modification puis restaurer la dernière version validée :

```bash
git diff
git restore diagnostic-reseau.ps1
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git diff
diff --git a/diagnostic-reseau.ps1 b/diagnostic-reseau.ps1
index ceeef17..c5a617e 100644
--- a/diagnostic-reseau.ps1
+++ b/diagnostic-reseau.ps1
@@ -10,4 +10,6 @@ Write-Host "Affichage des serveurs DNS"
 Get-DnsClientServerAddress
 Git add diagnostic-reseau.ps1
 Git commit -m "Ajout du diagnostic DNS"
-Git log --oneline
\ No newline at end of file
+Git log --oneline
+
+Write-Host "Test INCORRECT"
\ No newline at end of file

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git restore diagnostic-reseau.ps1
```

**Question 12 : Quel est l'intérêt de la commande « git restore » ?**

Elle revient sur une version enregistrée sur un projet.

**Question 13 : La modification supprimée avait-elle déjà été enregistrée dans un commit ? Justifier :**

Non, la modification avait été ajoutée sans faire de commit ensuite, sinon la commande « git restore » n'aurait rien changé au fichier « diagnostic-reseau.ps1 ».

---

## Partie 7 – Découverte de GitHub :

### 14. Se connecter à Github selon les modalités indiquées par l'enseignant :

**Question 14 : Peut-on utiliser Git sans GitHub ? Justifier :**

Oui, il est possible d'utiliser Git sans GitHub, car Git est un logiciel de gestion de versions local qui fonctionne de manière autonome sur sa propre machine, tandis que GitHub n'est qu'un service d'hébergement en ligne optionnel.

**Question 15 : Quelle différence faites-vous entre Git et GitHub ?**

Git est un logiciel de gestion de versions installé sur son ordinateur, tandis que GitHub est un site web qui héberge nos projets en ligne et facilite le travail en équipe.

### 15. Créer un nouveau dépôt GitHub nommé « CUB-Scripts-Administration » et choisir sa visibilité selon les consignes de l'enseignant :

Dépôt créé : `crysboisseau / CUB-Scripts-Administration` (Public).

**Question 16 : Quelle différence existe-t-il entre un dépôt public et un dépôt privé ?**

Le dépôt public peut être vu et utilisé par n'importe qui, pendant que le dépôt privé n'est visible et utilisable que par un nombre défini et limité de personnes.

**Question 17 : Pourquoi un administrateur systèmes doit-il être vigilant avant de publier un script sur un dépôt public ?**

L'administrateur doit être vigilant car il pourrait avoir des informations sensibles à ne pas divulguer au public sur le script.

**Question 18 : Donner trois exemples d'informations qui ne doivent pas être publiées dans un dépôt GitHub :**

- Mots de passe, clés cryptographiques, tokens, certificats.
- Données personnelles.
- Fichiers de configuration locale et variables d'environnement.

---

## Partie 8 – Relier le dépôt local au dépôt GitHub :

### 16. Vérifier les dépôts distants déjà configurés :

```bash
git remote -v
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git remote -v

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$
```

### 17. Associer le dépôt local au dépôt GitHub créé précédemment :

```bash
git remote add origin https://github.com/crysboisseau/CUB-Scripts-Administration.git
git remote -v
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git remote add origin https://github.com/crysboisseau/CUB-Scripts-Administration.git

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git remote -v
origin  https://github.com/crysboisseau/CUB-Scripts-Administration.git (fetch)
origin  https://github.com/crysboisseau/CUB-Scripts-Administration.git (push)

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$
```

**Question 19 : Que représente le nom « origin » ?**

Le nom par défaut (un alias ou un raccourci) attribué au dépôt distant (remote repository) depuis lequel un projet a été cloné ou vers lequel il est relié.

### 18. Vérifier le nom de la branche principale et, si nécessaire, la renommer « main » :

```bash
git branch
git branch -M main
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git branch
* master

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (master)
$ git branch -M main
```

### 19. Publier le dépôt local sur GitHub :

**Question 20 : Quelle commande permet d'envoyer les commits locaux vers GitHub ?**

La commande `git push`.

**Question 21 : Quelle différence existe-t-il entre « git commit » et « git push » ?**

La différence principale est que `git commit` enregistre votre travail sur votre ordinateur, tandis que `git push` l'envoie vers un serveur en ligne.

---

## Partie 9 – Modification et synchronisation avec GitHub :

### 20. Ajouter au script un test de la passerelle en adaptant l'adresse à votre environnement :

```powershell
Write-Host "Test de la passerelle"
Test-NetConnection 192.168.2.126
```

### 21. Créer un nouveau commit :

```bash
git status
git diff
git add diagnostic-reseau.ps1
git commit -m "Ajout du test de la passerelle"
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   diagnostic-reseau.ps1

no changes added to commit (use "git add" and/or "git commit -a")

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git diff
diff --git a/diagnostic-reseau.ps1 b/diagnostic-reseau.ps1
index ceeef17..ad47f3b 100644
--- a/diagnostic-reseau.ps1
+++ b/diagnostic-reseau.ps1
@@ -10,4 +10,7 @@ Write-Host "Affichage des serveurs DNS"
 Get-DnsClientServerAddress
 Git add diagnostic-reseau.ps1
 Git commit -m "Ajout du diagnostic DNS"
-Git log --oneline
\ No newline at end of file
+Git log --oneline
+
+Write-Host "Test de la passerelle"
+Test-NetConnection 192.168.2.126
\ No newline at end of file

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git add diagnostic-reseau.ps1

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git commit -m "Ajout du test de la passerelle"
[main fb31824] Ajout du test de la passerelle
 1 file changed, 4 insertions(+), 1 deletion(-)
```

**Question 22 : Avant d'exécuter « git push », cette modification est-elle déjà visible sur GitHub ? Pourquoi ?**

Non, `git commit` enregistre sur son ordinateur, `git push` enregistre sur un dépôt distant.

### 22. Publier la modification :

```bash
git push
git push --set-upstream origin main
```

Une fenêtre va s'ouvrir demandant les identifiants du compte étant propriétaire ou ayant les droits sur le dépôt.

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git push
fatal: The current branch main has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin main

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git push --set-upstream origin main
info: please complete authentication in your browser...
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 4 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (12/12), 1.20 KiB | 411.00 KiB/s, done.
Total 12 (delta 3), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (3/3), done.
To https://github.com/crysboisseau/CUB-Scripts-Administration.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

---

## Partie 10 – Récupération et mise à jour d'un dépôt :

### 23. Découvrir la commande permettant à un nouvel administrateur de récupérer un projet existant :

```bash
git clone https://github.com/crysboisseau/CUB-Scripts-Administration.git
```

```
etudiant@S406-06:~$ cd Documents/git/test/
etudiant@S406-06:~/Documents/git/test$ git clone https://github.com/crysboisseau/CUB-Scripts-Administration.git
Clonage dans 'CUB-Scripts-Administration'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 12 (delta 3), reused 12 (delta 3), pack-reused 0 (from 0)
Réception d'objets: 100% (12/12), fait.
Résolution des deltas: 100% (3/3), fait.
etudiant@S406-06:~/Documents/git/test$
```

**Question 23 : Quelle différence faites-vous entre « git init » et « git clone » ?**

`git init` crée un dépôt Git local entièrement vide ou initialise le suivi sur un dossier existant, tandis que `git clone` télécharge un projet déjà existant depuis un serveur distant avec tout son historique.

### 24. Découvrir la commande permettant de récupérer les nouvelles modifications publiées par un collègue :

```bash
git pull
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git pull
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Unpacking objects: 100% (3/3), 950 bytes | 47.00 KiB/s, done.
From https://github.com/crysboisseau/CUB-Scripts-Administration
   fb31824..5b97343  main       -> origin/main
Updating fb31824..5b97343
Fast-forward
 ptitfichefiche.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 ptitfichefiche.txt
```

**Question 24 : Pourquoi est-il important de récupérer les dernières modifications avant de commencer à travailler sur un projet partagé ?**

Il est essentiel de récupérer les dernières modifications (via un `git pull` ou un `fetch` suivi d'un `merge` par exemple) avant de commencer à travailler pour éviter les conflits de fusion (merge conflicts) et s'assurer que vous développez sur la version la plus récente et stable du projet.

---

## Partie 11 – Extension : travail avec des branches :

### 25. Afficher les branches du dépôt :

```bash
git branch
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git branch
* main
```

### 26. Créer puis activer une branche dédiée au diagnostic DNS :

```bash
git switch -c feature-diagnostic-dns
git branch
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-diagnostic-dns)
$ git switch -c feature-diagnostic-dns
fatal: a branch named 'feature-diagnostic-dns' already exists

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-diagnostic-dns)
$ git branch
* feature-diagnostic-dns
  main
```

**Question 25 : Comment Git indique-t-il la branche actuellement utilisée ?**

Git indique la branche actuellement utilisée en déplaçant un pointeur spécial appelé **HEAD** sur cette branche.

**Question 26 : Quel est l'intérêt de travailler dans une branche plutôt que directement dans « main » ?**

Travailler dans une branche dédiée plutôt que directement dans la branche principale (**main** ou **master**) est une bonne pratique fondamentale en développement informatique. Cela permet principalement d'**isoler vos modifications** pour ne pas perturber le code qui fonctionne déjà.

---

## Partie 12 – Modifier le projet dans une branche :

### 27. Dans la branche « feature-diagnostic-dns », ajouter au script :

```powershell
Write-Host "Test de résolution DNS"
Resolve-DnsName www.example.com
```

### 28. Valider la modification dans cette branche :

```bash
git add diagnostic-reseau.ps1
git commit -m "Ajout du test de résolution DNS"
git log --oneline --graph --all
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-diagnostic-dns)
$ git add diagnostic-reseau.ps1

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-diagnostic-dns)
$ git commit -m "Ajout du test de résolution DNS"
On branch feature-diagnostic-dns
nothing to commit, working tree clean

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-diagnostic-dns)
$ git log --oneline --graph --all
* e2b3e9a (HEAD -> feature-diagnostic-dns) Ajout du test de résolution DNS
* 5b97343 (origin/main, origin/HEAD, main) Add hello to ptitfichefiche.txt
* fb31824 Ajout du test de la passerelle
* 06a0ebd Ajout Affichage des serveurs DNS
* 7433ae4 Ajout du test TCP IP Local
* 8bdf457 Création du script de diagnostic réseau
```

### 29. Revenir sur la branche principale :

```bash
git switch main
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-diagnostic-dns)
$ git switch main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$
```

**Question 27 : La modification réalisée dans « feature-diagnostic-dns » est-elle présente dans « main » ? Pourquoi ?**

Elle est présente dans main, car le fichier modifié est présent dans main également.

---

## Partie 13 – Fusion d'une branche :

### 30. Depuis la branche « main », fusionner la branche de travail :

```bash
git merge feature-diagnostic-dns
git log --oneline --graph --all
```

```
ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git merge feature-diagnostic-dns
Updating 5b97343..e2b3e9a
Fast-forward
 diagnostic-reseau.ps1 | 5 ++++-
 1 file changed, 4 insertions(+), 1 deletion(-)

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git log --oneline --graph --all
* e2b3e9a (HEAD -> main, feature-diagnostic-dns) Ajout du test de résolution DNS
* 5b97343 (origin/main, origin/HEAD) Add hello to ptitfichefiche.txt
* fb31824 Ajout du test de la passerelle
* 06a0ebd Ajout Affichage des serveurs DNS
* 7433ae4 Ajout du test TCP IP Local
* 8bdf457 Création du script de diagnostic réseau
```

**Question 28 : Quel est le rôle de la commande « git merge » ?**

La commande `git merge` permet de combiner l'historique de deux branches en une seule, en intégrant les modifications d'une branche source dans votre branche courante.

### 31. Publier la branche principale mise à jour :

```bash
git push
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 419 bytes | 419.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/crysboisseau/CUB-Scripts-Administration.git
   5b97343..e2b3e9a  main -> main
```

---

## Partie 14 – Publier une branche et utiliser une Pull Request :

### 32. Créer une nouvelle branche :

```bash
git switch -c feature-informations-systeme
```

```
ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (main)
$ git switch -c feature-informations-systeme
Switched to a new branch 'feature-informations-systeme'

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-informations-systeme)
$
```

### 33. Ajouter au script :

```powershell
Write-Host "Informations système"
Get-ComputerInfo
```

### 34. Créer un commit puis publier la branche :

```bash
git add diagnostic-reseau.ps1
git commit -m "Ajout des informations système"
git push -u origin feature-informations-systeme
```

```
UCRT64:/c/CUB/Scripts

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-informations-systeme)
$ git add diagnostic-reseau.ps1

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-informations-systeme)
$ git commit -m "Ajout des informations système"
[feature-informations-systeme 18fb657] Ajout des informations système
 1 file changed, 4 insertions(+), 1 deletion(-)

ADM-SRV-WAC@ServeurWAC2 UCRT64 /c/CUB/Scripts (feature-informations-systeme)
$ git push -u origin feature-informations-systeme
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 397 bytes | 397.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
remote:
remote: Create a pull request for 'feature-informations-systeme' on GitHub by visiting:
remote:      https://github.com/crysboisseau/CUB-Scripts-Administration/pull/new/feature-informations-systeme
remote:
To https://github.com/crysboisseau/CUB-Scripts-Administration.git
 * [new branch]      feature-informations-systeme -> feature-informations-systeme
branch 'feature-informations-systeme' set up to track 'origin/feature-informations-systeme'.
```

**Question 29 : La branche apparaît-elle maintenant sur GitHub ?**

Oui.

**Question 30 : Quelle différence existe-t-il entre une branche uniquement locale et une branche publiée sur GitHub ?**

La principale différence entre une branche uniquement locale et une branche publiée sur GitHub réside dans leur **lieu de stockage**, leur **accessibilité** et leur **capacité de collaboration**.

### 35. Depuis GitHub, créer une Pull Request proposant l'intégration de « feature-informations-systeme » dans « main ».

- Consulter les fichiers et les lignes modifiés.
- Vérifier le contenu de la modification.
- Fusionner uniquement selon les consignes de l'enseignant.

**Question 31 : Quel est l'intérêt d'une Pull Request ?**

Une Pull Request (PR) – aussi appelée Merge Request sur GitLab – est un espace de discussion et de proposition pour intégrer des modifications de code d'une branche vers une branche principale.

**Question 32 : Pourquoi est-il préférable de faire vérifier une modification avant son intégration dans « main » ?**

Il est préférable de faire vérifier une modification avant son intégration dans la branche principale (« main ») pour garantir la **stabilité, la sécurité et la qualité du code**. Cette pratique, souvent formalisée par des Pull Requests ou Merge Requests, sert de filtre de sécurité pour l'ensemble du projet.

---

## Partie 15 – Extension : comprendre un conflit de fusion :

Lorsque deux personnes modifient la même partie d'un fichier, Git peut ne pas être capable de déterminer automatiquement quelle version conserver. Il signale alors un conflit de fusion.

```
<<<<<<< HEAD
Write-Host "Diagnostic réseau CUB"
=======
Write-Host "Diagnostic réseau avancé CUB"
>>>>>>> feature-affichage
```

**Question 33 : Pourquoi Git ne choisit-il pas automatiquement une des deux versions ?**

Git ne choisit pas automatiquement entre les deux versions car il ne peut pas deviner l'intention logique du développeur ni savoir quel choix est fonctionnellement correct pour le programme.

**Question 34 : Qui doit décider du contenu à conserver ?**

C'est au développeur (ou à l'équipe de développement) qui effectue la fusion de décider du contenu à conserver.

**Question 35 : Après correction manuelle du conflit, quelles opérations Git permettent d'enregistrer la résolution ?**

Pour enregistrer la résolution, vous devez exécuter deux commandes successives :

- `git add <fichier.extension>` : cette commande indexe le fichier et indique à Git que le conflit est résolu.
- `git commit` : cette commande crée le commit de fusion et enregistre définitivement les modifications.

---

## Partie 16 – Synthèse :

### 36. Compléter le rôle des commandes suivantes :

| Commandes | Rôles |
|---|---|
| `git init` | Initialise un nouveau dépôt Git local dans le dossier courant. |
| `git status` | Affiche l'état des fichiers (modifiés, suivis, en zone d'index/staging). |
| `git diff` | Affiche les modifications non indexées par rapport au dernier commit. |
| `git add` | Ajoute les modifications du répertoire de travail à la zone d'index (staging area). |
| `git commit` | Enregistre un instantané (snapshot) des fichiers indexés dans l'historique local. |
| `git log --oneline` | Affiche l'historique des commits de manière synthétique (une ligne par commit). |
| `git restore` | Annule les modifications locales non indexées d'un fichier. |
| `git clone` | Copie un dépôt distant existant sur ta machine locale. |
| `git pull` | Récupère et fusionne les dernières modifications du dépôt distant sur la branche courante. |
| `git push` | Envoie tes commits locaux vers le dépôt distant. |
| `git branch` | Liste, crée ou supprime des branches localement. |
| `git switch` | Bascule vers une autre branche de travail. |
| `git merge` | Fusionne la branche spécifiée dans la branche courante. |

### 37. Expliquer en quelques lignes la différence entre Git, GitHub, un commit, une branche, une fusion et une Pull Request.

- **Git** : c'est un logiciel de gestion de versions installé sur votre ordinateur qui permet de suivre, d'enregistrer et de gérer l'historique des modifications de votre code en local.
- **GitHub** : c'est une plateforme web (hébergée en ligne) qui stocke vos projets Git à distance pour faciliter le partage, le travail en équipe et la sauvegarde.
- **Un commit** : c'est une sauvegarde instantanée (un cliché) de l'état de vos fichiers à un moment donné, accompagnée d'un message explicatif.
- **Une branche** : c'est une copie de travail parallèle qui vous permet de développer une nouvelle fonctionnalité ou de corriger un bug sans casser le code principal.
- **Une fusion (Merge)** : c'est l'action technique qui consiste à assembler les modifications d'une branche dans une autre (souvent pour ramener une nouvelle fonction dans le tronc principal).
- **Une Pull Request (PR)** : c'est une demande de relecture créée sur GitHub ; elle permet à vos collègues de commenter, tester et valider votre code avant de l'intégrer officiellement.

### 38. Un administrateur exécute « git add . » puis « git commit », mais oublie « git push ». Les autres administrateurs peuvent-ils voir ce commit sur GitHub ? Justifier.

Non, car les commandes `git add .` et `git commit` ne changent que le répertoire local, le répertoire distant restera donc inchangé.

### 39. Un administrateur doit développer une fonctionnalité qui n'est pas encore validée. Doit-il modifier directement « main » ou créer une branche dédiée ? Justifier.

Il est préférable de créer une branche dédiée, afin de ne pas se perdre et créer des complications dans le cas où la fonctionnalité serait refusée par exemple.
