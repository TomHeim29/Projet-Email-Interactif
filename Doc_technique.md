# DOC TECHNIQUE PROJET #

Vous devez avoir Ubuntu 18.04 fraîchement installé !

Génèrez votre clé SSH : 
`ssh-keygen -t rsa -b 4096`

On achète un VPS chez OVH, on installe notre environnement Linux "Ubuntu 18.04" fraîchement installé OBLIGATOIREMENT,
on ajoute notre clé SSH dans la partie "Gestion des services" depuis OVH 

On se connecte en SSH via notre clé privée

ssh ubuntu@<NOTRE_IP>


#    CONFIGURATION ZONE DNS OVH        #

Aller dans "Tableau de bord", cliquez sur le nom de votre domaine, activer le DNSSEC, puis zone DNS, créez les entrées DNS de votre domaine :
- www IN A <IP>
- IN A <IP>
- IN TXT 1|www.<VOTRE_DOMAINE>

créer un sous-domaine appelé "box" qui sera redirigé vers l'ip de votre VPS, avec comme Type "A", "MX" et "TXT", respectivement :
- box IN A <IP>
- IN MX 10 box.<VOTRE_DOMAINE>.
- box IN TXT "v=spf1 a mx ip4:<IP> ~all"


création d'un sous-domaine "portainer"
- portainer IN A <IP>


création des headers SPF et TXT
- IN TXT "v=spf1 a mx ip4:<IP> ~all" sur votre domaine
- IN TXT "v=spf1 a mx ip4:<IP> ~all" sur votre sous-domaine "box"
- _dmarc IN TXT "v=DMARC1; p=quarantine"
- _dmarc.autoconfig IN TXT "v=DMARC1; p=reject"
- _dmarc.autodiscover IN TXT "v=DMARC1; p=reject"
- _dmarc.mta-sts IN TXT "v=DMARC1; p=reject"
- _dmarc.www IN TXT "v=DMARC1; p=reject"
  
Rajoutez dans la zone DNS les sous-domaines "mta", pour ajouter les "id", RDV sur ce site : https://aykevl.nl/apps/mta-sts/
- _mta-sts IN TXT "v=STSv1; id=<ID>"
- _smtp._tls IN TXT "v=TLSRPTv1; rua=mailto:mta-sts@<VOTRE_DOMAINE>"
- _smtp._tls.box IN TXT "v=TLSRPTv1; rua=mailto:mta-sts@<VOTRE_DOMAINE>"

Etendez votre domaine aux sous-domaines "autoconfig" et "autodiscover", respectivement :
- autoconfig IN CNAME <VOTRE_DOMAINE>.
- autodiscover IN CNAME <VOTRE_DOMAINE>.

Allez dans la section IP dans le panneau latéral de gauche, modifiez le reverse DNS, au niveau l'ipv4, remplacez vps-xxxx par votre nom de domaine.


#    INSTALLATION MAIL-IN-A-BOX        #

# on install Mail-in-a-Box (voir comment l'installer sous docker)
`curl -s https://mailinabox.email/setup.sh | sudo bash`


## Partie sécurité
on commente la bannière
`sed -i "10 s/smtpd_banner/#smtpd_banner/g" /etc/postfix/main.cf`
`systemctl restart postfix`


Renseignez l'email du nom de domaine, le nom de domaine "box" et les fuseaux horaires afin que l'infra de MiaB s'installe correctement.
Ensuite, allez sur votre nom de domaine pour commencer à configurer les certificats TLS de chaque domaine et qu'il soit sûr.

Rendez-vous sur box.<VOTRE_DOMAINE>/admin, et connectez-vous avez l'email et mot de passe que vous avez renseigné pendant l'installation de MiaB.


Esuite vous allez dans l'onglet System > Status check, pour vérifier que vous avez à peu près tout en vert ou jaune.
Cliquez à droite sur "Enable New-Version Check" pour mettre à jour MiaB, rebooter le système.

Vous devez récupérer la clé public de l'entête DKIM de MiaB :
- `less /home/user-data/mail/dkim/mail.txt`

Ensuite revenez dans la Zone DNS de OVH :
- `mail._domainkey IN TXT "<v=;h=;k=;s=>;p=<concat_2_clés>"`

RDV sur le dashboard du sous-domaine "box", onglet "system > TLS Certificates pour mettre à jour les certificats TLS de tous les sous-domaines :
Cliquez sur "Provision", MiaB va installer les certificats à l'aide de letsencryt, tous les certificats peuvent ne pas être installés, clqiuez à nouveau dessus pour que tous les sous-domaines passent au vert (voir screen).

Si vous revenez dans longlet System > Status Checks tout est passé au vert au niveau des erreurs TLS.


#    CREATION REDIRECTON NGINX        #

on fait une sauvegarde du fichier de conf de NGINX
`cp /etc/nginx/conf.d/local.conf /etc/nginx/conf.d/local.conf.old`


Dans le fichier "box.<VOTRE_DOMAINE>", dans la partie HTTP :
- ligne 23, commentez la ligne "root"
- ligne 33, remplacez "$request_uri" par "/admin"
- ligne 61, rajoutez le bloc suivant (ATTENTION AUX INDENTATIONS) :
  
```location / {
                proxy_set_header X-Forwarded-For $remote_addr;
                return 301 https://box.<VOTRE_DOMAINE>/admin;
        }
```

Dans le bloc " www.<VOTRE_DOMAINE>" :
- ligne 890, commentez la ligne
- ligne 900, supprimez "www" devant le nom de domaine et remplacez "$request_uri" par "/mail"
- ligne 929, remplacez la ligne entière par "rewrite ^/$ https://<VOTRE_DOMAINE>/mail permanent;"

- Ensuite ajoutez le bloc suivant (entre la ligne 929 et 930) :
        location / {
                # Redirect using the 'return' directive and the built-in
                # variable '$request_uri' to avoid any capturing, matching
                # or evaluation of regular expressions.
                proxy_set_header X-Forwarded-For $remote_addr;
                return 301 https://<VOTRE_DOMAINE>/mail;
        }
  
créer un nouveau fichier appelé "<IP_SERVEUR>.conf"
  
## <IP_SERVEUR>
  
```server {
        listen 80;
        listen [::]:80;
        server_name <IP_SERVEUR>;

        location / {
                return 301 https://<VOTRE_DOMAINE>/mail;
        }
}

server {
        listen 443 http2 ssl;
        listen [::]:443 http2 ssl;
        server_name <IP_SERVEUR>;

        ssl_certificate /home/user-data/ssl/ssl_certificate.pem;
        ssl_certificate_key /home/user-data/ssl/ssl_private_key.pem;

        rewrite ^/$  https://<VOTRE_DOMAINE>/mail permanent;


        location / {
                proxy_set_header X-Forwarded-For $remote_addr;
                return 301 https://<VOTRE_DOMAINE>/mail;
        }
}
```

Ensuite on sauvegarde ce nouveau fichier "local.conf" avec ses redirections :
`sudo cp /etc/nginx/conf.d/local.conf ../local.txt`
  
Dans le fichier /etc/nginx/conf.d/local.conf, 
il est conseillé de séparer le fichier en plusieurs blocs, dont un bloc est un sous domaine.
on découpe le fichier "local.conf" par bloc, chaque bloc étant un sous-domaine :

`touch <IP_SERVEUR>.conf`
  
`touch www.<VOTRE_DOMAINE>.conf`
  
`touch box.<VOTRE_DOMAINE>.conf`

`touch autoconfig.<VOTRE_DOMAINE>.conf`
  
`touch autodiscover.<VOTRE_DOMAINE>.conf`
  
`touch mta-sts.<VOTRE_DOMAINE>.conf`
  
`touch mta-sts.box.<VOTRE_DOMAINE>.conf`

`sudo chmod 664 /etc/nginx/conf.d/*`
  
`sudo systemctl restart nginx.service`


#    IMPORTATION CERTIFICAT TLS         #

on l'importe dans le magasin des certificats du serveur pour que le cadenas devienne vert même en auto-signé
  
`cp ../user-data/ssl/box.projetmailamp.site-selfsigned-20220314.pem /usr/local/share/ca-certificates/`
  
`cd /usr/local/share/ca-certificates/`
  
`mv box.projetmailamp.site-selfsigned-20220314.pem box.projetmailamp.site-selfsigned-20220314.crt`

`cp /etc/letsencrypt/live/box.projetmailamp.site/* /usr/local/share/ca-certificates/`
  
`mv portainer-CA.crt box.projetmailamp.site.crt`
  
`cd /usr/local/share/ca-certificates/`
  
`chmod 600 *`
  
`update-ca-certificates`
  

#    REMPLACER CRON MIAB                #

Description de la commande qui suit :
- De base MiaB, update le local.conf tous les jours à 3h49 et les redirections sont HS, donc :
    - Tous les 3 mois à 3h30 - lorsque les certificats doivent être renouvelés
    - sinon ça modifie le fichier /etc/nginx/conf.d/local.conf - et toutes les redirections sont HS
    - cp permet d'écraser la nouvelle conf et de revenir à notre config, le local.conf est prioritaire sur les autres 
    - dans le "local.txt" il y a toute la conf avec les redirections
    - puis on met à jour le système et ses paquets

Dans le cron de MIAB "/etc/cron.d/mailinabox-nightly", remplacez la dernière ligne par :
30 03 01 */3 *  root    (cd /root/mailinabox && management/daily_tasks.sh) && cp /etc/nginx/local.txt /etc/nginx/conf.d/local.conf && apt update -y && apt upgrade -y

permet d'afficher le DKIM généré
`cat /home/user-data/mail/dkim/mail.txt`
`dns_update --force`
  

#    SECURISATION SSH    #

on backup le fichier de conf sshd_config
`cp /etc/ssh/sshd_config /etc/ssh/sshd_config.old`


on ajoute le protocole SSHv2
`echo "Protocol 2" >> /etc/ssh/sshd_config`

on modifie le port SSH 22 en 2222
`sed -i "13 s/#Port 22/Port 2222/g" /etc/ssh/sshd_config`


on désactive l’utilisateur root en SSH
`sed -i "32 s/#PermitRootLogin prohibit-password/PermitRootLogin no/g" /etc/ssh/sshd_config`

on désactive l'authentification par password
`sed -i "56 s/PasswordAuthentication yes/PasswordAuthentication no/g" /etc/ssh/sshd_config`


on met décommente la ligne 33 "StrictModes yes"
`sed -i "33 s/#StrictModes yes//g" /etc/ssh/sshd_config`
`systemctl restart sshd.service`


on active le port 2222 sur le firewall
`ufw allow 2222`


# ENVOIE D'EMAIL AVEC SWAKS #
`swaks --auth-user "contact@<VOTRE_DOMAINE>" --auth-password "password_email" --server "box.projetmailamp.site:587" -f contact@<VOTRE_DOMAINE> --add-header 'Content-Type: multipart/alternative; boundary="----=_Part_80_1558614261.1649788279865"' --add-header 'List-Unsubscribe: <mailto:contact@<VOTRE_DOMAINE>>' --body survey.html --tls --h-Subject "Email AMP" --to <EMAIL_DESTINATAIRE>`
