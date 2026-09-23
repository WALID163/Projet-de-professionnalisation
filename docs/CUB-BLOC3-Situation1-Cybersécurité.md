# CONTEXTE CUB — BLOC 3 – CYBERSECURITE

*Crys 901 Boisseau*

## Table des matières

- [2. Expliquer ce qui a poussé le service RSSI à opter pour une solution UTM par rapport à un simple pare-feu stateful traditionnel](#2-expliquer-ce-qui-a-poussé-le-service-rssi-à-opter-pour-une-solution-utm-par-rapport-à-un-simple-pare-feu-stateful-traditionnel)
- [3. Donner 2 arguments en faveur d'un boîtier UTM Stormshield par rapport à ceux proposés par des entreprises concurrentes telles que Palo Alto ou CheckPoint](#3-donner-2-arguments-en-faveur-dun-boîtier-utm-stormshield-par-rapport-à-ceux-proposés-par-des-entreprises-concurrentes-telles-que-palo-alto-ou-checkpoint)
- [4. Réseau local unique et sécurité](#4-dans-le-schéma-proposé-dans-le-contexte-cub-expliquer-pourquoi-la-présence-dun-réseau-local-unique-au-sein-des-agences-pose-des-problèmes-de-sécurité)
- [5. Schéma logique](#5-réaliser-un-schéma-logique-représentant-votre-nouvelle-proposition)
- [6. Schéma physique](#6-réaliser-le-schéma-physique-de-votre-agence)
- [7. Plan d'affectation des ports par VLAN](#7-réaliser-un-plan-daffectation-des-port-par-vlan-pour-les-éléments-actifs-de-votre-agence)
- [8. Maquette Packet Tracer](#8-réaliser-une-maquette-de-la-nouvelle-infrastructure-du-site-à-laide-du-logiciel-packet-tracer)

---

## 2. Expliquer ce qui a poussé le service RSSI à opter pour une solution UTM par rapport à un simple pare-feu stateful traditionnel.

**Pare-feu stateful** : pare-feu intervenant essentiellement jusqu'à la couche 4 du modèle OSI. Ainsi, il inspecte les paquets IP ainsi que les en-têtes au niveau de la couche de transport et dresse l'inventaire des connexions actives, permettant ainsi d'utiliser « l'état » d'une connexion (nouvelle, active, non-existante) pour devenir une règle.

**Pare-feu UTM (Unified Threat Management)** : pare-feu qui fait la gestion unifiée des menaces, c'est une solution de sécurité tout-en-un, généralement une appliance de sécurité unique, qui fournit plusieurs fonctions de sécurité en un seul point du réseau.

Une appliance UTM réunit le plus souvent des fonctions telles que :

- logiciel antivirus,
- logiciel anti-espions,
- protection antispam,
- pare-feu réseau,
- prévention et détection des intrusions,
- filtrage des contenus et prévention des fuites.

Cette technologie de pare-feu positionne l'inspection de paquets au niveau de la couche applicative, la couche 7 du modèle OSI. Ainsi, si les informations sur les connexions et leur statut peuvent être utilisées pour définir des règles, ces dernières peuvent désormais intégrer des informations liées à des opérations menées dans le cadre d'un protocole précis. De plus, plutôt que de recourir à des fournisseurs ou appliances dédiés à chaque tâche de sécurité, les organisations peuvent regrouper toutes ces fonctions autour d'un seul et même fournisseur. L'administration est ainsi réduite à un seul segment ou à une seule équipe informatique utilisant une console centralisée qui facilite considérablement la lutte contre les nombreuses menaces actuelles.

Le service RSSI a opté pour une solution UTM car celle-ci offre plus de fonctions et couvre plus de couches (3 à 7) que le pare-feu stateful (3 à 4).

## 3. Donner 2 arguments en faveur d'un boîtier UTM Stormshield par rapport à ceux proposés par des entreprises concurrentes telles que Palo Alto ou CheckPoint.

**Souveraineté numérique** : Stormshield est une entreprise française, sous-filiale d'Airbus, dont le siège social se situe à Issy-les-Moulineaux. Ainsi, elle est soumise à la loi française et à la loi européenne (loi informatique et libertés, RGPD, etc.), échappant donc aux lois extraterritoriales étrangères américaines.

**Certifications ANSSI** : Stormshield possède la qualification standard ANSSI EAL4+ qui est fortement recommandée pour la protection des infrastructures critiques et la conformité à la directive.

## 4. Dans le schéma proposé dans le contexte CUB, expliquer pourquoi la présence d'un réseau local unique au sein des agences pose des problèmes de sécurité. Puis proposer une solution qui prenne en compte les différents services recensés dans les document 1.1 du dossier documentaire.

La présence d'un unique réseau local au sein des agences pose plusieurs problèmes de sécurité et aussi techniques.

**Sécurité :**

- Propagation des menaces : un logiciel malveillant peut facilement se propager dans le réseau sans être bloqué.
- Absence d'isolation : une personne se connectant par exemple sur un réseau invité a accès au reste du réseau.
- Contrôle d'accès limité : il est difficile d'empêcher un utilisateur ou équipement d'accéder à quelque chose dont il ne devrait pas avoir l'accès.

**Techniques :**

- Surcharge du trafic : tous les appareils sur le réseau partagent la même bande passante et ont le même domaine de diffusion, ce qui peut parfois ralentir le réseau pour tout le monde.
- Congestion : le transfert de fichiers volumineux entre deux appareils peut ralentir le réseau entier.

Pour remédier à ces problèmes, nous avons une table qui répertorie les services et leur nombre d'hôtes.

| Intitulé des services | VLAN | Nombre d'hôtes par service |
|---|---|---|
| Production | 52 | 60 hôtes |
| Clients | 10 | 16 hôtes |
| Administration systèmes et réseaux | 20 | 3 hôtes |

Avec l'aide de ce tableau, il sera possible de segmenter le réseau local (VLSM) en plusieurs réseaux (sous-réseaux), ce qui va permettre de séparer les services et de leur éviter de communiquer ensemble selon nos envies.

### Barcelone VLSM :

| Services | IP Réseau | Prem. IP | Passerelle | Broadcast | Nb hôtes |
|---|---|---|---|---|---|
| Production | 192.168.2.0/25 | 192.168.2.1 | 192.168.2.126 | 192.168.2.127 | 60 en 120+ |
| Clients | 192.168.2.128/26 | 192.168.2.129 | 192.168.2.190 | 192.168.2.191 | 16 en 32+ |
| Administration SR | 192.168.2.192/28 | 192.168.2.193 | 192.168.2.206 | 192.168.2.207 | 3 en 6+ |

### Barcelone — Table de routage :

**Switch de couche 3 :**

| Services | Destination | Masque | Passerelle | Interface | Type |
|---|---|---|---|---|---|
| Production | 192.168.2.0 | 255.255.255.128 | 192.168.2.126 | 192.168.2.126 | C |
| Clients | 192.168.2.128 | 255.255.255.192 | 192.168.2.190 | 192.168.2.190 | C |
| Administration | 192.168.2.192 | 255.255.255.240 | 192.168.2.206 | 192.168.2.206 | C |
| LAN 2 | 192.168.22.248 | 255.255.255.248 | 192.168.22.254 | 192.168.22.254 | C |
| NA | 0.0.0.0 | 0.0.0.0 | 192.168.22.254 | 192.168.22.253 | S* |

**Pare-feu Stormshield :**

| Services | Destination | Masque | Passerelle | Interface | Type |
|---|---|---|---|---|---|
| DMZ | 192.36.2.0 | 255.255.255.0 | 192.36.2.254 | 192.36.2.254 | C |
| WAN | 192.36.253.0 | 255.255.255.0 | 192.36.253.20 | 192.36.253.20 | C |
| LAN 2 | 192.168.22.248 | 255.255.255.248 | 192.168.22.254 | 192.168.22.254 | C |
| AGENCE B | 192.168.2.0 | 255.255.255.0 | 192.168.22.253 | 192.168.22.254 | S |
| NA | 0.0.0.0 | 0.0.0.0 | 192.168.36.254 | 192.168.36.20 | S* |

### Barcelone — Table de NAT :

| | **Avant translation** | | | | **Après translation** | | |
|---|---|---|---|---|---|---|---|
| **IP Src** | **Port Src** | **IP Dst** | **Port Dst** | **IP Src** | **Port Src** | **IP Dst** | **Port Dst** |
| 192.168.2.x/24 | X | X | X | 192.36.253.20 | X | X | X |
| 192.168.22.x/29 | X | X | X | 192.36.253.20 | X | X | X |

## 5. Réaliser un schéma logique représentant votre nouvelle proposition. Ce schéma ne concerne uniquement que le site dont vous avez la charge.

![Schéma Logique](https://raw.githubusercontent.com/WALID163/MilleNuits-mkdocs/refs/heads/main/images/Sch%C3%A9ma_logique_Barcelone_CUB_CB_WJ.png)

## 6. Réaliser le schéma physique de votre agence.

![Schéma Physique](https://raw.githubusercontent.com/WALID163/MilleNuits-mkdocs/refs/heads/main/images/Sch%C3%A9ma_physique_Barcelone_CUB_CB_WJ.png)

## 7. Réaliser un plan d'affectation des port par VLAN pour les éléments actifs de votre agence.

![Schéma Brassage](https://raw.githubusercontent.com/WALID163/Projet-de-professionnalisation/refs/heads/main/images/cub-schema-brassage-gp2.drawio.png)

## 8. Réaliser une maquette de la nouvelle infrastructure du site à l'aide du logiciel Packet Tracer. Le pare-feu du site sera représenté par un routeur.

ELEA Cybersécurité Situation 1.
