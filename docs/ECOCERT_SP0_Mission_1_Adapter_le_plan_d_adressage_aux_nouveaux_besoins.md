# ECOCERT - SP0 Mission 1 : Adapter le plan d'adressage aux nouveaux besoins

*Crys 801 Boisseau*

## Table des matières

- [Contexte](#contexte)
  - [Plan d'adressage](#plan-dadressage)
  - [VLSM (+20%)](#vlsm-20)
  - [Table de routage](#table-de-routage)
  - [ECOCERT Table de NAT](#ecocert-table-de-nat)

---

## Contexte

Actuellement la société ECOCERT France (www.ecocert.fr) dispose d'un réseau accueillant l'ensemble de ses services.

Chaque poste dispose d'une adresse IP fixe attribuée par l'administrateur réseau suivant un plan d'adressage rédigé par lui-même.

Plusieurs problèmes sont constatés :

- Depuis quelque temps, le trafic du réseau subit de nombreux ralentissements.
- L'installation d'une ferme de serveurs, permettant la virtualisation des OS via l'hyperviseur Proxmox, vient d'être mise en place dans l'architecture de la société par deux ingénieurs de la société Bull. Les serveurs virtuels hébergés dans cette ferme doivent être adressés dans la plage `172.16.52.0/24`.

### Plan d'adressage

Actuellement la société ECOCERT suivait le plan d'adressage ci-dessous :

| N°VLAN | Service(s) | Nombre d'hôtes (passerelle non comprise) |
|---|---|---|
| 11 | Administration (RH / Compta / Juridique / Secrétariat) | 45 + 20 % = 54 |
| 21 | Service de certification | 20 + 20 % = 24 |
| 31 | Service de référentiels | 12 + 20 % = 14 |
| 41 | Service de formations professionnelles | 8 + 20 % = 10 |
| 61 | Service d'expertise technique et de conseil | 15 + 20 % = 18 |
| 71 | Services techniques (Administration réseau et développement) | 10 + 20 % = 12 |
| 81 | Wifi-Visiteurs | 20 + 20 % = 24 |
| 52 | Serveurs | — |

### VLSM (+20%)

Afin de bien créer les VLSM par rapport au plan d'adressage, il faut commencer par les services avec le plus grand besoin en termes d'hôtes, à noter qu'il faut une marge de 20 %.

Administration a besoin de 45 hôtes en tout : 45 + 20 % = 45 * 1,2 = 54 : Nous avons donc 54 hôtes nécessaires sans compter la passerelle.

```
2⁰ = 1 ; 2¹ = 2 ; 2² = 4 ; 2³ = 8 ; 2⁴ = 16 ; 2⁵ = 32 ; 2⁶ = 64 ; 2⁷ = 128 ; 2⁸ = 256
```

On choisit donc 2⁶, car celui-ci est égal à 64 par rapport à 2⁵ qui est égal à 32 et est donc en-dessous de 54.

`32 - 6 = /26`

`192.168.2.0/26` :

- **Réseau** de Administration : `192.168.2.0/26`
- **Broadcast** de Administration : `192.168.2.63/26` (2⁶ = 64 : il y a donc 64 IP dans ce sous-réseau, le réseau commence par l'IP 192.168.2.0, donc 0 + 64 - 1, ce qui nous donne l'IP de broadcast).
- **Passerelle** de Administration : `192.168.2.1/26` (en fait -1 par rapport à l'IP de broadcast, dans le cas où la passerelle doit être la première IP).

Ensuite, on fait +1 sur l'IP de broadcast pour obtenir la première IP du prochain sous-réseau.

| Service(s) | Réseau | Prem. IP | Dern. IP | Passerelle | Broadcast |
|---|---|---|---|---|---|
| Administration | 192.168.2.0/26 | 192.168.2.2 | 192.168.2.62 | 192.168.2.1 | 192.168.2.63 |
| Service de certification | 192.168.2.64/27 | 192.168.2.66 | 192.168.2.94 | 192.168.2.65 | 192.168.2.95 |
| Wifi-Visiteurs | 192.168.2.96/27 | 192.168.2.98 | 192.168.2.126 | 192.168.2.97 | 192.168.2.127 |
| Service d'expertise technique et de conseil | 192.168.2.128/27 | 192.168.2.130 | 192.168.2.158 | 192.168.2.129 | 192.168.2.159 |
| Service de référentiels | 192.168.2.160/27 | 192.168.2.162 | 192.168.2.190 | 192.168.2.161 | 192.168.2.191 |
| Services techniques | 192.168.2.192/28 | 192.168.2.194 | 192.168.2.206 | 192.168.2.193 | 192.168.2.207 |
| Service de formations professionnelles | 192.168.2.208/28 | 192.168.2.210 | 192.168.2.222 | 192.168.2.209 | 192.168.2.223 |
| Serveurs | 172.16.52.0/24 | 172.16.52.1 | 172.16.52.252 | 172.16.52.253 | 172.16.52.255 |

### Table de routage

**Switch de couche 3 RTECOCERT :**

| Services | Destination | Masque | Passerelle | Interface | Type |
|---|---|---|---|---|---|
| Pare-feu Stormshield | 192.168.22.252 | 255.255.255.252 | 192.168.22.254 | 192.168.22.254 | C |
| Administration | 192.168.2.0 | 255.255.255.192 | 192.168.2.1 | 192.168.2.1 | C |
| Service de certification | 192.168.2.64 | 255.255.255.224 | 192.168.2.65 | 192.168.2.65 | C |
| Wifi-Visiteurs | 192.168.2.96 | 255.255.255.224 | 192.168.2.97 | 192.168.2.97 | C |
| Service d'expertise technique et de conseil | 192.168.2.128 | 255.255.255.224 | 192.168.2.129 | 192.168.2.129 | C |
| Service de référentiels | 192.168.2.160 | 255.255.255.224 | 192.168.2.161 | 192.168.2.161 | C |
| Services techniques | 192.168.2.192 | 255.255.255.240 | 192.168.2.193 | 192.168.2.193 | C |
| Service de formations professionnelles | 192.168.2.208 | 255.255.255.240 | 192.168.2.209 | 192.168.2.209 | C |
| Serveurs | 172.16.52.0 | 255.255.255.0 | 172.16.52.253 | 172.16.52.253 | C |
| NA | 0.0.0.0 | 0.0.0.0 | 192.168.22.253 | 192.168.22.254 | S* |

**Pare-feu Stormshield :**

| Services | Destination | Masque | Passerelle | Interface | Type |
|---|---|---|---|---|---|
| MODEM ADSL | 172.16.32.0 | 255.255.255.0 | 172.16.32.2 | 172.16.32.2 | C |
| SWITCH L3 | 192.168.22.252 | 255.255.255.252 | 192.168.22.253 | 192.168.22.253 | C |
| SWITCH L3 | 192.168.2.0 | 255.255.255.0 | 192.168.22.254 | 192.168.22.253 | S |
| NA | 0.0.0.0 | 0.0.0.0 | 172.16.32.253 | 172.16.32.2 | S* |

### ECOCERT Table de NAT

| | **Avant translation** | | | | **Après translation** | | |
|---|---|---|---|---|---|---|---|
| **IP Src** | **Port Src** | **IP Dst** | **Port Dst** | **IP Src** | **Port Src** | **IP Dst** | **Port Dst** |
| 192.168.2.x/24 | X | X | X | 172.16.32.2 | X | X | X |
| 192.168.22.x/30 | X | X | X | 172.16.32.2 | X | X | X |

---

## Schémas

### Schéma logique

![Schéma Logique]([https://raw.githubusercontent.com/WALID163/Projet-de-professionnalisation/refs/heads/main/images/ECOCERT%20-%20SP0%20Mission%201%20-%20Sch%C3%A9ma%20logique%20ECOCERT.png])

### Schéma physique

![Schéma Physique]([https://raw.githubusercontent.com/WALID163/Projet-de-professionnalisation/refs/heads/main/images/ECOCERT%20-%20SP0%20Mission%201%20-%20Sch%C3%A9ma%20physique%20ECOCERT.png])

### Schéma de brassage

![Schéma Brassage]([https://raw.githubusercontent.com/WALID163/Projet-de-professionnalisation/refs/heads/main/images/ECOCERT%20-%20SP0%20Mission%201%20-%20Sch%C3%A9ma%20brassage%20ECOCERT.png])
