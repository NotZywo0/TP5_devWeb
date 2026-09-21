Question 1.1 :

La requête renvoie le code HTTP suivant :
HTTP/1.1 200 OK
Date: Fri, 18 Sep 2026 23:30:29 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Transfer-Encoding: chunked

Le serveur indique donc que la requête a été traitée correctement avec un statut 200 OK.

Question 1.2 :

HTTP/1.1 200 OK
Content-Type: application/json
Date: Fri, 18 Sep 2026 23:32:08 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Content-Length: 20


Question 1.3 :

Parmi les informations envoyées par le navigateur, on retrouve notamment :
sec-ch-ua
"Google Chrome";v="153", "Not_A Brand";v="8", "Chromium";v="153"
sec-ch-ua-mobile
?0
sec-ch-ua-platform
"Windows"
upgrade-insecure-requests
1
user-agent
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/153.0.0.0 Safari/537.36


Question 1.4 :

Au départ, une erreur apparaît dans la console :
Error: ENOENT: no such file or directory, open 'C:\Users\goujo\OneDrive\Bureau\INFO\L2\S4\Dev Web\TP\devweb-tp5\index.html'
    at async open (node:internal/fs/promises:1360:25)
    at async Object.readFile (node:internal/fs/promises:2149:14) {
  errno: -4058,
  code: 'ENOENT',
  syscall: 'open',
  path: 'C:\\Users\\goujo\\OneDrive\\Bureau\\INFO\\L2\\S4\\Dev Web\\TP\\devweb-tp5\\index.html'
    }

ENOENT (No such file or directory): Commonly raised by fs operations to indicate that a component of the specified pathname does not exist. No entity (file or directory) could be found by the given path.

Pour gérer l'erreur côté serveur, le catch() a été modifié afin de retourner une réponse HTTP 500 :
function requestListener(_request, response) {
  fs.readFile("index.html", "utf8")
    .then((contents) => {
      response.setHeader("Content-Type", "text/html");
      response.writeHead(200);
      return response.end(contents);
    })
    .catch((error) => {
      console.error(error);
      response.writeHead(500);
      return response.end("<html><p>500: INTERNAL SERVER ERROR</p></html>");
    });
}

Après avoir renommé __index.html en index.html, le serveur trouve correctement le fichier et retourne une réponse 200 OK avec son contenu.


Question 1.5 :

Le même fonctionnement peut être écrit avec async/await :

async function requestListener(_request, response) {
  try {
    const contents = await fs.readFile("index.html", "utf8");
    response.setHeader("Content-Type", "text/html");
    response.writeHead(200);
    return response.end(contents);
  } catch (error) {
    console.error(error);
    response.writeHead(500);
    return response.end("<html><p>500: INTERNAL SERVER ERROR</p></html>");
  }
}


Question 1.6 :

L'installation de cross-env avec :

npm install cross-env --save

a entraîné l'ajout de cross-env dans les dépendances du projet. Elle a également créé node_modules/ ainsi que package-lock.json.

Pour nodemon, la commande utilisée est :

npm install nodemon --save-dev

Cette fois, nodemon est ajouté dans la section devDependencies, puisqu'il est principalement utilisé pendant le développement.


Question 1.7 :

http-dev:

Variable d'environnement : NODE_ENV=development 
Commande utilisée : nodemon 
Utilisation : développement

nodemon surveille les fichiers et redémarre automatiquement le serveur lorsqu'une modification est détectée

http-prod: 

Variable = NODE_ENV=production
Outil = node ( pas de rechargement automatique de la page web)
Usage = Production web

node lance juste le script une fois --> plus performant en prod


Question 1.8 :

URL : http://localhost:8000/index.html
Code HTTP : 200
URL : http://localhost:8000/random.html
Code HTTP : 200
URL : http://localhost:8000/
Code HTTP : 404
URL : http://localhost:8000/dont-exist
Code HTTP : 404

Pour gérer les différentes routes, le serveur a été modifié afin d'analyser l'URL avec split("/").

async function requestListener(request, response) {
  response.setHeader("Content-Type", "text/html");
  try {
    const contents = await fs.readFile("index.html", "utf8");
    const urlParts = request.url.split("/"); // ["", "random", "5"]

    switch (urlParts[1]) {
      case "":
      case "index.html":
        response.writeHead(200);
        return response.end(contents);

      case "random.html":
        response.writeHead(200);
        return response.end(
          `<html><p>${Math.floor(100 * Math.random())}</p></html>`,
        );

      case "random": {
        const nb = Number.parseInt(urlParts[2], 10);
        if (Number.isNaN(nb)) {
          response.writeHead(400);
          return response.end("<html><p>400: BAD REQUEST</p></html>");
        }
        const numbers = Array.from({ length: nb })
          .map(() => Math.floor(100 * Math.random()))
          .join("</li><li>");
        response.writeHead(200);
        return response.end(`<html><ul><li>${numbers}</li></ul></html>`);
      }

      default:
        response.writeHead(404);
        return response.end("<html><p>404: NOT FOUND</p></html>");
    }
  } catch (error) {
    console.error(error);
    response.writeHead(500);
    return response.end("<html><p>500: INTERNAL SERVER ERROR</p></html>");
  }
}

Grâce à cette modification, les routes / et /index.html utilisent toutes les deux le fichier index.html.

La route /random.html génère un nombre aléatoire compris entre 0 et 99.

Enfin, /random/n permet de générer n nombres aléatoires. Si la valeur fournie n'est pas un nombre entier valide, le serveur renvoie une erreur 400 Bad Request.


Partie 2 – Express
Question 2.1

Documentation utilisée
Express : https://expressjs.com/
http-errors : https://github.com/jshttp/http-errors
loglevel : https://github.com/pimterry/loglevel
morgan : https://github.com/expressjs/morgan

Deux scripts ont ensuite été ajoutés afin de pouvoir lancer le serveur Express selon l'environnement :

"express-dev": "cross-env NODE_ENV=development nodemon server-express.mjs",
"express-prod": "cross-env NODE_ENV=production node server-express.mjs"

Le premier est destiné au développement et utilise nodemon, tandis que le second démarre directement le serveur avec Node.js.


Question 2.2 :

Vérification des 3 routes :
Request URL http://localhost:8000/
Request method
GET
Status code
200 OK
Remote address
[::1]:8000
Referrer policy
no-referrer
HTTP/1.1 200 OK
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 18 Sep 2026 23:53:26 GMT
ETag: W/"3ac-1a0b6f0725b"
Content-Type: text/html; charset=utf-8
Content-Length: 940
Date: Sat, 19 Sep 2026 00:29:35 GMT
Connection: keep-alive
Keep-Alive: timeout=5

Request URL http://localhost:8000/index.html
Request method
GET
Status code
200 OK
Remote address
[::1]:8000
Referrer policy
no-referrer
HTTP/1.1 200 OK
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 18 Sep 2026 23:53:26 GMT
ETag: W/"3ac-1a0b6f0725b"
Content-Type: text/html; charset=utf-8
Content-Length: 940
Date: Sat, 19 Sep 2026 00:30:21 GMT
Connection: keep-alive
Keep-Alive: timeout=5

Request URL http://localhost:8000/random/5
Request method
GET
Status code
200 OK
Remote address
[::1]:8000
Referrer policy
no-referrer
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 81
ETag: W/"51-gKsBcyMZZjJX6YOit+hTEKc7Aaw"
Date: Sat, 19 Sep 2026 00:28:18 GMT
Connection: keep-alive
Keep-Alive: timeout=5


Question 2.3 :

Nouveaux par rapport au serveur HTTP natif :
X-Powered-By: Express (signature du framework)
ETag (pour le cache)
Content-Type avec charset=utf-8 (Express précise l'encodage)


Question 2.4 :

L'événement listening est déclenché quand le serveur a terminé de se lier au port et à l'hôte (après server.listen()), c'est-à-dire au moment où il commence à accepter des connexions. Il est asynchrone : le code situé après server.on("listening", ...) (comme console.info("File ... executed.")) s'exécute avant que le callback ne soit appelé.


Question 2.5 :

L'option index du middleware express.static, activée par défaut avec la valeur "index.html".


Question 2.6 :

Codes HTTP sur style.css
Premier chargement :	200 OK
Ctrl+R :	304 Not Modified
Ctrl+Shift+R :	200 OK

Justification :
- Express ajoute un ETag (empreinte du fichier) et Cache-Control
- Au Ctrl+R, le navigateur envoie If-None-Match: <etag> : si l'ETag est identique, le serveur répond 304 --> pas de corps, juste l'utilisation de la version en cache
- Au Ctrl+Shift+R, le navigateur vide le cache et redemande le fichier entier --> 200


Question 2.7 :

Différence d'affichage dev vs prod
En development (npm run express-dev), la page d'erreur affiche :
- Le code (ex: 404)
- Le message (Not Found)
- La stack trace complète (<pre>... avec les fichiers et lignes)

## En production (npm run express-prod), la stack trace est vide :
- Seuls le code et le message apparaissent
- Pas de détails internes (sécurité : on ne révèle pas la structure du code)

# const stack = app.get("env") === "development" ? error.stack : "";
--> app.get("env") lit NODE_ENV. Le stack n'est inclus qu'en dev.
