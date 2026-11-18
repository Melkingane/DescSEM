# MULTISCALE_AGGREGATION.md

## Décision d'architecture

Une même caractéristique peut être détectée à plusieurs fenêtres d'analyse.

Exemple :

```text
pente_douce à 25 m
pente_douce à 50 m
pente_douce à 100 m
```

Ces résultats ne sont pas trois caractéristiques différentes.
Ce sont plusieurs observations du même phénomène à des échelles différentes.

Le pipeline devient donc :

```text
Agent 1
│
├── observations 25 m
├── observations 50 m
└── observations 100 m
        │
        ▼
Agent 2 — étape A
consolidation par caractéristique
        │
        ▼
pente_douce consolidée
avec evidence=[25, 50, 100]
        │
        ▼
Agent 2 — étape B
overlay avec lacets, montée, descente, sinuosité...
        │
        ▼
portions finales
```

## Pourquoi conserver les résolutions ?

Une zone soutenue par plusieurs échelles peut être distinguée d'une zone
détectée uniquement à une échelle très locale.

Le système conserve donc :

- `resolutions_m`
- `n_evidence`
- `mean_confidence`
- `max_confidence`
- `multi_scale`
- les observations originales dans `evidence`

Aucune information multi-échelle n'est perdue pendant la fusion.

## Règle de fusion V1

Deux détections d'une même caractéristique sont consolidées si elles :

- se chevauchent ;
- ou sont séparées par au plus 10 m.

Les 10 m correspondent au pas de rééchantillonnage spatial de base.

Cette règle est volontairement simple et doit rester paramétrable pour les
expériences futures.

## Important

L'Agent 1 peut toujours sélectionner plusieurs fenêtres.

On NE force PAS :

```text
une caractéristique → une seule fenêtre
```

Au contraire, on autorise :

```text
une caractéristique → plusieurs échelles d'observation
```

puis l'Agent 2 consolide les preuves spatiales.
