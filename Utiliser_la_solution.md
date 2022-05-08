**WEBMAIL**

l'interface de webmail pour les utilisateurs est accessible depuis [https://projetmailamp.site]

**INTERFACE D'ADMINISTRATION** 

l'interface d'administration de la solution est accessible depuis [https://box.projetmailamp.site]

**MONITORING** 

1. Portainer est accessible via [https://portainer.projetmailamp.site]

2. On peut acceder à MUNIN depuis [https://box.projetmailamp.site/admin/munin/index.html]


**CREER UN UTILISATEUR** 

Pour créer un utilisateur, il faut se rendre sur la page d'administration  [https://box.projetmailamp.site], puis dans la partie "Mails & Users" --> "Users".

Une fois dans cette partie, il suffit de créer un utilisateur avec un mot de passe contenant minimum 8 charactères (chiffres et lettre).

Lorsque l'utilisateur a été créé, il suffit de se rendre sur la page de se connexion [https://projetmailamp.site] pour se connecter avec le nouvel utilisateur précédement créé. 

**ENVOYER UN MAIL AMP** 

Pour envoyer un mail AMP, il faut se connetcer en SSH au serveur en tant qu'administrateur. Ensuite, il faut utiliser swaks en renseignant ces paramètres : 

```
swaks --auth-user "contact@projetmailamp.site" --auth-password "motdepasseduserveur" --server "box.projetmailamp.site:587" 
--to tomheim2901@gmail.com -f contact@projetmailamp.site --add-header 'Content-Type: multipart/alternative; 
boundary="----=_Part_80_1558614261.1649788279865"' --add-header 'List-Unsubscribe: <mailto:contact@projetmailamp.site>'
--body hello_world.html --tls --h-Subject "TEST"
```

`--auth-user` --> le serveur utilisé pour envoyer le mail

`--auth-password` --> le mot de passe du serveur 

`--to` --> le destinataire

`--body` --> le mail avec le code AMP/HTML/Plain text que l'ont veut envoyer

`--h-subject` --> le sujet du mail qui s'affichera à la réception 

