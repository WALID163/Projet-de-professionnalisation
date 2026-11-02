# Activité Professionnelles

## 4ème Mission : Mise en place d'une partie de l'architecture réseau

## Table des matières

1. [Mise en place du firewall virtuel](#1-mise-en-place-du-firewall-virtuel)
2. [Mise en place de la borne wifi Ubiquiti](#2-mise-en-place-de-la-borne-wifi-ubiquiti)
   - [Installation des dépendances](#installation-des-dépendances)
   - [Téléchargement de l'installeur](#téléchargement-de-linstalleur)
   - [Exécution de l'installeur](#exécution-de-linstalleur)
   - [Post-installation](#post-installation)
   - [Adoption](#adoption)
   - [Paramétrage du contrôleur et configuration de la borne](#paramétrage-du-contrôleur-et-configuration-de-la-borne)
   - [Configuration switch L3](#configuration-switch-l3)

---

## 1) Mise en place du firewall virtuel

Copier un template → vérifier les cartes réseaux IN & OUT

| Paramètre | Valeur |
|---|---|
| Memory | 2.00 GiB |
| Processors | 1 (1 sockets, 1 cores) [x86-64-v2-AES] |
| BIOS | Default (SeaBIOS) |
| Display | Default |
| Machine | Default (i440fx) |
| SCSI Controller | VirtIO SCSI single |
| Hard Disk (ide0) | zfs-1:vm-20406-disk-0,size=10241M |
| CD/DVD Drive (ide2) | none,media=cdrom |
| Network Device (net0) | e1000=BC:24:11:8D:D7:95,bridge=WanSIO,firewall=1 |
| Network Device (net1) | e1000=BC:24:11:8D:15:77,bridge=ProjetD,firewall=1 |

Pour pouvoir accéder à l'interface WEB, le côté IN du Firewall devra être dans notre réseau.

Donc, depuis le terminal du Stormshield :

→ `vi ConfigFiles/network`

→ trouver l'interface ethernet correspondant au IN

→ Appuyer sur la touche `i` pour entrer en insertion, `Échap` pour quitter et `x` pour supprimer

→ Une fois modifié, taper `:wq`

→ `ennetwork`

### Configuration

→ Changer les paramètres généraux d'abord (nom, langue, date)

*(Capture : Système / Configuration — nom du firewall `StormshieldVIRTecocert`, langue et clavier en français, fuseau horaire Europe/Paris, serveurs NTP `ntp1.stormshieldcs.eu` et `ntp2.stormshieldcs.eu`.)*

Ensuite les paramètres réseau :

→ **Les interfaces**

| Interface | Port | Type | Adresse IPv4 | Commentaire |
|---|---|---|---|---|
| out | 1 | Ethernet, 1 Gbit/s | 172.16.32.2/24 | WANSIO |
| in | 2 | Ethernet, 1 Gbit/s | 192.168.22.253/30 | |

→ **Le routage**

- Passerelle par défaut (routeur) : `ECOCERT_OUT`
- Route statique : réseau de destination `RESEAU_INTERNE`, interface `in`, plan d'adressage `192.168.2.0/24`, passerelle `PasserelleL3`

La machine par défaut qui va servir de passerelle vers le WAN, ainsi que la route (IN) avec l'étendue réseau qui couvre nos VLAN (toujours créer de nouveaux objets si possible).

Une autre route également pour le VLAN SRV.

### Filtrage

Puis le filtrage (pour l'instant en **pass all**) : une seule règle, action *passer*, source *Any*, destination *Any*, port *Any*.

### NAT

**Nouvelle règle (masquerading)**

On réutilise l'objet qui englobe le réseau interne (déjà créé pour le routage), on l'applique à la source et, après translation, on applique le côté OUT du firewall en source.

On crée également une règle pour le réseau d'interconnexion entre le L3 et le firewall.

---

## 2) Mise en place de la borne wifi Ubiquiti

### Installation des dépendances

3.1. Mise à jour et installation des paquets requis. Actualise les dépôts et installe les composants nécessaires à la conteneurisation du service.

```bash
sudo apt-get update && sudo apt-get install podman slirp4netns
```

- `podman` : moteur de conteneurs sans démon central (daemonless), utilisé ici pour isoler UniFi OS.
- `slirp4netns` : outil gérant la mise en réseau des conteneurs Podman lancés dans l'espace utilisateur (rootless).

### Téléchargement de l'installeur

Récupération du binaire. Télécharge l'archive exécutable du serveur UniFi OS depuis les serveurs officiels.

```bash
cd /tmp
curl -O https://fw-download.ubnt.com/data/unifi-os-server/5172-linux-x64-5.1.42-12e9e3cf-8f8b-4e54-928c-76b80a10c8a4.42-x64
```

- `curl` : outil en ligne de commande de transfert de données réseau.
- `-O` : option (majuscule) ordonnant à `curl` de sauvegarder le fichier avec le même nom que sur le serveur distant.

### Exécution de l'installeur

Ajout des droits et lancement. Rend le binaire téléchargé exécutable, puis initie le processus de déploiement.

```bash
chmod +x 5172-linux-x64-5.1.42-12e9e3cf-8f8b-4e54-928c-76b80a10c8a4.42-x64
sudo ./5172-linux-x64-5.1.42-12e9e3cf-8f8b-4e54-928c-76b80a10c8a4.42-x64
```

### Post-installation

Vérification du fonctionnement du service `uosserver`. Vérifier le bon fonctionnement du contrôleur auprès du service `uosserver` sur Debian.

```bash
sudo systemctl status uosserver
```

### Adoption

Brancher la borne sur un port couvert par votre DHCP pour lui forcer une nouvelle IP, car de base son IP est `192.168.1.10`.

Une fois trouvée, il faut se connecter en SSH :

```bash
ssh ubnt@192.168.2.4
# mdp : ubnt
```

Il faudra ensuite lui demander d'envoyer une requête d'adoption vers le contrôleur wifi. Donc :

```bash
set-inform http://172.16.52.30:8080/inform
```

Puis, sur le serveur, il faudra accepter cette demande.

La borne est adoptée par le contrôleur.

### Paramétrage du contrôleur et configuration de la borne

Depuis la roue des paramètres, dans **System** : changer la langue, le format de l'heure et activer *network discovery* si nécessaire.

Dans les paramètres du contrôleur, dans **Réseau**, intégrer les réseaux à distribuer :

| Nom | VLAN ID | Type |
|---|---|---|
| Default | 1 | Gateway tiers |
| TECHNIQUE | 71 | Gateway tiers |
| WIFI-VISITEURS | 81 | Gateway tiers |

Il suffit de cliquer sur **Créer** et d'indiquer le nom ainsi que le VLAN ID.

Puis dans l'onglet **WiFi** : on crée les 2 SSID qui seront déployés.

| Nom | Réseau | APs diffuseurs | Bande radio | Sécurité |
|---|---|---|---|---|
| TECHNIQUE-GP2 | TECHNIQUE (71) | Tous les APs | 2,4 GHz / 5 GHz | WPA2 |
| WIFI-VISITEURS-GP2 | WIFI-VISITEURS (81) | Tous les APs | 2,4 GHz / 5 GHz | WPA2 |

Une fois les modifications apportées, il suffit de rechanger l'IP de la borne (paramètres IP, configuration **Statique**) :

| Paramètre | Valeur |
|---|---|
| Adresse IP | 192.168.1.2 |
| Masque de sous-réseau | 255.255.255.0 |
| Gateway | 192.168.1.254 |
| DNS préféré | 9.9.9.9 |
| DNS alternatif | 8.8.8.8 |

### Configuration switch L3

Pour que la borne wifi distribue les deux SSID, il va falloir modifier le cœur de réseau et utiliser le VLAN natif d'un de ses ports.

Pour pouvoir utiliser le VLAN 1, on lui attribue un réseau pour la borne, donc ici :

```
interface vlan 1
 ip address 192.168.1.254 255.255.255.0
 no shutdown
```

> Pas besoin d'`ip helper` ou quoi que ce soit : c'est la passerelle des deux autres ID qui s'occupera de la distribution DHCP.

Puis :

```
interface GigabitEthernet1/0/11
 description BORNE WIFI
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 1,71,81
```

À partir de là, la borne est fonctionnelle.

*(Capture : la borne `BorneWifiAP` apparaît dans le contrôleur avec un client connecté, un iPhone.)*

Le VLAN 1 est utilisé pour recevoir son trafic de management sans tag.
