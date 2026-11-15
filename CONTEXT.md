# CONTEXT.md — PIR Route Agent

## 1. Contexte du projet

Ce projet de recherche vise à annoter automatiquement des portions d'itinéraires à partir de caractéristiques spatiales et topographiques.

Entrée principale :

```text
un itinéraire GPX
```

Sortie cible :

```text
portions d'itinéraire
+
annotations structurées
+
description naturelle
```

La difficulté scientifique centrale est que toutes les caractéristiques ne s'observent pas à la même échelle spatiale.

Exemples :
- une pente forte peut être détectée sur une fenêtre courte ;
- une montée soutenue nécessite une fenêtre plus longue ;
- une série de lacets nécessite d'observer plusieurs changements de direction ;
- une sinuosité générale nécessite une autre échelle.

La discrétisation / fenêtre d'analyse ne doit donc pas être figée.

## 2. Objectif de la V1

La V1 doit fonctionner sur :

```text
1 fichier GPX
```

Caractéristiques initiales :

```text
- forte_pente
- pente_douce
- montee
- descente
- lacets
- sinuosite
```

Hors périmètre V1 car nécessitant des données externes :

```text
- forêt
- ombre
- panorama
- proximité d'eau
- type de terrain
- POI
```

Ces éléments pourront être ajoutés plus tard avec OpenStreetMap, BD TOPO, MNT ou d'autres sources.

## 3. Architecture conceptuelle

```text
GPX
 │
 ▼
Parsing
 │
 ▼
Rééchantillonnage spatial
 │
 ▼
Route normalisée
 │
 ▼
AGENT 1
Choix des outils et des échelles
 │
 ├── pente
 ├── montée
 ├── descente
 ├── lacets
 └── sinuosité
 │
 ▼
Annotations atomiques
 │
 ▼
AGENT 2
Agrégation spatiale déterministe
 │
 ▼
Portions + ensemble d'annotations
 │
 ▼
AGENT 3
Verbalisation LLM
 │
 ▼
Description naturelle
```

## 4. Principe fondamental : LLM ≠ calculateur géométrique

Le LLM ne doit jamais calculer lui-même :
- une pente ;
- une distance ;
- une courbure ;
- une longueur ;
- un nombre de lacets ;
- une intersection géométrique.

Ces opérations doivent être réalisées par des fonctions Python déterministes.

Le rôle du LLM dans l'Agent 1 est de choisir :

```text
quel outil utiliser
+
quelle échelle ou fenêtre tester
```

Exemple :

```text
Feature : lacets
        ↓
Agent 1
        ↓
"tester 75 m, 150 m et 250 m"
        ↓
Python
        ↓
detect_switchbacks(...)
        ↓
résultats
        ↓
Agent 1
        ↓
choix / interprétation
```

## 5. Rééchantillonnage de base

Les points GPX sont souvent irrégulièrement espacés.

Exemple brut :

```text
0 m
7 m
19 m
34 m
52 m
```

Représentation normalisée :

```text
0 m
10 m
20 m
30 m
40 m
50 m
```

Résolution de base initiale proposée :

```text
10 m
```

Cette résolution constitue la représentation commune de la route.

## 6. Fenêtres d'analyse initiales

```text
forte_pente :
10 / 25 / 50 m

montee :
50 / 100 / 200 m

descente :
50 / 100 / 200 m

lacets :
75 / 150 / 250 m

sinuosite :
50 / 100 / 200 m
```

Ces valeurs ne sont pas définitives.

Elles doivent être :
- configurables ;
- testables ;
- comparables ;
- conservées dans les résultats.

## 7. Agent 1 — Orchestrateur d'analyse

Responsabilité :
- recevoir une caractéristique cible ;
- connaître les outils disponibles ;
- connaître les fenêtres autorisées ;
- choisir quel détecteur appeler ;
- choisir quelle(s) fenêtre(s) tester.

Exemple d'entrée :

```json
{
  "feature": "lacets",
  "route_length_m": 8200,
  "base_spacing_m": 10,
  "allowed_windows_m": [75, 150, 250]
}
```

Exemple de sortie :

```json
{
  "detector": "switchbacks",
  "windows_to_test_m": [75, 150, 250]
}
```

Python exécute ensuite réellement les détecteurs.

## 8. Détecteurs déterministes

Dossier :

```text
src/detectors/
```

Fichiers initiaux :

```text
slope.py
ascent.py
descent.py
switchbacks.py
sinuosity.py
```

Exemple d'interface :

```python
detect_switchbacks(
    route,
    window_m=150,
    step_m=25
)
```

## 9. Annotation atomique

Toutes les sorties doivent suivre une structure commune.

Exemple :

```json
{
  "feature": "forte_pente",
  "start_m": 1200,
  "end_m": 1425,
  "resolution_m": 25,
  "confidence": 0.91,
  "metrics": {
    "mean_slope_pct": 16.4
  }
}
```

Exemple lacets :

```json
{
  "feature": "lacets",
  "start_m": 1325,
  "end_m": 1650,
  "resolution_m": 150,
  "confidence": 0.87,
  "metrics": {
    "direction_changes": 5
  }
}
```

Ces objets doivent être représentés par des modèles Pydantic.

## 10. Agent 2 — Agrégation spatiale

Agent 2 est déterministe dans la V1.

Exemple :

```text
forte_pente :
1000 ---------------- 1500

lacets :
        1250 ---------------- 1750
```

Bornes :

```text
1000
1250
1500
1750
```

Résultat :

```text
1000–1250
{forte_pente}

1250–1500
{forte_pente, lacets}

1500–1750
{lacets}
```

Les portions adjacentes ayant exactement le même ensemble d'annotations peuvent être fusionnées.

## 11. Agent 3 — Verbalisation

Agent 3 utilise le LLM.

Entrée exemple :

```json
{
  "start_m": 1250,
  "end_m": 1500,
  "annotations": [
    "forte_pente",
    "lacets"
  ],
  "metrics": {
    "mean_slope_pct": 17.2,
    "switchbacks": 4
  }
}
```

Sortie exemple :

```text
Entre 1,25 km et 1,5 km, l'itinéraire grimpe fortement
à travers une succession de lacets.
```

Contrainte :
le LLM ne doit jamais inventer une information absente des données structurées.

## 12. Structure logicielle cible

```text
project/
│
├── data/
│   ├── input/
│   └── output/
│
├── src/
│   ├── __init__.py
│   ├── gpx_loader.py
│   ├── resampling.py
│   │
│   ├── detectors/
│   │   ├── __init__.py
│   │   ├── slope.py
│   │   ├── ascent.py
│   │   ├── descent.py
│   │   ├── switchbacks.py
│   │   └── sinuosity.py
│   │
│   ├── models.py
│   ├── ollama_client.py
│   ├── agent1.py
│   ├── agent2.py
│   └── agent3.py
│
├── tests/
├── main.py
├── ENVIRONMENT.md
├── CONTEXT.md
├── requirements.txt
├── environment.yml
└── README.md
```

## 13. Modèles de données

Créer au minimum :

```python
class RoutePoint(BaseModel):
    distance_m: float
    latitude: float
    longitude: float
    elevation_m: float | None

class Annotation(BaseModel):
    feature: str
    start_m: float
    end_m: float
    resolution_m: float
    confidence: float
    metrics: dict

class Portion(BaseModel):
    start_m: float
    end_m: float
    annotations: list[str]
    metrics: dict
```

## 14. Ollama

Serveur :

```text
http://localhost:11434
```

Premier modèle :

```text
llama3.1:8b
```

Tous les appels passent par :

```text
src/ollama_client.py
```

Le reste du code ne dépend pas directement d'Ollama.

## 15. Priorité de développement pour Copilot / Claude

### Étape A

Créer :

```text
src/models.py
src/gpx_loader.py
src/resampling.py
```

et leurs tests.

### Étape B

Créer deux détecteurs seulement :

```text
slope.py
switchbacks.py
```

et leurs tests.

### Étape C

Créer :

```text
agent2.py
```

pour fusionner les annotations.

### Étape D

Créer :

```text
ollama_client.py
agent3.py
```

et produire une description naturelle.

### Étape E

Créer une première version de :

```text
agent1.py
```

qui choisit parmi des fenêtres prédéfinies.

## 16. Premier milestone

Première démonstration complète :

```text
1 GPX
+
2 caractéristiques :
- pente
- lacets
```

Pipeline :

```text
route_test.gpx
     ↓
parse
     ↓
resample 10 m
     ↓
slope detector
switchback detector
     ↓
annotations
     ↓
agent2
     ↓
portions
     ↓
agent3
     ↓
description française
```

Ce milestone doit fonctionner avant d'ajouter les autres caractéristiques.

## 17. Tests obligatoires

Prévoir :

```text
test_gpx_loader.py
test_resampling.py
test_slope.py
test_switchbacks.py
test_agent2.py
test_ollama_client.py
```

Le système ne doit pas dépendre du LLM pour tester :
- parsing ;
- calculs ;
- agrégation.

## 18. Principes de conception

1. Séparer calcul déterministe et raisonnement LLM.
2. Aucun calcul géométrique dans un prompt.
3. Aucun texte libre entre modules si un JSON structuré suffit.
4. Utiliser Pydantic pour les objets échangés.
5. Rendre toutes les fenêtres configurables.
6. Conserver la résolution utilisée dans chaque annotation.
7. Conserver les métriques brutes ayant conduit à une décision.
8. Tester d'abord un seul GPX.
9. Ajouter les sources externes après validation de la V1.
10. Ne pas ajouter de framework agentique tant qu'il n'est pas nécessaire.

## 19. Hors périmètre immédiat

Ne pas implémenter maintenant :

```text
PostgreSQL
PostGIS
LangChain
LangGraph
RAG
embeddings
vector database
web application
API REST
Docker
```
