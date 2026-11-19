# Test rapide — Agent IA 2 V2

Premier passage recommandé :

```python
RUN_VECTOR_AGENT = True
RUN_VISION_AGENT = False
PRIMARY_OUTPUT_METHOD = "vector_agent"
```

Puis lancer le notebook du haut vers le bas.

Pour le test vision :

```bash
ollama pull qwen3-vl:4b
```

puis :

```python
RUN_VISION_AGENT = True
PRIMARY_OUTPUT_METHOD = "vision_agent"
```

Le notebook conserve aussi la référence Python `INTERSECT`
afin de mesurer précision, rappel et IoU linéaire.
