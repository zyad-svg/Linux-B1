## TP 5

Partie 1 – Prise en main et sécurisation
1) Accès à l’interface
Adresse IP du LAN : 192.168.52.99/24 (interface LAN de pfSense, utilisée pour accéder au WebGUI).
Adresse IP du WAN : 10.0.2.15/24 (interface WAN de pfSense côté “Internet/VirtualBox”).
​
Pourquoi utilise-t-on HTTPS ?
On utilise HTTPS pour chiffrer l’administration (identifiants + actions dans l’interface), c'est pour éviter qu’un tiers sur le réseau intercepte ou modifies les échange entre le navigateur et pfSense.

Pourquoi changer les identifiants par défaut sur un pare-feu ?
Parce que les identifiants par défaut sont connus et souvent testés en premier : si quelqu’un accède au compte admin, il obtient un contrôle total sur le pare-feu (règles, NAT, services).

2) Sécurisation de l’accès administrateur
Où se gèrent les utilisateurs ?
Dans pfSense : System > User Manager (onglet Users).
​
​

Qu’est-ce qu’un mot de passe robuste ?
Un mot de passe robuste est long, unique, difficile à deviner (pas d’infos perso), et idéalement sous forme de phrase de passe avec diversité de caractères, ce qui réduit le risque d’attaque par dictionnaire/brute force.
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

