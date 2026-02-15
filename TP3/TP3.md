B1 Linux — TP3 : Cryptographie avec OpenSSL

A) Base64

Génération d’un fichier binaire
J’ai commencé par créer un fichier binaire data.bin de 100 Ko de données aléatoires, afin d’avoir un contenu non-textuel (octets quelconques) typique d’un flux binaire. La commande utilisée est :

dd if=/dev/urandom of=data.bin bs=1k count=100

Ensuite j’ai vérifié la taille du fichier pour confirmer qu’on est bien sur 100 Ko :

ls -l data.bin

Constat attendu : data.bin fait environ 102400 octets (100×1024) et son contenu est illisible lorsqu’on tente un affichage direct, car il s’agit de données aléatoires.

Encodage Base64
J’ai encodé le fichier binaire data.bin en Base64 dans data.b64 :

openssl base64 -e -in data.bin -out data.b64

J’ai ensuite affiché le contenu de data.b64 :

cat data.b64

Constat : data.b64 est un texte “propre” composé de caractères ASCII (lettres, chiffres, +, / et = pour le padding), éventuellement avec des retours à la ligne selon la sortie. On est donc passé d’un flux binaire à un flux texte.

J’ai comparé les tailles :

ls -l data.bin data.b64

Constat important : data.b64 est plus gros que data.bin. L’augmentation est normale parce que Base64 encode 3 octets en 4 caractères, donc on a un surcoût d’environ 33% (plus un léger overhead dû aux retours à la ligne éventuels).

Décodage et restauration
J’ai décodé data.b64 pour reconstruire un fichier binaire data_restored.bin :

openssl base64 -d -in data.b64 -out data_restored.bin

Pour vérifier rigoureusement que data_restored.bin est strictement identique à data.bin, j’ai utilisé une comparaison binaire :

diff -s data.bin data_restored.bin

Constat : la commande doit indiquer que les fichiers sont identiques. Une méthode encore plus “crypto-compatible” serait de comparer des empreintes SHA-256, mais ici diff suffit à valider l’identité byte-à-byte dans le cadre du TP.

Réponses aux questions
Base64 n’est pas un chiffrement : c’est un encodage. Il n’y a pas de notion de secret, de clé ou de mot de passe. Toute personne qui possède data.b64 peut revenir au binaire original via un décodage Base64. Le but est la compatibilité (faire passer du binaire sur des canaux texte), pas la confidentialité.

La taille change après encodage car l’encodage Base64 représente les données binaires sous une forme texte moins dense (3 octets → 4 caractères), ce qui augmente la taille d’environ 33%.

Le pourcentage d’augmentation est approximativement 33% (en pratique un peu plus selon les retours à la ligne).

La méthode la plus rigoureuse pour vérifier l’identité de deux fichiers est soit une comparaison byte-à-byte (diff/cmp), soit la comparaison d’empreintes cryptographiques (ex: SHA-256) : si les empreintes sont identiques, la probabilité d’une différence non détectée devient négligeable.

B) Chiffrement symétrique — AES

Création du message
J’ai créé un fichier confidentiel.txt contenant mon nom, la date du jour et au moins cinq lignes de texte, afin de simuler un contenu réellement confidentiel. Exemple de contenu :

Nom : <Ton Nom>
Date : <YYYY-MM-DD>
Ligne 1 : …
Ligne 2 : …
Ligne 3 : …
Ligne 4 : …
Ligne 5 : …

J’ai affiché le fichier pour vérifier qu’il est correct :

cat confidentiel.txt

Chiffrement AES-256 avec sel + dérivation robuste
J’ai chiffré confidentiel.txt en AES 256 bits, avec un sel et PBKDF2 (dérivation robuste de clé) en utilisant SHA-256 comme fonction de hachage de dérivation :

openssl enc -e -salt -in confidentiel.txt -out confidentiel.enc -aes256 -pbkdf2 -md sha256

Pendant l’exécution, OpenSSL demande un mot de passe : ce mot de passe ne devient pas directement la clé AES, il sert à dériver une clé via PBKDF2. PBKDF2 ajoute un coût (itérations) pour ralentir les attaques par bruteforce et rendre les mots de passe faibles plus difficiles à casser. La doc précise que -pbkdf2 active PBKDF2 (avec un nombre d’itérations par défaut) et que -md définit le digest utilisé pour la dérivation.

J’ai vérifié que confidentiel.enc est bien binaire. Un test simple est d’essayer d’afficher :

cat confidentiel.enc

Constat : l’affichage donne des caractères incohérents/illisibles, ce qui montre que le résultat est bien un flux binaire (et non du texte).

Déchiffrement et vérification
J’ai déchiffré le fichier chiffré vers confidentiel_dechiffre.txt :

openssl enc -d -in confidentiel.enc -aes256 -pbkdf2 -md sha256 -out confidentiel_dechiffre.txt

Il est important de réutiliser les mêmes paramètres (algorithme, PBKDF2, digest) : si une option critique change, la clé dérivée ne correspond plus et le déchiffrement échoue ou produit un contenu corrompu. La doc OpenSSL rappelle que les options doivent être cohérentes entre chiffrement et déchiffrement.

J’ai ensuite comparé confidentiel_dechiffre.txt avec l’original :

diff -s confidentiel.txt confidentiel_dechiffre.txt

Constat : les fichiers doivent être identiques.

Analyse : chiffrement deux fois avec le même mot de passe
J’ai rechiffré le même fichier confidentiel.txt une seconde fois avec le même mot de passe, vers un autre fichier (ex: confidentiel2.enc) en utilisant exactement la même commande de chiffrement. Puis j’ai comparé confidentiel.enc et confidentiel2.enc.

Constat attendu : les deux fichiers chiffrés sont différents, malgré un clair identique et un mot de passe identique. La raison principale est l’utilisation du sel (-salt) qui introduit de l’aléatoire ; OpenSSL indique d’ailleurs que le sel est stocké avec le ciphertext quand il est généré aléatoirement, pour permettre le déchiffrement ultérieur.

Réponses aux questions
Les deux fichiers chiffrés sont différents parce que le sel aléatoire fait que la dérivation de clé (et les paramètres internes comme l’IV) changent à chaque chiffrement, empêchant de reconnaître qu’on a chiffré deux fois le même contenu.

Le rôle du sel est d’empêcher les attaques efficaces par dictionnaire pré-calculé et d’éviter qu’un même mot de passe donne toujours le même résultat sur un même fichier. OpenSSL insiste que -salt doit être utilisé lors d’une dérivation depuis un mot de passe, sinon on facilite des attaques par dictionnaire.

Si une option change au déchiffrement (mauvais algorithme, oubli de -pbkdf2, digest différent, etc.), la clé dérivée ne sera pas la bonne : on obtient un échec ou un résultat incohérent. La doc souligne aussi que certains paramètres (ex: longueur de sel avec -saltlen) doivent correspondre entre chiffrement et déchiffrement.

On utilise PBKDF2 pour rendre la dérivation de clé coûteuse et donc ralentir fortement les tentatives massives de mots de passe. OpenSSL précise que -iter permet d’augmenter le nombre d’itérations et donc le coût pour un attaquant.

La différence entre encodage et chiffrement : l’encodage change la représentation (sans secret), le chiffrement protège la confidentialité en nécessitant une clé/mot de passe.

C) Cryptographie asymétrique — RSA

Génération de clés RSA 2048 bits
J’ai généré une clé privée RSA 2048 bits :

openssl genrsa -out rsa_private.pem 2048

Le fichier est en format PEM (texte Base64 avec en-têtes), donc affichable :

cat rsa_private.pem

J’ai inspecté les paramètres mathématiques de la clé privée :

openssl rsa -in rsa_private.pem -text -noout

Constat : on voit le modulo (n), l’exposant public (e), mais aussi des paramètres privés supplémentaires (exposant privé d, facteurs premiers p et q, etc.). Ces éléments doivent rester secrets, car ils permettent de déchiffrer et de signer.

J’ai ensuite protégé la clé privée par chiffrement (passphrase) afin que, même si le fichier est copié, il ne soit pas utilisable immédiatement sans mot de passe. (La commande dépend des options demandées par l’enseignant, mais le principe est : “clé privée chiffrée + passphrase”.)

Export de la clé publique
J’ai extrait la clé publique à partir de la clé privée :

openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem

J’ai affiché la clé publique :

cat rsa_public.pem

Puis j’ai affiché ses paramètres :

openssl rsa -in rsa_public.pem -pubin -text -noout

Observation : la clé publique contient essentiellement le modulo (n) et l’exposant public (e). Contrairement à la clé privée, elle ne contient pas les paramètres secrets (d, p, q).

Chiffrement / déchiffrement d’un fichier
J’ai créé un petit fichier secret.txt (quelques caractères). Puis je l’ai chiffré avec la clé publique :

openssl pkeyutl -encrypt -in secret.txt -inkey rsa_public.pem -pubin -out secret.enc

Ensuite je l’ai déchiffré avec la clé privée :

openssl pkeyutl -decrypt -in secret.enc -inkey rsa_private.pem -out secret_dechiffre.txt

J’ai vérifié que secret_dechiffre.txt correspond exactement à secret.txt :

diff -s secret.txt secret_dechiffre.txt

Réponses aux questions
La clé privée ne doit jamais être partagée car elle permet de déchiffrer tous les messages chiffrés pour toi et de produire des signatures valides qui prouvent (cryptographiquement) que “toi” tu as signé. Si quelqu’un l’a, il peut se faire passer pour toi.

RSA n’est pas adapté au chiffrement de gros fichiers car c’est coûteux, lent, et limité en taille de message chiffrable (selon padding). En pratique, on utilise RSA pour chiffrer une petite donnée (souvent une clé AES), puis AES pour chiffrer le gros document (schéma hybride).

Les différences observées entre paramètres public et privé : la clé publique expose les éléments nécessaires à chiffrer/vérifier (n, e), la clé privée contient des paramètres supplémentaires permettant déchiffrer/signer (d, p, q, etc.).

Le rôle du modulo (n) dans RSA est central : il définit le domaine mathématique des opérations (arithmétique modulaire) et apparaît dans la clé publique et la clé privée.

On utilise souvent RSA pour chiffrer une clé AES plutôt qu’un document entier parce que c’est plus performant et plus adapté : AES est très rapide et fait du chiffrement de masse, RSA sert à protéger une petite clé de session.

D) Signature numérique (tip : dgst)

Création, empreinte et signature
J’ai créé un fichier contrat.txt, puis généré une empreinte (hash). Pour une signature, OpenSSL peut directement calculer le hash et signer en une commande. La doc openssl dgst indique qu’on peut signer un fichier avec -sign et vérifier avec -verify, et que le résultat de vérification est “Verified OK” ou “Verification Failure”.

Signature (SHA-256) :

openssl dgst -sha256 -sign rsa_private.pem -out contrat.sig contrat.txt

Vérification et test de modification
Vérification avec la clé publique :

openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt

Constat : la commande doit afficher “Verified OK” si le fichier n’a pas été modifié.

Ensuite j’ai modifié légèrement contrat.txt (un caractère). J’ai relancé la vérification.

Constat : la vérification échoue (“Verification Failure”), car la signature correspond au hash de l’ancienne version du fichier ; dès que le contenu change, le hash change, donc la signature ne correspond plus.

Réponses aux questions
Après modification du fichier, la vérification échoue. Cela arrive parce que la signature numérique dépend du hash du document : un changement, même minime, modifie l’empreinte et invalide la signature.

Le rôle du hachage dans la signature est de transformer un document de taille quelconque en empreinte de taille fixe, qui est ensuite signée avec la clé privée. La doc rappelle que openssl dgst “output the message digest” et “generates and verifies digital signatures using message digests”.

La différence entre signature numérique et chiffrement : la signature sert à prouver l’intégrité et l’authenticité (et potentiellement la non-répudiation), alors que le chiffrement sert à garantir la confidentialité.