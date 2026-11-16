# Mission 1 : Installation et configuration d'un serveur de gestion de configurations centralisées

## Table des matières

1. [Installation Ansible](#1-installation-ansible)
   - [1.1) Connexion SSH par paire de clés](#11-permissions-pour-ansible-de-se-connecter-en-ssh-au-compte-administrateur-local-etudiant-via-une-authentification-par-paire-de-clés)
   - [1.2) Commandes en root via sudo](#12-ansible-doit-pouvoir-effectuer-des-commandes-ou-des-tâches-en-tant-que-root-à-laide-de-la-commande-sudo)
   - [1.3) Configuration d'Ansible avant mise en place des playbooks](#13-configuration-dansible-avant-mise-en-place-des-playbooks)
   - [1.4) Configuration SSH par clés sur l'ensemble des postes](#14-on-doit-pouvoir-sassurer-que-lensemble-des-postes-disposent-dune-configuration-ssh-avec-une-authentification-par-clés-et-dans-le-cas-contraire-réaliser-les-modifications-sur-les-postes-concernés)
   - [1.5) Installation d'une application par le gestionnaire de paquets](#15-on-doit-pouvoir-installer-une-nouvelle-application-par-le-gestionnaire-de-paquet-sur-lensemble-de-la-salle)
   - [1.6) Mises à jour automatiques](#16on-doit-pouvoir-réaliser-les-mises-à-jour-automatiquement-sur-lensemble-de-la-salle)
   - [1.7) Politique de complexité des mots de passe](#17-enfin-on-doit-pouvoir-sassurer-que-la-politique-de-complexité-des-mots-de-passe-locaux-sont-conformes-aux-recommandations-de-la-cnil-et-de-lanssi)

---

## 1) Installation Ansible

On met à jour le système et on installe les dépendances de Ansible, ainsi que la solution :

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

En cas de paquet impossible à trouver, il existe un moyen par **PIPX** (Python).

PIPX est un gestionnaire d'applications Python isolées. Pour éviter qu'Ansible installe ses propres versions de bibliothèques Python dans le système et initie des conflits, PIPX enferme Ansible dans un environnement virtuel fermé afin qu'il ne touche pas à la machine.

```bash
sudo apt install pipx
pipx install --include-deps ansible   # Installation propre d'Ansible et des dépendances
pipx ensurepath                       # Ajout de pipx aux chemins d'accès des commandes (bin)
```

Recharger la configuration du terminal :

```bash
source ~/.bashrc
ansible --version
pipx upgrade ansible                  # Pour mettre à jour
```

Résultat de `ansible --version` :

```
ansible [core 2.21.5]
  config file = None
  configured module search path = ['/home/etudiant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/etudiant/.local/share/pipx/venvs/ansible/lib/python3.13/site-packages/ansible
  ansible collection location = /home/etudiant/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/etudiant/.local/bin/ansible
  python version = 3.13.5 (main, Aug 10 2026, 12:06:59) [GCC 14.2.0] (/home/etudiant/.local/share/pipx/venvs/ansible/bin/python)
  jinja version = 3.1.6
  pyyaml version = 6.0.3 (with libyaml v0.2.5)
```

---

## 1.1) Permissions pour Ansible de se connecter en SSH au compte administrateur local etudiant via une authentification par paire de clés

**Création d'une paire de clés SSH :**

```bash
ssh-keygen -t ed25519    # Format moderne
```

**Choisir l'emplacement où conserver la clé** (au choix ou par défaut).

```bash
ssh-copy-id nom@ip_cible
```

Dans notre cas : `etudiant@172.16.52.10` (pour l'exemple → à mettre normalement sur la machine cible, autre que l'hôte Ansible).

Puis entrer le mot de passe de la machine : les prochaines connexions seront désormais automatiques, sans mot de passe à renseigner.

Exemple de sortie de `ssh-keygen` :

```
etudiant@CONFECOCERT:/$ ssh-keygen -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/etudiant/.ssh/id_ed25519):
Enter passphrase for "/home/etudiant/.ssh/id_ed25519" (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/etudiant/.ssh/id_ed25519
Your public key has been saved in /home/etudiant/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:X5Rn9hEaDdPLDt0+27psmSo9jAflVwry0lCY8M4+XTM etudiant@CONFECOCERT
```

**Sur la machine cible**, la clé publique apparaît dans `authorized_keys` :

```
etudiant@KEADHCP:~/.ssh$ cat authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEVJFg8/+uuziikcHFAbeNLtsUevslzC9tQn0bbQDC1a etudiant@CONFECOCERT
```

---

## 1.2) Ansible doit pouvoir effectuer des commandes ou des tâches en tant que root à l'aide de la commande sudo

Comme Ansible va devoir exécuter des tâches administratives (installer des paquets, modifier des fichiers système), le compte `etudiant` sur la machine cible doit pouvoir utiliser `sudo` sans que Linux ne lui réclame son mot de passe à chaque action.

```bash
echo "etudiant ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible-etudiant
sudo chmod 0440 /etc/sudoers.d/ansible-etudiant
```

**Ou simplement :**

```bash
sudo visudo
```

Ligne où il y a `%sudo` :

```
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL) NOPASSWD: ALL
```

---

## 1.3) Configuration d'Ansible avant mise en place des playbooks

Comme nous avons installé Ansible via pipx, nous y avons accès depuis n'importe quel dossier. Il sera donc nécessaire de créer un répertoire pour notre configuration, inventaire, etc.

```bash
mkdir ansible/{inventories,playbooks}
```

Puis, dans le dossier `ansible`, créer la configuration :

```bash
nano ansible.cfg
```

```ini
[defaults]
inventory = ./inventories/inventaire.ini
host_key_checking = True
retry_files_enabled = False
```

Dans cet exemple, on indique à Ansible où se situe l'inventaire des machines cibles, on active la vérification de l'identité SSH des serveurs et on désactive la création de fichiers `.retry` après un échec.

### L'inventaire

Il sera utilisé par Ansible pour savoir sur quelles machines il doit appliquer ses playbooks.

Dans le dossier `inventories` nouvellement créé :

```bash
nano inventaire.ini
```

```ini
[serveurs]
192.168.2.211

[serveurs:vars]
ansible_user=etudiant
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Ici, j'indique les machines à atteindre ainsi que l'utilisateur à utiliser pour Ansible. La dernière ligne indique le fichier de la clé privée du serveur pour qu'il puisse se connecter via sa paire.

### Les playbooks

Avant toute chose, nous allons créer un playbook de test afin d'essayer si le service fonctionne.

Dans le dossier `playbooks` :

```bash
nano test.yml
```

> Le YAML est sensible à l'indentation.

```yaml
---
- name: Test de création de fichier
  hosts: serveurs
  become: false

  tasks:
    - name: Créer un fichier texte
      ansible.builtin.copy:
        dest: /home/etudiant/test.txt
        content: |
          Bonjour, test ansible réussi
        mode: '0644'
```

Exécution :

```bash
ansible-playbook playbooks/test.yml
```

```
PLAY [Test de création de fichier] ***********************************

TASK [Gathering Facts] ***********************************************
[WARNING]: Host '192.168.2.211' is using the discovered Python interpreter at '/usr/bin/python3.13', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.21/reference_appendices/interpreter_discovery.html for more information.
ok: [192.168.2.211]

TASK [Créer un fichier texte] ****************************************
changed: [192.168.2.211]

PLAY RECAP ***********************************************************
192.168.2.211 : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

Normalement, vous trouverez le fichier `.txt` créé à l'endroit indiqué dans le playbook.

---

## 1.4) On doit pouvoir s'assurer que l'ensemble des postes disposent d'une configuration SSH avec une authentification par clés et, dans le cas contraire, réaliser les modifications sur les postes concernés

```bash
ansible-playbook playbooks/1-ssh.yml
```

```
PLAY [Securisation SSH des serveurs] *********************************

TASK [Gathering Facts] ***********************************************
ok: [192.168.2.211]

TASK [Verifier que les informations SSH sont disponibles] ************
ok: [192.168.2.211] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Installer la cle publique sur le serveur] **********************
ok: [192.168.2.211]

TASK [Activer l'authentification par cle publique] *******************
ok: [192.168.2.211]

TASK [Verifier la configuration SSH] *********************************
ok: [192.168.2.211]

TASK [Tester une connexion avec la cle privee] ***********************
ok: [192.168.2.211 -> localhost]

TASK [Desactiver l'authentification par mot de passe] ****************
changed: [192.168.2.211]

RUNNING HANDLER [Reload SSH] *****************************************
changed: [192.168.2.211]

PLAY RECAP ***********************************************************
192.168.2.211 : ok=8 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

---

## 1.5) On doit pouvoir installer une nouvelle application par le gestionnaire de paquet sur l'ensemble de la salle

```bash
ansible-playbook playbooks/2-Installation-application.yml
```

```
PLAY [Installation d'une application] ********************************

TASK [Gathering Facts] ***********************************************
ok: [192.168.2.211]

TASK [Verifier le nom du paquet] *************************************
ok: [192.168.2.211] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Actualiser les depots] *****************************************
ok: [192.168.2.211]

TASK [Installer l'application] ***************************************
changed: [192.168.2.211]

TASK [Demarrer et activer le service] ********************************
ok: [192.168.2.211]

PLAY RECAP ***********************************************************
192.168.2.211 : ok=5 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

---

## 1.6) On doit pouvoir réaliser les mises à jour automatiquement sur l'ensemble de la salle

```bash
ansible-playbook playbooks/3-Mise-a-jour.yml
```

```
PLAY [Mise a jour des serveurs Debian] *******************************

TASK [Gathering Facts] ***********************************************
ok: [192.168.2.211]

TASK [Actualiser les depots] *****************************************
ok: [192.168.2.211]

TASK [Installer les mises a jour disponibles] ************************
changed: [192.168.2.211]

TASK [Verifier si un redemarrage est necessaire] *********************
ok: [192.168.2.211]

TASK [Signaler un redemarrage necessaire] ****************************
skipping: [192.168.2.211]

TASK [Redemarrer le serveur si demande] ******************************
skipping: [192.168.2.211]

PLAY RECAP ***********************************************************
192.168.2.211 : ok=4 changed=1 unreachable=0 failed=0 skipped=2 rescued=0 ignored=0
```

---

## 1.7) Enfin, on doit pouvoir s'assurer que la politique de complexité des mots de passe locaux sont conformes aux recommandations de la CNIL et de l'ANSSI

```
TASK [Installer les outils de controle des mots de passe] ************
changed: [192.168.2.211]

TASK [Verifier la presence du profil PAM pwquality] ******************
ok: [192.168.2.211]

TASK [Verifier que le profil PAM existe] *****************************
ok: [192.168.2.211] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Verifier si pwquality est deja active dans PAM] ****************
ok: [192.168.2.211]

TASK [Activer le module pwquality dans PAM] **************************
skipping: [192.168.2.211]

TASK [Creer le dossier de configuration] *****************************
changed: [192.168.2.211]

TASK [Appliquer la politique de complexite] **************************
changed: [192.168.2.211]

TASK [Verifier la presence du module PAM] ****************************
ok: [192.168.2.211]

TASK [Afficher le resultat] ******************************************
ok: [192.168.2.211] => {
    "msg": "La politique de complexite des mots de passe est configuree sur 192.168.2.211."
}

PLAY RECAP ***********************************************************
192.168.2.211 : ok=9 changed=3 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0
```

---

## Commandes principales d'Ansible

| Commande | Rôle |
|---|---|
| `ansible` | Exécuter des commandes sur les serveurs |
| `ansible-playbook` | Exécuter des playbooks |
| `ansible-inventory` | Afficher / gérer l'inventaire |
| `ansible-galaxy` | Gérer les rôles et collections |
| `ansible-vault` | Chiffrer des données sensibles |
| `ansible-config` | Consulter la configuration |
| `ansible-doc` | Consulter la documentation des modules |
