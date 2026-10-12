# ECOCERT - SP0 Mission 3 : Mise en place d'un serveur de gestion d'incidents et d'un serveur NAS

*Crys 801 Boisseau*

> **Légende :** les passages précédés de la mention **« Complément »** ont été ajoutés pour combler des étapes absentes du document source (captures non fournies ou étapes sautées). Le reste du contenu provient du PDF d'origine.

## Table des matières

- [Caractéristiques VM](#caractéristiques-vm)
- [Comparaison d'OS](#comparaison-dos)
- [Mise en place du serveur GLPI](#mise-en-place-du-serveur-glpi)
  - [Configuration réseau et hostname](#configuration-réseau-et-hostname)
  - [Base de données GLPI](#base-de-données-glpi)
  - [Installation de GLPI et configuration d'Apache](#installation-de-glpi-et-configuration-dapache)
  - [Sécurisation de PHP](#sécurisation-de-php)
  - [Finalisation via l'assistant web](#finalisation-via-lassistant-web)
- [Mise en place du serveur NAS](#mise-en-place-du-serveur-nas)
  - [Configuration réseau de TrueNAS](#configuration-réseau-de-truenas)
  - [Connexion à l'interface web](#connexion-à-linterface-web)
  - [RAID 5](#raid-5)
  - [NTP](#ntp)
  - [Active Directory](#active-directory)
  - [Dataset, ACL et partage](#dataset-acl-et-partage)
  - [Client Windows](#client-windows)

---

## Caractéristiques VM

### Caractéristiques du serveur de gestion d'incidents et de parc informatique

- **Nom :** GLPIECOCERT
- **IP :** 172.16.52.40
- **Système d'exploitation :** *(voir « Comparaison d'OS »)*
- **Service en charge :** gestion d'inventaire et de tickets d'incident
- La gestion des tickets est sous la responsabilité de deux administrateurs réseau :
  - **Olivier TONDET** (responsable des solutions techniques d'accès et des éléments d'interconnexion)
  - **Pamela TREMO** (responsable des services et des systèmes serveurs)
- L'inventaire est réalisé en grande partie par une remontée automatique des composants des postes de travail.

### Caractéristiques du serveur NAS de sauvegarde

- **Nom :** NASECOCERT
- **IP :** 172.16.52.20
- **Système d'exploitation :** *(voir « Comparaison d'OS »)*
- **Service en charge :** gestion des sauvegardes
- Une configuration en **RAID 5** doit être prévue avec au moins **20 Go** disponibles pour les données à stocker.
- Un accès via l'**authentification par l'Active Directory** doit être paramétré.

## Comparaison d'OS

### GLPI

Pour faire notre VM, il est nécessaire d'avoir un système d'exploitation, mais lequel choisir ?

Les plus communs sont Windows et Linux :

- **Windows :** utilise beaucoup de ressources, n'est pas open source, a beaucoup de processus activés en arrière-plan.
- **Linux :** utilise peu de ressources, est open source pour la plupart de ses distributions, a un nombre minime de processus en arrière-plan.

Le meilleur système d'exploitation à choisir pour notre VM est donc Linux, il faut désormais choisir la distribution.

Il existe plusieurs distributions, les plus utilisées sont :

- **Ubuntu :** idéal pour les débutants, utilisation plus graphique.
- **Linux Mint :** idéal pour les personnes voulant arrêter Windows, utilisation plus graphique.
- **Debian :** très bonne stabilité, consomme très peu de ressources et est parfait pour lancer des serveurs.

Nous faisons donc le choix d'une distribution **Debian** pour le serveur GLPI, **Debian 13** en particulier.

### NAS

Il existe des moyens de créer un NAS avec une distribution Linux comme Debian par exemple. Mais il existe aussi des systèmes d'exploitation créés exprès pour les NAS, comme **TrueNAS**, qui est un système d'exploitation libre, gratuit et performant.

Nous utiliserons donc **TrueNAS**, **TrueNAS SCALE** en particulier.

---

## Mise en place du serveur GLPI

### Configuration réseau et hostname

> **Complément :** cette étape n'était pas détaillée dans le document source. Les valeurs reprennent le plan d'adressage du VLAN Serveurs (`172.16.52.0/24`, passerelle `172.16.52.253`) ; le nom de l'interface (`ens18`) et les serveurs DNS sont à adapter à l'environnement.

```bash
sudoedit /etc/network/interfaces
```

```
# The primary network interface
allow-hotplug ens18
auto ens18
iface ens18 inet static
address 172.16.52.40
netmask 255.255.255.0
gateway 172.16.52.253
```

```bash
sudoedit /etc/hostname
```

```
GLPIECOCERT
```

```bash
sudoedit /etc/hosts
```

```
127.0.0.1   localhost
127.0.1.1   glpi.local.ecocert2.fr   GLPIECOCERT
```

```bash
sudo systemctl restart networking
sudo shutdown -r now
```

### Base de données GLPI

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install apache2 php8.4-fpm mariadb-server
sudo apt install php8.4-{curl,gd,intl,mysql,zip,bcmath,mbstring,xml,bz2}
sudo apt install php8.4-ldap
sudo mariadb-secure-installation
sudo mysql -u root -p
```

```sql
CREATE DATABASE dbglpi_ecocert;
GRANT ALL PRIVILEGES ON dbglpi_ecocert.* TO adm_glpiecocert@localhost IDENTIFIED BY "xxxxxxxxxxxxxxx";
FLUSH PRIVILEGES;
EXIT
```

### Installation de GLPI et configuration d'Apache

```bash
cd /tmp
wget https://github.com/glpi-project/glpi/releases/download/11.0.4/glpi-11.0.4.tgz
sudo tar -xzvf glpi-11.0.4.tgz
sudo mv glpi /var/www/html/
sudo chown -R www-data:www-data /var/www/html/glpi
sudo chown -R www-data:www-data /var/www/html/glpi/files
sudo chmod +x /var /var/www /var/www/html /var/www/html/glpi /var/www/html/glpi/public
sudoedit /etc/apache2/sites-available/glpi.conf
```

**Virtual host :**

```apache
<VirtualHost *:8080>
    ServerName glpi.local.ecocert2.fr
    DocumentRoot /var/www/html/glpi/public
    <Directory /var/www/html/glpi/public>
        Require all granted
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>
</VirtualHost>
```

Écoute du port 8080 dans `/etc/apache2/ports.conf` :

```bash
sudoedit /etc/apache2/ports.conf
```

```apache
Listen 8080
```

Activation du site et des modules :

```bash
sudo a2ensite glpi.conf
sudo a2dissite 000-default.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php8.4-fpm
sudo systemctl reload apache2
```

### Sécurisation de PHP

```bash
sudoedit /etc/php/8.4/fpm/php.ini
```

Dans ce fichier, rechercher l'option `session.cookie_httponly` (CTRL+W avec nano) et indiquer la valeur `on` pour l'activer, afin de protéger les cookies de GLPI.

```ini
; Whether or not to add the httpOnly flag to the cookie, which makes it
; inaccessible to browser scripting languages such as JavaScript.
; https://php.net/session.cookie-httponly
session.cookie_httponly = on
```

Toujours pour renforcer la sécurité, configurer la directive `session.cookie_samesite` avec la valeur `Lax` conseillée par la documentation de GLPI. Elle contrôle la façon dont le navigateur envoie le cookie de session, ce qui bloque notamment certaines attaques CSRF (Cross-Site Request Forgery).

```ini
; Add SameSite attribute to cookie to help mitigate Cross-Site Request Forgery (CSRF/XSRF)
; Current valid values are "Strict", "Lax" or "None". When using "None",
; make sure to include the quotes, as `none` is interpreted like `false` in ini files.
; https://tools.ietf.org/html/draft-west-first-party-cookies-07
session.cookie_samesite = Lax
```

Enregistrer le fichier. Par la suite, d'autres modifications pourront être nécessaires, notamment pour augmenter la taille des uploads sur GLPI (`upload_max_filesize` limité à 2 Mo par défaut), etc. La directive `session.cookie_secure` devra aussi passer sur `on` une fois GLPI accessible en HTTPS.

Pour appliquer les modifications, redémarrer PHP-FPM :

```bash
sudo systemctl restart php8.4-fpm.service
```

### Finalisation via l'assistant web

> **Complément :** étapes non présentes dans le document source.

**1. Vérifier que les services tournent et que le port 8080 est en écoute :**

```bash
sudo systemctl status apache2 php8.4-fpm mariadb
sudo ss -tlnp | grep 8080
```

**2. Lancer l'assistant d'installation** depuis un navigateur : `http://172.16.52.40:8080`

1. Choisir la langue puis accepter la licence.
2. Cliquer sur **Installer** et vérifier que tous les prérequis sont validés.
3. Renseigner la connexion à la base de données :

| Paramètre | Valeur |
|---|---|
| Serveur SQL | localhost |
| Utilisateur SQL | adm_glpiecocert |
| Mot de passe SQL | *(celui défini dans le `GRANT`)* |
| Base de données | dbglpi_ecocert |

4. Laisser l'assistant initialiser la base, puis terminer l'installation.

**3. Sécuriser les comptes par défaut :** GLPI crée des comptes de démonstration (`glpi`, `tech`, `normal`, `post-only`). Il faut changer leurs mots de passe ou les désactiver.

**4. Créer les deux administrateurs des tickets** (*Administration → Utilisateurs*) avec le profil **Super-Admin** :

| Utilisateur | Rôle |
|---|---|
| Olivier TONDET | Responsable des solutions techniques d'accès et des éléments d'interconnexion |
| Pamela TREMO | Responsable des services et des systèmes serveurs |

**5. Activer l'inventaire automatique** (*Configuration → Inventaire*), puis installer l'agent GLPI sur les postes de travail en le pointant vers le serveur (URL d'inventaire généralement de la forme `http://172.16.52.40:8080/front/inventory.php`, à vérifier selon la configuration).

---

## Mise en place du serveur NAS

### Configuration réseau de TrueNAS

**Configurer l'IP du TrueNAS en statique :**

| Paramètre | Valeur |
|---|---|
| name | ens18 |
| description | *(vide)* |
| ipv4_dhcp | No |
| ipv6_auto | No |
| aliases | 172.16.52.20/24 |
| mtu | *(non défini)* |

**Configurer le hostname, la gateway et les DNS du TrueNAS :**

| Paramètre | Valeur |
|---|---|
| hostname | NASECOCERT |
| domain | local |
| ipv4gateway | 172.16.52.253 |
| ipv6gateway | *(vide)* |
| nameserver1 | 172.16.52.1 |
| nameserver2 | 172.16.20.11 |
| nameserver3 | 8.8.8.8 |

### Connexion à l'interface web

- URL : `http://172.16.52.20/ui/signin`
- Nom d'utilisateur : `truenas_admin`

> **Complément :** la page de connexion signale une connexion non sécurisée (HTTP). Il est recommandé de basculer l'interface en HTTPS une fois la configuration terminée.

### RAID 5

Création du volume depuis *Stockage → Créer un volume* (assistant de création de volume) :

| Paramètre | Valeur |
|---|---|
| Nom du volume | DataPool |
| Layout | RAIDZ1 |
| Taille du disque | 10 GiB (HDD) |
| Largeur | 3 |
| Nombre de VDEVs | 1 |

> **Complément :** le **RAIDZ1** est l'équivalent ZFS du **RAID 5** (1 disque de parité). Avec 3 disques de 10 GiB, la capacité utile est d'environ **20 GiB** (2 disques de données + 1 de parité), ce qui correspond à l'exigence de 20 Go minimum. Cette capacité est légèrement réduite par les réservations ZFS : à vérifier dans le tableau de bord du stockage après création.

### NTP

Même configuration que sur l'AD Windows (synchronisation indispensable pour l'authentification Kerberos de l'Active Directory).

| Adresse | Burst | IBurst | Préférer | Poll Min | Poll Max |
|---|---|---|---|---|---|
| 0.fr.pool.ntp.org | Non | Oui | Non | 6 | 10 |
| 1.fr.pool.ntp.org | Non | Oui | Non | 6 | 10 |

### Active Directory

*Identifiants → Services d'annuaire → Configurer Active Directory*

| Paramètre | Valeur |
|---|---|
| Type d'identifiant | Kerberos User |
| Nom d'utilisateur | NASADMIN |
| Mot de passe | *(masqué)* |
| Nom d'hôte TrueNAS | TrueNAS |
| Nom de domaine | local.ecocert2.fr |

> **Complément :** pour que la jonction au domaine fonctionne, le NAS doit pouvoir résoudre `local.ecocert2.fr` via ses serveurs DNS et avoir une heure synchronisée avec le contrôleur de domaine (d'où le paramétrage NTP ci-dessus). Une fois le service connecté, les utilisateurs et groupes du domaine sont disponibles dans l'éditeur d'ACL (préfixe `LOCAL\`).

### Dataset, ACL et partage

**Créer un dataset** : `/mnt/DataPool/Partage` (nom = `Partage`).

**ACL** (éditeur d'ACL, chemin `/mnt/DataPool/Partage`) :

| Paramètre | Valeur |
|---|---|
| Propriétaire | truenas_admin |
| Groupe propriétaire | LOCAL\utilisateurs du domaine |

| Entrée | Type | Autorisation |
|---|---|---|
| owner@ - truenas_admin | Autoriser | Contrôle total |
| group@ - LOCAL\utilisateurs du domaine | Autoriser | Modifier |
| Utilisateur - LOCAL\nasadmin | Autoriser | Contrôle total |

Options utilisées : types d'autorisations et de flags en mode **Basique**, flag **Hériter**, case **Valider l'ACL effective** cochée, puis **Sauvegarder ACL**.

> **Complément :** pour que le dataset soit accessible depuis les clients Windows, il faut également créer un partage SMB : *Partages → Windows (SMB) Shares → Ajouter*, avec le chemin `/mnt/DataPool/Partage` et le nom `Partage`, puis activer le service SMB.

### Client Windows

Connecter un lecteur réseau sur le partage :

```
\\172.16.52.20\Partage
```

Saisir son utilisateur et son mot de passe pour modifier et ajouter des fichiers selon ses droits.

> **Complément :** *Explorateur → Ce PC → Connecter un lecteur réseau*, cocher « Se connecter avec d'autres informations d'identification » et utiliser un compte du domaine (par exemple `LOCAL\nasadmin`). Pour valider les droits, créer puis modifier un fichier dans le partage avec un compte du groupe « utilisateurs du domaine » (droit *Modifier*).
