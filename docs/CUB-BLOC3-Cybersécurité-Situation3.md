# CONTEXTE CUB — BLOC 3 – Situation 3 – CYBERSECURITE

*Crys 901 Boisseau*

## Table des matières

1. [Rédiger la table de routage du pare-feu Stormshield de votre agence](#1-rédiger-la-table-de-routage-du-pare-feu-stormshield-de-votre-agence)
2. [Déterminer quelle adresse IP du WAN doit servir de passerelle pour aller sur Internet ? Puis créer un objet réseau afin que cette adresse IP soit représentée dans l'interface d'administration](#2-déterminer-quelle-adresse-ip-du-wan-doit-servir-de-passerelle-pour-aller-sur-internet--puis-créer-un-objet-réseau-afin-que-cette-adresse-ip-soit-représentée-dans-linterface-dadministration)
3. [Utiliser cet objet afin de pouvoir implémenter la table de routage sur votre pare-feu](#3-utiliser-cet-objet-afin-de-pouvoir-implémenter-la-table-de-routage-sur-votre-pare-feu)
4. [Proposer et paramétrer une solution technique permettant aux adresses IP privées de votre site de pouvoir communiquer sur le réseau WAN public et Internet](#4-proposer-et-paramétrer-une-solution-technique-permettant-aux-adresses-ip-privées-de-votre-site-de-pouvoir-communiquer-sur-le-réseau-wan-public-et-internet)
5. [Peut-on joindre le pare-feu général CUB puis les serveurs présents dans sa DMZ. Proposer une analyse des résultats obtenus](#5-peut-on-joindre-le-pare-feu-général-cub-puis-les-serveurs-présents-dans-sa-dmz-proposer-une-analyse-des-résultats-obtenus)
6. [Proposer et paramétrer une solution technique permettant aux services WEB et FTP de votre DMZ d'être interrogé par le réseau WAN](#6-proposer-et-paramétrer-une-solution-technique-permettant-aux-services-web-et-ftp-de-votre-dmz-dêtre-interrogé-par-le-réseau-wan)
8. [Pour conclure, lister les objets réseaux implicites et explicites dont vous avez eu besoin de mobiliser précédemment](#8-pour-conclure-lister-les-objets-réseaux-implicites-et-explicites-dont-vous-avez-eu-besoin-de-mobiliser-précédemment)

---

## 1. Rédiger la table de routage du pare-feu Stormshield de votre agence :

| Destination | Masque | Passerelle | Interface | Type |
|---|---|---|---|---|
| 192.36.2.0 | 255.255.255.0 | 192.36.2.254 | 192.36.2.254 | C |
| 192.36.253.0 | 255.255.255.0 | 192.36.253.20 | 192.36.253.20 | C |
| 192.168.22.248 | 255.255.255.248 | 192.168.22.254 | 192.168.22.254 | C |
| 192.168.2.0 | 255.255.255.0 | 192.168.22.253 | 192.168.22.254 | S |
| 0.0.0.0 | 0.0.0.0 | 192.168.36.254 | 192.168.36.20 | S* |

## 2. Déterminer quelle adresse IP du WAN doit servir de passerelle pour aller sur Internet ? Puis créer un objet réseau afin que cette adresse IP soit représentée dans l'interface d'administration :

**Objet réseau créé sur le Stormshield :**

| Propriété | Valeur |
|---|---|
| Nom de l'objet | RT_CUB |
| Adresse IPv4 | 192.36.253.254 |
| Adresse MAC | 01:23:45:67:89:ab (Facultatif) |
| Résolution | Aucune (IP statique) |
| Commentaire | — |

## 3. Utiliser cet objet afin de pouvoir implémenter la table de routage sur votre pare-feu :

**Configuration générale :**

| Paramètre | Valeur |
|---|---|
| Passerelle par défaut (routeur) | RT_CUB |

## 4. Proposer et paramétrer une solution technique permettant aux adresses IP privées de votre site de pouvoir communiquer sur le réseau WAN public et Internet :

Pour ce faire, nous allons paramétrer le NAT sur le Stormshield afin que le réseau `192.168.2.0/24`, qui englobe tous nos sous-réseaux, puisse communiquer avec le WAN public et Internet.

**Règles NAT :**

| N° | État | Source | Interface source | Destination | Translation | | Interface de destination |
|---|---|---|---|---|---|---|---|
| 1 | on | BARCELONE_LAN1_CUE | Any interface : ou | Any | Fire | ephemeral_fw | Any |
| 2 | on | Network_in | Any interface : ou | Any | Fire | ephemeral_fw | Any |

## 5. Peut-on joindre le pare-feu général CUB puis les serveurs présents dans sa DMZ. Proposer une analyse des résultats obtenus :

```
etudiant@S406-06:~$ ping 192.36.253.254
PING 192.36.253.254 (192.36.253.254) 56(84) bytes of data.
64 bytes from 192.36.253.254: icmp_seq=1 ttl=63 time=1.08 ms

etudiant@S406-06:~$ ping 192.36.2.20
PING 192.36.2.20 (192.36.2.20) 56(84) bytes of data.
64 bytes from 192.36.2.20: icmp_seq=1 ttl=63 time=1.81 ms
```

Nous arrivons à ping le pare-feu général CUB et les serveurs de la DMZ depuis un PC sur le VLAN 20 Administrateur.

## 6. Proposer et paramétrer une solution technique permettant aux services WEB et FTP de votre DMZ d'être interrogé par le réseau WAN :

Les IP des machines sur la DMZ sont des IP publiques. S'il s'agissait d'IP privées, une translation NAT se serait imposée, mais ce n'est donc pas le cas ici.

## 8. Pour conclure, lister les objets réseaux implicites et explicites dont vous avez eu besoin de mobiliser précédemment :

- Switch L2
- Switch L3
- Proxmox
- Stormshield
