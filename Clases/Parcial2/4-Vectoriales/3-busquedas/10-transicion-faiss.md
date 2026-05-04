# Transición hacia FAISS

### Objetivo

Implementar una estructura de búsqueda más eficiente que la búsqueda lineal, introduciendo un índice vectorial simplificado que reduzca el número de comparaciones.

### Construcción de un índice por agrupación simple

En lugar de comparar contra todos los vectores, se construye una estructura de agrupación por categorías aproximadas.

La idea es simular cómo un índice reduce el espacio de búsqueda.

```python
indice = {
    "animales": [],
    "vehiculos": [],
    "velocidad": []
}
```

### Clasificación de vectores en el índice

Cada texto se asigna a una categoría aproximada según su contenido.

```python
def clasificar(texto):
    if "gato" in texto or "perro" in texto or "felino" in texto:
        return "animales"
    if "auto" in texto or "coche" in texto:
        return "vehiculos"
    if "rápido" in texto or "lento" in texto:
        return "velocidad"
    return "otros"
```

### Construcción del índice

Se distribuyen los elementos de la base en el índice.

```python
for texto in base:
    categoria = clasificar(texto)
    indice[categoria].append(texto)

print({k: len(v) for k, v in indice.items()})
```

### Embeddings del índice

Se generan embeddings por separado para cada grupo.

```python
indice_vec = {
    k: [embedding_frase_1(t) for t in v]
    for k, v in indice.items()
}
```

# Nueva estrategia de búsqueda

En lugar de buscar en toda la base, primero se selecciona una categoría probable.

```python
def seleccionar_categoria(query):
    if "gato" in query or "perro" in query or "felino" in query:
        return "animales"
    if "auto" in query or "coche" in query:
        return "vehiculos"
    if "rápido" in query or "lento" in query:
        return "velocidad"
    return "animales"
```

---

# Búsqueda sobre un subconjunto reducido

```python
def buscar_indexado(query):
    categoria = seleccionar_categoria(query)
    q_vec = embedding_frase_1(query)

    resultados = []

    for texto, vec in zip(indice[categoria], indice_vec[categoria]):
        sim = coseno(q_vec, vec)
        resultados.append((texto, sim))

    return sorted(resultados, key=lambda x: x[1], reverse=True)
```

---

# Ejecución comparativa

```python
print(buscar_indexado("gato rápido")[:5])
```

---

# Comparación con búsqueda lineal

```python
print(buscar_lineal("gato rápido")[:5])
print(buscar_indexado("gato rápido")[:5])
```

---

# Reducción del espacio de búsqueda

Se mide el número de comparaciones efectivas.

```python
def contar_comparaciones_indexado(query):
    categoria = seleccionar_categoria(query)
    return len(indice[categoria])

print(contar_comparaciones_indexado("gato rápido"))
```

---

# Comparación con el sistema completo

```python
print("Total base:", len(base))
print("Subconjunto indexado:", len(indice["animales"]))
```

---

# Impacto en rendimiento

```python
import time

start = time.time()
buscar_indexado("gato rápido")
end = time.time()

print("Tiempo indexado:", end - start)
```

---

# Comparación directa de escalabilidad

```python
start = time.time()
buscar_lineal("gato rápido")
end = time.time()

print("Lineal:", end - start)

start = time.time()
buscar_indexado("gato rápido")
end = time.time()

print("Indexado:", end - start)
```

---

# Limitación del índice manual

El sistema funciona, pero presenta problemas estructurales:

* la clasificación es rígida
* depende de reglas manuales
* no captura relaciones complejas

---

# Necesidad de índices automáticos

En sistemas reales, no se usan reglas manuales como:

```python
if "gato" in texto:
```

En su lugar, se usan estructuras que aprenden la organización del espacio vectorial.

---

# Introducción a índices reales

Los sistemas modernos no usan diccionarios manuales, sino estructuras como:

* clustering automático
* grafos de vecindad
* particiones dinámicas del espacio

Estas estructuras se construyen directamente sobre embeddings.

---

# FAISS como implementación real

FAISS permite construir estos índices de forma eficiente.

Su funcionamiento se basa en:

* agrupar vectores similares
* reducir candidatos de búsqueda
* aproximar vecinos cercanos sin comparar todo

---

# Diferencia clave con el sistema construido

Sistema manual:

* reglas explícitas
* categorías fijas
* lógica determinista

Sistema real:

* agrupación basada en geometría
* sin etiquetas humanas
* optimización estadística

---

# Cambio conceptual

El problema deja de ser:

“cómo comparo vectores”

y pasa a ser:

“cómo organizo el espacio para no tener que comparar todos los vectores”

