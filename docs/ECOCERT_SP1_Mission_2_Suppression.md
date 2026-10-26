# ECOCERT – SP1 – Mission 2 : Suppression des comptes des utilisateurs ayant quitté ECOCERT

**Auteur :** Crys Boisseau
**Date :** 12/10/2026
**Périmètre :** suppression des comptes AD des anciens collaborateurs.

> **Note :** ce script correspond à la **partie 2** du fichier `Gestion-Utilisateurs.ps1` décrit dans la Mission 1 (le `Import-Module ActiveDirectory` du début du fichier s'applique aussi à cette partie). Il se lance en même temps que la partie 1, à la main, depuis le dossier qui contient `SP1anciensUtilisateursEcocert.csv`.
> Les commentaires sont ajoutés à des fins pédagogiques ; avec PowerShell 5.1, enregistrer le script en UTF-8 avec BOM si les accents sont conservés.

## Règles appliquées

- Suppression du compte AD, sans demande de confirmation

---

## Gestion-Utilisateurs.ps1 – partie 2 : suppression des anciens utilisateurs

```powershell
# 2. SUPPRESSION DES ANCIENS UTILISATEURS
$anciens = Import-Csv -Path ".\SP1anciensUtilisateursEcocert.csv" -Delimiter ","   # Lit le CSV des anciens collaborateurs (dossier courant, séparateur virgule)

foreach ($ligne in $anciens) {   # Traite chaque ligne du CSV
    $nom = $ligne.nom.Trim()   # Nom, sans espaces parasites
    $prenom = $ligne.prenom.Trim()   # Prénom, sans espaces parasites
    $samAccountName = "$prenom.$nom".ToLower()   # Login prenom.nom en minuscules (même règle qu'à la création)

    Remove-ADUser -Identity $samAccountName -Confirm:$false   # Supprime le compte AD sans demande de confirmation
}   # Fin de la boucle sur les lignes
```

---

> ## 📋 Résumé du fonctionnement – Mission 2
>
> **Objectif :** supprimer les comptes des collaborateurs ayant quitté ECOCERT, listés dans `SP1anciensUtilisateursEcocert.csv`.
>
> **Déroulement :**
> 1. Le script est lancé à la main, à la suite de la création des comptes (Mission 1).
> 2. Il lit `SP1anciensUtilisateursEcocert.csv`.
> 3. Pour chaque ligne du CSV :
>    - nettoie nom et prénom (`Trim`) ;
>    - reconstruit le login `prenom.nom` en minuscules ;
>    - supprime le compte AD correspondant avec `Remove-ADUser`, sans confirmation.
> 4. Si le login n'existe pas dans l'AD, PowerShell affiche une erreur « objet introuvable » en rouge, puis passe à la ligne suivante.
>
> **Points clés :** script très court, facile à expliquer, la règle de login est la même qu'à la création.
>
> **Limites à connaître :**
> - la suppression est **définitive** et sans confirmation ;
> - le login doit correspondre exactement à `prenom.nom` ; une ligne mal formée (ex. `TERRA Idriss,Yves`), un nom accentué ou un compte déjà supprimé provoque une erreur et le compte n'est pas supprimé ;
> - le CSV doit être séparé par des virgules ;
> - relancé une seconde fois, le script affiche une erreur « objet introuvable » pour chaque compte déjà supprimé.
