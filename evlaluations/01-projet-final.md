# Projet de session — MLOps, expérimentations et déploiement léger d’un modèle ML

## 1. Contexte du projet

Dans ce projet de session, vous devez réaliser un mini-projet complet de Machine Learning en appliquant une démarche MLOps simple.

L’objectif n’est pas seulement d’entraîner un modèle, mais de montrer une démarche professionnelle :

1. Choisir un problème de Machine Learning.
2. Préparer les données.
3. Entraîner plusieurs modèles ou plusieurs variantes d’un modèle.
4. Lancer plusieurs expérimentations.
5. Suivre les paramètres, les métriques et les résultats avec MLflow.
6. Comparer les runs.
7. Sélectionner le meilleur modèle.
8. Exposer le modèle avec une API FastAPI.
9. Créer une petite interface Streamlit pour tester le modèle.

Le choix du sujet est libre, mais il doit correspondre à un problème réel ou réaliste.

---

# 2. Choix du problème

Chaque équipe doit choisir un problème parmi les catégories suivantes :

## Option 1 — Régression

Exemples :

* Prédire le prix d’une maison.
* Prédire la consommation d’énergie.
* Prédire la qualité d’un vin.
* Prédire le temps de livraison.
* Prédire une note ou un score numérique.

Dans ce cas, la variable cible est numérique.

Exemples de métriques possibles :

* RMSE
* MAE
* R²

---

## Option 2 — Classification binaire

Exemples :

* Prédire si un client va quitter une entreprise ou non.
* Prédire si un courriel est spam ou non.
* Prédire si une transaction est frauduleuse ou non.
* Prédire si un patient présente un risque ou non.
* Prédire si un étudiant réussit ou échoue.

Dans ce cas, il y a deux classes possibles.

Exemples de métriques possibles :

* Accuracy
* Precision
* Recall
* F1-score
* Matrice de confusion
* ROC-AUC si applicable

---

## Option 3 — Classification multiclasse

Exemples :

* Classer une fleur selon son espèce.
* Classer un type de client.
* Classer un produit dans une catégorie.
* Classer un niveau de risque : faible, moyen, élevé.
* Classer un type de panne ou d’erreur.

Dans ce cas, il y a plus de deux classes possibles.

Exemples de métriques possibles :

* Accuracy
* Precision macro
* Recall macro
* F1-score macro
* Matrice de confusion

---

# 3. Données utilisées

Vous pouvez utiliser :

1. Un jeu de données public.
2. Un jeu de données fourni par Scikit-Learn.
3. Un jeu de données Kaggle.
4. Un fichier CSV trouvé sur une source fiable.
5. Un jeu de données simulé, seulement si vous expliquez clairement comment il a été généré.

Le jeu de données doit contenir suffisamment de données pour permettre un entraînement et une évaluation raisonnables.

Vous devez indiquer dans votre rapport :

* le nom du dataset ;
* la source ;
* le nombre de lignes ;
* le nombre de colonnes ;
* la variable cible ;
* le type de problème ;
* les principales variables explicatives ;
* les transformations appliquées.

---

# 4. Travail demandé

## Partie 1 — Préparation du projet

Vous devez créer un projet Python structuré proprement.

Structure minimale attendue :

```text
projet-mlops/
│
├── data/
│   └── dataset.csv
│
├── notebooks/
│   └── exploration.ipynb
│
├── src/
│   ├── train.py
│   ├── predict.py
│   └── preprocessing.py
│
├── api/
│   └── main.py
│
├── app/
│   └── streamlit_app.py
│
├── models/
│
├── requirements.txt
├── README.md
└── rapport.pdf
```

La structure peut être adaptée, mais elle doit rester claire et professionnelle.

---

# 5. Expérimentations obligatoires avec MLflow

Vous devez utiliser MLflow pour suivre vos expérimentations.

Chaque expérimentation doit enregistrer :

1. Le nom de l’expérience.
2. Le nom du run.
3. Les paramètres du modèle.
4. Les métriques obtenues.
5. Le modèle entraîné.
6. Les informations importantes du projet.

Exemple de paramètres à suivre :

```text
model_name
alpha
max_depth
n_estimators
learning_rate
test_size
random_state
```

Exemple de métriques à suivre :

```text
accuracy
precision
recall
f1_score
rmse
mae
r2
```

Les métriques dépendent du type de problème choisi.

---

# 6. Nombre minimal de runs

Vous devez produire au minimum :

## Minimum obligatoire

* 1 expérience MLflow ;
* 5 runs différents ;
* au moins 2 modèles différents ou 5 configurations différentes du même modèle.

Exemples acceptés :

### Exemple pour un problème de régression

Vous pouvez comparer :

1. Linear Regression
2. Ridge
3. Lasso
4. ElasticNet avec alpha = 0.1
5. ElasticNet avec alpha = 0.5

### Exemple pour une classification binaire

Vous pouvez comparer :

1. Logistic Regression
2. Decision Tree
3. Random Forest
4. KNN
5. SVM

### Exemple pour une classification multiclasse

Vous pouvez comparer :

1. Logistic Regression
2. Random Forest
3. Decision Tree
4. Gradient Boosting
5. KNN

Chaque run doit avoir un nom clair.

Exemples :

```python
with mlflow.start_run(run_name="random_forest_depth_5"):
    ...
```

```python
with mlflow.start_run(run_name="elasticnet_alpha_0_1_l1_0_5"):
    ...
```

Le nom du run doit permettre de comprendre rapidement ce qui a été testé.

---

# 7. Comparaison des résultats

Vous devez comparer vos runs dans un tableau.

Exemple :

| Run    | Modèle              | Paramètres principaux | Métrique 1 | Métrique 2 | Résultat |
| ------ | ------------------- | --------------------- | ---------: | ---------: | -------- |
| run_01 | Logistic Regression | C=1.0                 |       0.84 |       0.81 | Correct  |
| run_02 | Decision Tree       | max_depth=4           |       0.79 |       0.76 | Moyen    |
| run_03 | Random Forest       | n_estimators=100      |       0.88 |       0.86 | Meilleur |

Vous devez expliquer :

* quel modèle donne le meilleur résultat ;
* pourquoi vous avez choisi ce modèle ;
* quelles sont les limites de votre modèle ;
* quelles améliorations seraient possibles.

---

# 8. API FastAPI

Vous devez créer une API avec FastAPI pour permettre d’utiliser le modèle sélectionné.

L’API doit contenir au minimum :

## Route obligatoire

```text
POST /predict
```

Cette route doit recevoir des données en entrée et retourner une prédiction.

Exemple de réponse attendue :

```json
{
  "prediction": "classe_1",
  "model_name": "RandomForestClassifier"
}
```

ou pour une régression :

```json
{
  "prediction": 7.82,
  "model_name": "ElasticNet"
}
```

L’API doit charger le meilleur modèle sauvegardé et utiliser ce modèle pour prédire une nouvelle donnée.

---

# 9. Interface Streamlit

Vous devez créer une interface Streamlit simple permettant de tester votre modèle.

L’interface doit permettre :

1. D’entrer les valeurs nécessaires.
2. D’envoyer les données à l’API FastAPI.
3. D’afficher la prédiction retournée par l’API.
4. D’afficher quelques informations simples sur le modèle utilisé.

L’interface n’a pas besoin d’être complexe.

L’objectif est de montrer que le modèle peut être utilisé dans une petite application.

---

# 10. Communication entre Streamlit et FastAPI

Attention : Streamlit ne doit pas refaire directement la prédiction dans son propre code.

Streamlit doit communiquer avec FastAPI.

Architecture attendue :

```text
Utilisateur
    ↓
Interface Streamlit
    ↓
Requête HTTP
    ↓
API FastAPI
    ↓
Modèle ML
    ↓
Prédiction
    ↓
Retour vers Streamlit
```

Exemple de logique attendue :

```python
response = requests.post(
    "http://localhost:8000/predict",
    json=input_data
)
```

---

# 11. Livrables attendus

Chaque équipe doit remettre :

## 1. Le code source complet

Le projet doit contenir :

* le code d’entraînement ;
* le code de prédiction ;
* le code FastAPI ;
* le code Streamlit ;
* le fichier `requirements.txt` ;
* le fichier `README.md`.

---

## 2. Le suivi MLflow

Vous devez fournir des preuves de vos expérimentations MLflow :

* captures d’écran de l’interface MLflow ;
* liste des runs ;
* comparaison des métriques ;
* meilleur run identifié.

---

## 3. Le rapport final

Le rapport doit contenir :

1. Présentation du problème choisi.
2. Description du dataset.
3. Type de problème : régression, classification binaire ou classification multiclasse.
4. Préparation des données.
5. Modèles testés.
6. Description des expérimentations MLflow.
7. Tableau comparatif des runs.
8. Choix du meilleur modèle.
9. Description de l’API FastAPI.
10. Description de l’interface Streamlit.
11. Limites du projet.
12. Améliorations possibles.
13. Conclusion.

---

## 4. Démonstration

Vous devez être capables de démontrer :

1. Le lancement de MLflow.
2. L’affichage des runs dans MLflow.
3. Le lancement de FastAPI.
4. Le test de la route `/predict`.
5. Le lancement de Streamlit.
6. Une prédiction faite depuis l’interface Streamlit.

---

# 12. Commandes minimales attendues

Le projet doit pouvoir être lancé avec des commandes claires dans le README.

Exemple :

```bash
pip install -r requirements.txt
```

```bash
mlflow ui
```

```bash
python src/train.py
```

```bash
uvicorn api.main:app --reload
```

```bash
streamlit run app/streamlit_app.py
```

---

# 13. Barème proposé

| Partie évaluée                              | Description                                                                   | Points |
| ------------------------------------------- | ----------------------------------------------------------------------------- | -----: |
| Choix du problème et compréhension du sujet | Problème clair, type ML bien identifié, objectif compréhensible               |   10 % |
| Dataset et préparation des données          | Source, nettoyage, séparation train/test, transformations pertinentes         |   15 % |
| Expérimentations MLflow                     | Plusieurs runs, paramètres suivis, métriques suivies, noms de runs clairs     |   20 % |
| Modèles et comparaison des résultats        | Au moins 5 runs, comparaison sérieuse, meilleur modèle justifié               |   15 % |
| Qualité du code et structure du projet      | Projet organisé, code lisible, fichiers bien séparés, README clair            |   10 % |
| Rapport final                               | Explications claires, tableaux, captures, analyse des résultats               |   15 % |
| FastAPI + Streamlit                         | API fonctionnelle, interface simple, communication entre Streamlit et FastAPI |   15 % |
| Total                                       |                                                                               |  100 % |

---

# 14. Détail de la partie FastAPI + Streamlit — 15 %

| Élément                                                | Points |
| ------------------------------------------------------ | -----: |
| API FastAPI fonctionnelle avec route `/predict`        |    5 % |
| Chargement correct du meilleur modèle                  |    3 % |
| Interface Streamlit simple et utilisable               |    3 % |
| Communication Streamlit vers FastAPI avec requête HTTP |    3 % |
| Affichage clair de la prédiction                       |    1 % |
| Total                                                  |   15 % |

---

# 15. Contraintes importantes

Vous devez respecter les contraintes suivantes :

1. Le projet doit utiliser Python.
2. Le projet doit utiliser MLflow.
3. Le projet doit contenir plusieurs runs.
4. Le projet doit comparer les résultats.
5. Le projet doit sauvegarder ou enregistrer le meilleur modèle.
6. Le projet doit contenir une API FastAPI.
7. Le projet doit contenir une interface Streamlit.
8. Streamlit doit communiquer avec FastAPI.
9. Le projet doit être reproductible avec un fichier `requirements.txt`.
10. Le README doit expliquer clairement comment exécuter le projet.

---

# 16. Critères de réussite

Un bon projet doit montrer que vous êtes capables de :

* comprendre un problème de Machine Learning ;
* choisir des métriques adaptées ;
* entraîner plusieurs modèles ;
* organiser plusieurs expérimentations ;
* utiliser MLflow pour suivre les runs ;
* comparer les résultats ;
* sélectionner un modèle final ;
* exposer un modèle avec FastAPI ;
* créer une interface simple avec Streamlit ;
* expliquer clairement votre démarche.

---

# 17. Exemple de sujet accepté

## Sujet : Prédiction de la qualité du vin

Type de problème :

```text
Régression
```

Objectif :

```text
Prédire la qualité d’un vin à partir de ses caractéristiques chimiques.
```

Exemples de modèles :

```text
Linear Regression
Ridge
Lasso
ElasticNet
Random Forest Regressor
```

Métriques :

```text
RMSE
MAE
R²
```

Runs MLflow possibles :

```text
linear_regression_baseline
ridge_alpha_0_1
ridge_alpha_1_0
elasticnet_alpha_0_1_l1_0_5
random_forest_100_trees
```

---

# 18. Exemple de sujet accepté

## Sujet : Prédiction du départ d’un client

Type de problème :

```text
Classification binaire
```

Objectif :

```text
Prédire si un client risque de quitter une entreprise.
```

Exemples de modèles :

```text
Logistic Regression
Decision Tree
Random Forest
KNN
Gradient Boosting
```

Métriques :

```text
Accuracy
Precision
Recall
F1-score
Matrice de confusion
```

Runs MLflow possibles :

```text
logistic_regression_baseline
decision_tree_depth_3
decision_tree_depth_5
random_forest_100_trees
gradient_boosting_baseline
```

---

# 19. Exemple de sujet accepté

## Sujet : Classification d’espèces de fleurs

Type de problème :

```text
Classification multiclasse
```

Objectif :

```text
Prédire l’espèce d’une fleur à partir de ses mesures.
```

Exemples de modèles :

```text
Logistic Regression
Decision Tree
Random Forest
KNN
SVM
```

Métriques :

```text
Accuracy
Precision macro
Recall macro
F1-score macro
Matrice de confusion
```

Runs MLflow possibles :

```text
logistic_regression_baseline
knn_k_3
knn_k_5
random_forest_50_trees
random_forest_100_trees
```

---

# 20. Remarque 

Le but du projet n’est pas d’obtenir le modèle parfait.

Le but est de démontrer une démarche complète :

```text
données → entraînement → expérimentations → suivi MLflow → comparaison → sélection → API → interface
```

Vous devez montrer que vous savez expérimenter, analyser, comparer et présenter un modèle de Machine Learning de façon professionnelle.


# Exemples: 

- https://towardsdatascience.com/how-to-dockerize-machine-learning-applications-built-with-h2o-mlflow-fastapi-and-streamlit-a56221035eb5/

