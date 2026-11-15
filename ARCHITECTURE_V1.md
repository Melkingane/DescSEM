# ARCHITECTURE_V1.md

## Objectif

Construire un pipeline expérimental permettant d'annoter des portions d'un itinéraire GPX avec des caractéristiques topographiques / géométriques observées à différentes échelles, puis de produire une description naturelle de ces portions.

## Pipeline

```text
GPX
 ↓
Parser GPX
 ↓
Rééchantillonnage spatial régulier (10 m)
 ↓
Agent 1
 ↓
Détecteurs Python déterministes
 ↓
Annotations atomiques
 ↓
Agent 2 déterministe
 ↓
Portions homogènes
 ↓
Agent 3 LLM
 ↓
Description naturelle
```

## Agent 1

Agent 1 est un orchestrateur.

Il NE calcule PAS :
- pente ;
- distance ;
- courbure ;
- changement de direction ;
- sinuosité.

Il choisit :
- un détecteur ;
- une ou plusieurs fenêtres autorisées.

Les détecteurs font ensuite les calculs.

### Configuration initiale

```python
FEATURE_CONFIG = {
    "forte_pente": {
        "detector": "slope",
        "allowed_windows_m": [10, 25, 50],
    },
    "pente_douce": {
        "detector": "slope",
        "allowed_windows_m": [25, 50, 100],
    },
    "montee": {
        "detector": "slope",
        "allowed_windows_m": [50, 100, 200],
    },
    "descente": {
        "detector": "slope",
        "allowed_windows_m": [50, 100, 200],
    },
    "lacets": {
        "detector": "switchbacks",
        "allowed_windows_m": [75, 150, 250],
    },
    "sinuosite": {
        "detector": "sinuosity",
        "allowed_windows_m": [50, 100, 200],
    },
}
```

Ces paramètres sont des valeurs de départ expérimentales, pas des vérités définitives.

## Agent 2

Agent 2 est déterministe.

Il transforme des intervalles éventuellement chevauchants en portions homogènes.

Exemple :

```text
forte_pente : 1000–1500
lacets      : 1250–1750
```

devient :

```text
1000–1250 → forte_pente
1250–1500 → forte_pente + lacets
1500–1750 → lacets
```

## Agent 3

Agent 3 est un LLM de verbalisation.

Entrée :

```json
{
  "start_m": 1250,
  "end_m": 1500,
  "annotations": ["forte_pente", "lacets"],
  "metrics": {
    "mean_slope_pct": 17.2,
    "direction_changes": 4
  }
}
```

Sortie :

```text
Entre 1,25 km et 1,5 km, l'itinéraire grimpe fortement à travers une succession de lacets.
```

Le modèle n'a pas le droit d'inventer d'information extérieure.

## Données externes

Hors V1 :
- forêt ;
- ombre ;
- eau ;
- panorama ;
- POI ;
- occupation du sol.

Ces caractéristiques seront intégrées plus tard avec OSM, BD TOPO, MNT ou d'autres données spatiales.

## Architecture logicielle cible

```text
src/
├── models.py
├── gpx_loader.py
├── resampling.py
├── ollama_client.py
├── agent1.py
├── agent2.py
├── agent3.py
└── detectors/
    ├── slope.py
    ├── ascent.py
    ├── descent.py
    ├── switchbacks.py
    └── sinuosity.py
```

## Principe de recherche

Toutes les fenêtres d'analyse doivent être :
- enregistrées ;
- configurables ;
- comparables ;
- reproductibles.

Le projet doit permettre d'étudier comment la granularité spatiale modifie la détection des caractéristiques.
