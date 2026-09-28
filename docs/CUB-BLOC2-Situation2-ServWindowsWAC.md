# CONTEXTE CUB — BLOC 2 – Administration des systèmes

*Crys 901 Boisseau*

## Table des matières

- [Partie 1 – Installation du serveur Windows 2025 avec bureau pour WAC](#partie-1--installation-du-serveur-windows-2025-avec-bureau-pour-wac)
  1. [Créer une VM Windows serveur 2025 avec bureau](#1-créer-une-vm-windows-serveur-2025-avec-bureau)
  2. [Réaliser un « sysprep » de votre serveur pour réinitialiser les SID](#2-réaliser-un--sysprep--de-votre-serveur-pour-réinitialiser-les-sid)
  3. [Modifier et vérifier la configuration IP de votre VM](#3-modifier-et-vérifier-la-configuration-ip-de-votre-vm)
  4. [Réaliser les modifications de base du système](#4-réaliser-les-modifications-de-base-du-système)
- [Partie 2 – Installation de Windows Admin Center (WAC)](#partie-2--installation-de-windows-admin-center-wac)
  5. [Suivre la procédure – Fiche de procédure 2 : WAC – Windows Admin Center](#5-suivre-la-procédure--fiche-de-procédure-2--wac--windows-admin-center)
  6. [Ajouter les serveurs Windows dans WAC](#6-ajouter-les-serveurs-windows-dans-wac)
  7. [Créer un compte « administrateurWAC1 » pour administrer le WAC](#7-créer-un-compte--administrateurwac1--pour-administrer-le-wac-mot-de-passe--etudiant_007)
  8. [Vérifier via WAC les éléments suivants](#8-vérifier-via-wac-les-éléments-suivants)

---

## Partie 1 – Installation du serveur Windows 2025 avec bureau pour WAC :

### 1. Créer une VM Windows serveur 2025 avec bureau :

**Virtual Machine 20505 (BLOC2-AdminSys-WAC1) on node 'pve2'**

| Paramètre | Valeur |
|---|---|
| Memory | 4.00 GiB |
| Processors | 1 (1 sockets, 1 cores) [x86-64-v2-AES] |
| BIOS | OVMF (UEFI) |
| Display | Default |
| Machine | pc-q35-10.1 |
| SCSI Controller | VirtIO SCSI single |
| CD/DVD Drive (ide0) | none,media=cdrom |
| CD/DVD Drive (ide2) | none,media=cdrom |
| Hard Disk (scsi0) | zfs-1:vm-20505-disk-1,iothread=1,size=32G |
| Network Device (net0) | virtio=BC:24:11:DC:95:02,bridge=ProjetB,firewall=1 |
| EFI Disk | zfs-1:vm-20505-disk-0,efitype=4m,ms-cert=2023k,pre-enrolled-keys=1,size=1M |
| TPM State | zfs-1:vm-20505-disk-2,size=4M,version=v2.0 |

### 2. Réaliser un « sysprep » de votre serveur pour réinitialiser les SID :

Le lancement direct de `sysprep.exe` échoue (la commande n'est pas reconnue), car PowerShell ne charge aucune commande depuis l'emplacement actuel. Il faut donc le lancer avec `.\`, depuis son répertoire :

```
PS C:\Windows\System32\Sysprep> sysprep.exe
sysprep.exe : Le terme «sysprep.exe» n'est pas reconnu comme nom d'applet de commande, fonction,
fichier de script ou programme exécutable. Vérifiez l'orthographe du nom, ou si un chemin d'accès
existe, vérifiez que le chemin d'accès est correct et réessayez.
Au caractère Ligne:1 : 1
+ sysprep.exe
+ ~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (sysprep.exe:String)
    + FullyQualifiedErrorId : CommandNotFoundException

Suggestion [3,General]: La commande sysprep.exe est introuvable, mais elle existe à l'emplacement
actuel. Windows PowerShell ne charge aucune commande à partir de l'emplacement actuel par défaut.
Si vous approuvez cette commande, entrez « .\sysprep.exe » à la place. Pour en savoir plus, voir
"get-help about_Command_Precedence".

PS C:\Windows\System32\Sysprep> .\sysprep.exe
PS C:\Windows\System32\Sysprep>
```

**Outil de préparation du système (Sysprep) v3.14 :**

| Paramètre | Valeur |
|---|---|
| Action de nettoyage du système | Entrer en mode OOBE (Out-of-Box Experience) |
| Généraliser | Non coché |
| Options d'extinction | Redémarrer |

### 3. Modifier et vérifier la configuration IP de votre VM :

**Propriétés de : Protocole Internet version 4 (TCP/IPv4)**

- Option sélectionnée : **Utiliser l'adresse IP suivante**

| Paramètre | Valeur |
|---|---|
| Adresse IP | 192.168.8.6 |
| Masque de sous-réseau | 255.255.255.128 |
| Passerelle par défaut | 192.168.8.126 |

### 4. Réaliser les modifications de base du système :

#### 4.1. Mettre à jour le système :

```
PS C:\WINDOWS\system32> install-module PSWindowsUpdate
PS C:\WINDOWS\system32> Get-WindowsUpdate
PS C:\WINDOWS\system32> Install-WindowsUpdate
```

#### 4.2. Modifier le nom du serveur :

```
PS C:\WINDOWS\system32> Rename-Computer

applet de commande Rename-Computer à la position 1 du pipeline de la commande
Fournissez des valeurs pour les paramètres suivants :
NewName: ServeurWAC2
```

#### 4.3. Modifier le nom de l'utilisateur « Administrateur » local par « ADM-SRV-WAC » et un mot de passe respectant les recommandations de l'ANSSI :

```
PS C:\WINDOWS\system32> Rename-LocalUser -Name Administrateur -NewName ADM-SRV-WAC
PS C:\WINDOWS\system32> Set-LocalUser -Name ADM-SRV-WAC -Password (Get-Credential).password

applet de commande Get-Credential à la position 1 du pipeline de la commande
Fournissez des valeurs pour les paramètres suivants :
Credential
```

Une fenêtre « Demande d'informations d'identification » s'ouvre alors, demandant le nom d'utilisateur (**ADM-SRV-WAC**) et le nouveau mot de passe à définir.

#### 4.4. Modifier le serveur de temps pour être synchronisé sur le même que vos autres serveurs Windows (« 0.fr.pool.ntp.org 1.fr.pool.ntp.org » et forcer une première synchronisation) :

```
PS C:\WINDOWS\system32> w32tm /config /manualpeerlist:"0.fr.pool.ntp.org 1.fr.pool.ntp.org" /syncfromflags:manual /reliable:yes /update
La commande s'est terminée correctement.
PS C:\WINDOWS\system32>
```

---

## Partie 2 – Installation de Windows Admin Center (WAC) :

### 5. Suivre la procédure – Fiche de procédure 2 : WAC – Windows Admin Center :

**5) Windows Administration Center**

Menu démarrer → se rendre sur **Windows Admin Center** → lancer **Windows Admin Center setup** :

1. Installation personnalisée
2. Accès distant
3. Formulaire HTML
4. Port : `10443`
5. Certificat TLS auto-signé 60 jours
6. Nom de domaine : `wacX.local.barcelone.cub.`
7. Autoriser l'accès à n'importe quel ordinateur
8. WinRM sur HTTPS = HTTP, mécanisme de communication par défaut
9. MàJ auto = installer les mises à jour automatiquement

Depuis l'interface WAC :

- Ajouter → Nom du serveur → Utiliser un autre compte pour cette connexion → entrer les identifiants.

Afin de tester si la connexion fonctionne, se connecter à n'importe quel serveur à partir de l'onglet PowerShell.

### 6. Ajouter les serveurs Windows dans WAC :

Interface WAC : `https://wac1.local.barcelone.cub.sioplc.fr:10443` — *5 élément(s)*

| Nom | Type | Dernier connecté | Gestion en tant que | État Azure Arc |
|---|---|---|---|---|
| ServeurAD0 | Serveurs | Jamais | ADM-SRV-00 | Inconnu |
| ServeurAD1 | Serveurs | Jamais | ADM-SRV-00 | Inconnu |
| ServeurDHCP0 | Serveurs | 16/09/2026 17:28:36 | ADM-SRV-00 | Non installé(e) |
| ServeurDHCP1 | Serveurs | Jamais | ADM-SRV-00 | Inconnu |
| serveurwac2 [Passerelle] | Serveurs | Jamais | ADM-SRV-00 | Inconnu |

### 7. Créer un compte « administrateurWAC1 » pour administrer le WAC (mot de passe : etudiant_007) :

`ADM-SRV-WAC` → `etudiant_007`

### 8. Vérifier via WAC les éléments suivants :

#### État des mises à jour :

Paramètres → Mises à jour : **Mises à jour du Centre d'administration Windows — À jour** ✅

#### Paramétrage réseaux des serveurs :

**serveurwac2 → Pare-feu → Vue d'ensemble**

| Nom | Statut | Action entrante par défaut | Action sortante par défaut |
|---|---|---|---|
| Domain | Activé | Bloquer | Autoriser |
| Private | Activé | Bloquer | Autoriser |
| Public | Activé | Bloquer | Autoriser |

#### Configuration réseaux des serveurs :

**serveurwac2 → Réseaux** (1 élément)

| Nom | Description | Statut | Adresse IPv4 |
|---|---|---|---|
| Ethernet | Red Hat VirtIO Ethernet Adapter | Actif | 192.168.2.6 |

#### Rôles et fonctionnalités installées :

**serveurwac2 → Rôles et fonctionnalités** (264 éléments)

| Nom | État | Type |
|---|---|---|
| Rôles | 1 sur 90 installé(s) | |
| Accès à distance | 0 sur 3 installé(s) | Role |
| Attestation d'intégrité de l'appareil | Disponible | Role |
| Hyper-V | Disponible | Role |
| Serveur de télécopie | Disponible | Role |
| Serveur DHCP | Disponible | Role |

#### Listes des utilisateurs et des groupes paramétrés :

**serveurwac2 → Utilisateurs et groupes locaux → Utilisateurs** (4 éléments)

| Nom | Description |
|---|---|
| ADM-SRV-WAC | Compte d'utilisateur d'administration |
| DefaultAccount | Compte utilisateur géré par le système. |
| Invité | Compte d'utilisateur invité |
| WDAGUtilityAccount | Compte d'utilisateur géré et utilisé par le système pour le scénario… |
