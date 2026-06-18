# chap01 - Récapitulatif étape par étape : bases de MLflow avec un premier exemple

> [!TIP]
> **Objectif du chap01 — Faire tourner MLflow pour la toute première fois.**
>
> Tu vas :
>
> 1. Construire et démarrer un serveur MLflow **100 % dans Docker** sans installer Python sur la machine hôte.
> 2. Ouvrir l’interface MLflow à l’adresse `http://localhost:5000`.
> 3. Créer **un seul run minimal** avec `mlflow.start_run` et `mlflow.log_metric("foo", 1)` directement depuis le conteneur grâce à `docker compose exec`.
> 4. Voir ce run apparaître dans l’interface MLflow et comprendre où sont stockés la **base SQLite** (`./database/mlflow.db`) et les **artefacts** (`./mlruns/`) grâce aux bind mounts.
>
> À la fin, tu sauras démarrer proprement MLflow, l’arrêter avec `docker compose down`, et reconnaître la pile `MLflow + bind mounts + SQLite`, qui servira de base à tous les chapitres suivants.

## Exécution du projet : 100 % Docker, aucun Python sur la machine hôte

```bash
# Avant de commencer, assure-toi d’avoir lu les fichiers Dockerfile
# et docker-compose.yml afin de comprendre comment le serveur MLflow
# est configuré.
#
# Le serveur MLflow est configuré pour utiliser :
# - un dossier local pour stocker les artefacts ;
# - une base SQLite pour stocker les métadonnées de suivi.

# 1. Créer les dossiers database/ et mlruns/ dans le répertoire courant.
#    Ces dossiers serviront à stocker les données de suivi MLflow
#    et les artefacts.
#
#    --> CETTE ÉTAPE EST OBLIGATOIRE.
#        Si les dossiers n’existent pas sur la machine hôte,
#        les bind mounts définis dans docker-compose.yml peuvent échouer
#        ou être créés avec le propriétaire root.
#        Dans ce cas, le fichier SQLite pourrait ne pas être accessible
#        en écriture.

cd chap01-mlflow-step-by-step-recap-hello-mlflow-basics
mkdir database mlruns

# 2. Exécuter ensuite les commandes suivantes pour :
#    - démarrer le serveur MLflow ;
#    - lancer le script hello_mlflow.py ;
#    - arrêter le serveur MLflow à la fin.

docker compose up -d --build

# Cette commande démarre le serveur MLflow en mode détaché.
# Cela permet de laisser MLflow tourner en arrière-plan pendant
# que le script hello_mlflow.py est exécuté.
#
# Vérifie ensuite cette adresse :
# http://localhost:5000
#
# Au début, l’interface MLflow est vide.
# Tu devrais seulement voir l’expérience "Default".

# python hello_mlflow.py
#
# Cette commande ne fonctionnera PAS correctement dans ce contexte.
# Pourquoi ?
#
# Parce que le serveur MLflow tourne à l’intérieur d’un conteneur Docker.
# Le script hello_mlflow.py essaie de se connecter au serveur avec l’adresse
# localhost.
#
# Depuis ta machine hôte, localhost ne pointe pas toujours vers le conteneur.
# De plus, MLflow n’est peut-être même pas installé sur ta machine hôte.
#
# La bonne solution est donc d’exécuter hello_mlflow.py À L’INTÉRIEUR
# du conteneur MLflow.

docker compose exec -d mlflow python hello_mlflow.py

# Cette commande exécute hello_mlflow.py dans le conteneur mlflow.
# Ainsi, le script peut se connecter au serveur MLflow avec :
# http://localhost:5000
#
# Cette fois, localhost désigne bien le serveur MLflow lui-même,
# car le script tourne dans le même conteneur que le serveur.
#
# Les données de l’expérience sont donc correctement enregistrées.

# Vérifie à nouveau l’interface :
# http://localhost:5000
#
# Tu devrais maintenant voir les données de l’expérience dans l’interface MLflow.
# Clique sur le nom de l’expérience "hello_mlflow", puis sur le run
# "my_first_run" pour voir les paramètres et les métriques enregistrés.

# 3. Arrêter le serveur MLflow lorsque tu as terminé.

docker compose down
```

## Ce qui est créé sur ta machine hôte

Après l’exécution du script, les bind mounts remplissent deux vrais dossiers à côté du fichier `docker-compose.yml` :

```text
chap01-mlflow-step-by-step-recap-hello-mlflow-basics/
├── database/
│   └── mlflow.db                    <- métadonnées SQLite
└── mlruns/
    └── 0/
        └── <run_id>/                <- artefacts
```

Tu peux ouvrir ces dossiers avec l’explorateur de fichiers.

## Pourquoi `set_tracking_uri("http://localhost:5000")` fonctionne-t-il ?

Parce que le script est exécuté **à l’intérieur** du conteneur `mlflow`.

Dans ce contexte, `localhost:5000` désigne le serveur MLflow lui-même.

Autrement dit :

```text
script Python dans le conteneur
        |
        | se connecte à
        v
serveur MLflow dans le même conteneur
```

Donc l’adresse `http://localhost:5000` fonctionne correctement.

## Arrêter et nettoyer le projet

```bash
docker compose down

# Optionnel : supprimer les dossiers locaux pour tout réinitialiser.
# Attention : cela supprime la base SQLite et les artefacts MLflow.

rm -rf database mlruns
```

## Récapitulatif

```bash
cd chap01-mlflow-step-by-step-recap-hello-mlflow-basics
mkdir database mlruns
docker compose up -d --build
docker compose exec -d mlflow python hello_mlflow.py
docker compose down
```

## Récapitulatif complémentaire

```bash
cd chap01-mlflow-step-by-step-recap-hello-mlflow-basics
docker compose up -d
docker compose exec -d mlflow python hello_mlflow.py

# Pour entrer dans un conteneur Docker :
# docker exec -it <id> bash
#
# Pour lister les fichiers à l’intérieur du conteneur :
# ls
#
# Pour récupérer l’identifiant <id> du conteneur :
# docker ps
#
# Pour sortir du conteneur :
# exit

docker compose down
```

> Équivalent de `docker exec`

```bash
docker compose exec mlflow bash
```

Cette commande permet d’ouvrir un terminal directement dans le service `mlflow` défini dans `docker-compose.yml`.

---

# Dépannage — tuer les processus zombies

<details>
   <summary>Dépannage</summary>

Sur Windows, pour arrêter le processus qui utilise le port **5000**, commence par exécuter :

```bat
netstat -ano | findstr :5000
```

Tu verras un résultat ressemblant à ceci :

```bat
TCP    127.0.0.1:5000    0.0.0.0:0    LISTENING    12345
```

Le dernier nombre correspond au **PID**, c’est-à-dire l’identifiant du processus.

Dans cet exemple, le PID est :

```text
12345
```

Tu peux ensuite arrêter ce processus avec :

```bat
taskkill /PID 12345 /F
```

Séquence complète :

```bat
netstat -ano | findstr :5000
taskkill /PID 12345 /F
```

Pour vérifier quelle application utilise ce PID avant de l’arrêter :

```bat
tasklist | findstr 12345
```

Version PowerShell :

```powershell
Get-NetTCPConnection -LocalPort 5000
Stop-Process -Id 12345 -Force
```

Le port **5000** peut être vu comme une porte utilisée par une application.

Si Flask, MLflow, FastAPI ou un autre serveur utilise déjà cette porte, une nouvelle application ne peut pas démarrer sur le même port.

La commande `netstat` permet de trouver qui utilise cette porte.

La commande `taskkill` permet ensuite d’arrêter ce processus.

</details>
