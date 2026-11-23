# Nouvelle organisation du 03bis

## 03bis-A — Méthode 0 stable

Source fonctionnelle : `03_BIS_TERRAIN_FOREST_OSM_CORRIGE.ipynb` du GitHub.

Rôle :
- récupérer les surfaces forêt OSM ;
- gérer les grandes forêts englobantes ;
- reconstruire les polygones ;
- mesurer la couverture du corridor ;
- produire les annotations officielles pour le notebook 04.

Sorties :
- `terrain_forest_annotations.json` — sortie officielle ;
- `forest_osm_union.geojson` — géométrie partagée avec les expériences ;
- `forest_osm_context.json` — métadonnées de récupération.

## 03bis-B — Expériences Agent IA 2

Ne refait pas Overpass et ne modifie jamais la sortie officielle.

Contient :
- référence SIG `INTERSECT` ;
- Test A : Agent IA vectoriel ;
- Test B1 : raisonnement spatial visuel sur carte OSM abstraite ;
- Test B2 : prévu plus tard, vraie reconnaissance visuelle sur orthophoto/satellite.

## Pourquoi deux notebooks ?

La méthode 0 est déjà fonctionnelle. Les expériences IA sont encore exploratoires.
Les séparer évite qu'une expérience :
- casse la récupération OSM ;
- remplace accidentellement `terrain_forest_annotations.json` ;
- oblige à refaire les appels réseau ;
- rende difficile la comparaison scientifique.

Le lien entre les deux est explicite :

```text
03bis-A
  └── forest_osm_union.geojson
           ↓
03bis-B
```
