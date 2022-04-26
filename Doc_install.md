# on met à jour le dépôt et les paquets
apt update -y && apt upgrade -y

# on install docker et docker-compose
apt install -y docker.io 
apt install -y docker-compose
apt-get install -y software-properties-common && add-apt-repository ppa:certbot/certbot && apt-get update -y && sudo apt-get install -y certbot
apt install -y python3-cerbot-nginx python3-certbot-apache 




# on install notre outil de monitoring / supervision "Portainer"
docker volume create portainer_data
#docker run -d -p 9443:9443 --name portainer--restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.11.1
ufw allow 9443
sudo docker run -d -p 127.0.0.1:9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.11.1
# --ssl --sslcert /certs/portainer.crt --sslkey /certs/portainer.key

# on créer notre certificat SSL auto-signé
#openssl req -newkey rsa:2048 -nodes -keyout portainer-key.pem -x509 -days 365 -out portainer-certificate.pem
certbot --nginx --agree-tos --redirect --hsts --staple-ocsp --email contact@projetmailamp.site -d portainer.projetmailamp.site





# on l'importe dans le magasin des certificats du serveur pour que le cadenas devienne vert tout en étant auto-signé
#cp ../user-data/ssl/box.projetmailamp.site-selfsigned-20220314.pem /usr/local/share/ca-certificates/
#cd /usr/local/share/ca-certificates/
#mv box.projetmailamp.site-selfsigned-20220314.pem box.projetmailamp.site-selfsigned-20220314.crt

cp /etc/letsencrypt/live/portainer.projetmailamp.site/* /usr/local/share/ca-certificates/
mv portainer-CA.crt portainer.projetmailamp.site.crt
cd /usr/local/share/ca-certificates/
chmod 600 *
update-ca-certificates





# on install Mail-in-a-Box 
curl -s https://mailinabox.email/setup.sh | sudo bash



# on configure le reverse proxy "Nginx"
cp /etc/nginx/conf.d/local.conf /etc/nginx/conf.d/local.conf.old
