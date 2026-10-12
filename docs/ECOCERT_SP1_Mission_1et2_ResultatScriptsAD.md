# ECOCERT – SP1 – Missions 1 et 2 : Résultats des scripts Active Directory

**Auteur :** Crys Boisseau
**Date :** 12/10/2026
**Périmètre :** captures de vérification après exécution des scripts de création / modification (Mission 1) et de suppression / archivage (Mission 2) des comptes utilisateurs.

> **Note :** le document source est composé uniquement de captures d'écran. Elles sont décrites ci-dessous à partir de ce qu'on y lit ; les noms tronqués à l'écran sont signalés par « … ».

Éléments vérifiés : `C:\` (arborescence générale), `C:\Anciens Collaborateurs`, `C:\Données`, `C:\Ecocert` (`Etat` et `Logs`), console *Utilisateurs et ordinateurs Active Directory*.

---

## 1. Racine du disque local `C:\`

Le dossier contient **10 éléments**. Les trois dossiers créés ou modifiés par les scripts ECOCERT sont ceux du 12/10/2026.

| Nom | Modifié le | Type |
|---|---|---|
| **Anciens Collaborateurs** | 12/10/2026 15:56 | Dossier de fichiers |
| **Données** | 12/10/2026 15:56 | Dossier de fichiers |
| **Ecocert** | 12/10/2026 15:37 | Dossier de fichiers |
| inetpub | 21/05/2026 16:36 | Dossier de fichiers |
| PerfLogs | 01/04/2024 09:02 | Dossier de fichiers |
| ProgramData | 07/09/2026 16:28 | Dossier de fichiers |
| Programmes | 21/05/2026 15:30 | Dossier de fichiers |
| Programmes (x86) | 01/04/2024 18:43 | Dossier de fichiers |
| Utilisateurs | 21/05/2026 15:29 | Dossier de fichiers |
| Windows | 12/10/2026 10:54 | Dossier de fichiers |

---

## 2. `C:\Anciens Collaborateurs` – archivage des comptes supprimés (Mission 2)

Le dossier contient **4 éléments** : deux collaborateurs archivés, chacun présent en deux dossiers (le dossier d'origine et un dossier suffixé de l'horodatage `20261012-155655`).

| Nom | Modifié le | Type |
|---|---|---|
| `serre.rayan` | 12/10/2026 15:51 | Dossier de fichiers |
| `serre.rayan_20261012-155655` | 12/10/2026 15:56 | Dossier de fichiers |
| `terra.idriss` | 12/10/2026 15:51 | Dossier de fichiers |
| `terra.idriss_20261012-155655` | 12/10/2026 15:56 | Dossier de fichiers |

---

## 3. `C:\Données` – dossiers privés des utilisateurs (Mission 1)

Le dossier contient **44 éléments**, un dossier privé `nom.prenom` par utilisateur, tous modifiés le **12/10/2026 à 15:51**. La capture n'en montre que les 14 premiers :

| Nom | Modifié le |
|---|---|
| `arear.leia` | 12/10/2026 15:51 |
| `arema.lourdsini` | 12/10/2026 15:51 |
| `aumont.victor` | 12/10/2026 15:51 |
| `basselle.fatima` | 12/10/2026 15:51 |
| `benzema.hassan` | 12/10/2026 15:51 |
| `carre.manon` | 12/10/2026 15:51 |
| `clairin.sofia` | 12/10/2026 15:51 |
| `delage.benoit` | 12/10/2026 15:51 |
| `deleis.jean` | 12/10/2026 15:51 |
| `dezael.emma` | 12/10/2026 15:51 |
| `dupont.rodrigues` | 12/10/2026 15:51 |
| `falegeant.renaud` | 12/10/2026 15:51 |
| `ferne.mohamed` | 12/10/2026 15:51 |
| `fosseneuve.mona` | 12/10/2026 15:51 |
| … | … |

---

## 4. `C:\Ecocert` – fichiers de fonctionnement des scripts

Le dossier contient **2 éléments**, tous deux modifiés le 12/10/2026 à 15:37 :

| Nom | Modifié le | Type |
|---|---|---|
| `Etat` | 12/10/2026 15:37 | Dossier de fichiers |
| `Logs` | 12/10/2026 15:37 | Dossier de fichiers |

### 4.1 `C:\Ecocert\Etat` – empreintes SHA256 des CSV

Ces fichiers servent à `Run-Auto.ps1` pour détecter si un CSV a changé depuis la dernière exécution.

| Nom | Modifié le | Type | Taille |
|---|---|---|---|
| `SP1anciensUtilisateursEcocert.csv.sha256` | 12/10/2026 15:37 | Fichier SHA256 | 1 Ko |
| `SP1utilisateursEcocert.csv.sha256` | 12/10/2026 15:37 | Fichier SHA256 | 1 Ko |

### 4.2 `C:\Ecocert\Logs` – journaux

| Nom | Modifié le | Type | Taille |
|---|---|---|---|
| `ecocert_2026-10.log` | 12/10/2026 15:56 | Document texte | 23 Ko |
| `suppressions.csv` | 12/10/2026 15:56 | Fichier CSV | 2 Ko |

---

## 5. Active Directory – domaine `local.ecocert2.fr`

Console *Utilisateurs et ordinateurs Active Directory*.

**Arborescence des conteneurs et OU :**

- Requêtes enregistrées
- `local.ecocert2.fr`
  - Builtin
  - Compta
  - Computers
  - Domain Controllers
  - ForeignSecurityPrincipals
  - Juridique
  - Managed Service Accounts
  - RH
  - Secretariat
  - ServiceCertif
  - ServiceExper
  - ServiceForm
  - ServiceRef
  - **ServiceTech** *(sélectionnée)*
  - Users

**Contenu de l'OU `ServiceTech`** (7 utilisateurs) :

| Nom | Type |
|---|---|
| JARA Nadia | Utilisateur |
| LEPIN Aude | Utilisateur |
| LONG Anissa | Utilisateur |
| MATAAH Ya… | Utilisateur |
| MOREAU Ka… | Utilisateur |
| SALLA Mehdi | Utilisateur |
| SWAIN Jules | Utilisateur |

---

> ## 📋 Résumé des résultats – Missions 1 et 2
>
> **Objectif :** vérifier que les scripts ont bien produit les éléments attendus sur le serveur.
>
> **Constats :**
> 1. **Mission 1 (création / modification)** : `C:\Données` contient 44 dossiers privés `nom.prenom`, et les utilisateurs sont rangés dans des OU par service (ex. 7 comptes dans `ServiceTech`).
> 2. **Mission 2 (suppression)** : `C:\Anciens Collaborateurs` contient les dossiers de `serre.rayan` et `terra.idriss`, chacun avec une copie suffixée de l'horodatage `20261012-155655`. Le fichier `suppressions.csv` est présent dans les logs.
> 3. **Automatisation** : `C:\Ecocert\Etat` contient les deux fichiers `.sha256` (un par CSV), et `C:\Ecocert\Logs` contient le journal mensuel `ecocert_2026-10.log` (23 Ko).
