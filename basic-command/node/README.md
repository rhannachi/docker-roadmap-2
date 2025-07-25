## Sommaire
* [Résumé des uses cases de l’application](uses_cases.md)

--- 

## Dockerfile
### `build` - Build image Docker
Construit une image Docker à partir du Dockerfile du dossier courant et lui donne le nom (tag) node-img.
```
$ docker build -t node-img .
```

---

### `image ls` - Afficher la liste des images Docker
```
$ docker image ls
```

---

### `run` - Lancer un conteneur à partir d'une image
- **`-d`** : Lance le conteneur en **arrière-plan** (mode détaché)
- **`-p 4000:80`** : **Redirige** le port 4000 de votre machine vers le port 80 du conteneur
- **`--name  node-app`** : **Nomme** le conteneur ` node-app`
- **`node-img:latest`** : Utilise l'**image** `node-img` version `latest`

```
$ docker run -d -p 4000:80 --name  node-app node-img:latest
```

L’option `--rm` dans la commande docker run sert à supprimer automatiquement le conteneur dès qu’il s’arrête ou termine son exécution.
```
$ docker run --rm -d -p 4000:80 --name node-app node-img:latest

$ docker stop node-app

$ docker ps -a
# => il n'y a aucun conteneur "node-app" arrêté ou existant, car il a été supprimé automatiquement grâce à l'option --rm.
```

---

### `exec` - Executer un container en Mode Interactif
Pour entrer dans votre conteneur en mode interactif (s’il tourne en arrière-plan) :
```
$ docker exec -it node-app sh
```

Pour entrer dans votre conteneur en mode interactif au moment de son lancement:
```
$ docker run -it -p 4000:80 --name node-app node-img:latest sh
```

---

### `ps` - Pour afficher la liste des conteneurs Docker en cours d’exécution:
```
$ doker ps

$ docker container ls
```

---

### `stop` `kill` - Arrêter un conteneur:
La méthode recommandée pour arrêter un conteneur en douceur:
```
$ docker stop <ID-CONTAINER>
```

Une solution de dernier recours pour forcer l’arrêt immédiat:
```
$ docker kill <ID-CONTAINER>
```

Pour forcer l’arrêt de tous les conteneurs en cours, on peut combiner docker kill avec la liste des conteneurs actifs :
```
$ docker kill $(docker ps -q)
```

---

### `rmi` - Supprimer une image par son ID
```
$ docker rmi <IMAGE_ID or IMAGE_NAME>
```

---

### `rm` - Supprimer un conteneur existant
Lister tous les conteneurs (y compris les arrêtés)
```
$ docker ps -a
```
```
$ docker rm <CONTAINER_ID or CONTAINER_NAME>
```

---

### `start` - Démarrer un conteneur

`docker run` permet de créer un nouveau conteneur à partir d’une image.
Pour démarrer un ou plusieurs conteneurs Docker qui sont actuellement arrêtés, sans en créer de nouveaux, il faut utiliser la commande `docker start`.

- Utilisez `docker start` pour relancer un conteneur stoppé (STATUS => Exited).
- Utilisez `docker exec` pour interagir ou diagnostiquer un conteneur déjà en fonctionnement (STATUS => Up).

```
$ docker build -t node-img .

$ docker run -d -p 3000:80 --name node-app node-img:latest

$ docker stop node-app
```
``` 
$ docker start node-app
```

---

### `attach` -  Se connecter à la sortie standard (STDOUT)
La commande `docker attach` permet de connecter ton terminal aux flux d’entrée (STDIN), de sortie (STDOUT) et d’erreur (STDERR) d’un conteneur Docker déjà en cours d’exécution.
```
$ docker build -t node-img .

# attach mode
$ docker run -p 3000:80 --name node-app node-img:latest
# => voir ce qui se passe à l’intérieur, comme si tu étais devant son terminal. 
# (c’est-à-dire la sortie standard – STDOUT – du processus principal défini par la commande CMD ou ENTRYPOINT du Dockerfile).

```
``` 
$ docker build -t node-img .

$ docker run -d -p 3000:80 --name node-app node-img:latest

# attach mode
$ docker attach node-app
# => la sortie standard – STDOUT – du processus principal défini par la commande CMD ou ENTRYPOINT du Dockerfile
```

Tu peux également suivre les logs d’un conteneur en temps réel avec la commande suivante :
```
$ docker logs -f node-app
```

Pour tout conteneur à l’arrêt, tu peux le redémarrer de cette façon tout en affichant sa sortie standard (stdout) :
```
# attach mode
$ docker start -a node-app
```

### Variable d'environnement dans Docker

Pour lancer un conteneur Docker avec des variables d’environnement, on utilise l’option `-e` (ou `--env`) dans la commande `docker run`. Cela permet de définir des paires clé-valeur accessibles à l’intérieur du conteneur.

#### Exemple 1 : Lancer un conteneur avec la variable `PORT` par défaut à 80

```
$ docker run -p 3000:80 --name node-app node-img
```

Ici, le port 80 du conteneur est exposé et mappé sur le port 3000 de l’hôte. La variable `PORT` est définie dans l’image Docker (via `ENV PORT=80` dans le Dockerfile).

#### Exemple 2 : Modifier la variable d’environnement `PORT` à 88 et ajouter une autre variable

```
$ docker run -p 3000:88 -e PORT=88 -e VAR_1=var_1 --name node-app node-img
```

- `-e PORT=88` : remplace la valeur par défaut de `PORT` par 88 dans le conteneur.
- `-e VAR_1=var_1` : ajoute une variable d’environnement supplémentaire.


#### Exemple 3 : Utiliser un fichier d’environnement (`.env`)

Fichier `file.env` :

```
PORT=88
VAR_1=var_1
```

Commande pour lancer le conteneur avec ce fichier :

```
$ docker run -p 3000:88 --env-file ./file.env --name node-app node-img
```

### `build-arg` – Utilisation des arguments lors du build

```dockerfile
FROM node:current-alpine3.22

ARG DEFAULT_PORT=80

WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
ENV PORT=$DEFAULT_PORT
EXPOSE $PORT

CMD ["node", "server.js"]
```

#### Exemple 1 : Construire et lancer un conteneur sans modifier `DEFAULT_PORT`

```
$ docker build -t node-img .
$ docker run --rm -p 3000:80 --name node-app node-img:latest
```
- Ici, le port par défaut est 80 (défini par `ARG DEFAULT_PORT=80`), donc le conteneur écoutera sur le port 80.

#### Exemple 2 : Construire l’image en modifiant `DEFAULT_PORT`

```
$ docker build --build-arg DEFAULT_PORT=88 -t node-img .
$ docker run -p 3000:88 --name node-app node-img:latest
```

- Ici, le port d’écoute du conteneur sera 88, car la valeur de l’argument de build a été surchargée lors du build.


### Explications importantes

- **`ARG`** permet de définir des variables **uniquement disponibles pendant la construction de l’image** (build-time).
- Tu peux fournir une nouvelle valeur à un argument avec l’option `--build-arg` lors du `docker build`.

**Résumé** :

- Utilise `ARG` pour personnaliser la construction de l’image sans modifier le Dockerfile.
- Utilise `--build-arg` pour passer des valeurs au moment du build.
- Utilise `ENV` pour que les variables soient disponibles dans le conteneur à l’exécution.
- Les arguments de build sont très utiles pour rendre tes images Docker plus flexibles et réutilisables, tout en évitant de "figer" des valeurs dans le Dockerfile.
