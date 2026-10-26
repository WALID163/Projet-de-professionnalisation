# ECOCERT – SP1 – Mission 1 : Création des comptes utilisateurs

**Auteur :** Crys Boisseau
**Date :** 12/10/2026
**Périmètre :** création des comptes Active Directory à partir du fichier CSV, selon la norme interne ECOCERT.

> **Note :** les commentaires ci-dessous sont ajoutés à des fins pédagogiques. Si le code est réutilisé tel quel avec PowerShell 5.1, enregistrer le fichier en UTF-8 avec BOM (ou retirer les accents des commentaires).

## Norme interne ECOCERT appliquée

- Login : `prenom.nom` en minuscules (sAMAccountName **et** UPN)
- UPN : `prenom.nom@local.ecocert1.lan`
- Mail : `p.nom@local.ecocert1.lan` (initiale du prénom + nom)
- OU = nom du service (l'OU doit déjà exister dans l'annuaire)
- Mobile au format `+33 x xx xx xx xx`
- Mot de passe initial identique pour tous, à changer à la 1ère connexion

Fichier de cette mission : `Gestion-Utilisateurs.ps1` (un seul fichier, composé de deux parties : la création, décrite ici, et la suppression, décrite dans la Mission 2).

Le script se lance à la main, dans PowerShell en administrateur, **depuis le dossier qui contient les CSV** (les chemins `.\` sont relatifs au dossier courant).

---

## 1. Gestion-Utilisateurs.ps1 – partie 1 : création des comptes

```powershell
Import-Module ActiveDirectory   # Charge le module AD (New-ADUser, Remove-ADUser...)

# 1. CRÉATION DES UTILISATEURS
$utilisateurs = Import-Csv -Path ".\SP1utilisateursEcocert.csv" -Delimiter ","   # Lit le CSV des utilisateurs (dossier courant, séparateur virgule)

foreach ($ligne in $utilisateurs) {   # Traite chaque ligne du CSV
    $nom = $ligne.nom.Trim()   # Nom, sans espaces parasites
    $prenom = $ligne.prenom.Trim()   # Prénom, sans espaces parasites
    $service = $ligne.service.Trim()   # Service (détermine l'OU)
    $telMobileBrut = $ligne.telMobile   # Numéro de mobile tel que saisi dans le CSV
    
    $samAccountName = "$prenom.$nom".ToLower()   # Login prenom.nom en minuscules
    $userPrincipalName = "$samAccountName@local.ecocert1.lan"   # UPN = login@domaine
    $email = "$($prenom.Substring(0,1)).$nom".ToLower() + "@local.ecocert1.lan"   # Mail : initiale du prénom + nom + domaine
    
    $telephone = $null   # Téléphone vide par défaut
    if (-not [string]::IsNullOrEmpty($telMobileBrut) -and $telMobileBrut -ne "NaN") {   # Si un numéro est renseigné (ni vide, ni "NaN")
        $telNet = $telMobileBrut.ToString().Replace(".", "").Replace(" ", "")   # Retire les points et les espaces
        if ($telNet.StartsWith("0")) {   # Si le numéro commence par 0
            $telNet = $telNet.Substring(1)   # On retire le 0 initial
        }   # Fin du if
        $telephone = "+33 $(([string]$telNet)[0]) $(([string]$telNet).Substring(1,2)) $(([string]$telNet).Substring(3,2)) $(([string]$telNet).Substring(5,2)) $(([string]$telNet).Substring(7,2))"   # Découpe en 1-2-2-2-2 chiffres derrière "+33"
    }   # Fin du if téléphone

    $ouPath = "OU=$service,DC=local,DC=ecocert1,DC=lan"   # Chemin de l'OU du service (elle doit déjà exister)
    $password = ConvertTo-SecureString "Ecocert2026!" -AsPlainText -Force   # Mot de passe initial converti en SecureString

    # Crée le compte avec : nom affiché "prénom nom", login, UPN, OU du service,
    # mail, mobile, mot de passe initial, compte actif et changement du mot de passe obligatoire à la 1ère connexion.
    New-ADUser -Name "$prenom $nom" `
               -SamAccountName $samAccountName `
               -UserPrincipalName $userPrincipalName `
               -Path $ouPath `
               -EmailAddress $email `
               -MobilePhone $telephone `
               -AccountPassword $password `
               -Enabled $true `
               -ChangePasswordAtLogon $true
}   # Fin de la boucle sur les lignes
```

La suite du fichier (« 2. SUPPRESSION DES ANCIENS UTILISATEURS ») est décrite dans la Mission 2.

---

> ## 📋 Résumé du fonctionnement – Mission 1
>
> **Objectif :** créer les comptes AD à partir du fichier `SP1utilisateursEcocert.csv`.
>
> **Déroulement :**
> 1. Le script `Gestion-Utilisateurs.ps1` est lancé à la main, en administrateur, depuis le dossier contenant les CSV.
> 2. Il charge le module `ActiveDirectory`, puis lit `SP1utilisateursEcocert.csv`.
> 3. Pour chaque ligne du CSV :
>    - nettoie nom, prénom et service (`Trim`) ;
>    - construit le login `prenom.nom` en minuscules, l'UPN `login@local.ecocert1.lan` et le mail `p.nom@local.ecocert1.lan` ;
>    - convertit le mobile au format `+33 x xx xx xx xx` (exemple : `06 65 43 65 78` devient `+33 6 65 43 65 78`) ; si le numéro est vide ou égal à `NaN`, aucun mobile n'est renseigné ;
>    - place le compte dans l'OU portant le nom du service ;
>    - **crée** le compte (nom affiché `prénom nom`) avec le mot de passe initial, compte actif, changement du mot de passe obligatoire à la 1ère connexion.
>
> **Points clés :** script court et lisible, tenant en un seul fichier.
>
> **Limites à connaître :**
> - les OU des services doivent exister avant l'exécution ;
> - relancé une seconde fois, le script affiche une erreur « compte déjà existant » pour chaque utilisateur ;
> - le CSV doit être séparé par des virgules et les numéros de mobile doivent être des numéros français à 10 chiffres ;
> - une erreur sur une ligne affiche un message en rouge, puis le script passe à la ligne suivante.
