1. Analyse des logs SSH
J’ai commencé par me connecter en SSH à la VM via port forwarding VirtualBox (ssh vboxuser@127.0.0.1 -p 2222). J’ai ensuite vérifié les logs de connexion SSH dans /var/log/auth.log pour voir les connexions réussies et tentatives échouées. Cela permet de détecter des attaques par force brute ou des problèmes d’authentification.

<img src=image/screen1.png>

2. Transfert de fichiers sécurisés
Pour le transfert, j’ai créé un fichier test.txt sur Windows et je l’ai envoyé vers la VM avec SCP via le port 2222 (scp -P 2222 test.txt vboxuser@127.0.0.1:/tmp/) Sur la VM, j’ai vérifié l’arrivée du fichier avec ls -l et cat. ça montre le fonctionnement de SCP par dessus SSH.

<img src=image/screen2.png>
<img src=image/screen3.png>

3. Tunnels SSH (redirection de port local)
J’ai crée un tunnel SSH local pour accéder au port 80 de la VM depuis Windows (ssh -L 8080:localhost:80 vboxuser@127.0.0.1 -p 2222). ça permet d’accéder à un service distant non exposé directement. Le test curl http://localhost:8080 a renvoyé la page Nginx.

<img src=image/screen4.png>
<img src=image/screen5.png>

4. Installation et vérification Nginx
J’ai installé Nginx (sudo apt install nginx) et vérifié qu’il tournait (systemctl status nginx) et écoutait bien sur le port 80 (sudo ss -lntp | grep ':80'). Nginx est actif et en écoute sur 0.0.0.0:80.

<img src=image/screen6.png>
<img src=image/screen7.png>

5. Configuration Nginx HTTP (site statique)
J’ai créé un dossier web /srv/mon-site/html avec une page index.html, puis un server block dans /etc/nginx/sites-available/mon-site. J’ai activé le site avec un lien symbolique vers sites-enabled, supprimé le site default, testé la config (nginx -t) et rechargé (systemctl reload nginx). Le site est maintenant accessible via le tunnel.

<img src=image/screen8.png>
<img src=image/screen9.png>

6. HTTPS et certificats SSL/TLS
J’ai généré un certificat auto signé avec openssl req -x509 (valide 365 jours). J’ai modifié la config Nginx pour écouter sur 443 ssl avec les fichiers .crt et .key, et ajouté un bloc 80 qui redirige vers HTTPS (return 301 https://$host$request_uri;). Test config et reload réussis.

<img src=image/screen10.png>
<img src=image/screen11.png>

7. Test HTTPS via tunnel SSH
J’ai créé un tunnel pour le port 443 (ssh -L 8443:localhost:443) et testé avec curl -k https://localhost:8443 (le -k ignore l’avertissement certificat auto-signé). Le site s’affiche bien en HTTPS, prouvant le chiffrement TLS.

<img src=image/screen13.png>

8. Firewall et sécurité (UFW)
J’ai installé UFW (sudo apt install ufw), autorisé les ports SSH (22), HTTP (80) et HTTPS (443), activé le firewall (ufw enable) et vérifié le statut. Seuls les ports nécessaires sont ouverts, les autres sont bloqués par défaut.

<img src=image/screen14.png>

9. Vérification finale des logs
Pour finir, j’ai vérifié les logs Nginx (access.log) pour les requêtes web et auth.log pour les connexions SSH. Tout est cohérent avec les tests effectués.

<img src=image/screen15.png>

