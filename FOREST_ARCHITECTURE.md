# FOREST_ARCHITECTURE_V4.md

## Nouvelle stratégie

```text
route GPX
   ├─ bbox locale → petites forêts / frontières proches
   ├─ points tous les 200 m → is_in → surfaces englobantes → pivot → géométrie OSM
   └─ si couverture < 10 % → bbox élargie progressive
                         ↓
                  union / déduplication
                         ↓
               couche forêt OSM complète
                         ↓
                 corridor ±10 m
                         ↓
              annotations `foret`
```

### Pourquoi `is_in`

`is_in(lat,lon)` cherche les surfaces Overpass qui contiennent le point.
La frontière de la forêt n'a donc pas besoin de traverser la petite bbox.

### Pourquoi le fallback élargi

Overpass ne crée pas une `area` pour chaque relation OSM.
Une grande relation forestière non matérialisée comme `area` peut donc échapper à `is_in`.
Les bbox élargies servent de secours pour récupérer directement sa frontière.

### Compatibilité

La sortie reste `data/output/terrain_forest_annotations.json`.
Le notebook 04 V2 ne change pas.
