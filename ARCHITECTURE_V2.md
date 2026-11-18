# ARCHITECTURE_V2.md — contraintes sémantiques + terrain OSM

## Pipeline

```text
02 — préparation GPX
        │
        ├──────────────────────────────┐
        ▼                              ▼
03 — Agent 1                    03bis — Terrain OSM
géométrie/topographie           forêt
        │                              │
        └──────────────┬───────────────┘
                       ▼
                04 — Agent 2
        consolidation multi-échelles
        + contraintes sémantiques
                       │
                       ▼
                portions finales
                       │
                       ▼
                05 — Agent 3
                 verbalisation
```

## Caractéristiques V2

### Direction de pente

```text
montee XOR descente
```

Une portion finale ne peut avoir qu'une seule direction.

### Intensité de pente

```text
pente_douce XOR forte_pente
```

Une portion finale ne peut avoir qu'une seule intensité.

### Géométrie

```text
lacets
```

`sinuosite` est retirée de la V2.

### Terrain

```text
foret
```

La forêt est détectée indépendamment avec OpenStreetMap.

## Pourquoi le notebook 03bis ?

Le GPX contient latitude, longitude et altitude, mais ne décrit pas directement
l'occupation du sol.

Le terrain dépend donc d'une source géographique externe.

Le notebook 03bis produit les mêmes objets conceptuels que le notebook 03 :

```json
{
  "feature": "foret",
  "family": "terrain",
  "start_m": 1000,
  "end_m": 1450,
  "resolution_m": 10,
  "confidence": 0.84,
  "metrics": {
    "source": "OpenStreetMap"
  }
}
```

Le notebook 04 peut donc fusionner des annotations provenant de sources différentes.

## Forêt OSM

Tags utilisés :

```text
natural=wood
landuse=forest
landcover=trees
```

Le détecteur :

1. télécharge les polygones boisés autour de la route ;
2. projette les géométries dans un CRS UTM métrique ;
3. construit un corridor de 10 m autour de chaque segment de route ;
4. calcule la proportion du corridor couverte par la forêt ;
5. classe le segment en forêt si cette proportion dépasse 50 % ;
6. fusionne les segments forestiers consécutifs.

## Résolution des contradictions dans l'Agent 2

Les détections multi-échelles peuvent se chevaucher.

Exemple :

```text
montee 100 m
descente 50 m
```

sur le même intervalle.

Le système agrège les preuves et calcule un score par caractéristique.

Un petit bonus est accordé quand plusieurs résolutions soutiennent la même
caractéristique.

Si les scores de deux caractéristiques incompatibles sont nettement différents,
le meilleur score gagne.

S'ils sont proches, la pente locale du segment sert d'arbitre déterministe.

Le LLM n'intervient pas dans cette résolution.

## Sortie structurée d'une portion

```json
{
  "start_m": 300,
  "end_m": 620,
  "slope_direction": "montee",
  "slope_intensity": "forte_pente",
  "geometry_features": ["lacets"],
  "terrain_features": ["foret"],
  "annotations": [
    "montee",
    "forte_pente",
    "lacets",
    "foret"
  ]
}
```

Cette représentation rend structurellement impossibles :

```text
montee + descente
pente_douce + forte_pente
```

## Agent 3

L'Agent 3 reçoit la structure finale et la verbalise.

Exemple :

```text
Entre 300 et 620 m, l'itinéraire effectue une forte montée en forêt
à travers une succession de lacets.
```

L'Agent 3 n'a pas le droit de déduire :

- ombre ;
- panorama ;
- nature du sentier ;
- humidité ;
- type d'essence forestière ;

sauf si une source future fournit explicitement cette information.
