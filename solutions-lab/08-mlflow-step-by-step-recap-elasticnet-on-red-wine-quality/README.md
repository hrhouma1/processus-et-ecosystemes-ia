# chap03 - Récapitulatif étape par étape : premier pipeline ElasticNet sur red-wine-quality



> [!TIP]
> **Objectif du chap03 — Premier vrai pipeline ML tracé dans MLflow.**
>
> Tu vas :
>
> 1. Reprendre la même pile à un seul service `mlflow` que dans les chap01 et chap02, avec les bind mounts `./database`, `./mlruns` et `./data`.
> 2. Entraîner un modèle **ElasticNet** sur le dataset `red-wine-quality.csv` en passant `--alpha` et `--l1_ratio` en ligne de commande.
> 3. Logger dans MLflow les **paramètres**, les **métriques** (`rmse`, `mae`, `r2`) et le **modèle** avec `mlflow.sklearn.log_model`.
> 4. Lancer **trois runs successifs** avec des hyperparamètres différents (`0.1/0.1`, `0.5/0.5`, `0.9/0.1`) et les comparer dans l’interface MLflow (`http://localhost:5000`).
>
> À la fin, tu sauras lancer le *même* script avec des arguments différents, comparer les runs côte à côte dans MLflow, et identifier le meilleur jeu d’hyperparamètres. C’est le **squelette de code** que tous les chapitres suivants vont enrichir : multi-services, `log_artifacts`, signatures, registry, etc.

Ce laboratoire montre comment exécuter plusieurs expériences MLflow avec différents hyperparamètres.

Tu vas entraîner le même modèle ElasticNet trois fois, mais chaque run utilisera des valeurs différentes de :

```text
alpha
l1_ratio
```

L’objectif est de comparer les résultats dans l’interface MLflow.

---

## Ce qui est nouveau par rapport au chap02 — ElasticNet sur Red Wine Quality avec Docker

* Un vrai pipeline de Machine Learning : `pd.read_csv` → `train_test_split` → `ElasticNet.fit` → calcul de `rmse / mae / r2`.
* `mlflow.sklearn.log_model(lr, "mymodel")` pour sauvegarder le modèle entraîné dans MLflow.



# Arrêter d’abord les conteneurs déjà en cours d’exécution

Avant de commencer ce laboratoire, assure-toi qu’aucun autre conteneur MLflow n’est déjà en cours d’exécution sur le port `5000`.

## Méthode 1 — Arrêter les conteneurs d’un autre projet

Va dans le dossier de l’autre projet :

```bash
cd other-project
docker compose down
```

Cette commande arrête et supprime les conteneurs créés par ce projet.

---

## Méthode 2 — Utiliser Docker Desktop

Tu peux aussi ouvrir **Docker Desktop** et faire manuellement les étapes suivantes :

```text
1. Aller dans Containers
2. Trouver le conteneur en cours d’exécution
3. L’arrêter
4. Le supprimer si nécessaire
```

Cette méthode est utile si tu ne te souviens plus quel dossier a démarré le conteneur.

---

# 3. Démarrer le projet #3


Crée les dossiers nécessaires :

```bash
mkdir -p database mlruns
```

Ou sur Windows PowerShell :

```powershell
New-Item -ItemType Directory -Force database, mlruns
```

Tu peux aussi créer les deux dossiers manuellement : `database` et `mlruns`.

Ensuite, démarre le serveur MLflow :

```bash
docker compose up -d --build
```

Ouvre l’interface MLflow :

```text
http://localhost:5000
```

À ce stade, l’interface peut être vide ou afficher uniquement l’expérience par défaut. C’est normal.

> **Attention importante :** crée les dossiers `database/` et `mlruns/` avant de démarrer Docker.

Si tu les oublies, arrête Docker, crée les dossiers, puis redémarre avec `--build` :

```bash
docker compose down
mkdir -p database mlruns
docker compose up -d --build
```

L’option `--build` force Docker à reconstruire l’image au lieu d’utiliser le cache.

---

# 4. Lancer l’expérience 1

Lance le modèle avec :

```text
alpha = 0.1
l1_ratio = 0.1
```

Commande :

```bash
docker compose exec mlflow python train_with_mlflow.py --alpha 0.1 --l1_ratio 0.1
```

Ensuite, vérifie :

```text
http://localhost:5000
```

Tu devrais voir un nouveau run MLflow.

---

# 5. Lancer l’expérience 2

Lance le modèle avec :

```text
alpha = 0.5
l1_ratio = 0.5
```

Commande :

```bash
docker compose exec mlflow python train.py --alpha 0.5 --l1_ratio 0.5
```

Ensuite, vérifie à nouveau :

```text
http://localhost:5000
```

Tu devrais maintenant voir un autre run.

---

# 6. Lancer l’expérience 3

Lance le modèle avec :

```text
alpha = 0.9
l1_ratio = 0.9
```

Commande :

```bash
docker compose exec mlflow python train.py --alpha 0.9 --l1_ratio 0.9
```

Ensuite, vérifie à nouveau :

```text
http://localhost:5000
```

Tu devrais maintenant voir trois runs différents.

---

# 7. Comparer les runs dans MLflow

Dans l’interface MLflow, compare les runs à l’aide de :

```text
Parameters
Metrics
Artifacts
Model output
```

L’idée importante est la suivante :

```text
Chaque run utilise le même script d’entraînement, mais avec des hyperparamètres différents.
MLflow enregistre chaque run séparément.
Cela permet de comparer quelle configuration donne les meilleurs résultats.
```

Par exemple :

```text
Run 1 : alpha = 0.1, l1_ratio = 0.1
Run 2 : alpha = 0.5, l1_ratio = 0.5
Run 3 : alpha = 0.9, l1_ratio = 0.9
```

---

# 8. Arrêter les conteneurs

Lorsque tu as terminé :

```bash
docker compose down       # conserve tous les runs
docker compose down -v    # supprime tout
```

---

## Attention importante — créer d’abord les dossiers

Sois vigilant : si les deux dossiers `database/` et `mlruns/` ne sont pas créés avant le démarrage de Docker, MLflow peut ne pas sauvegarder correctement les données d’expérience.

Tu peux ouvrir l’interface MLflow à l’adresse :

```text
http://localhost:5000
```

mais tu risques de ne pas voir tes runs, tes métriques, tes paramètres ou tes artefacts.

Avant d’exécuter Docker, crée les deux dossiers manuellement :

```bash
mkdir -p database mlruns
```

Sur Windows PowerShell :

```powershell
New-Item -ItemType Directory -Force database, mlruns
```

Si tu as déjà démarré Docker sans créer ces dossiers, fais ceci :

```bash
docker compose down
mkdir -p database mlruns
docker compose up -d --build
```

Sur Windows PowerShell :

```powershell
docker compose down
New-Item -ItemType Directory -Force database, mlruns
docker compose up -d --build
```

L’option `--build` est importante ici.

Elle force Docker à reconstruire l’image au lieu de réutiliser l’ancienne version en cache.

Sans `--build`, Docker peut réutiliser l’ancienne configuration en cache, et ta correction peut ne pas être appliquée correctement.

---

# Récapitulatif final des commandes

```bash
git clone https://github.com/inskillflow/mlops-beginner-level-01-en.git

cd mlops-beginner-level-01-en/chap01-mlflow-step-by-step-recap-hello-mlflow-basics
# terminé

cd ../chap02-mlflow-step-by-step-recap-printing-the-tracking-uri
# terminé

cd ../chap03-mlflow-step-by-step-recap-elasticnet-on-red-wine-quality
# début du projet #3

mkdir -p database mlruns

docker compose up -d --build

docker compose exec mlflow python train.py --alpha 0.1 --l1_ratio 0.1
docker compose exec mlflow python train.py --alpha 0.5 --l1_ratio 0.5
docker compose exec mlflow python train.py --alpha 0.9 --l1_ratio 0.9

docker compose down
```

Version PowerShell :

```powershell
git clone https://github.com/inskillflow/mlops-beginner-level-01-en.git

cd mlops-beginner-level-01-en/chap01-mlflow-step-by-step-recap-hello-mlflow-basics
# terminé

cd ../chap02-mlflow-step-by-step-recap-printing-the-tracking-uri
# terminé

cd ../chap03-mlflow-step-by-step-recap-elasticnet-on-red-wine-quality
# début du projet #3

New-Item -ItemType Directory -Force database, mlruns

docker compose up -d --build

docker compose exec mlflow python train.py --alpha 0.1 --l1_ratio 0.1
docker compose exec mlflow python train.py --alpha 0.5 --l1_ratio 0.5
docker compose exec mlflow python train.py --alpha 0.9 --l1_ratio 0.9

docker compose down
```

---

# Entrer manuellement dans le conteneur

Liste d’abord les conteneurs en cours d’exécution :

```bash
docker ps
```

Ensuite, entre dans le conteneur MLflow :

```bash
docker exec -it <container_id> bash
```

À l’intérieur du conteneur :

```bash
ls
python train_with_mlflow.py --alpha 0.1 --l1_ratio 0.1
exit
```

L’équivalent avec Docker Compose est plus simple :

```bash
docker compose exec mlflow bash
```

---

# Pourquoi ne pas utiliser `-d` avec `docker compose exec` ?

Tu peux parfois voir cette commande :

```bash
docker compose exec -d mlflow python train.py --alpha 0.1 --l1_ratio 0.1
```

Elle fonctionne, mais elle lance le script en mode détaché.

## Version recommandée :

```bash
docker compose exec mlflow python train.py --alpha 0.1 --l1_ratio 0.1
```

De cette manière, si une erreur se produit, elle apparaît immédiatement dans la console.

---

# Que se passe-t-il si j’ai oublié de créer `mlruns` et `database` ?

Si tu as oublié de créer ces dossiers avant de démarrer Docker, tu peux rencontrer des problèmes avec :

```text
les permissions de la base SQLite
le stockage des métadonnées MLflow
le stockage des artefacts
des dossiers appartenant à root
des erreurs de bind mount
```

En termes simples :

```text
MLflow a besoin d’un endroit pour stocker les informations d’expérience et les artefacts.
Le dossier database/ stocke les métadonnées SQLite.
Le dossier mlruns/ stocke les artefacts des runs.
Si ces dossiers sont absents ou créés incorrectement, MLflow peut ne pas réussir à écrire les données correctement.
```

---

# Comment corriger le problème

## Étape 1 — Arrêter les conteneurs

```bash
docker compose down
```

## Étape 2 — Créer les dossiers nécessaires

Linux, macOS, Git Bash :

```bash
mkdir -p database mlruns
```

Windows PowerShell :

```powershell
New-Item -ItemType Directory -Force database, mlruns
```

## Étape 3 — Reconstruire et redémarrer les conteneurs

```bash
docker compose up -d --build
```

L’option `--build` force Docker à reconstruire l’image.

C’est utile lorsque :

```text
le Dockerfile a changé
les dépendances ont changé
l’environnement doit être rafraîchi
le conteneur précédent a été créé incorrectement
```

## Étape 4 — Relancer le script d’entraînement

```bash
docker compose exec mlflow python train.py --alpha 0.1 --l1_ratio 0.1
```

Ensuite, ouvre :

```text
http://localhost:5000
```

---

# Dépannage 1 : le port 5000 est déjà utilisé

Sur Windows CMD :

```bat
netstat -ano | findstr :5000
tasklist | findstr 12345
taskkill /PID 12345 /F
```

Sur PowerShell :

```powershell
Get-NetTCPConnection -LocalPort 5000
Stop-Process -Id 12345 -Force
```

Remplace `12345` par le PID affiché par la commande.

Explication simple :

Le port `5000` est comme une porte. Si une autre application utilise déjà cette porte, MLflow ne peut pas démarrer sur le même port. Tu dois donc soit arrêter l’autre application, soit changer le port utilisé par MLflow.

---

# Dépannage 2 : Docker Desktop ne démarre pas

Ouvre **PowerShell en tant qu’administrateur**, puis exécute ceci :

```powershell
# 1. Arrêter les processus Docker Desktop
Get-Process *docker* -ErrorAction SilentlyContinue | Stop-Process -Force

# 2. Arrêter le service Docker Desktop
Stop-Service com.docker.service -Force -ErrorAction SilentlyContinue

# 3. Forcer l’arrêt du backend WSL utilisé par Docker
wsl --shutdown
```

Attends ensuite **10 à 15 secondes**.

Pour redémarrer Docker Desktop :

```powershell
Start-Service com.docker.service
Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"
```

Si Docker est encore bloqué, utilise la version plus forte :

```powershell
taskkill /F /IM "Docker Desktop.exe"
taskkill /F /IM "com.docker.backend.exe"
taskkill /F /IM "com.docker.service.exe"
taskkill /F /IM "dockerd.exe"
wsl --shutdown
```

Ensuite, redémarre Docker Desktop manuellement depuis le menu Démarrer.

Ne supprime pas encore les dossiers Docker. Essaie d’abord l’arrêt forcé avec `wsl --shutdown`.
