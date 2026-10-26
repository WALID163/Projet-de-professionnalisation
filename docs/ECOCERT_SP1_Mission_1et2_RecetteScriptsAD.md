# ECOCERT SP1 Mission1et2 – RecetteScriptsAD

## Modèle de recette de validation d'un prototype

---

### 1. Les utilisateurs sont créés et placés dans l'OU de leur service.

**Vérification sur l'AD :** `Gestionnaire de serveur > Outils > Utilisateurs et ordinateurs Active Directory`

- [X] Réussite
- [ ] Échec

En cas d'échec, proposer une hypothèse expliquant le dysfonctionnement.

................................................................................................................................................................

................................................................................................................................................................

................................................................................................................................................................

---

### 2. Les comptes ont le login, l'adresse mail et le mobile attendus.

**Commande utilisée :** `Get-ADUser -Identity nadia.jara -Properties EmailAddress, MobilePhone`

**Remarque :** résultat attendu : login `nadia.jara`, mail `n.jara@local.ecocert1.lan`, mobile au format `+33 x xx xx xx xx`.

- [X] Réussite
- [ ] Échec

En cas d'échec, proposer une hypothèse expliquant le dysfonctionnement.

................................................................................................................................................................

................................................................................................................................................................

................................................................................................................................................................

---

### 3. Le mot de passe doit être changé à la première connexion.

**Vérification sur l'AD :** `Utilisateurs et ordinateurs Active Directory > Propriétés de l'utilisateur > onglet Compte`

**Remarque :** la case « L'utilisateur doit changer le mot de passe à la prochaine ouverture de session » doit être cochée.

- [X] Réussite
- [ ] Échec

En cas d'échec, proposer une hypothèse expliquant le dysfonctionnement.

................................................................................................................................................................

................................................................................................................................................................................

................................................................................................................................................................

---

### 4. Les utilisateurs supprimés n'existent plus dans l'AD.

**Commande utilisée :** `Get-ADUser -Identity serre.rayan`

**Remarque :** le résultat attendu est une erreur « objet introuvable ». Pour une ligne mal formée du CSV (ex. `TERRA Idriss,Yves`), le login reconstitué n'existe pas : une erreur s'affiche et le compte reste dans l'AD.

- [X] Réussite
- [ ] Échec

En cas d'échec, proposer une hypothèse expliquant le dysfonctionnement.

................................................................................................................................................................

................................................................................................................................................................

................................................................................................................................................................

---
