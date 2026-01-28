# TP 2 - Exploration réseau

## 1. Affichage des infos et calculs
Pour ce TP, j’ai commencé par afficher les informations réseau de mon PC avec la commande ipconfig /all dans l’invite de commandes Windows.
J’ai ainsi pu voir que ma carte Wi-Fi est une Intel Wi-Fi 6 AX201, avec l’adresse IP 10.33.70.22, un masque de sous-réseau en 255.255.240.0 et une passerelle en 10.33.79.254.


Avec le masque /20, le pas est de 16 sur le troisième octet. Comme mon IP est dans les 70, je fais partie du réseau allant de 10.33.64.0 à 10.33.79.255. L’adresse réseau est donc 10.33.64.0 et l’adresse de broadcast est 10.33.79.255. La passerelle correspond au routeur du réseau, c’est elle qui permet à mon PC de sortir du réseau local pour accéder à Internet.

## 2. Modification d'IP et Scan réseau
J’ai ensuite modifié mon adresse IP manuellement. Les adresses utilisables dans ce réseau vont de 10.33.64.1 à 10.33.79.253, j’ai donc choisi une adresse libre dans cette plage et je l’ai configurée dans les paramètres IPv4 de Windows.
Avant de faire ce changement, j’ai scanné le réseau avec la commande nmap -sn 10.33.64.0/20 afin d’éviter de prendre l’adresse IP d’un autre appareil déjà connecté.



Après ça, j’ai fait un test en modifiant la passerelle par défaut. J’ai remplacé la vraie passerelle (10.33.79.254) par une adresse prise au hasard dans le réseau, par exemple 10.33.70.15. Résultat : plus d’accès à Internet. Cela montre que sans la bonne adresse du routeur, mon PC ne sait pas où envoyer les paquets pour sortir du réseau local.

## 3. DHCP et DNS
J’ai aussi observé le fonctionnement du DHCP. C’est lui qui attribue automatiquement une adresse IP à mon ordinateur. D’après ipconfig /all, le serveur DHCP est 10.33.79.254 et le bail est valable pendant environ 10 heures. Une fois ce temps écoulé, le PC doit redemander une adresse IP. J’ai pu forcer ce renouvellement avec les commandes ipconfig /release et ipconfig /renew.

Enfin, j’ai testé le DNS, qui sert à traduire les noms de domaine en adresses IP. Sur ma machine, les serveurs DNS utilisés sont 8.8.8.8 (Google) et 1.1.1.1 (Cloudflare). Avec la commande nslookup, j’ai testé la résolution de noms comme google.com, qui renvoie plusieurs adresses IP car Google utilise plusieurs serveurs, et ynov.com, qui renvoie l’IP du serveur web de l’école. J’ai aussi fait des tests de résolution inverse en entrant directement des adresses IP, ce qui permet d’obtenir des noms de domaine souvent liés à des fournisseurs d’accès.


Bilan
En conclusion, ce TP m’a permis de mieux comprendre comment fonctionne une connexion réseau : l’adresse IP sert à identifier la machine, le masque définit la taille du réseau, la passerelle permet l’accès à Internet, le DHCP gère tout automatiquement avec des baux, et le DNS permet de naviguer sur le web sans avoir à retenir les adresses IP.