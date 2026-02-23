## TP 5

Partie 1 – Prise en main et sécurisation
1) Accès à l’interface
Adresse IP du LAN : 192.168.52.99/24 (interface LAN de pfSense, utilisée pour accéder au WebGUI).
Adresse IP du WAN : 10.0.2.15/24 (interface WAN de pfSense côté “Internet/VirtualBox”).
​
Pourquoi utilise-t-on HTTPS
On utilise HTTPS pour chiffrer l’administration (identifiants + actions dans l’interface), c'est pour éviter qu’un tiers sur le réseau intercepte ou modifies les échange entre le navigateur et pfSense.

Pourquoi changer les identifiants par défaut sur un pare-feu ?
Parce que les identifiants par défaut sont connus et souvent testé en premier : si quelqu’un accede au compte admin, il obtient un controle totale sur le pare-feu.

2) Sécurisation de l’accès administrateur
Où se gèrent les utilisateurs ?
Dans pfSense : System > User Manager (onglet Users).
​

Qu’est-ce qu’un mot de passe robuste ?
Un mot de passe robuste est long unique difficile à deviner pas d’infos perso, et idéalement sous forme de phrase de passe avec différent charactére , ce qui réduit le risque d’attaque par dictionnaire/brute force.
​

Pourquoi sécuriser en priorité l’accès admin sur un équipement réseau ?
Parce que l’accès admin donne des privilèges critiques : modifier les règles de filtrage, ouvrir l’accès depuis l’extérieur, rediriger le trafic, ou désactiver des protections.

<img src=image/2.png> 

​

​

Partie 2 – Comprendre les interfaces réseau
3) Vérification des interfaces
Quelle interface permet l’accès Internet ?
L’interface WAN (sur ton installation : WAN = em0).
​
​

Quelle interface correspond au réseau interne ?
L’interface LAN (sur ton installation : LAN = em1).
​
​

Que se passerait-il si les interfaces étaient inversées ?
Le réseau interne pourrait se retrouver exposé côté “WAN”, tu pourrais perdre l’accès à l’interface d’administration, et les règles/NAT seraient appliqués sur la mauvaise interface (donc soit ça ne marche plus, soit c’est dangereux).
​


<img src=image/1.png>
<img src=image/3.png>



Partie 3 – Configuration des services réseau
4) DHCP
Ce que j’ai fait (procédure courte) :
Dans Services > DHCP Server > LAN j’ai activé le serveur DHCP sur l’interface LAN et defini la plage d’attribution 192.168.52.10 a 192.168.52.100.
​
Puis j’ai vérifié l’attribution d’un bail dans Status > DHCP Leases
​
​
​<img src=image/dhcpserver.png>
<img src=image/dhcpleases.png>


Pourquoi utiliser DHCP plutôt qu’une IP fixe ?
DHCP automatise l'attribution des parametres reseau (IP/masque/passerelle/DNS), reduit les erreurs humaine et evite les conflits d’IP quand il y a plusieurs machine.
Quelle plage d’adresses choisir ?
Une plage dans le sous-réseau du LAN assez grande pour les clients, en gardant des IP hors plage pour les IP fixes/réservations. Ici : 192.168.52.10 → 192.168.52.100.
​
​
Quelles adresses faut-il éviter d’inclure dans la plage ?
L’IP de pfSense gateway LAN et les IP qu’on veut garder en statique serveurs/services/réservations DHCP pour eviter des doublons.
Vérification : Ubuntu obtient elle automatiquement une adresse IP ?
Oui : pfSense affiche un bail DHCP actif pour Ubuntu avec l’IP 192.168.52.10.*

5) DNS
Ce que j’ai fait  :
Dans Services > DNS Resolver j’ai activé le DNS Resolver (Unbound) pour que pfSense puisse résoudre les noms de domaine pour les clients du LAN.
​
​
J’ai validé le bon fonctionnement avec un ping IP + un ping nom de domaine depuis Ubuntu.
​

Où mettre les screens :

​<img src=image/dnssolver.png>
​
​<img src=image/pinggoogle.png>
​

Questions :

Pourquoi un pare-feu peut-il jouer le rôle de serveur DNS ?
Parce que pfSense intègre un service DNS (DNS Resolver/Unbound) capable de répondre aux requêtes DNS des machines du LAN.
​

Que se passe-t-il si le DNS ne fonctionne pas mais que le ping vers 8.8.8.8 fonctionne ?
La connectivité IP vers Internet est OK (routage/NAT), mais la résolution des noms est KO : les sites “par nom” ne marchent pas (ex google.com), alors que les accès par IP peuvent marcher.

Partie 4 – Autoriser l’accès Internet
6) Règles de pare-feu (LAN → Internet)
Ce que j’ai fait / constaté :
Dans Firewall > Rules > LAN, une règle “Default allow LAN to any rule” autorise le trafic sortant depuis le réseau LAN vers n’importe quelle destination (règle évaluée dans l’ordre, de haut en bas, premier match).
​
​
Cette règle permet à Ubuntu d’accéder à Internet (tests ping IP et ping DNS OK).
​

Où mettre le screen :

​<img src=image/lannet.png>
​

Questions :

Quelle doit être la source ?
La source doit être le réseau LAN , c'est les machines internes (Ubuntu).
​
​
Quelle doit être la destination ?
La destination doit être WAN net, pour autoriser la sortie vers Internet.
​
​

Faut-il autoriser tous les protocoles ?
On peut autoriser any pour valider l’accès Internet, mais en pratique on limite au besoin (ex DNS/HTTP/HTTPS) pour réduire la surface d’attaque.
​

Tests à indiquer  :

Ping pfSense (gateway) : OK (connectivité LAN).

Ping 8.8.8.8 OK (Internet IP).
​

Ping google.com: OK (DNS OK).
​

Si ça ne fonctionne pas : où regarder ?

Status > System Logs > Firewall pour voir les paquet bloques et la regle associée Tracker/Matched Rule.

Vérifier Firewall > Rules > LAN (ordre des règles haut en bas) et s’assurer qu’une règle Pass existe avant un posible blocage.
​

Vérifier le NAT sortant (Outbound).
​

7) NAT sortant
Ce que j’ai fait/vu :
Dans Firewall > NAT > Outbound, le mode est en Automatic outbound NAT, ce qui génère automatiquement les règles NAT pour traduire le trafic des réseaux internes vers l’adresse WAN.
​

​<img src=image/natoutbund.png>
​

Questions 

Pourquoi le NAT est nécessaire avec une interface WAN en NAT ?
Parce que les IP du LAN sont privee et non routable sur Internet le NAT traduit les connexion sortante en utilisant l’adresse WAN pour permettre l’accès Interne.
​

Différence entre NAT automatique et manuel ?
Automatic: pfSense genere et maintient les regles automatiquement  Manual : l’admin crée et gere toutes les regles plus flexible, mais plus de risques d’erreur.
​

Comment vérifier qu’une traduction d’adresse a lieu ?
En constatant que les clients du LAN accèdent à Internet et en vérifiant la page Outbound NAT (mode automatique + règles automatiques affichées) ; on peut aussi correler avec les logs/états si nécessaire.
​
​

