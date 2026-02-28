# TP OpenVPN - Mise en place d'un serveur

## preparation du systeme
pour commencer jai mis a jour mon serveur ubuntu et installer les paquets necessaire avec la commande `sudo apt update` et `sudo apt install openvpn easy-rsa -y`.
 
​<img src=image/install.png>
 
## partie 1 : comprendre la pki et creation
 
**questions de theorie:**
- a quoi sert une CA ?
 sa sert a signer les certificats du serveur et des clients pour prouver leur identité. cest la base de la confiance de notre vpn, tout le monde lui fait confiance.
- diff entre clé privé et certificat ? 
la clé privé faut absolument la garder secrete sur la machine, elle sert a prouver notre identité mathématiquement. le certificat cest public, on l'envoi aux autres pour quil puisse verifier notre identité.
- pourquoi le serveur vpn a besoin de certificat ? 
pour prouver aux clients qui se connecte au vrai serveur et pas a un pirate qui ecoute le reseau. et inversement le serveur s'en sert pour verifier les clients.
 
**creation de linfrastructure easy-rsa:**
jai preparer un dossier pour easy-rsa et lancer un init-pki pour avoir un environnement propre. ensuite jai generer la CA, la clé serveur, le certif client, les parametres DH et la clé TLS.
 
​<img src=image/screen1.png>
 
​<img src=image/screen2.png>
 
 
**questions easy-rsa:**
- ou easy-rsa cree ses fichiers ? 
il les met par defaut dans le sous-dossier `pki/` la ou on a lancer l'init.

- que contient le dossier pki/ ? 
ya plusieurs trucs mais le plus important cest le dossier `private` (qui a les cles privés), le dossier `issued` (avec les certificats signés) et `reqs` (les demandes). ya aussi le ca.crt a la racine de pki.

- diff entre gen-req et sign-req ?
 gen-req sa fabrique juste la clé privé et une demande de signature (.req). sign-req sa prend cette demande et utilise la CA pour la valider et faire le vrai certificat final (.crt).

- si on oublie de signer ? 
le certificat marchera pas. le serveur vpn refusera la connexion du client parce que le certificat naura pas la signature de la CA de confiance.


## partie 2 : configuration du serveur openvpn

### 2.1 mise en place du fichier server.conf
jai copier les certificats et cles generés avec easy-rsa dans `/etc/openvpn/server/` pour avoir un chemin clair et standard. ensuite jai creer le fichier `/etc/openvpn/server/server.conf` avec un serveur en udp sur le port 1194, une interface tunnel `tun` et un reseau vpn dedié en `10.8.0.0/24`.

​<img src=image/partie2-1.png>

​<img src=image/partie2-2.png>


**questions**
- `dev tun` signifie quon crée une interface tunnel IP (couche 3) pour faire passer du trafic routé
- udp vs tcp : udp est souvent plus adapté pour un vpn (moins de surcouche), tcp peut aider si udp est bloqué mais peut faire perdre en perf
- la plage IP du vpn doit etre une plage privée differente du LAN pour eviter les conflits (ex 10.8.0.0/24)

### 2.2 routage et nat
pour permettre aux clients du vpn davoir acces a internet, jai activé le forwarding ipv4 avec `net.ipv4.ip_forward=1`. ensuite jai ajouté une regle nat (MASQUERADE) sur linterface qui sort sur internet, pour que le trafic du reseau 10.8.0.0/24 soit traduit avec lip du serveur.

​<img src=image/partie2-3.png>
​


<img src=image/partie2-4.png>

**questions**
- ip_forward se configure avec sysctl, et de facon permanente dans `/etc/sysctl.conf` (ou sysctl.d)
- pour voir les regles nat : `iptables -t nat -S`
- masquerader est necessaire car les ip du vpn sont privées et ne peuvent pas sortir telles quelles sur internet

### 2.3 demarrage et debug du service
jai demarré le service openvpn avec systemd et verifier l'etat. en cas derreur, jai utiliser `journalctl -u openvpn-server@server` pour lire les logs et corriger les chemins des certificats si besoin.

​<img src=image/partie2-5.png>


​<img src=image/partie2-6.png>


**questions**
- les logs dun service systemd se voient avec `journalctl -u <service>`
- `systemctl status` donne un resume rapide, `journalctl` donne le detail des logs


## tests et validation
 
une fois mon fichier `client1.ovpn` complet (avec toutes les cles inline), je lai recuperer sur mon pc windows. je lai ensuite importer dans le logiciel officiel openvpn connect en utilisant l'option "File".
 
la connexion sest lancé directement. l'interface de l'application est passé au vert (statut "Connected"), ce qui prouve que lauto-authentification via les certificats de la pki fonctionne bien.
 
 
pour valider que le reseau marche reellement, jai fait quelques tests dans linvite de commande (cmd) :
1. avec un `ipconfig`, jai pu voir que ma carte openvpn avais bien recu l'adresse ip `10.8.0.2` (distribué par le serveur ubuntu).
2. jai lancer un `ping 8.8.8.8` qui a repondu avec succes. ca confirme que la regle NAT iptables (masquerade) de la partie 2 fait le job et permet de sortir sur internet depuis le tunnel vpn.
 
 
**questions:**
- comment verifier que votre trafic passe par le VPN ?
 la methode la plus simple cest daller sur un site comme mon-ip.com depuis le navigateur windows. si l'ip affiché est celle de la vm ubuntu et non plus celle de ma box internet, cest que tout mon trafic web passe bien dans le tunnel. on peu aussi utiliser la commande `tracert 8.8.8.8` pour verifier que le premier saut correspond a lip de la passerelle vpn (`10.8.0.1`).

- que se passe-t-il si le port 1194 est bloqué ? 
la connexion vpn echouera. le client va envoyer des trames udp dans le vide et openvpn affichera une erreur "connection timeout" car il n'aura aucune reponse du serveur. pour contourner les blocages (souvent dans les ecoles ou wifi publics), la technique consiste a passer le serveur en tcp sur le port 443 pour simuler du trafic web (https) classique.

