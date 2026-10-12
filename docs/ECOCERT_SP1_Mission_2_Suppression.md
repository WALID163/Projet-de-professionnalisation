# ECOCERT – SP1 – Mission 2 : Suppression des comptes des utilisateurs ayant quitté ECOCERT

**Auteur :** Crys Boisseau
**Date :** 12/10/2026
**Périmètre :** suppression des comptes AD des anciens collaborateurs, archivage de leur dossier privé et journalisation.

> **Note :** ce script s'appuie sur `Common.ps1` (configuration et fonctions partagées : `Write-Log`, `ConvertTo-Ascii`, `Import-CsvAuto`, `Set-FolderAcl`), décrit dans la Mission 1. Il est déclenché automatiquement par `Run-Auto.ps1` / la tâche planifiée (également décrits dans la Mission 1) lorsque `SP1anciensUtilisateursEcocert.csv` change.
> Les commentaires sont ajoutés à des fins pédagogiques ; avec PowerShell 5.1, enregistrer le script en UTF-8 avec BOM si les accents sont conservés.

## Règles appliquées

- Suppression du compte AD
- Déplacement du dossier privé vers `C:\Anciens Collaborateurs` (accès administrateurs uniquement)
- Journal daté de chaque suppression dans `suppressions.csv` (dossier des logs)

---

## Remove-Utilisateurs.ps1

```powershell
<#
.SYNOPSIS
  Supprime les comptes listes dans SP1anciensUtilisateursEcocert.csv.
.DESCRIPTION
  - supprime le compte AD
  - deplace son dossier prive vers "Anciens Collaborateurs" (acces admins uniquement)
  - consigne chaque suppression, datee, dans suppressions.csv (dossier des logs)
  Utiliser -WhatIf pour simuler.
#>
[CmdletBinding(SupportsShouldProcess)]   # Active -WhatIf / -Confirm (mode simulation)
param([string]$CsvPath = (Join-Path $PSScriptRoot 'SP1anciensUtilisateursEcocert.csv'))   # Chemin du CSV, par défaut celui du dossier du script

. (Join-Path $PSScriptRoot 'Common.ps1')   # Charge la configuration et les fonctions communes

$journal = Join-Path $Cfg.DossierLogs 'suppressions.csv'   # Chemin du journal des suppressions

function Add-Journal {   # Ajoute une ligne au journal des suppressions
    param($Login, $Nom, $Prenom, $Ou, $Mail, $Archive, $Statut)   # Informations à consigner
    [pscustomobject]@{   # Crée l'objet représentant la ligne du journal
        DateSuppression = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'   # Date et heure de l'opération
        Login = $Login; Nom = $Nom; Prenom = $Prenom; OU = $Ou; Mail = $Mail   # Identité et OU d'origine du compte
        DossierArchive = $Archive; Statut = $Statut; Operateur = $env:USERNAME   # Dossier d'archive, résultat et utilisateur ayant lancé le script
    } | Export-Csv -Path $journal -Append -NoTypeInformation -Delimiter ';' -Encoding UTF8   # Ajoute la ligne au CSV (séparateur ;, UTF-8)
}   # Fin de Add-Journal

# Retrouve le compte ; tolere les lignes mal formees du type "TERRA Idriss,Yves"
# en essayant toutes les combinaisons prenom.nom des mots de la ligne.
function Find-User {   # Recherche le compte AD correspondant à une ligne du CSV
    param([string]$Nom, [string]$Prenom)   # Nom et prénom lus dans le CSV
    $props = 'mail', 'HomeDirectory', 'GivenName', 'Surname'   # Attributs supplémentaires à charger
    $sam = '{0}.{1}' -f (ConvertTo-Ascii $Prenom), (ConvertTo-Ascii $Nom)   # Login attendu : prenom.nom
    $upn = "$sam@$($Cfg.DomaineDNS)"   # UPN attendu
    $nomQ = $Nom -replace "'", "''"; $prenomQ = $Prenom -replace "'", "''"   # Échappe les apostrophes pour le filtre AD
    $u = Get-ADUser -Filter "SamAccountName -eq '$sam' -or UserPrincipalName -eq '$upn' -or SamAccountName -eq '$nomQ $prenomQ'" -Properties $props |   # Cherche par login, UPN ou ancien login "NOM Prenom"
         Select-Object -First 1   # Garde le premier résultat
    if ($u) { return $u }   # Compte trouvé directement : on le retourne

    $mots = @("$Nom $Prenom" -split '\s+' | Where-Object { $_ } | ForEach-Object { ConvertTo-Ascii $_ })   # Plan B : découpe la ligne en mots normalisés
    $trouves = @()   # Liste des comptes candidats
    foreach ($a in $mots) { foreach ($b in $mots) {   # Teste toutes les paires de mots
        if ($a -ne $b) {   # Évite de combiner un mot avec lui-même
            $c = Get-ADUser -Filter "SamAccountName -eq '$a.$b' -or UserPrincipalName -eq '$a.$b@$($Cfg.DomaineDNS)'" -Properties $props   # Cherche le login a.b
            if ($c) { $trouves += $c }   # Ajoute le compte trouvé aux candidats
        }   # Fin du if
    } }   # Fin des deux boucles
    $trouves = @($trouves | Sort-Object SamAccountName -Unique)   # Supprime les doublons
    if ($trouves.Count -eq 1) {   # Un seul candidat : sans ambiguïté
        Write-Log WARN "Ligne '$Nom,$Prenom' mal formee : compte retenu = $($trouves[0].SamAccountName)"   # Signale la correction automatique
        return $trouves[0]   # Retourne ce compte
    }   # Fin du if
    return $null   # Aucun ou plusieurs candidats : compte introuvable (par sécurité, on ne supprime rien)
}   # Fin de Find-User

if (-not (Test-Path $Cfg.DossierArchive)) {   # Si le dossier d'archives n'existe pas
    New-Item -ItemType Directory -Path $Cfg.DossierArchive -Force | Out-Null   # On le crée
    Set-FolderAcl -Path $Cfg.DossierArchive -Compte $null    # Droits : administrateurs + SYSTEM uniquement
}   # Fin du if

$rows = Import-CsvAuto -Path $CsvPath   # Lit le CSV des anciens collaborateurs
foreach ($row in $rows) {   # Traite chaque ligne
    try {   # Une erreur sur une ligne ne bloque pas les suivantes
        $nom = "$($row.nom)".Trim(); $prenom = "$($row.prenom)".Trim()   # Nom et prénom nettoyés
        $u = Find-User $nom $prenom   # Recherche le compte AD
        if (-not $u) {   # Compte non trouvé
            Write-Log WARN "Compte introuvable : $nom $prenom"   # Avertissement dans le log
            Add-Journal '' $nom $prenom '' '' '' 'Introuvable'   # Consigne l'échec dans le journal
            continue   # Passe à la ligne suivante
        }   # Fin du if
        if (-not $PSCmdlet.ShouldProcess($u.SamAccountName, 'Supprimer le compte et archiver le dossier')) { continue }   # En mode -WhatIf : simule seulement

        $ou = $u.DistinguishedName -replace '^CN=(?:\\.|[^,\\])+,', ''   # Mémorise l'OU d'origine (DN sans le "CN=...")
        $nomDossier = if ($u.HomeDirectory) { Split-Path $u.HomeDirectory -Leaf }   # Nom du dossier = dernier élément du chemin du lecteur P:
                      else { '{0}.{1}' -f (ConvertTo-Ascii $u.Surname), (ConvertTo-Ascii $u.GivenName) }   # Sinon, reconstitue nom.prenom
        $source = Join-Path $Cfg.DossierPartage $nomDossier   # Chemin actuel du dossier privé
        $dest   = Join-Path $Cfg.DossierArchive $nomDossier   # Chemin de destination dans les archives
        if (Test-Path $dest) { $dest = '{0}_{1}' -f $dest, (Get-Date -Format 'yyyyMMdd-HHmmss') }   # Si le nom existe déjà, ajoute la date pour ne rien écraser

        $archive = ''   # Chemin d'archive (vide si aucun dossier)
        if (Test-Path $source) {   # Si le dossier privé existe
            Move-Item -Path $source -Destination $dest   # Le déplace vers les archives
            Set-FolderAcl -Path $dest -Compte $null   # Restreint l'accès aux administrateurs et SYSTEM
            $archive = $dest   # Mémorise l'emplacement d'archive
        } else {   # Sinon
            Write-Log WARN "$($u.SamAccountName) : dossier $source introuvable"   # Avertit que le dossier est absent
        }   # Fin du if/else

        Remove-ADUser -Identity $u -Confirm:$false   # Supprime le compte AD sans demande de confirmation
        Add-Journal $u.SamAccountName $u.Surname $u.GivenName $ou $u.mail $archive 'Supprime'   # Consigne la suppression dans le journal
        Write-Log INFO "SUPPRIME : $($u.SamAccountName) - dossier archive : $archive"   # Trace dans le log
    } catch {   # En cas d'erreur sur cette ligne
        Write-Log ERREUR "$($row.nom) $($row.prenom) : $($_.Exception.Message)"   # Consigne le message d'erreur
    }   # Fin du catch
}   # Fin de la boucle
```

---

> ## 📋 Résumé du fonctionnement – Mission 2
>
> **Objectif :** supprimer proprement les comptes des collaborateurs ayant quitté ECOCERT, listés dans `SP1anciensUtilisateursEcocert.csv`, sans perdre leurs données.
>
> **Déroulement :**
> 1. `Run-Auto.ps1` détecte (empreinte SHA256) que le CSV a changé et lance `Remove-Utilisateurs.ps1`.
> 2. Le script crée, si besoin, le dossier `C:\Anciens Collaborateurs`, accessible uniquement aux administrateurs et à SYSTEM.
> 3. Pour chaque ligne du CSV :
>    - `Find-User` cherche le compte par login `prenom.nom`, UPN ou ancien format `NOM Prenom` ; si la ligne est mal formée (ex. `TERRA Idriss,Yves`), il teste les combinaisons de mots et ne retient le compte que s'il n'y a **qu'un seul** candidat ;
>    - si le compte est introuvable, un avertissement est écrit et la ligne est notée « Introuvable » dans le journal, sans rien supprimer ;
>    - le dossier privé est déplacé vers les archives (suffixe daté en cas de doublon) et ses droits sont restreints aux administrateurs ;
>    - le compte AD est supprimé ;
>    - la suppression est consignée dans `suppressions.csv` (date, login, nom, prénom, OU d'origine, mail, dossier d'archive, statut, opérateur).
> 4. Les événements sont aussi écrits dans le log mensuel `C:\Ecocert\Logs\ecocert_AAAA-MM.log`.
>
> **Points clés :** simulation possible avec `-WhatIf`, traçabilité complète, aucune perte de données (archivage avant suppression), une erreur sur une ligne n'arrête pas le traitement des suivantes.
