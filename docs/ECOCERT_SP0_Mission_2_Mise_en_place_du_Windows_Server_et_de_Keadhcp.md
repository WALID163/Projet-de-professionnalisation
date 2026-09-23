Mise en place dans un environnement virtuel d'un contrôleur de domaine et d'un service DHCP
Table des matières
Mise en place d'un windows serveur virtuel avec proxmox
1.2) Mise en place du service DNS sur windows serveur 2025
1.3) Mise en place du service Active Directory (AD) ainsi que la promotion en contrôleur de domaine
1.4) Ajout d'utilisateurs via script
Mise en place d'un serveur DHCP sous DEBIAN 13
1) Mise en place d'un windows serveur virtuel avec proxmox

Sur Proxmox créer une VM avec une image ISO d'un Windows server version (récent de préférence) ou clonez un template.

Lancer la VM.

Puis se rendre directement dans les paramètres réseau pour l'introduire dans notre infrastructure :

1.2) Mise en place du service DNS sur windows serveur 2025

Ouvrir le gestionnaire de serveur puis cliquer sur gérer → puis suivant jusqu'à choisir le rôle du serveur que l'on souhaite, dans notre cas le service DNS sera coché.

→ Puis faire suivant jusqu'à l'installation.

1.3) Mise en place du service Active Directory (AD) ainsi que la promotion en contrôleur de domaine

Pour promouvoir le serveur en contrôleur de domaine, il sera nécessaire d'ajouter le service Active Directory.

→ De la même manière que pour le serveur DNS, il faudra cliquer sur gérer → puis suivant jusqu'à choisir le rôle du serveur (Active Directory).

→ Puis au niveau du drapeau avec le symbole danger en jaune, l'option de promouvoir le serveur sera présente.

→ Ajouter une nouvelle forêt.

    → Préciser le nom de domaine en question.

1.4) Ajout d'utilisateurs via script

Pour peupler l'AD si → beaucoup d'utilisateurs à entrer → l'utilisation d'un script et d'un CSV facilitera la tâche.

Au préalable, dans PowerShell Admin, effectuer :
