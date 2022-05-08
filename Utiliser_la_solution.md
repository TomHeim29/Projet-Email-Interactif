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

***Etape 1.***

En premier lieu, il faut créer un fichier dans lequel on va y mettre notre code AMP/HTML/Plain text créé, pour l'intégrer dans notre mail. Pour ce faire, vous pouvez utiliserle playground de google pour être sur que le code AMP soit correct. Voici la syntaxe d'un mail : 

```html
------=_Part_80_1558614261.1649788279865
Content-Type: text/plain; charset="UTF-8"; format=flowed; delsp=yes

Hello World in plain text !

------=_Part_80_1558614261.1649788279865
Content-Type: text/x-amp-html; charset="UTF-8"

<!doctype html>
<html amp4email data-css-strict>
<head>
  <meta charset="utf-8">
  <script async src="https://cdn.ampproject.org/v0.js"></script>
  <script async custom-element="amp-form" src="https://cdn.ampproject.org/v0/amp-form-0.1.js"></script>
  <style amp4email-boilerplate>body{visibility:hidden}</style>
  <style amp-custom>
    .container {
      max-width: 500px;
      margin: auto;
      font-family: sans-serif;
      padding: 1em;
      text-align: center;
    }

    .block {
    display: block;
      width: 100%;
    }

    .m1 {
      margin: 1em 0;
    }

    label {
      margin-bottom: 0.5em;
    }
  </style>
</head>
  <body>
  <div class="container">
    <div>
      <amp-img class="m1" width="600" height="314" layout="responsive" src="https://amp.dev/static/img/sharing/default-$      <p>Nous espérons que vous avez passé un bon moment !</p>
    </div>

    <hr>

    <form method="post" action-xhr="https://tomheim.fr/amp/index.php">
            <div class="m1">
                <p>Pouvez-vous noter ce cours Ynov ?</p>
                <input type="radio" id="rating1" name="rating" value="Super" on="change:step2.show" required>
                <label for="rating1">Super</label>

                <input type="radio" id="rating2" name="rating" value="Satisfait" on="change:step2.show">
                <label for="rating2">Satisfait</label>

                <input type="radio" id="rating3" name="rating" value="Pas ouf !" on="change:step2.show">
                <label for="rating3">Pas ouf !</label>
        </div>
        <div class="m1" id="step2" hidden>
                <label class="block" for="info">Que voulez-vous dire de plus ?</label>
                <textarea class="block" id="info" name="name" rows="5"></textarea>
        </div>
        <div class="m1" id="step2" hidden>
                <label class="block" for="info">Que voulez-vous dire de plus ?</label>
                <textarea class="block" id="info" name="name" rows="5"></textarea>
        </div>
        <input type="submit" value="Envoyer !">
        <input type="reset" value="Effacer">

              <div class="m1" submit-success>
                Merci pour votre retour.
      </div>
    </form>
  </div>
</body>
</html>
```

*Etape 2.*

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

