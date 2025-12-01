# Guide de déploiement avec Docker - Exercice Pratique

Ce guide vous accompagne pour déployer une application web complète avec Docker.

## 📋 Prérequis

- Docker et Docker Compose installés sur votre machine
- Un éditeur de texte (VS Code, Sublime Text, etc.)
- Les fichiers du projet déjà présents (code backend et frontend)

##  Architecture du projet

Le projet est composé de 3 services :
- **Backend** : API Python (Flask/FastAPI)
- **Base de données** : PostgreSQL
- **Frontend + Nginx** : Application React servie par Nginx


##  Exercice 1 : Compléter le Backend Dockerfile

**Fichier à modifier : `backend/Dockerfile`**

### Objectif
Créer une image Docker qui exécute votre application Python.

### 📚 Instructions Docker disponibles
- `FROM` : définir l'image de base
- `WORKDIR` : définir le répertoire de travail dans le conteneur
- `COPY` : copier des fichiers de votre machine vers le conteneur
- `RUN` : exécuter une commande pendant la construction de l'image
- `EXPOSE` : documenter le port utilisé par l'application
- `CMD` : définir la commande à exécuter au démarrage du conteneur

### 💡 Étapes à suivre

**1. Choisir l'image de base**
- Quelle image Python utiliser ? (version 3.11, variante slim pour réduire la taille)
- Syntaxe : `FROM image:tag`

**2. Définir le répertoire de travail**
- Où placer votre application dans le conteneur ? (convention : `/app`)
- Syntaxe : `WORKDIR /chemin`

**3. Installer les dépendances Python**
- Quel fichier contient vos dépendances ? (`requirements.txt`)
- Comment optimiser le cache Docker ? (copier d'abord requirements.txt seul, puis installer, puis copier le reste)
- Quelle commande pip utiliser ? (pensez à `--no-cache-dir` pour réduire la taille)
- Syntaxe : 
  - `COPY fichier_source destination`
  - `RUN commande`

**4. Copier le code de l'application**
- Comment copier tout le contenu du dossier backend ? (`.` représente tout)
- Syntaxe : `COPY source destination`

**5. Exposer le port**
- Sur quel port votre application Flask/FastAPI écoute-t-elle ? (généralement 5000)
- Syntaxe : `EXPOSE numéro_port`

**6. Définir la commande de démarrage**
- Comment lancer votre application Python ? (`python app.py`)
- Format recommandé : array JSON → `["commande", "arg1", "arg2"]`
- Syntaxe : `CMD ["executable", "param1", "param2"]`

### Vérification
Votre Dockerfile devrait contenir environ 6-7 lignes d'instructions.

---

##  Exercice 2 : Compléter le Frontend Dockerfile

**Fichier à modifier : `frontend/Dockerfile`**

### Objectif
Créer une image optimisée avec un build multi-stage : construire l'application React, puis la servir avec Nginx.

### 📚 Instructions Docker supplémentaires
- `FROM image AS nom` : créer une étape nommée (multi-stage)
- `COPY --from=nom` : copier des fichiers depuis une étape précédente

### 💡 Étapes à suivre

**ÉTAPE 1 : Builder (Construction de l'application React)**

**1. Créer l'étape de build**
- Quelle image Node.js utiliser ? (version 18, variante alpine pour la légèreté)
- Comment nommer cette étape ? (ex: `builder`)
- Syntaxe : `FROM image:tag AS nom_etape`

**2. Définir le répertoire de travail**
- Convention : `/app`

**3. Installer les dépendances npm**
- Quels fichiers copier en premier ? (`package.json` et `package-lock.json`)
- Quelle commande npm utiliser pour une installation propre ? (`npm ci`)
- Option pour exclure les dépendances de dev ? (`--omit=dev`)
- Syntaxe pour copier plusieurs fichiers : `COPY package*.json ./`

**4. Copier le code source et builder**
- Copier tout le code source
- Quelle commande npm lance le build ? (`npm run build`)
- Où se trouve le résultat du build ? (généralement dans `/app/build`)

**ÉTAPE 2 : Production (Servir avec Nginx)**

**5. Créer l'étape production**
- Quelle image utiliser pour servir des fichiers statiques ? (`nginx:alpine`)
- Pas besoin de nommer cette étape (c'est la dernière)

**6. Copier le build depuis l'étape précédente**
- D'où copier ? (depuis l'étape `builder`, dossier `/app/build`)
- Où copier ? (dans le dossier par défaut de Nginx : `/usr/share/nginx/html`)
- Syntaxe : `COPY --from=nom_etape source destination`

### Vérification
Votre Dockerfile devrait contenir environ 8-9 lignes, réparties en 2 étapes distinctes (2 instructions `FROM`).

---

## Exercice 3 : Compléter le docker-compose.yml

**Fichier à modifier : `docker-compose.yml`**

### Objectif
Orchestrer les 3 services (backend, db, nginx) et définir leurs interactions.

### Propriétés Docker Compose importantes

**Pour tous les services :**
- `build` : instructions pour construire une image
  - `context` : chemin vers le dossier contenant le Dockerfile
  - `dockerfile` : nom du Dockerfile (optionnel si nommé "Dockerfile")
- `image` : utiliser une image existante du Docker Hub
- `container_name` : nom du conteneur
- `ports` : mapper les ports `"port_hote:port_conteneur"`
- `volumes` : monter des volumes ou des fichiers
- `env_file` : charger des variables d'environnement depuis un fichier
- `depends_on` : définir les dépendances entre services
  - `condition` : attendre une condition (ex: `service_healthy`)
- `restart` : politique de redémarrage (ex: `unless-stopped`)
- `mem_limit` : limite de mémoire (ex: `512m`, `1g`)
- `cpus` : limite de CPU (ex: `0.5`, `1.0`)

**Spécifique à la base de données :**
- `healthcheck` : vérifier que le service est prêt
  - `test` : commande à exécuter (format: `["CMD-SHELL", "commande"]`)
  - `interval` : fréquence des vérifications (ex: `15s`)
  - `timeout` : délai maximum d'attente (ex: `10s`)
  - `retries` : nombre de tentatives (ex: `5`)

### Structure à compléter

```yaml
services:
  # Service 1 : backend
  
  # Service 2 : db
  
  # Service 3 : nginx

volumes:
  # Volumes nommés
```

### Service 1 : Backend

**Questions à vous poser :**
1. Comment construire l'image ?
   - Quel est le contexte (dossier) ? → `./backend`
   - Quel fichier Dockerfile utiliser ?

2. Comment nommer le conteneur ? → `backend`

3. Quelles variables d'environnement charger ?
   - Fichier : `.env.local.backend`

4. De quel service dépend le backend ?
   - Service : `db`
   - Condition : attendre que `db` soit en bonne santé (`service_healthy`)

5. Quelles limites de ressources appliquer ?
   - Mémoire : 512 Mo
   - CPU : 0.5

6. Quelle politique de redémarrage ? → `unless-stopped`

**Structure suggérée :**
```yaml
backend:
  build:
    context: ?
    dockerfile: ?
  container_name: ?
  env_file:
    - ?
  depends_on:
    ?:
      condition: ?
  mem_limit: ?
  cpus: ?
  restart: ?
```

### Service 2 : Database (PostgreSQL)

**Questions à vous poser :**
1. Quelle image utiliser ?
   - Image officielle : `postgres`
   - Version : `13`

2. Comment nommer le conteneur ? → `db`

3. Quelles variables d'environnement charger ?
   - Fichier : `.env.local.db`

4. Comment vérifier que PostgreSQL est prêt ? (healthcheck)
   - Commande test : `pg_isready -U postgres` (format CMD-SHELL)
   - Vérifier toutes les 15 secondes (`interval`)
   - Timeout de 10 secondes
   - 5 tentatives maximum (`retries`)

5. Où stocker les données de manière persistante ?
   - Volume nommé : `postgres_data`
   - Destination dans le conteneur : `/var/lib/postgresql/data`

6. Quelles limites de ressources ?
   - Mémoire : 1 Go
   - CPU : 1.0

**Structure suggérée :**
```yaml
db:
  image: ?
  container_name: ?
  env_file:
    - ?
  healthcheck:
    test: ["CMD-SHELL", "?"]
    interval: ?
    timeout: ?
    retries: ?
  volumes:
    - ?:/var/lib/postgresql/data
  mem_limit: ?
  cpus: ?
  restart: ?
```

### Service 3 : Nginx (Frontend)

**Questions à vous poser :**
1. Comment construire l'image ?
   - Contexte : `./frontend`

2. Comment nommer le conteneur ? → `nginx`

3. Quels ports exposer ?
   - Port de votre machine : `8088`
   - Port du conteneur : `80`
   - Format : `"hote:conteneur"`

4. Quel fichier de configuration monter ?
   - Source : `./nginx/default.conf`
   - Destination : `/etc/nginx/conf.d/default.conf`

5. De quel service dépend nginx ?
   - Service : `backend` (pas besoin de condition)

6. Quelles limites de ressources ?
   - Mémoire : 256 Mo
   - CPU : 0.25

**Structure suggérée :**
```yaml
nginx:
  build:
    context: ?
  container_name: ?
  ports:
    - "?:?"
  volumes:
    - ?:?
  depends_on:
    - ?
  mem_limit: ?
  cpus: ?
  restart: ?
```

### Volumes

**Question finale :**
- Quel volume nommé déclarer pour PostgreSQL ?
  - Nom : `postgres_data`

**Structure suggérée :**
```yaml
volumes:
  ?:
```

### Vérification

Votre fichier docker-compose.yml devrait :
- Définir 3 services