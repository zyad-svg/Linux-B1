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

Partie 5 – Filtrage
8. Blocage d’un site spécifique (perdu.com)
Ce que j'ai fait :
j'ai identifie l'adresse  du site perdu.com (Cloudflare). J'ai ensuite crér un alias FQDN qui va vers ce site dans Firewall > Aliases . après j'ai créé une regle dans Firewall > Rules > LAN pour bloquer tout le trafic sortant vers cet alias et j'ai active la journalisation des paquets sur cette règle.

​<img src=image/site_bloque.png>

​<img src=image/SITE_BLOQUE2.png>

​<img src=image/preuvesite.png>

​<img src=image/preuvesite2.png>


reponse 
Faut-il bloquer par IP ou par nom de domaine ?
C'est mieux  de bloquer par nom de domaine. Les adresses IP des sites web, surtout ceux utilisant des reseaux de distribution de contenu  comme Cloudflare, peuvent changer fréquemment ou utiliser de nombreuses adresses. En créant un alias FQDN, pfsense ce charge de resoudre et mettre à jour l'IP automatiquement en arrière plan.

Que se passe-t-il si le site utilise HTTPS ?
Si le site utilise HTTPS, le pare-feu classique ne peut pas lire le contenu de l'URL car le trafic est chiffre il  voit que le trafic sur le port 443. mais, le blocage fonctionne quand même car le pare-feu intervient au niveau réseau (couches 3 et 4) il bloque la coo vers l'adresse IP de destination avant même que la négociation HTTPS ne commence

Pourquoi le blocage par IP peut-il être contourné ?
Le blocage par IP stricte est pas fiable pour les grand sites web. Le site peut modifier ses enregistrements DNS pour pointer vers une nouvelle adresse IP et l'utilisateur  peut utiliser un VPN ou un proxy pour contourner la restriction locale en plus, bloquer l'IP d'un grand CDN peut bloquer involontairement pleins d'autres sites legitimes hebergés sur la même infrastructure.

9. Blocage d’une catégorie de sites (jeux d’argent)
Ce que j'ai configuré
pour creer une solution maintenable pour bloqué plusieur sites de la même catégorie, j'ai créé un alias nommé GAMBLING de type Host dans Firewall > Aliases J'y ai ajouté les différents noms de domaine des sites cibles (betclic.fr, winamax.fr, pokerstars.fr). Ensuite, dans Firewall > Rules > LAN j'ai créé une unique règle de blocage ciblée sur cet alias, en m'assurant de la placer tout en haut de la liste pour qu'elle soit prioritaire sur la règle d'autori sation générale.


​<img src=image/gambling1.png>

​<img src=image/journalblocage.png>


Pourquoi ne pas créer une règle par site ?
Créer une regle de pare feu individuel pour chaqu site web rendrait la table de filtrage très longue, illisible, complexe à analyser et difficile a gardé. L'utilisation d'un alias permet de regrouper et de centraliser tous les sites d'une categorie dans un seul objet qui n'est appellé que par une seule regle de pare-feu dcp pour ajouter ou supprimer un site de la liste de blocage il suffit de modifier l'alias sans avoir besoin de manipuler les règles de filtrage ou de risquer de casser leur ordre

Où se créent les alias ?
Les alias se creent et se gerent depuis le menu principal de pfsense en naviguant dans Firewall > Aliases.

Comment vérifier qu’une règle bloque réellement le trafic ?
pour verifier le bon fonctionnement d'une regle de blocage on doit effectuer deux etapes. D'abord, il faut s'assurer que la journalisation est activée dans les paramètres de la règle (option "Log packets that are handled by this rule"). après apres avoir effectue une tentative de connexion depuis une machine cliente il faut se rendre dans le menu Status > System Logs > Firewall. Les paquets arretes y apparaissent avec une icone de croix rouge et on peut verifier que l'action a bien été declenchee par la bonne rgle en inspectant l'identifiant associe.


11. Règles horaires
Ce que j'ai fait :
Pour cette partie j'ai crée un horaire dans Firewall > schedules pour la pause du midi de 12h a 14h
Apres je suis retourné modifier ma regle de blocage de l'exo 9 pour lui appliquer ce planning dans les options avancés tout en bas
Du coup maintenant la regle s'active que pendant ces heures la


​<img src=image/pause_12.png>

​<img src=image/pause_2.png>


Questions :

Pourquoi les règles horaires sont-elles utiles en entreprise ?
en entreprise c'est super  pour gerer la productivité des employés et la bande passante
Par eemple on peut bloquer les reseau sociaux ou youtube pendant les heures de travail mais les laisser ouvert pendant la pause du midi
Ca sert aussi beaucoup pour la securite pour bloquer completement les acces au reseau la nuit ou le week end quand ya personne au bureau.
Comme ca on limite les risques d'attaques quand le service info est pas la et ca evite de devoir activer et desactiver les regles a la main tout les jours.


12. Serveur web local
j'ai d'abord installé un serveur web Apache basique sur ma machine Ubuntu (sudo apt install apache2) apres je suis allé dans pfsense pour faire deux regles sur l'interface LAN.
La première regle (en haut) autorise tout le monde (Source : Any) à accéder à l'IP de mon Ubuntu (Destination : 192.168.52.10) mais uniquement sur le port 80 (HTTP).
La deuxième règle (juste en dessous) bloque absolument tout le reste du trafic vers l'IP de mon Ubuntu. Comme les règles sont lues de haut en bas, le port 80 passe, et le reste se fait jeter.

​<img src=image/regles_lan.png>

Filtrer par IP source ?
Dans ce cas precis on a mis l'IP source sur "Any"  parce qu'un serveur web est cense être accessible a tous par definition. Par contre, si c'était une interface d'administration privée, là on filtrerait avec l'IP source de l'admin pour bloquer les autres.

Filtrer par port ?
Oui, c'est super important. Le but c'est de ne laisser ouvert que le strict minimum (ici le port TCP 80 pour le web). La deuxième règle bloque tous les autres ports. Ça reduit a fond la surface d'attaque : si le serveur a une faille sur un autre service, le pare-feu empêchera les pirates d'y accéder.

Pourquoi le pare-feu protège-t-il le LAN même en réseau interne ?
Le pare-feu ne sert pas qu'à bloquer les hackers d'Internet. En interne, il permet de faire de la segmentation isoler les machines les unes des autres. Si un PC d'un employé dans le LAN chope un virus, le pare-feu va bloquer les flux bizarres et empêcher le virus de se propager vers nos serveurs importants. C'est ce qu'on appelle empêcher les mouvements latéraux dans le réseau 


13. Logs et analyse
Pour cette partie j'ai juste analysé les logs dans Status > System Logs > Firewall qu'on a generé dans les exos avant avec nos test de ping et les sites bloqués.

Question:

Différence entre paquet bloqué et autorisé :
Dans les logs c'est assez simple a voir Si le paquet est bloquer ya une petite croix rouge dans la colonne action, et si c'est autorisé (pass) c'est un check vert. De base pfsense logue que ce qui est bloqué, si on veut voir ce qui passe faut penser a cocher la case log packet dans la regle verte.

Identifier quelle règle a déclenché le blocage :
Pour trouver quelle regle exactement a bloqué le truc, suffit de cliquer ou de passer la souris sur la croix rouge dans le log. Ca ouvre une info bulle avec le nom de la regle (genre "Default deny rule" ou le nom qu'on a mis comme "Block GAMBLING") et un Tracker ID (un numero unique pour la regle).

Comprendre le sens du trafic :
Dans pfsense la plupart des regles s'appliquent sur le trafic qui rentre dans l'interface Inbound. Si par hasard le paquet est bloqué a la sortie de l'interface Outbound ca se voit dans les logs parce qu'il ya un petit symbole en forme de fleche  a coté du nom de l'interface .



14. DMZ
Ce que j'ai fait :
Pour faire la DMZ j'ai du d'abord eteindre la vm pfsense pour lui rajouter une 3eme carte reseau dans virtualbox que j'ai mis en reseau interne (DMZ_net). Apres j'ai redemarré et je suis allé dans interfaces > assignments pour l'ajouter. Je l'ai renommé en DMZ et je lui ai donné l'ip 192.168.100.1 pour qu'elle soit dans un sous reseau different de mon LAN.
Ensuite je suis allé dans firewall > rules > DMZ pour faire les regles de securité. J'ai fais une premiere regle en haut qui bloque tout le trafic de la DMZ vers le LAN (pour pas qu'un hacker puisse rentrer dans le reseau privé) et une deuxieme regle en dessous qui autorise la DMZ a aller partout ailleurs (pour qu'elle ait quand meme internet).


​<img src=image/DMZ1.png>


Questions :

Qu'est ce qu'une DMZ ?
C'est une zone demilitarisée en gros c'est un reseau a part qui est entre internet et notre vrai reseau privé (LAN). On met dedans tout les serveurs qui doivent etre accessible depuis l'exterieur comme les serveurs web ou mail.

Pourquoi isoler un serveur ?
Parce que un serveur web ouvert sur internet c'est la premiere cible des hackers. Si on le met dans le LAN normal et qu'il se fait pirater le hacker a direct acces aux pc des employés et aux donnees sensibles. En l'isolant dans une DMZ meme si le serveur tombe le hacker reste coincé dedans et peut pas aller plus loin.

Une machine en DMZ peut-elle accéder au LAN ?
Non surtout pas c'est tout l'interet de la DMZ. On met des regles de pare feu tres stricte (comme ma regle block de tout a l'heure) pour bloquer completement les connexions qui viennent de la DMZ et qui essayent d'aller vers le LAN.

Le LAN peut-il accéder librement à la DMZ ?
Oui en general le LAN a le droit d'aller vers la DMZ pour que les dev puissent mettre a jour le site web par exemple. Vu que le pare feu est "stateful" il laisse passer la connexion dans un sens et il autorise juste la reponse a revenir dans l'autre sens.


15. Filtrage MAC
Le filtrage MAC est-il réellement sécurisé ?
Non le filtrage par adresse MAC c'est pas consideré comme une mesure de securité fiable. C'est tout au plus une petite barrière administrative pour empêcher les utilisateurs honnete  de brancher n'importe quel appareil sur le réseau.
​

Pourquoi est-il facilement contournable ?
Il est très facile à contourner parce que les adresses MAC circulent  sur le réseau (non chiffrées). Un attaquant n'a qua écouter le trafic réseau avec un logiciel comme Wireshark ou Airodump-ng pour capturer l'adresse MAC d'un appareil autorise qui communique avec le routeur. Après il lui suffit de modifier sa propre adresse MAC  via les parametres de son système d'exploitation ou avec un outil comme macchanger sous Linux, pour se faire passer pour l'appareil légitime et entrer sur le réseau.
​
