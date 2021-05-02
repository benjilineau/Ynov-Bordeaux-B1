# Projet Infra/Si
> Abadie Arthur
> Gauthier Donatien
> Gélineau Benjamin 
> Souliac Raphaël



## Installation


### Prérequis

- GNS3 
- Une machine virtuelle sous GNS3 VM
- Une machine virtuelle sous windows server 2019
- Une machine virtuelle sous Windows 10 


### Préparation de GNS3

- Sur GNS3 Cliquer sur “Edit” ensuite “**Preferences**” puis “**GNS3 VM**”. Ensuite dans “**virtualization engine**” sélectionner VirtualBox puis dans “**VM Name**” choisir la VM créée auparavant  et enfin appuyer sur “**OK**”

- Télécharger image du routeur (cisco c 3640) sur ```https://drive.google.com/drive/folders/102jxZ9ECpe6ZFtXYdK_81iEVuuFoGOGR``` Cliquer sur “New Template” et  “Manually create a template”. Aller sur Dynamips => IOS router. Cliquer sur New et Run this IOS router on my local computer puis finir l’installation de l’image en sélectionnant l’image déja existante et en suivant les consignes d’installation.
  
- Importer les machines virtuelles windows server 2019 et windows 10 en allant sur “**Edit**”; “**Preferences**” puis soit "**Virtualbox vms**" ou "**Vmware vms**" selon votre hyperviseur. Cliquer sur "**new**" puis sélectionner votre machinne virtuelle correspondant au serveur et répéter l'opération pour la machine sous windows 10

- Reproduire le schéma ci-dessous ou réaliser le vôtre:![](https://i.imgur.com/zCmeszc.png)


### Step 2: Configuration du serveur

#### DNS et Active Directory

- Dans “**Tableau de bord**” cliquer sur “**Rôles des serveurs**” puis cocher les cases “**Services AD DS**” et “**Serveur DNS**”

- Dans “**DNS**” choisir "**type de zone principal**" dans l’assistant nouvelle zone
- Ensuite choisir de répliquer vos données DNS dans votre domaine créé 
- Cocher la zone de recherche inversée IPV4 
  puis choisir l’ID du réseau
 
- Enfin n’autoriser que les mises à jour dynamiques sécurisées 


- Ensuite aller dans “**AD DS**” puis dans “**configuration de déploiement**” pour y ajouter “**une nouvelle forêt**” et choisir le nom du domaine racine.


- Maintenant dans “**option du contrôleur de domaine**” cocher les cases “**Serveur DNS**” et “**Catalogue global**” et pour le niveau fonctionnel de la nouvelle forêt et domaine racine choisir “**Windows server 2016**” puis Le mot de passe du DSRM.


- Créer un utilisateur sur windows server dans “**Utilisateurs et ordinateurs Active Directory**” puis dans votre domaine local aller dans “**Users**” et faire un nouvel objet en lui donnant le nom désiré ainsi que son nom d’ouverture de session 
- Ensuite choisir votre mot de passe et cocher les cases "**l'utilisateur ne peut pas changer de mot de passe**" et "**le mot de passe n'expire jamais**"

### Step 3: Partage de fichier


- Pour le partage de fichier ouvrer l’Assistant nouveau partage situé dans Service de fichier et de stockage => partage .Cliquer sur "**Tâches**" puis "**Nouveau partage**" pour lancer l’assistant.
 
- Sélectionner le profil "**Partage SMB**" – Rapide puis sélectionner l’emplacement du dossier partagé. 
- Nommer ensuite le dossier. Pour finir activer l’énumération basée sur l'accès et autoriser la mise en cache du partage.Le partage devrait être créé.
 
- Vous pourrez gérer les permissions directemment en allant dans les propriétés du dossier partagé dans le windows server.

- Pour accéder au partage il vous suffit de vous connecter à un ordinateur windows appartenant au domaine que vous avez créé précédemment puis dans la barre de recherche taper "**Executer**" puis "**\\NomduServeur\NomDossierPartagé**"
![](https://i.imgur.com/Ud59aZP.png)
![](https://i.imgur.com/kHsUeC5.png)





### Step 4: VPN


Maintenant nous allons créer un VPN afin d'avoir accès au partage à partir du moment où vous avez internet.


- Premièrement aller dans l'assistant d'ajout de rôle sur Windows server, puis aller dans service de rôle et cocher les cases "**VPN** et **Routage**" 
- Aller dans configuration de l'accès distant et choisir "**déployer VPN uniquement**". Puis dans l'assistant d'installation choisir la "**configuration personnalisé**" et cocher "**Accès VPN, NAT**, et **Routage réseau** " 

- Dans les propriétés du serveur, dans sécurité entrer votre clé prépartagée et appliquer. Puis dans IPv4 activé le routage IPv4 et entrer une nouvelle plage d'adresse IPv4

- Modifier les services "**Sécurité IP(IKE)**";"**Sécurité IP(IKE NAT Traversal)**"; "**Passerelle VPN(L2TP)**"; "**Passerelle VPN(PPTP)**" en mettant pour tous en adresse privée 127.0.0.1.
![](https://i.imgur.com/FVsPUhe.png)

- Pour se connecter au vpn il faut ensuite aller dans les paramètres windows puis ajouter une connexion vpn.
![](https://i.imgur.com/GgtC311.png)
![](https://i.imgur.com/InUGfUQ.png)

### Step 5 : DHCP

- Dans “**Tableau de bord**” cliquer sur “**Rôles des serveurs**” puis cocher la case "**Serveur DHCP**" 
- Il n'y a plus qu'à faire suivant jusqu'à installer le serveur DHCP
- Suite à cela en haut à droite dans les notifications (représenter par un petit drapeau) puis cliquer sur "**Terminer la configuration DHCP**"
- Cliquer sur Suivant pour arriver dans les "**Autorisation**" Il suffit une fois sur cette page de cliquer sur le bouton "**Valider**" pour avoir la création de groupe et l'autorisation du DHCP auprès de L'Active Directory
- On se déplace ensuite dans le menu démarrer de windows puis dans les "**Outils d'administration Windows**" pour cliquer sur "**DHCP**" 
- Une fois dans le menu on développe notre serveur à droite puis il suffit de faire un clique droit sur "**IPv4**" puis sur "**Nouvelle étendue**"
- On rentre une adresse IP de début qui est 192.168.10.200 et une adresse de fin a 192.168.10.209 et on a plus qu'à continuer
- Il suffit ensuite d'appuyer 3 fois sur "**Suivant**" et ajouter l'adresse du routeur qui est de 192.168.10.1 plus qu'à faire encore une fois 3 fois suivant et notre étendue DHCP est configurée
- Il n'y a plus qu'a aller dans le centre Réseau et partage du patron et de l'employé cliquer sur "**Ethernet**" qui est en surbrillance puis sur "**Propriétés**" puis sur "**Protocole Internet version 4**" et cocher "**Obtenir une adresse IP automatiquement**" et "**Obtenir les adresses DNS automatiquement**" 
- C'est fini la vm est maintenant reliée à notre service DHCP !



### Step 6 : Sécurité

- Afin d'améliorer la sécurité des ordinateurs sous windows, nous vous conseillons de télécharger le logiciel usb disk security windows afin de vous prémunir de clé usb malveillantes et béneficier par la même occasion d'une navigation sur internet sécurisé.
- ![](https://i.imgur.com/SuFr4Bd.png)