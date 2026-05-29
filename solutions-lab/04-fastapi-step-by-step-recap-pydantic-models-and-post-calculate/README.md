#  Step-by-step recap: Pydantic models + POST /calculate

> [!TIP]
> **Objectif - Passer des paramètres d'URL à un VRAI body JSON validé par Pydantic.**
>
> Tu vas :
> 1. Garder la pile `fastapi + streamlit` du chap28 (réseau `calc-net`, hot reload, `depends_on`).
> 2. Ajouter dans `main.py` deux **Pydantic models** : `CalculationRequest` (entrée) et `CalculationResponse` (sortie), plus une `Operation(Enum)` pour limiter les opérations valides à `add | subtract | multiply | divide`.
> 3. Créer un **endpoint `POST /calculate`** qui reçoit un JSON `{"a": ..., "b": ..., "operation": "..."}` et retourne un JSON typé.
> 4. Garder les anciens `GET /add/{a}/{b}` etc. pour rester compatible avec chap27/28 (ils renvoient maintenant le même `CalculationResponse`).
> 5. Côté Streamlit, basculer sur `requests.post(url, json=payload)` au lieu d'un `GET` avec params dans l'URL — et **afficher la réponse 422 Pydantic** quand la validation échoue, pour bien voir comment FastAPI sécurise tes entrées.
>
> À la fin, tu sais utiliser Pydantic pour : valider les entrées, typer les sorties, générer automatiquement la doc Swagger avec exemples — et tu sais distinguer `GET` (paramètres dans l'URL) de `POST` (paramètres dans le body JSON).

## Structure du projet

```
fastapi-step-by-step-recap-pydantic-models-and-post-calculate/
├── fastapi/
│   └── Dockerfile
├── streamlit/
│   ├── Dockerfile
│   └── requirements.txt
├── docker-compose.yml          <- 2 services + reseau "calc-net" (idem chap28)
├── main.py                     <- backend + Pydantic models + POST /calculate
├── app.py                      <- front Streamlit qui POSTe du JSON
└── README.md
```

## What's 

| | chapitre précédent | ce chapitre  |
|---|---|---|
| Endpoint principal | `GET /add/{a}/{b}` | `POST /calculate` |
| Paramètres | dans l'URL (path) | dans le **body JSON** |
| Validation | Python implicite (`float`) | **Pydantic** (`BaseModel`, `Enum`) |
| Erreur de saisie | `200 OK` avec valeur cassée | **`422 Unprocessable Entity`** détaillé |
| Sortie | `dict` Python lâche | **`response_model=CalculationResponse`** typé |
| Swagger UI | endpoints listés | endpoints **+ exemples + schémas Pydantic** |

## Le concept clé : Pydantic

```python
from enum import Enum
from pydantic import BaseModel, Field


class Operation(str, Enum):
    add = "add"
    subtract = "subtract"
    multiply = "multiply"
    divide = "divide"


class CalculationRequest(BaseModel):
    a: float = Field(..., examples=[10])
    b: float = Field(..., examples=[2])
    operation: Operation


class CalculationResponse(BaseModel):
    operation: Operation
    a: float
    b: float
    result: float


@app.post("/calculate", response_model=CalculationResponse)
def calculate(payload: CalculationRequest):
    ...
```

Avec ce code :
- Si tu envoies `{"a": "abc", "b": 2, "operation": "add"}` → FastAPI répond **`422`** automatiquement avec un message expliquant que `a` doit être un nombre.
- Si tu envoies `{"a": 1, "b": 2, "operation": "modulo"}` → **`422`** car `modulo` n'est pas dans l'`Enum`.
- Le Swagger UI à `http://localhost:8000/docs` montre maintenant le **schéma JSON attendu** + un **bouton "Try it out"** qui pré-remplit avec les `examples=[10]` / `examples=[2]`.

## Run it (100% Docker, no Python on the host)

```bash
cd fastapi-step-by-step-recap-pydantic-models-and-post-calculate

# 1. Build + up des deux services :
docker compose up -d --build

# 2. Verifier les URLs :
#    Swagger UI (POST /calculate visible)  : http://localhost:8000/docs
#    Streamlit                              : http://localhost:8501

# 3. Tester l'endpoint POST en CLI :
curl -X POST http://localhost:8000/calculate \
  -H "Content-Type: application/json" \
  -d '{"a": 10, "b": 2, "operation": "divide"}'
# {"operation":"divide","a":10.0,"b":2.0,"result":5.0}

# 4. Tester la validation Pydantic (volontairement cassee) :
curl -X POST http://localhost:8000/calculate \
  -H "Content-Type: application/json" \
  -d '{"a": "abc", "b": 2, "operation": "add"}'
# 422 Unprocessable Entity + detail explicite

curl -X POST http://localhost:8000/calculate \
  -H "Content-Type: application/json" \
  -d '{"a": 1, "b": 2, "operation": "modulo"}'
# 422 Unprocessable Entity (Operation non valide)

# 5. Tester division par zero (HTTPException 400 cote backend) :
curl -X POST http://localhost:8000/calculate \
  -H "Content-Type: application/json" \
  -d '{"a": 1, "b": 0, "operation": "divide"}'
# {"detail":"Division par zero impossible"}

# 6. Stop :
docker compose down
```

## GET vs POST — quand utiliser quoi ?

| | `GET` | `POST` |
|---|---|---|
| Paramètres dans | URL (querystring ou path) | body JSON |
| Idempotent (= rejouable) | oui (lecture) | pas forcément (création) |
| Cacheable côté navigateur | oui | non |
| Limite de taille | URL (~2000 caractères) | body (Mo) |
| Body JSON structuré | non | oui |
| Mots de passe / secrets | **non** (visible dans logs) | mieux (mais HTTPS reste indispensable) |

> [!IMPORTANT]
> Pour une calculatrice 2-opérandes, GET marche très bien. On bascule en POST pour **apprendre le pattern** : payload structuré + validation Pydantic + réponses typées. C'est la façon standard d'écrire une API moderne.

## Recap

```bash
cd fastapi-step-by-step-recap-pydantic-models-and-post-calculate
docker compose up -d --build
# -> http://localhost:8501  (front)
# -> http://localhost:8000/docs  (backend Swagger avec POST /calculate)
docker compose down
```

## Recap +

```bash
cd fastapi-step-by-step-recap-pydantic-models-and-post-calculate
docker compose up -d --build

# Tester directement depuis le conteneur Streamlit :
docker compose exec streamlit python -c "import requests; print(requests.post('http://fastapi:8000/calculate', json={'a': 7, 'b': 6, 'operation': 'multiply'}).json())"
# {'operation': 'multiply', 'a': 7.0, 'b': 6.0, 'result': 42.0}

# Volontairement casser pour voir l'erreur 422 detaillee :
docker compose exec streamlit python -c "import requests; r = requests.post('http://fastapi:8000/calculate', json={'a': 'abc', 'b': 6, 'operation': 'multiply'}); print(r.status_code); print(r.json())"

docker compose down
```

# Troubleshooting - killing the zombies

<details>
   <summary> Troubleshooting </summary>

> Sur Windows, pour tuer le processus qui occupe les ports **8000** ou **8501** :

```bat
netstat -ano | findstr :8000
netstat -ano | findstr :8501
taskkill /PID 12345 /F
```

> Version PowerShell :

```powershell
Get-NetTCPConnection -LocalPort 8000
Get-NetTCPConnection -LocalPort 8501
Stop-Process -Id 12345 -Force
```

> **Erreur 422 Unprocessable Entity ?** C'est attendu et c'est le but du chapitre : Pydantic refuse les entrées invalides. Lis le champ `detail` de la réponse pour savoir précisément quel champ a posé problème.

</details>
