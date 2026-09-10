# BLOC 2 – Exploitation des services – Activité 0 – Mise en place d'une machine Debian 13

*Crys Boisseau 901 Walid Joual*

## Table des matières

- [Expression des besoins](#expression-des-besoins)
  - [Fiche technique de la VM](#fiche-technique-de-la-vm)
  - [Outils à installer sur la VM](#outils-à-installer-sur-la-vm)
  - [Gestion des logs](#gestion-des-logs)
  - [Système de versioning pour /etc](#système-de-versioning-pour-etc)
- [Authentification multifacteur TOTP](#authentification-multifacteur-totp)

---

## Expression des besoins

Le DSI et le RSSI de l'entreprise CUB souhaitent disposer dans chaque agence d'une machine virtuelle Linux Debian 13 de base conforme au référentiel contenu dans le schéma directeur et qui puisse être clonée dès qu'un serveur devra être mis en production sur la ferme de serveurs Proxmox de l'organisation.

### Fiche technique de la VM

| Paramètre | Valeur |
|---|---|
| Memory | 4.00 GiB |
| Processors | 2 (1 sockets, 2 cores) [x86-64-v2-AES] |
| BIOS | Default (SeaBIOS) |
| Display | Default |
| Machine | Default (i440fx) |
| SCSI Controller | VirtIO SCSI single |
| CD/DVD Drive (ide2) | none,media=cdrom |
| Hard Disk (scsi0) | zfs-1:vm-20402-disk-0,iothread=1,size=50G |
| Network Device (net0) | virtio=BC:24:11:26:8A:0F,bridge=ProjetB |

| Paramètre | Valeur |
|---|---|
| Name | BLOC2-ExploitServ-Modele-Debian13 |
| Start at boot | No |
| Start/Shutdown order | order=any |
| OS Type | Linux 7.x - 2.6 Kernel |
| Boot Order | scsi0, ide2, net0 |
| Use tablet for pointer | Yes |
| Hotplug | Disk, Network, USB |
| ACPI support | Yes |
| KVM hardware virtualization | Yes |
| Freeze CPU at startup | No |
| Use local time for RTC | Default (Enabled for Windows) |
| RTC start date | now |
| SMBIOS settings (type1) | uuid=640a2db9-c5d2-46f0-b028-adc91285271b |
| QEMU Guest Agent | Default (Disabled) |
| Protection | No |
| Spice Enhancements | none |
| VM State storage | Automatic |
| AMD SEV | Default (Disabled) |
| Intel TDX | Default (Disabled) |

### Outils à installer sur la VM

```bash
etudiant@template:~$ sudo apt install htop tcpdump tmux && sudo apt upgrade
```

Afin de vérifier que htop est bien installé, taper la commande :

```bash
etudiant@template:~$ htop
```

Pour tcpdump :

```bash
etudiant@template:~$ sudo tcpdump
```

Pour tmux :

```bash
etudiant@template:~$ sudo tmux
```

### Gestion des logs

La gestion des logs devra se faire via des fichiers texte présents dans `/var/log/` et par le daemon rsyslog.

```bash
etudiant@template:~$ sudo apt install rsyslog
```

Activer daemon rsyslog et vérifier son status.

```bash
etudiant@template:~$ sudo systemctl start rsyslog
etudiant@template:~$ sudo systemctl status rsyslog
```

### Système de versioning pour /etc

Le répertoire système `/etc` devra pouvoir bénéficier du système de versioning nommé etckeeper.

```bash
etudiant@template:~$ sudo apt install etckeeper
etudiant@template:~$ etckeeper --version
Version: 1.18.22
```

---

## Authentification multifacteur TOTP

La connexion SSH avec le compte etudiant devra se faire à l'aide d'une authentification multifacteur TOTP. Un accès SSH avec authentification par simple mot de passe devra être possible avec l'utilisateur **adminbastion** (cet utilisateur doit pouvoir utiliser toutes les commandes systèmes avec l'outil sudo).

```bash
sudo apt install qrencode
```

On s'assurera également que la connexion SSH entre le poste client et le serveur Debian est pleinement opérationnelle. Si ce n'est pas le cas, il sera impératif d'installer le service OpenSSH sur le serveur Debian 12 (`sudo apt install openssh-server`).

```bash
ssh etudiant@192.168.1.90
etudiant@192.168.1.90's password:
Linux serveur 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x...

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Jan  3 19:34:10 2024 from 192.168.1.85
etudiant@serveur:~$
```

Dans un premier temps, il sera nécessaire d'installer les paquets permettant de mettre en œuvre le mécanisme d'OTP nommé OATH sur le serveur.

```bash
sudo apt install libpam-oath oathtool
```

Puis, nous allons définir un secret sous forme hexadécimal qui sera utilisé par le générateur TOTP/HOTP en lien avec l'utilisateur étudiant et qui sera soumis à la double authentification.

```bash
sudo -i
KEY=$(openssl rand -hex 20)
echo "HOTP/T30/6 etudiant - ${KEY}" >> /etc/security/users.oath
chown root /etc/security/users.oath
chown 600 /etc/security/users.oath
```

Nous allons maintenant configurer PAM (Pluggable Authentication Modules), le service qui contrôle les authentifications sur le serveur Debian.

```bash
nano /etc/pam.d/sshd
```

```
# PAM configuration for the Secure Shell service

# Standard Un*x authentication.
#@include common-auth

auth required pam_unix.so nullok_secure
auth required pam_oath.so usersfile=/etc/security/users.oath window=30 digits=6
```

Nous commentons la ligne `@include common-auth` car elle empêche l'authentification OTP même en cas de connexion avec le bon mot de passe.

La ligne suivante impose l'authentification par mot de passe stocké en local sur le système et interdit les mots de passe vides.

La dernière impose une fois l'authentification par mot de passe réussie, une deuxième authentification par OTP. Nous faisons référence au fichier contenant le (ou les) nom d'utilisateur concerné ainsi que le secret servant à générer des mots de passe à usage unique. Ce mot de passe disposera de 6 chiffres et aura une durée de validité de 30 secondes.

Si vous avez un utilisateur, par exemple, **adminbastion** et que vous souhaitez uniquement une simple authentification par mot de passe pour lui et un mot de passe + code à usage unique pour les autres, il faudra ajouter une ligne supplémentaire à la configuration initiale proposée dans `/etc/pam.d/sshd`.

```
# PAM configuration for the Secure Shell service

# Standard Un*x authentication.
#@include common-auth

auth required pam_unix.so nullok_secure
auth [success=1 default=ignore] pam_succeed_if.so user = adminbastion
auth required pam_oath.so usersfile=/etc/security/users.oath window=30 digits=6
```

**La 2ème ligne permet de dire que si l'utilisateur adminbastion a correctement saisi son mot de passe alors PAM ignore l'authentification TOTP avec OATH.**

Il nous reste maintenant à éditer le fichier de configuration du service SSH afin de définir l'usage de l'authentification 2FA.

```bash
nano /etc/ssh/sshd_config
```

```
ChallengeResponseAuthentication yes
#KbdInteractiveAuthentication no
UsePAM yes
```

Nous nous assurons de commenter la ligne `KbdInteractiveAuthentication` et d'avoir les deux autres lignes activées avec la valeur `yes`. Nous pouvons ensuite redémarrer le service SSH.

```bash
systemctl restart ssh
```

Enfin, il nous reste à récupérer le secret en base 32 qui nous permettra ensuite de générer sur le poste client un QR code pour notre application Android.

```bash
cat /etc/security/users.oath
```

```
HOTP/T30/6 etudiant - 65f43c705ce51c9c058ec8bb4b7f64b656681866
root@serveur:~# oathtool -v -d 6 65f43c705ce51c9c058ec8bb4b7f64b656681866
Hex secret: 65f43c705ce51c9c058ec8bb4b7f64b656681866
Base32 secret: MX2DY4C44UOJYBMOZC5UW73EWZLGQGDG
Digits: 6
Window size: 0
Start counter: 0x0 (0)
```

Il s'agit maintenant de paramétrer correctement la machine cliente et l'application Android FreeOTP+ afin de rendre opérationnel l'authentification SSH 2FA.

Nous allons d'abord générer sur le poste client (Ubuntu ou Kali) un fichier png contenant un QR code que nous soumettrons à l'application FreeOTP+.

```bash
qrencode -o etudiant.png 'otpauth://totp/etudiant@192.168.1.90?secret='
```

```bash
ls -l
...
-rw-r--r-- 1 etudiant etudiant      471  3 janv. 23:08 etudiant.png
```

Nous pouvons ouvrir ce fichier PNG sur le poste client puis ouvrir l'application FreeOTP+ sur le smartphone.

Sélectionner l'icône « Appareil photo » en bas à droite. Puis prenez en photo le QR code présent sur l'écran du client. Une nouvelle configuration pour votre utilisateur et votre serveur est automatiquement créée.

Nous pouvons ensuite supprimer le fichier PNG contenant le QR code, car il contient le secret à ne pas compromettre.

Sur le client, nous pouvons lancer une connexion SSH vers le serveur avec le compte etudiant. Après avoir entré votre mot de passe, un OTP vous est demandé. Dans l'application FreeOTP+, sélectionnez la nouvelle configuration. Celle-ci vous fournit un code de 6 chiffres valable 30 secondes.

```bash
ssh etudiant@192.168.1.90
(etudiant@192.168.1.90) Password:
(etudiant@192.168.1.90) One-time password (OATH) for `etudiant`:
Linux serveur 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x...

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Jan  3 22:17:13 2024 from 192.168.1.85
etudiant@serveur:~$
```
