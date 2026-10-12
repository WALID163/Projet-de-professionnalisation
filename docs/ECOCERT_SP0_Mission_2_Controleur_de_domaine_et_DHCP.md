# ECOCERT – SP0 – Mission 2 : Mise en place dans un environnement virtuel d'un contrôleur de domaine et d'un service DHCP

**Auteur :** Walid Joual
**Date :** 12/10/2026
**Périmètre :** déploiement d'un Windows Server 2025 virtuel sous Proxmox (DNS, Active Directory, promotion en contrôleur de domaine), puis d'un serveur DHCP Kea sous Debian 13.

> **Note :** les captures d'écran du document source sont décrites ou retranscrites sous forme de blocs de code. Les valeurs illisibles à l'écran sont signalées.

## Paramètres de l'infrastructure

- Hyperviseur : Proxmox (nœud `pve2`), VM `20403 (SIO2-AP-GP2-ADECOCERT)`
- Serveur Windows : Windows Server 2025, nom `WIN-LRT7KS9E9EC`
- Domaine : `local.ecocert2.fr` (nouvelle forêt)
- Serveur DHCP : Debian 13, interface `ens18`, IP `172.16.52.2/24`, passerelle `172.16.52.254`
- DHCP : Kea (`kea-dhcp4-server`)

---

## 1. Mise en place d'un Windows Server virtuel avec Proxmox

1. Sur Proxmox, créer une VM avec l'image ISO d'un Windows Server (version récente de préférence) **ou** cloner un template.
2. Lancer la VM.
3. Se rendre directement dans les paramètres réseau pour l'introduire dans l'infrastructure : *Propriétés de : Protocole Internet version 4 (TCP/IPv4)* → « Utiliser l'adresse IP suivante ».

Paramètres visibles sur la capture :

| Champ | Valeur |
|---|---|
| Adresse IP | `172.16.52.x` (dernier octet peu lisible) |
| Masque de sous-réseau | `255.255.255.0` |
| Passerelle par défaut | `172.16.52.254` |
| Serveur DNS préféré | `8.8.8.8` (peu lisible) |

### 1.2 Mise en place du service DNS sur Windows Server 2025

1. Ouvrir le **Gestionnaire de serveur**, puis cliquer sur **Gérer**.
2. Cliquer sur *Suivant* jusqu'au choix des rôles du serveur : cocher **DNS**.
3. Cliquer sur *Suivant* jusqu'à l'installation.

### 1.3 Mise en place du service Active Directory (AD) et promotion en contrôleur de domaine

Pour promouvoir le serveur en contrôleur de domaine, il faut d'abord ajouter le rôle **Active Directory**.

1. De la même manière que pour le DNS : **Gérer** → *Suivant* jusqu'au choix du rôle **Services de domaine Active Directory**.
2. Au niveau du **drapeau avec le symbole danger jaune**, l'option *Promouvoir ce serveur en contrôleur de domaine* apparaît (configuration post-déploiement requise pour `WIN-LRT7KS9E9EC`).
3. Dans l'assistant de configuration : **Ajouter une nouvelle forêt**.
4. Préciser le nom de domaine racine : `local.ecocert2.fr`.

### 1.4 Ajout d'utilisateurs via script

Pour peupler l'AD avec beaucoup d'utilisateurs, l'utilisation d'un script et d'un CSV facilite la tâche.

1. Au préalable, dans PowerShell en administrateur :

```powershell
Set-ExecutionPolicy RemoteSigned
```

2. Lancer le script.
3. Se rendre dans l'annuaire et vérifier l'apparition des utilisateurs.

---

## 2. Mise en place d'un serveur DHCP sous Debian 13

### 2.1 Paramètres réseau de la VM

Toujours en premier, vérifier les paramètres réseau de la VM (ne pas oublier `bridge=projet.X` dans *Hardware*).

```bash
cd /etc/network
sudoedit interfaces
```

Faire une configuration qui permet d'accéder à Internet (dans notre cas) :

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens18
iface ens18 inet static
        address 172.16.52.2
        netmask 255.255.255.0
        gateway 172.16.52.254
```

Puis modifier `resolv.conf`, ou créer un fichier `resolv.conf.head` qui écrasera `resolv.conf` :

```bash
cd /etc/
sudoedit resolv.conf.head
```

Y mettre n'importe quel DNS public (ou autre) pour avoir une résolution de noms.

### 2.2 Mise en place d'un service DHCP (Kea DHCP)

**Installation de Kea :**

```bash
sudo apt-get update            # met à jour Debian
sudo apt upgrade
sudo apt-get install kea-dhcp4-server
sudo systemctl status kea-dhcp4-server.service
```

Extrait de la sortie : `Active: active (running)` (Kea DHCPv4 version 2.6.3). **Active = fonctionne.**

> À ce stade, le journal indique `DHCPSRV_NO_SOCKETS_OPEN` (aucune interface configurée) et le sous-réseau par défaut `192.0.2.0/24` : c'est normal, la configuration reste à faire.

**Configuration de Kea et entrée des étendues réseau :**

```bash
cp /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.bkp   # backup avant toute modification
sudoedit /etc/kea/kea-dhcp4.conf
```

**Configuration de base classique** : on y indique la carte réseau que le serveur doit écouter, les durées de bail et la base de données où ils sont stockés.

```json
"Dhcp4": {

    "interfaces-config": {
        "interfaces": [ "ens18" ]
    },

    "valid-lifetime": 691200,
    "renew-timer": 345600,
    "rebind-timer": 604800,

    "authoritative": true,

    "lease-database": {
        "type": "memfile",
        "persist": true,
        "name": "/var/lib/kea/kea-leases4.csv",
        "lfc-interval": 3600
    }
}
```

**Étendues type :**

- `subnet` = le sous-réseau
- `pool` = la plage disponible
- `option-data` = regroupe le DNS, le domaine et la passerelle

```json
##------------------------------------------------------------
##  VLAN 11 ADMINISTRATION
##------------------------------------------------------------

"subnet4": [
 {
  "id": 11,
  "subnet": "192.168.2.0/26",
  "pools": [
   { "pool": "192.168.2.2 - 192.168.2.62" }
  ],
  "option-data": [
   {
    "name": "domain-name-servers",
    "data": "172.16.52.1"
   },
   {
    "name": "domain-search",
    "data": "local.ecocert2.fr"
   },
   {
    "name": "routers",
    "data": "192.168.2.1"
   }
  ]
 },
##------------------------------------------------------------
##  VLAN 21 SERVICE DE CERTIFICATION
##------------------------------------------------------------
```

### 2.3 Recharger le service Kea

```bash
sudo systemctl restart kea-dhcp4-server.service
sudo systemctl status kea-dhcp4-server.service
```

Le statut affiche `active (running)` : **les services Kea sont actifs.**

**En cas d'erreur :**

```bash
sudo journalctl -xe | grep kea
```

Puis vérifier les logs.

---

> ## 📋 Résumé du fonctionnement – SP0 Mission 2
>
> **Objectif :** mettre en place, dans un environnement virtuel Proxmox, un contrôleur de domaine Windows et un serveur DHCP Debian.
>
> **Déroulement :**
> 1. Création (ou clonage) d'une VM Windows Server 2025 sous Proxmox, puis configuration de son adresse IP fixe.
> 2. Installation des rôles **DNS** et **Active Directory**, puis promotion en contrôleur de domaine avec création d'une nouvelle forêt `local.ecocert2.fr`.
> 3. Ajout des utilisateurs en masse via un script PowerShell et un CSV (après `Set-ExecutionPolicy RemoteSigned`), puis vérification dans l'annuaire.
> 4. Sur Debian 13 : configuration réseau statique (`ens18`, `172.16.52.2/24`) et DNS via `resolv.conf.head`.
> 5. Installation de `kea-dhcp4-server`, sauvegarde du fichier de configuration, puis déclaration de l'interface, des baux et des étendues (ex. VLAN 11 Administration : `192.168.2.0/26`, plage `.2` à `.62`, DNS `172.16.52.1`, domaine `local.ecocert2.fr`, passerelle `192.168.2.1`).
> 6. Redémarrage du service et contrôle avec `systemctl status` ; en cas d'erreur, consulter `journalctl`.
>
> **Points clés :** faire une copie de sauvegarde (`.bkp`) avant toute modification de la configuration Kea, et vérifier que le DHCP écoute bien la bonne interface réseau.
