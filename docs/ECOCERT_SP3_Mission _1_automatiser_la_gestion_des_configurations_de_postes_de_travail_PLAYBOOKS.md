# Mission 1 : Installation et configuration d'un serveur de gestion de configurations centralisées

## PLAYBOOKS

---

## Playbook 1 : `1-ssh.yml`

### Explication ligne par ligne

Ce playbook sécurise SSH en trois temps : déployer la clé publique, vérifier qu'elle fonctionne, puis seulement après désactiver les mots de passe. Cet ordre évite de se verrouiller hors du serveur.

### En-tête du play

```yaml
---
- name: Securisation SSH des serveurs
  hosts: serveurs
  become: true
  serial: 1
```

- `hosts: serveurs` : groupe ciblé dans l'inventaire.
- `become: true` : élévation de privilèges (sudo), nécessaire pour écrire dans `/etc/ssh`.
- `serial: 1` : un serveur à la fois. Si la configuration SSH casse l'accès sur un serveur, le play s'arrête et les autres restent intacts. C'est une protection importante pour ce type de modification.

```yaml
  vars:
    ssh_public_key_file: "{{ lookup('env', 'HOME') }}/.ssh/id_ed25519.pub"
    ssh_private_key_file: "{{ lookup('env', 'HOME') }}/.ssh/id_ed25519"
```

Deux variables contenant le chemin de la paire de clés. `lookup('env', 'HOME')` lit la variable d'environnement `HOME` sur la machine de contrôle (celle qui lance Ansible), donc `/home/etudiant`. `id_ed25519` est une clé de type Ed25519 (moderne et recommandée).

### Tâche 1 : vérifications préalables

```yaml
    - name: Verifier que les informations SSH sont disponibles
      ansible.builtin.assert:
        that:
          - ansible_user is defined
          - ssh_public_key_file is file
          - ssh_private_key_file is file
        fail_msg: "Utilisateur SSH ou paire de cles introuvable."
```

Le play s'arrête si une condition est fausse :

- `ansible_user is defined` : un utilisateur de connexion est bien défini dans l'inventaire.
- `ssh_public_key_file is file` : le test Jinja `file` vérifie que le fichier de clé publique existe (côté machine de contrôle).
- `ssh_private_key_file is file` : idem pour la clé privée.

### Tâche 2 : installer la clé publique

```yaml
    - name: Installer la cle publique sur le serveur
      ansible.posix.authorized_key:
        user: "{{ ansible_user }}"
        key: "{{ lookup('file', ssh_public_key_file) }}"
        state: present
```

Le module `authorized_key` (collection `ansible.posix`) ajoute la clé dans `~/.ssh/authorized_keys` de l'utilisateur.

- `user` : le compte qui recevra la clé.
- `key` : `lookup('file', ...)` lit le contenu du fichier `.pub` sur la machine de contrôle.
- `state: present` : ajoute la clé si elle est absente (idempotent, pas de doublon).

### Tâche 3 : activer l'authentification par clé

```yaml
    - name: Activer l'authentification par cle publique
      ansible.builtin.copy:
        dest: /etc/ssh/sshd_config.d/00-ansible.conf
        owner: root
        group: root
        mode: '0644'
        content: |
          PubkeyAuthentication yes
      notify: Reload SSH
```

Crée un petit fichier de configuration dans `sshd_config.d/` (chargé par `Include` dans `sshd_config` sur Debian récent). Il contient une seule directive : `PubkeyAuthentication yes`.

`notify: Reload SSH` déclenche le handler défini en bas, mais seulement si le fichier a été modifié. Précision importante : un handler s'exécute à la fin du play (sur chaque hôte), pas immédiatement.

### Tâche 4 : valider la configuration

```yaml
    - name: Verifier la configuration SSH
      ansible.builtin.command: sshd -t
      changed_when: false
```

`sshd -t` teste la syntaxe de la configuration sans rien appliquer. S'il y a une erreur, la commande retourne un code non nul, la tâche échoue et le play s'arrête avant que le handler ne recharge SSH avec une configuration invalide. `changed_when: false` indique qu'un test ne modifie rien.

### Tâche 5 : tester une connexion par clé

```yaml
    - name: Tester une connexion avec la cle privee
      ansible.builtin.command:
        argv:
          - ssh
          - -i
          - "{{ ssh_private_key_file }}"
          - -o
          - BatchMode=yes
          - -o
          - StrictHostKeyChecking=accept-new
          - -p
          - "{{ ansible_port | default(22) | string }}"
          - "{{ ansible_user }}@{{ ansible_host | default(inventory_hostname) }}"
          - "true"
      delegate_to: localhost
      become: false
      changed_when: false
```

Lance sur la machine de contrôle la commande :

```bash
ssh -i ~/.ssh/id_ed25519 -o BatchMode=yes -o StrictHostKeyChecking=accept-new -p 22 user@serveur true
```

- `-i` : utilise explicitement la clé privée.
- `BatchMode=yes` : interdit toute demande de mot de passe. Si la clé ne marche pas, la connexion échoue au lieu d'attendre une saisie. C'est ce qui prouve que l'authentification par clé fonctionne.
- `StrictHostKeyChecking=accept-new` : accepte automatiquement l'empreinte d'un serveur jamais vu, mais refuse si l'empreinte a changé (protection contre une usurpation).
- `-p` : port, pris dans `ansible_port` ou 22 par défaut (`| string` convertit en texte, requis dans `argv`).
- `user@hôte` : `ansible_host` s'il existe, sinon le nom d'inventaire.
- `true` : commande distante qui ne fait rien et retourne 0.
- `delegate_to: localhost` : la tâche s'exécute sur la machine de contrôle, pas sur le serveur (c'est elle qui joue le rôle de client SSH).
- `become: false` : pas de sudo, on doit utiliser vos clés dans votre `~/.ssh`.
- `changed_when: false` : simple test.

### Tâche 6 : désactiver les mots de passe

```yaml
    - name: Desactiver l'authentification par mot de passe
      ansible.builtin.copy:
        dest: /etc/ssh/sshd_config.d/99-ansible-hardening.conf
        owner: root
        group: root
        mode: '0644'
        content: |
          PasswordAuthentication no
          KbdInteractiveAuthentication no
      notify: Reload SSH
```

Cette tâche n'est atteinte que si le test de la tâche 5 a réussi.

- `PasswordAuthentication no` : interdit la connexion par mot de passe.
- `KbdInteractiveAuthentication no` : ferme aussi la porte dérobée du mode « clavier interactif » (PAM), qui permettrait sinon de saisir un mot de passe.
- Le préfixe `99-` place ce fichier en dernier dans l'ordre de lecture.

### Handlers

```yaml
  handlers:
    - name: Reload SSH
      ansible.builtin.service:
        name: ssh
        state: reloaded
```

Un handler n'est exécuté que s'il a été notifié, et une seule fois même si plusieurs tâches le notifient. Ici, `reloaded` recharge la configuration sans couper les sessions existantes. Sur Debian, le service s'appelle `ssh` (et non `sshd`).

### Points d'attention

1. **Priorité des fichiers dans `sshd_config.d/`** : OpenSSH applique la première valeur rencontrée pour une directive, dans l'ordre alphabétique. Si un fichier plus ancien (par exemple `50-cloud-init.conf`) contient `PasswordAuthentication yes`, il sera lu avant `99-ansible-hardening.conf` et gagnera. Après exécution, vérifiez avec :
   ```bash
   sudo sshd -T | grep -i -E "passwordauth|kbdinteractive|pubkeyauth"
   ```
2. **Timing du handler** : le rechargement a lieu à la fin du play. Donc le test de la tâche 5 se fait avant le rechargement, alors que le mot de passe est encore actif. Le test prouve que la clé fonctionne, mais pas que le serveur refusera les mots de passe. Pour cela, ajoutez `- ansible.builtin.meta: flush_handlers` après la tâche 6 si vous voulez recharger immédiatement.
3. **Pas de validation de la 2e configuration** : `sshd -t` est exécuté avant la tâche 6, donc il ne valide pas le fichier de durcissement. Une meilleure pratique est d'ajouter `validate: sshd -t -f %s` dans le module `copy`, ou de refaire un `sshd -t` après.
4. **Dépendance** : le module `ansible.posix.authorized_key` nécessite la collection `ansible.posix` (`ansible-galaxy collection install ansible.posix`).

### Utilisation

```bash
ansible-playbook 1-ssh.yml
```

> Gardez toujours une session SSH ouverte pendant le test, au cas où un problème surviendrait.

---

## Playbook 2 : `2-Installation-application.yml`

### Explication ligne par ligne

### En-tête du play

```yaml
---
- name: Installation d'une application
```

Début du document YAML, puis début d'un play nommé (le nom s'affiche à l'exécution).

```yaml
  hosts: serveurs
```

Cible : le groupe `serveurs` de l'inventaire.

```yaml
  become: true
```

Élévation de privilèges (sudo), nécessaire pour installer des paquets et gérer des services.

```yaml
  vars:
    application_package: apache2
    application_service: apache2
```

Deux variables avec des valeurs par défaut :

- `application_package` : le nom du paquet à installer.
- `application_service` : le nom du service à démarrer.

Elles sont séparées car le nom du paquet et celui du service ne sont pas toujours identiques (par exemple, le paquet `openssh-server` fournit le service `ssh`). Pour `apache2`, les deux noms coïncident.

```yaml
  tasks:
```

Début de la liste des tâches.

### Tâche 1 : vérifier le nom du paquet

```yaml
    - name: Verifier le nom du paquet
      ansible.builtin.assert:
        that:
          - application_package | length > 0
        fail_msg: "Aucun paquet n'a ete indique."
```

Le module `assert` fait échouer le playbook si la condition est fausse.

- `application_package | length > 0` : le filtre `length` compte les caractères de la variable. Si elle est vide (par exemple `-e application_package=""`), le playbook s'arrête avec le message d'erreur.

C'est une protection : sans elle, `apt` serait appelé avec un nom vide, ce qui produirait une erreur peu claire.

### Tâche 2 : actualiser les dépôts

```yaml
    - name: Actualiser les depots
      ansible.builtin.apt:
        update_cache: true
        cache_valid_time: 3600
```

Équivalent de `apt update`.

- `update_cache: true` : met à jour la liste des paquets disponibles.
- `cache_valid_time: 3600` : si le cache a été actualisé il y a moins de 3600 secondes (1 heure), la mise à jour est sautée. Cela fait gagner du temps et rend la tâche idempotente.

À comparer avec le playbook `3-Mise-a-jour.yml`, qui utilisait `cache_valid_time: 0` pour forcer l'actualisation à chaque fois.

### Tâche 3 : installer l'application

```yaml
    - name: Installer l'application
      ansible.builtin.apt:
        name: "{{ application_package }}"
        state: present
```

- `name: "{{ application_package }}"` : le nom vient de la variable (donc `apache2` par défaut). Les guillemets sont obligatoires en YAML quand une valeur commence par `{{`.
- `state: present` : installe le paquet s'il est absent, ne fait rien s'il est déjà là (idempotent). Pour forcer la dernière version, on utiliserait `state: latest`.

### Tâche 4 : démarrer et activer le service

```yaml
    - name: Demarrer et activer le service
      ansible.builtin.service:
        name: "{{ application_service }}"
        state: started
        enabled: true
      when: application_service | length > 0
```

Le module `service` gère les services système :

- `state: started` : démarre le service s'il est arrêté (sans le redémarrer s'il tourne déjà).
- `enabled: true` : l'active au démarrage de la machine (équivalent de `systemctl enable`).

La clause `when: application_service | length > 0` fait que la tâche n'est exécutée que si un nom de service est fourni. Cela permet d'installer une bibliothèque ou un outil sans service (par exemple `htop` ou `curl`) en passant `application_service=""`.

### Utilisation

Installation par défaut (apache2) :

```bash
ansible-playbook 2-Installation-application.yml
```

Installer une autre application en surchargeant les variables :

```bash
ansible-playbook 2-Installation-application.yml -e "application_package=nginx application_service=nginx"
```

Installer un paquet sans service :

```bash
ansible-playbook 2-Installation-application.yml -e "application_package=htop application_service="
```

### Remarques

- Sur Debian, l'installation d'un paquet comme `apache2` démarre généralement le service automatiquement, mais la tâche 4 garantit qu'il est bien démarré et activé au boot.
- Le playbook ne gère pas le pare-feu : si `ufw` est actif, il faudra ouvrir les ports (80/443) dans un autre playbook.
- Pour un paquet avec un nom de service différent, pensez à fournir les deux variables, sinon la tâche 4 échouera (service introuvable).

---

## Playbook 3 : `3-Mise-a-jour.yml`

### Explication ligne par ligne

### En-tête du play

```yaml
---
- name: Mise a jour des serveurs Debian
```

Début du document YAML, puis début d'un play nommé (le nom est affiché à l'exécution).

```yaml
  hosts: serveurs
```

Cible : le groupe `serveurs` de l'inventaire.

```yaml
  become: true
```

Élévation de privilèges (sudo), indispensable pour mettre à jour le système.

```yaml
  serial: 1
```

Le play est exécuté un serveur à la fois : Ansible termine toutes les tâches sur le serveur 1 avant de passer au serveur 2, etc. C'est une bonne pratique pour les mises à jour : si un serveur plante, les autres ne sont pas touchés, et vous évitez que tous les serveurs redémarrent en même temps.

```yaml
  vars:
    reboot_if_required: false
```

Déclare une variable avec une valeur par défaut `false` : par défaut, le playbook ne redémarre pas les serveurs. On peut la modifier à l'exécution (voir plus bas).

```yaml
  tasks:
```

Début de la liste des tâches.

### Tâche 1 : actualiser les dépôts

```yaml
    - name: Actualiser les depots
      ansible.builtin.apt:
        update_cache: true
        cache_valid_time: 0
```

Équivalent de `apt update`.

- `update_cache: true` : met à jour la liste des paquets disponibles.
- `cache_valid_time: 0` : force la mise à jour même si le cache a été rafraîchi récemment (par défaut, Ansible peut sauter l'étape si le cache est récent).

### Tâche 2 : installer les mises à jour

```yaml
    - name: Installer les mises a jour disponibles
      ansible.builtin.apt:
        upgrade: dist
        update_cache: true
        dpkg_options: force-confdef,force-confold
```

- `upgrade: dist` : équivalent de `apt dist-upgrade` (ou `full-upgrade`). Contrairement à un simple `upgrade`, il peut installer de nouveaux paquets ou en supprimer pour résoudre les dépendances (utile pour les changements de noyau, par exemple).
- `update_cache: true` : refait un `apt update` (redondant avec la tâche 1, mais sans danger).
- `dpkg_options: force-confdef,force-confold` : évite les questions interactives sur les fichiers de configuration modifiés :
  - `force-confdef` : si dpkg peut décider seul, il le fait.
  - `force-confold` : en cas de conflit, conserve votre ancienne configuration plutôt que d'écraser avec celle du paquet.

Sans ces options, une mise à jour pourrait rester bloquée en attente d'une réponse.

### Tâche 3 : vérifier si un redémarrage est nécessaire

```yaml
    - name: Verifier si un redemarrage est necessaire
      ansible.builtin.stat:
        path: /var/run/reboot-required
      register: reboot_required
```

Sur Debian/Ubuntu, le fichier `/var/run/reboot-required` est créé après certaines mises à jour (noyau, libc, etc.). Le module `stat` teste son existence et le résultat est stocké dans `reboot_required`.

> **Note :** ce fichier est créé par le paquet `update-notifier-common` (présent par défaut sur Ubuntu, mais pas toujours sur Debian). Sur un Debian minimal, il peut ne jamais exister même si un redémarrage serait utile. Vérifiez qu'il est installé si vous comptez sur ce mécanisme.

### Tâche 4 : signaler le besoin de redémarrage

```yaml
    - name: Signaler un redemarrage necessaire
      ansible.builtin.debug:
        msg: "Un redemarrage est necessaire sur {{ inventory_hostname }}."
      when: reboot_required.stat.exists
```

Affiche un message si le fichier existe. `{{ inventory_hostname }}` est remplacé par le nom du serveur. Cette tâche est purement informative.

### Tâche 5 : redémarrer si demandé

```yaml
    - name: Redemarrer le serveur si demande
      ansible.builtin.reboot:
        reboot_timeout: 600
        msg: "Redemarrage effectue par Ansible"
      when:
        - reboot_required.stat.exists
        - reboot_if_required | bool
```

Le module `reboot` redémarre la machine, puis attend qu'elle soit de nouveau joignable avant de continuer.

- `reboot_timeout: 600` : attend jusqu'à 600 secondes (10 min) le retour du serveur.
- `msg` : message de diffusion affiché aux utilisateurs connectés.

Les deux conditions du `when` doivent être vraies (liste = ET logique) :

1. le fichier de redémarrage existe ;
2. la variable `reboot_if_required` vaut vrai (le filtre `| bool` convertit proprement une chaîne comme `"true"` en booléen, utile quand la valeur vient de la ligne de commande).

### Utilisation

Mise à jour sans redémarrage (comportement par défaut) :

```bash
ansible-playbook 3-Mise-a-jour.yml
```

Mise à jour avec redémarrage automatique si nécessaire :

```bash
ansible-playbook 3-Mise-a-jour.yml -e reboot_if_required=true
```

### Remarques

- Grâce à `serial: 1`, si un serveur échoue, Ansible arrête généralement le play pour la suite (comportement par défaut), ce qui protège le reste du parc.
- La tâche 1 est en partie redondante avec les `update_cache` de la tâche 2, mais cela ne pose pas de problème.
- La tâche « Installer les mises a jour » sera toujours signalée « changed » seulement si des paquets ont réellement été mis à jour : le module est idempotent.

---

## Playbook 4 : `4-politique-de-securite`

### Explication ligne par ligne

### En-tête du playbook

```yaml
---
```

Marqueur de début d'un document YAML. Obligatoire par convention.

```yaml
- name: Configuration de la politique des mots de passe
```

Début d'un play (une liste en YAML commence par `-`). Le `name` est simplement une description affichée à l'exécution.

```yaml
  hosts: serveurs
```

Les machines ciblées : le groupe `serveurs` défini dans votre inventaire Ansible.

```yaml
  become: true
```

Élévation de privilèges (équivalent de `sudo`) pour toutes les tâches. Nécessaire car on modifie des fichiers système.

```yaml
  tasks:
```

Début de la liste des tâches, exécutées dans l'ordre.

### Tâche 1 : installation des paquets

```yaml
    - name: Installer les outils de controle des mots de passe
      ansible.builtin.apt:
        name:
          - libpam-pwquality
          - cracklib-runtime
        state: present
        update_cache: true
```

Utilise le module `apt` (Debian/Ubuntu). Deux paquets à installer :

- `libpam-pwquality` : le module PAM qui contrôle la qualité des mots de passe.
- `cracklib-runtime` : les dictionnaires pour détecter les mots de passe trop faibles.

`present` = installer s'il n'est pas déjà là. `update_cache: true` = faire un `apt update` avant.

### Tâche 2 : vérifier la présence du profil PAM

```yaml
    - name: Verifier la presence du profil PAM pwquality
      ansible.builtin.stat:
        path: /usr/share/pam-configs/pwquality
      register: pam_profile
```

Le module `stat` récupère des informations sur un fichier. Le résultat est stocké (`register`) dans la variable `pam_profile`. Ce fichier est le profil que `pam-auth-update` utilise pour activer le module.

### Tâche 3 : assertion

```yaml
    - name: Verifier que le profil PAM existe
      ansible.builtin.assert:
        that:
          - pam_profile.stat.exists
        fail_msg: "Le profil PAM pwquality est introuvable."
```

Le module `assert` fait échouer le playbook si la condition est fausse. Ici : si le fichier n'existe pas, on s'arrête avec le message d'erreur. Ça évite de continuer sur une base cassée.

### Tâche 4 : vérifier si pwquality est déjà actif

```yaml
    - name: Verifier si pwquality est deja active dans PAM
      ansible.builtin.command:
        argv:
          - grep
          - -q
          - pam_pwquality.so
          - /etc/pam.d/common-password
      register: pam_check
      failed_when: false
      changed_when: false
```

Exécute `grep -q pam_pwquality.so /etc/pam.d/common-password`. L'option `-q` (quiet) n'affiche rien : seul le code retour compte (0 = trouvé, 1 = absent). `argv` passe les arguments sous forme de liste, plus sûr qu'une chaîne de commande.

- `register` : stocke le résultat (dont le code retour `rc`).
- `failed_when: false` : la tâche n'échoue jamais, même si `grep` renvoie 1.
- `changed_when: false` : une simple lecture ne doit pas être comptée comme une modification.

### Tâche 5 : activer le module

```yaml
    - name: Activer le module pwquality dans PAM
      ansible.builtin.command:
        argv:
          - pam-auth-update
          - --enable
          - pwquality
      when: pam_check.rc != 0
      changed_when: true
```

Exécute `pam-auth-update --enable pwquality`, qui modifie les fichiers `/etc/pam.d/common-*` pour insérer le module.

- `when` : n'exécute la tâche que si `grep` n'a pas trouvé le module (code retour différent de 0). Cela rend le playbook idempotent.
- `changed_when: true` : la tâche est toujours marquée « changed » quand elle s'exécute.

### Tâche 6 : créer le dossier de configuration

```yaml
    - name: Creer le dossier de configuration
      ansible.builtin.file:
        path: /etc/security/pwquality.conf.d
        state: directory
        owner: root
        group: root
        mode: '0755'
```

Crée le répertoire `pwquality.conf.d` (s'il n'existe pas), appartenant à root, avec les droits `rwxr-xr-x` (lecture pour tous, écriture pour root seul). Les fichiers de ce dossier complètent `/etc/security/pwquality.conf`.

### Tâche 7 : appliquer la politique

```yaml
    - name: Appliquer la politique de complexite
      ansible.builtin.copy:
        dest: /etc/security/pwquality.conf.d/99-ansible.conf
        owner: root
        group: root
        mode: '0644'
        backup: true
        content: |
          minlen = 14
          ucredit = -1
          lcredit = -1
          dcredit = -1
          ocredit = -1
          maxrepeat = 3
          maxsequence = 3
          difok = 4
          dictcheck = 1
          usercheck = 1
          gecoscheck = 1
          retry = 3
          enforcing = 1
          enforce_for_root
```

Le module `copy` crée un fichier à partir du contenu fourni. Le préfixe `99-` garantit qu'il est lu en dernier (et donc prioritaire). Droits `rw-r--r--`. `backup: true` sauvegarde l'ancienne version avant écrasement.

Le `|` indique un bloc de texte multiligne, conservé tel quel.

**Contenu du fichier :**

| Paramètre | Signification |
|---|---|
| `minlen = 14` | Longueur minimale de 14 caractères |
| `ucredit = -1` | Au moins 1 majuscule (valeur négative = minimum obligatoire) |
| `lcredit = -1` | Au moins 1 minuscule |
| `dcredit = -1` | Au moins 1 chiffre |
| `ocredit = -1` | Au moins 1 caractère spécial |
| `maxrepeat = 3` | Pas plus de 3 fois le même caractère consécutif (ex. `aaaa` refusé) |
| `maxsequence = 3` | Pas plus de 3 caractères en séquence (ex. `abcd`, `1234` refusés) |
| `difok = 4` | Au moins 4 caractères différents de l'ancien mot de passe |
| `dictcheck = 1` | Refuse les mots du dictionnaire |
| `usercheck = 1` | Refuse un mot de passe contenant le nom d'utilisateur |
| `gecoscheck = 1` | Refuse un mot de passe contenant des éléments du champ GECOS (nom complet, etc.) |
| `retry = 3` | 3 tentatives avant échec de la commande `passwd` |
| `enforcing = 1` | Le refus est effectif (0 = simple avertissement) |
| `enforce_for_root` | Applique aussi les règles à root |

### Tâche 8 : vérification finale

```yaml
    - name: Verifier la presence du module PAM
      ansible.builtin.command:
        argv:
          - grep
          - pam_pwquality.so
          - /etc/pam.d/common-password
      changed_when: false
```

Même `grep` qu'avant, mais sans `-q` : la ligne trouvée est affichée. Si le module est absent, `grep` renvoie 1 et la tâche échoue (pas de `failed_when: false` ici), ce qui sert de contrôle final.

### Tâche 9 : message de fin

```yaml
    - name: Afficher le resultat
      ansible.builtin.debug:
        msg: "La politique de complexite des mots de passe est configuree sur {{ inventory_hostname }}."
```

Affiche un message. `{{ inventory_hostname }}` est une variable Jinja2 remplacée par le nom de chaque serveur dans l'inventaire.

### Remarques

- Le fichier s'appelle `4-politique-de-securite` sans extension `.yml` : ça fonctionne, mais l'extension est conseillée.
- Les tâches `command` avec `grep` pourraient être remplacées par le module `ansible.builtin.lineinfile` ou `ansible.builtin.shell`, mais l'approche choisie est correcte et idempotente.
- **Attention** : `enforce_for_root` s'applique aussi à root, ce qui peut bloquer un changement de mot de passe si celui-ci ne respecte pas la politique.

Pour lancer le playbook :

```bash
ansible-playbook 4-politique-de-securite
```
