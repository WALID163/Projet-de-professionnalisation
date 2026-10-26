# ECOCERT – SP1 – Missions 1 et 2 : Résultats des scripts Active Directory

**Auteur :** Crys Boisseau
**Date :** 12/10/2026
**Périmètre :** vérification après exécution du script `Gestion-Utilisateurs.ps1` : création des comptes utilisateurs (Mission 1) et suppression des comptes des anciens collaborateurs (Mission 2).

> **Note :** ce document décrit le résultat attendu après exécution du script unique. Les logins et mails des exemples sont déduits des règles du script ; les noms tronqués à l'écran sont signalés par « … ».

Éléments vérifiés : console *Utilisateurs et ordinateurs Active Directory*, comptes supprimés.

---

## 1. Active Directory – domaine `local.ecocert1.lan`

Console *Utilisateurs et ordinateurs Active Directory*.

**Arborescence des conteneurs et OU :** les OU des services existent avant l'exécution (le script ne les crée pas).

- Requêtes enregistrées
- `local.ecocert1.lan`
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

**Contenu de l'OU `ServiceTech`** (7 utilisateurs) : le nom affiché est `prénom nom`, la liste est donc triée par prénom.

| Nom | Type |
|---|---|
| Anissa LONG | Utilisateur |
| Aude LEPIN | Utilisateur |
| Jules SWAIN | Utilisateur |
| Ka… MOREAU | Utilisateur |
| Mehdi SALLA | Utilisateur |
| Nadia JARA | Utilisateur |
| Ya… MATAAH | Utilisateur |

**Attributs attendus** (exemples déduits du script) :

| Nom | Login et UPN | Mail |
|---|---|---|
| Anissa LONG | `anissa.long@local.ecocert1.lan` | `a.long@local.ecocert1.lan` |
| Aude LEPIN | `aude.lepin@local.ecocert1.lan` | `a.lepin@local.ecocert1.lan` |
| Jules SWAIN | `jules.swain@local.ecocert1.lan` | `j.swain@local.ecocert1.lan` |
| Mehdi SALLA | `mehdi.salla@local.ecocert1.lan` | `m.salla@local.ecocert1.lan` |
| Nadia JARA | `nadia.jara@local.ecocert1.lan` | `n.jara@local.ecocert1.lan` |

Tous les comptes sont actifs, avec le mot de passe initial à changer à la première connexion.

---

## 2. Anciens collaborateurs – comptes supprimés (Mission 2)

Les comptes listés dans `SP1anciensUtilisateursEcocert.csv` sont supprimés définitivement de l'AD.

| Login recherché | Résultat attendu |
|---|---|
| `serre.rayan` | Compte supprimé (`Get-ADUser` renvoie « objet introuvable ») |
| `terra.idriss` | À vérifier : la ligne `TERRA Idriss,Yves` du CSV est mal formée : le login `yves.terra idriss` n'existe pas, le script affiche une erreur « objet introuvable » et le compte reste dans l'AD |

---

> ## 📋 Résumé des résultats – Missions 1 et 2
>
> **Objectif :** vérifier que le script a bien produit les éléments attendus dans l'annuaire.
>
> **Constats :**
> 1. **Mission 1 (création)** : les utilisateurs sont rangés dans des OU par service (ex. 7 comptes dans `ServiceTech`), avec le login `prenom.nom`, le mail `p.nom@local.ecocert1.lan` et le mot de passe initial à changer à la 1ère connexion.
> 2. **Mission 2 (suppression)** : les comptes des anciens collaborateurs bien formés dans le CSV (ex. `serre.rayan`) n'existent plus dans l'AD ; une ligne mal formée (ex. `TERRA Idriss,Yves`) provoque une erreur et son compte est conservé.
