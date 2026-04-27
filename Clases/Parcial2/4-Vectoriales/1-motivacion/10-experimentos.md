# Experimentos

Probar múltiples queries:

```python
print(buscar("felino"))
print(buscar("animal"))
print(buscar("auto"))
print(buscar("perro grande"))
```

Luego modificar reglas del embedding en vivo:

* agregar nuevas palabras
* cambiar pesos

### Introducción a problemas reales

Agregar ambigüedad:

```python
base.append("banco financiero")
base.append("banco de parque")

vectores = [embedding_frase(t) for t in base]

print(buscar("banco"))
```

Resultado esperado:
inconsistente


