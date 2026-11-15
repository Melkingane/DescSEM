# ENVIRONMENT.md — PIR Route Agent

## Objectif

Ce document décrit l'environnement de développement de référence du projet.

L'objectif est que deux développeurs puissent reproduire le même environnement sur Windows 11 et exécuter le projet depuis VS Code.

## Système cible

- Windows 11
- Visual Studio Code
- Git
- Miniforge / Conda
- Python 3.11
- Ollama
- LLM local via Ollama

## IDE

IDE de référence :

```text
Visual Studio Code
```

Extensions recommandées :

- Python — Microsoft
- Pylance — Microsoft
- Jupyter — Microsoft

Le développement principal doit être réalisé en fichiers `.py`.

Les notebooks sont utilisés pour :
- documentation ;
- expérimentation ;
- validation ponctuelle ;
- démonstrations.

## Environnement Python

Nom :

```text
pir-route
```

Création :

```powershell
conda create -n pir-route python=3.11 -y
```

Activation :

```powershell
conda activate pir-route
```

Vérification :

```powershell
python --version
```

Résultat attendu :

```text
Python 3.11.x
```

## Dépendances Python de la V1

```powershell
python -m pip install --upgrade pip
python -m pip install numpy pandas gpxpy shapely pyproj scipy pydantic requests matplotlib pytest
```

| Package | Utilisation |
|---|---|
| numpy | calcul numérique |
| pandas | résultats tabulaires et annotations |
| gpxpy | parsing GPX |
| shapely | géométrie |
| pyproj | projections et coordonnées |
| scipy | filtres, lissage, calcul scientifique |
| pydantic | validation des structures d'échange |
| requests | appels HTTP à Ollama |
| matplotlib | visualisation / debug |
| pytest | tests |

## LLM local

Moteur :

```text
Ollama
```

Serveur local :

```text
http://localhost:11434
```

Modèle initial :

```text
llama3.1:8b
```

Téléchargement :

```powershell
ollama pull llama3.1:8b
```

Test :

```powershell
ollama run llama3.1:8b
```

Le modèle pourra être remplacé sans changer l'architecture globale.

## Isolation du LLM

Tout appel au modèle doit être centralisé dans :

```text
src/ollama_client.py
```

Aucun autre module ne doit appeler directement l'API Ollama.

Cette règle permet de remplacer facilement le backend LLM.

## Reproductibilité

Tous les développeurs doivent utiliser :

```text
Python 3.11
```

Les dépendances doivent être enregistrées dans :

```text
requirements.txt
```

et/ou :

```text
environment.yml
```

Ne pas versionner :

```text
.conda/
.venv/
__pycache__/
*.pyc
modèles locaux Ollama
fichiers temporaires
sorties volumineuses
```

## Vérification minimale

La machine est prête si :

```powershell
conda activate pir-route
python --version
python -c "import numpy, pandas, gpxpy, shapely, pyproj, scipy, pydantic, requests; print('OK')"
ollama --version
ollama list
```

Puis Python doit pouvoir appeler :

```text
http://localhost:11434/api/chat
```

## Non nécessaire dans la V1

Ne pas installer pour l'instant :

- LangChain
- LangGraph
- PostgreSQL
- PostGIS
- Docker
- pgvector
- FAISS
- Chroma
- FastAPI
- OpenAI SDK
