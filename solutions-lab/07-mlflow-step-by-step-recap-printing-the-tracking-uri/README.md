# chap02 - Récapitulatif étape par étape : afficher le Tracking URI


> [!TIP]
> **Objectif du chap02 — Savoir *où* MLflow écrit, avant même de logger.**
>
> Tu vas :
>
> 1. Reprendre exactement la pile du chap01 : un seul conteneur `mlflow`, avec les bind mounts `./database` et `./mlruns`.
> 2. Ajouter **une seule ligne** dans le script : `print("Tracking URI:", mlflow.get_tracking_uri())`.
> 3. Lancer `docker compose exec mlflow python train.py` et **lire l’URI affichée** dans la console.
> 4. Comparer cette URI avec ce que tu vois sur le disque (`./database/mlflow.db`, `./mlruns/0/...`) pour confirmer où les runs sont enregistrés.
>
> À la fin, tu comprends que **MLflow choisit son backend en fonction de l’URI courante** : `file:///...` par défaut, puis `http://...` plus tard.
> C’est la base de toute la suite, notamment à partir du chap04, où cette URI sera passée avec la variable `MLFLOW_TRACKING_URI`.

## Ce qui est nouveau par rapport au chap01

Une seule ligne est ajoutée :

```python
print("Tracking URI:", mlflow.get_tracking_uri())
```

Cette ligne permet d’afficher l’adresse utilisée par MLflow pour savoir où enregistrer les expériences, les runs, les paramètres, les métriques et les artefacts.

## Exécution du projet : 100 % Docker, aucun Python sur la machine hôte

```bash
# Avant de commencer, assure-toi d’avoir lu les fichiers Dockerfile
# et docker-compose.yml afin de comprendre comment le serveur MLflow
# est configuré.
#
# Le serveur MLflow persiste :
# - les métadonnées dans une base SQLite ;
# - les artefacts dans des dossiers locaux montés avec des bind mounts.

# 1. Créer les dossiers database/ et mlruns/ dans le répertoire courant.
#
#    CETTE ÉTAPE EST OBLIGATOIRE.
#    Les bind mounts définis dans docker-compose.yml s’attendent à trouver
#    ces dossiers sur la machine hôte.

cd chap02-mlflow-step-by-step-recap-printing-the-tracking-uri
mkdir database mlruns

# 2. Démarrer le serveur MLflow en mode détaché.

docker compose up -d --build

# Vérifie ensuite cette adresse :
# http://localhost:5000
#
# Au début, l’interface MLflow est vide.
# Tu devrais seulement voir l’expérience "Default".

# python hello_mlflow.py
#
# Cette commande ne fonctionnera PAS correctement dans ce contexte.
#
# Raisons possibles :
# - MLflow n’est peut-être pas installé sur la machine hôte ;
# - le dossier local `mlflow/` peut masquer le package Python `mlflow`
#   si tu exécutes le script directement depuis l’hôte ;
# - le serveur MLflow tourne dans un conteneur Docker.
#
# La bonne solution est donc d’exécuter le script À L’INTÉRIEUR
# du conteneur mlflow.

docker compose exec -d mlflow python hello_mlflow.py

# Sortie attendue, visible avec :
# docker compose logs mlflow
#
# Exemple :
#   Tracking URI: http://localhost:5000
#   Done. Open http://localhost:5000 to see your run.

# Actualise ensuite l’interface :
# http://localhost:5000
#
# L’expérience "hello_mlflow" devrait apparaître avec un nouveau run
# nommé "my_first_run".

# 3. Arrêter le serveur MLflow.

docker compose down
```

## Ce qui est créé sur ta machine hôte

Après l’exécution, tu obtiens une structure similaire à celle-ci :

```text
chap02-mlflow-step-by-step-recap-printing-the-tracking-uri/
├── database/
│   └── mlflow.db
└── mlruns/
    └── 0/
        └── <run_id>/
            └── ...
```

Le dossier `database/` contient la base SQLite utilisée par MLflow pour enregistrer les métadonnées.

Le dossier `mlruns/` contient les artefacts liés aux runs.

## Nettoyage complet du projet

```bash
docker compose down

# Supprimer complètement la base SQLite et les artefacts persistés.
# Attention : cette commande efface les données MLflow locales.

rm -rf database mlruns
```

## Récapitulatif

> Supposons que tu es encore dans le premier projet.
>
> Avant de passer au deuxième projet, tu dois d’abord exécuter `docker compose down` pour arrêter le serveur MLflow du chap01.
>
> Ensuite, tu peux te déplacer dans le dossier du chap02 et répéter les mêmes étapes :
>
> * créer les dossiers nécessaires ;
> * démarrer le serveur MLflow ;
> * exécuter le script `hello_mlflow.py` ;
> * vérifier l’interface MLflow ;
> * arrêter le serveur.

```bash
docker compose down

cd ../chap02-mlflow-step-by-step-recap-printing-the-tracking-uri

mkdir database mlruns

docker compose up -d --build

docker compose exec -d mlflow python hello_mlflow.py

# Attention :
# Cette commande exécute le script en arrière-plan, en mode détaché.
# Elle ne montre donc pas directement les messages print dans le terminal.

docker compose exec mlflow python hello_mlflow.py

# Cette commande exécute le script au premier plan.
# Elle affiche donc les messages print au début et à la fin de l’exécution
# du fichier Python.

# Vérifie les logs dans ton terminal.
# Vérifie aussi l’interface MLflow :
# http://localhost:5000

docker compose down
```

## Pourquoi la deuxième commande affiche les print ?

Cette commande :

```bash
docker compose exec -d mlflow python hello_mlflow.py
```

lance le script en mode détaché grâce à l’option `-d`.

Cela veut dire que Docker démarre le script en arrière-plan et te rend immédiatement la main dans le terminal.

Donc, les messages `print()` ne sont pas affichés directement dans ta console.

Pour voir les messages, tu peux utiliser :

```bash
docker compose logs mlflow
```

Ou bien exécuter le script sans `-d` :

```bash
docker compose exec mlflow python hello_mlflow.py
```

Dans ce cas, le script s’exécute au premier plan, et les messages `print()` apparaissent directement dans le terminal.

## À retenir

Le chap02 ne change presque rien par rapport au chap01.

La grande nouveauté est cette ligne :

```python
print("Tracking URI:", mlflow.get_tracking_uri())
```

Elle permet de voir clairement où MLflow envoie les données de tracking.

Dans ce chapitre, l’URI affichée est généralement :

```text
Tracking URI: http://localhost:5000
```

Cela signifie que le script Python envoie les informations vers le serveur MLflow disponible sur le port `5000`.

Comme le script est exécuté dans le conteneur `mlflow`, l’adresse `localhost:5000` désigne le serveur MLflow à l’intérieur du même conteneur.

C’est une notion très importante pour comprendre la suite du cours.
