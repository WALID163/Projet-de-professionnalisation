# Mise en place dans un environnement virtuel d'un contrôleur de domaine et d'un service DHCP

## Table des matières

1. [Mise en place d'un windows serveur virtuel avec proxmox](#1-mise-en-place-dun-windows-serveur-virtuel-avec-proxmox)
   - 1.2) [Mise en place du service DNS sur windows serveur 2025](#12-mise-en-place-du-service-dns-sur-windows-serveur-2025)
   - 1.3) [Mise en place du service Active Directory (AD) ainsi que la promotion en contrôleur de domaine](#13-mise-en-place-du-service-active-directory-ad-ainsi-que-la-promotion-en-contrôleur-de-domaine)
   - 1.4) [Ajout d'utilisateurs via script](#14-ajout-dutilisateurs-via-script)
2. [Mise en place d'un serveur DHCP sous DEBIAN 13](#2-mise-en-place-dun-serveur-dhcp-sous-debian-13)

---

## 1) Mise en place d'un windows serveur virtuel avec proxmox

Sur Proxmox créer une VM avec une image ISO d'un Windows server version (récent de préférence) ou clonez un template.

Lancer la VM.

Puis se rendre directement dans les paramètres réseau pour l'introduire dans notre infrastructure :

### 1.2) Mise en place du service DNS sur windows serveur 2025

Ouvrir le gestionnaire de serveur puis cliquer sur **gérer** → puis **suivant** jusqu'à choisir le rôle du serveur que l'on souhaite, dans notre cas le service **DNS** sera coché.

→ Puis faire **suivant** jusqu'à l'installation.

### 1.3) Mise en place du service Active Directory (AD) ainsi que la promotion en contrôleur de domaine

Pour promouvoir le serveur en contrôleur de domaine, il sera nécessaire d'ajouter le service Active Directory.

→ De la même manière que pour le serveur DNS, il faudra cliquer sur **gérer** → puis **suivant** jusqu'à choisir le rôle du serveur (**Active Directory**).

→ Puis au niveau du drapeau avec le symbole danger en jaune, l'option de promouvoir le serveur sera présente.

→ Ajouter une nouvelle forêt.

&nbsp;&nbsp;&nbsp;&nbsp;→ Préciser le nom de domaine en question.

### 1.4) Ajout d'utilisateurs via script

Pour peupler l'AD si → beaucoup d'utilisateurs à entrer → l'utilisation d'un script et d'un CSV facilitera la tâche.

Au préalable, dans PowerShell Admin, effectuer :

```powershell
Set-ExecutionPolicy RemoteSigned
```

Puis lancer le script.

→ Se rendre dans l'annuaire et vérifier l'apparition des individus.

---

## 2) Mise en place d'un serveur DHCP sous DEBIAN 13

### 1) Paramètres réseau de la VM

Toujours en premier, vérifier les paramètres réseau de la VM (ne pas oublier `bridge=projet.X` dans hardware).

```bash
cd /etc/network
sudoedit interfaces
```

Faire une configuration qui nous permet d'accéder à internet (dans notre cas).

Puis modifier `resolv.conf` ou créer un fichier `resolv.conf.head` qui écrasera `resolv.conf` :

```bash
cd /etc/
sudoedit resolv.conf.head
```

Mettre n'importe quelle DNS publique ou autre pour avoir une résolution de nom.

### 2) Mise en place d'un service DHCP (Kea DHCP)

**Installation de Kea :**

```bash
sudo apt-get update      # pour mettre à jour notre debian
sudo apt upgrade
sudo apt-get install kea-dhcp4-server
sudo systemctl status kea-dhcp4-server.service
```

`Active` = fonctionne

**Ensuite il s'agira de configurer Kea et d'entrer nos étendues réseau :**

```bash
cp /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.bkp   # faire une backup avant toute modif
sudoedit /etc/kea/kea-dhcp4.conf
```

Conf de base classique : on y met la carte réseau que le serveur doit écouter, les baux, ainsi que la base de données où ils seront stockés.

**Puis entrer les étendues type :**

- `subnet` = le sous-réseau
- `pool` = la plage disponible
- `option data` = regroupe le DNS, le domaine et la passerelle

**Recharger le service Kea :**

```bash
sudo systemctl restart kea-dhcp4-server.service
sudo systemctl status kea-dhcp4-server.service
```

Les services Kea sont actifs.

**En cas d'erreurs :**

```bash
sudo journalctl -xe | grep kea
```

Puis vérifier les logs.
