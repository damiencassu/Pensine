# Mémo openssl

## Création certificats numériques x509 pour serveur Web (HTTPS/TLS)

1. Génération clef privée

 Pour créer une clef RSA (4096 bits recommandés) :
 
    openssl genrsa [-des3] 4096 > servweb.key
 
 > Remarque : [-des3] : Permet de chiffrer la clef (chiffrement symétrique par bloc 3DES). 
 > Une passphrase pour déchiffrer la clef à chaque utilisation sera demandée.
 
 Protéger la clef (uniquement le droit de lecture pour le propriétaire)
 
    chmod 400 servweb.key

2. Créer une demande de certificat (Certificate Signing Request)

Pour créer la demande de certificat pour la clef privée générée à l'étape 1 :

    openssl req -new -key servweb.key > servweb.csr

Répondre au questionnaire qui permet de renseigner les différents champs du futur certificat (code pays, nom organisation...).

>Remarque : 
> * Le champ COMMON NAME doit contenir le nom de domaine à certifier
> * Le fichier servweb.csr contient également le clef publique à certifier (associée à la clef privée servweb.key)

3. Faire certifier la clef par une autorité de certification reconnue (utilisation en production) ou auto-signer le certificat (en phase de test)

## Création d'une autorité de certification de test

4. Génération de la clef privée de l'autorité

Comme précédemment :

    openssl genrsa [-des3] 4096 > ca.key

5. Création du certificat auto-signé de l'autorité

 Pour créer le certificat au format x509 valable 1 an :

    openssl req -new -x509 -days 365 -key ca.key > ca.crt
 
Répondre au questionaire qui permet de renseigner les différents champs du futur certificat de l'autorité (code pays, nom organisation).

>Remarque : Le champ COMMON NAME doit contenir le nom de l'autorité
 
6. Signature d'un certificat par l'autorité de certification de test

Enfin la signature d'une demande de certificat par l'autorité de test :
 
    openssl x509 -req -in servweb.csr -out servweb.crt -CA ca.crt -CAkey ca.key -CAcreateserial -CAserial ca.srl

>Remarque : L'option CAcreateserial n'est nécessaire que pour la première signature, par la suite seul CAserial est nécessaire.

servweb.crt contient désormais la clef publique certifiée du serveur. Le serveur web doit posséder servweb.crt pour s'identifier auprès des clients **ET** servweb.key pour s'authentifier.
 
## Génération et signature de certificats SAN

7. Création d'un CSR avec des champs SAN

Par défaut, openssl n'intégre pas de champs SAN dans les CSRs. Il faut forcer leur intégration en utilisant un fichier de configuration local comme présenté ci-après. Par exemple pour créer un CSR avec SANs pour un serveur apache en localhost, il faut rajouter une entrée *DNS.1 = localhost* au paragraphe *[alt_names]*. On peut également rajouter une entrée SAN pour l'adresse de loopback avec une entrée *IP.1 = 127.0.0.1*.

Exemple de fichier de configuration san.cf:

       [ req ]
       default_bits       = 2048
       distinguished_name = req_distinguished_name
       req_extensions     = req_ext
       [ req_distinguished_name ]
       countryName                 = Country Name (2 letter code)
       stateOrProvinceName         = State or Province Name (full name)
       localityName               = Locality Name (eg, city)
       organizationName           = Organization Name (eg, company)
       commonName                 = Common Name (e.g. server FQDN or YOUR name)
       [ req_ext ]
       subjectAltName = @alt_names
       [alt_names]
       IP.1   = 127.0.0.1
       DNS.1   = localhost

Plus généralement, toutes les IPs (IP.1, IP.2 ... IP.n) et tous les FQDNs (DNS.1, DNS.2 ... DNS.n) déclarés comme entrée SAN permettent d'obtenir un cadenas vert lorsque l'URL du navigateur correspond à l'un des SAN du certificat (sous réserve que le certificat soit valide).

Pour générer le CSR avec SAN, on utilise la commande 

       openssl req -config san.cf -new -key servweb.key > servweb.csr
       
8. Signature d'un CSR avec conservation des SAN

Par défaut, les certificats signés avec la commande openssl x509 comme présenté plus haut (6) n'ont pas de champs SAN (même si présents dans le CSR). Pour obtenir un certificat avec SANs il faut une nouvelle fois utiliser un fichier de configuration local combiné au module ca.

8.1 Fichier de configuration

Le fichier de configuration suivant permet de conserver les champs SANs lors de la signature d'un CSR (directive *copy_extensions = copyall*). Dans un répertoire *ca* il faut créer un fichier *index.txt* vide, un répertoire *newcerts*, et placer le fichier de série (*ca.srl*), la clef publique et la clef privée de l'autorité (*ca.crt* et *ca.key*).

Exemple de fichier de configuration san_ca.cf:

       [ca]
       default_ca = CA_default

       [CA_default]
       dir = ./ca
       database = $dir/index.txt
       new_certs_dir = $dir/newcerts
       serial = $dir/ca.srl
       private_key = $dir/ca.key
       certificate = $dir/ca.crt
       default_days = 3650
       default_md = sha256
       policy = policy_anything
       copy_extensions = copyall

       [policy_anything]
       countryName = optional
       stateOrProvinceName = optional
       localityName = optional
       organizationName = optional
       organizationalUnitName = optional
       commonName = supplied
       emailAddress = optional

Ensuite il suffit de signer un CSR avec SAN avec la commande suivante

       openssl ca -config san_ca.cf -in servweb.csr -out servweb.crt

On obtient ainsi un certificat *servweb.crt* avec SANs.

## Bonus

### Lire le contenu d'un CSR ou d'un certificat (avec affichage des éventuels SANs)

Pour un CSR :

      openssl req -in servweb.csr -text -noout 
      
Pour un Certificat :

      openssl x509 -in servweb.crt -text -noout

### Vérifier le status de révocation d'un certificat

1. OCSP

Prérequis :
* Certificat de l'autorité émettrice du certificat à vérifier (PEM)
* Certificat à vérifier (PEM)

      openssl ocsp -verify_other <certificat_autorite_emettrice> -issuer <certificat_autorité_emettrice> -cert  <certificat_a_verifier> -text -url <url_repondeur_ocsp_autorite_emettrice> -header "Host"="<FQDN  repondeur_ocsp_autorite_emettrice> 


Exemple d'utilisation :

      openssl ocsp -verify_other DigiCert_SHA2_High_Assurance_Server_CA.pem  -issuer DigiCert_SHA2_High_Assurance_Server_CA.pem -cert github-com.pem -text -url  http://ocsp.digicert.com -header "Host"="ocsp.digicert.com" 



2. CRL

Prérequis : 
* CRL de l'autorité émettrice 
* Numéro de série du certificat à vérifier

      openssl crl -inform DER -text -in <crl_autorite_emettrice> | grep <numero_serie_certificat_a_verifier> 


Exemple utilisation : 

      openssl crl -inform DER -text -in Digicert.crl | grep 0557C80B282683A17B0A114493296B79




