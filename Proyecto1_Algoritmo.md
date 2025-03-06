# Implementación del algoritmo A*

Para la realizacion de este proyecto, se tomo como referencia los
apuntes y el codigo base proporcionados por el profesor en su página,
asi como diversos ejercicios realizados en clase que funjieron un papel
importante para la comprensión de la forma de trabjar del algoritmo.

**Enlace al codigo proporcionado como base, asi como a los apuntes teóricos:** [Apuntes IA](https://ealcaraz85.github.io/IA.io/#orgbe4d9ff)

---

## 1. Importación de Librerías

```python
import pygame
from queue import PriorityQueue
import math
import time
```

- `pygame`: Se usa para crear la interfaz gráfica y manejar eventos del usuario.
- `PriorityQueue`: Implementa una cola de prioridad para manejar los nodos en la búsqueda A*.
- `math`: Se utiliza para operaciones matemáticas básicas.
- `time`: Se emplea para agregar retardos en la visualización.

---

## 2. Configuración Inicial

```python
ANCHO_VENTANA = 600
VENTANA = pygame.display.set_mode((ANCHO_VENTANA, ANCHO_VENTANA))
pygame.display.set_caption("Visualización de A*")
pygame.font.init()
```

- Se define el tamaño de la ventana y se inicializa la fuente para textos en `pygame`.

### Definición de Colores

```python
BLANCO = (255, 255, 255)
NEGRO = (0, 0, 0)
GRIS = (128, 128, 128)
VERDE = (0, 255, 0)
ROJO = (255, 0, 0)
NARANJA = (255, 165, 0)
PURPURA = (128, 0, 128)
AZUL_TENUE = (173, 216, 230)
TURQUESA = (64, 224, 208)
AMARILLO = (255, 255, 0)
```

Se definen colores en formato RGB para representar los distintos estados de los nodos en la visualización.

---

## 3. Clase `Nodo`

Cada celda de la cuadrícula es un objeto `Nodo` que tiene información sobre su posición, estado y costos heurísticos.

### Constructor

```python
class Nodo:
    def __init__(self, fila, col, ancho, total_filas):
        self.fila = fila
        self.col = col
        self.x = fila * ancho
        self.y = col * ancho
        self.color = BLANCO
        self.ancho = ancho
        self.total_filas = total_filas
        
        self.g = float("inf")
        self.h = 0
        self.f = float("inf")
        self.vecinos = []
        self.previo = None
```

- Se almacena la posición y el color del nodo.
- Se inicializan las variables `g` (costo desde el inicio), `h` (heurística), y `f` (suma de ambas).
- Se guarda una referencia a los nodos vecinos y al nodo previo en la ruta.

### Métodos de Estado y Modificación

Incluyen funciones para obtener la posición, cambiar el estado del nodo (inicio, fin, pared, camino, etc.), y restablecer valores.

Ejemplo:

```python
def hacer_pared(self):
    self.color = NEGRO
```

Este método hace que el nodo sea una pared, impidiendo el paso a través de él.

### Método `actualizar_vecinos()`

Determina los vecinos accesibles del nodo considerando las direcciones cardinales y diagonales, con diferentes costos.

```python
def actualizar_vecinos(self, grid):
    direcciones = [
        (1, 0, 10), (-1, 0, 10), (0, 1, 10), (0, -1, 10),
        (1, 1, 14), (1, -1, 14), (-1, 1, 14), (-1, -1, 14)
    ]
    for df, dc, costo in direcciones:
        nueva_fila, nueva_col = self.fila + df, self.col + dc
        if (0 <= nueva_fila < self.total_filas and
            0 <= nueva_col < self.total_filas and
            not grid[nueva_fila][nueva_col].es_pared()):
            self.vecinos.append((grid[nueva_fila][nueva_col], costo))
```

---

## 4. Implementación del Algoritmo A*

### Funciones Auxiliares

#### Heurística Manhattan

```python
def heuristica(p1, p2):
    x1, y1 = p1
    x2, y2 = p2
    return 10 * (abs(x1 - x2) + abs(y1 - y2))
```

Calcula la distancia Manhattan entre dos puntos multiplicada por 10 debido a que ese es el costo manejado para este caso en particular al movernos entre vecinos.

#### Reconstrucción del Camino

```python
def reconstruir_camino(nodo_actual, dibujar_func):
    while nodo_actual.previo:
        nodo_actual = nodo_actual.previo
        if not nodo_actual.es_inicio():
            nodo_actual.hacer_camino()
        dibujar_func()
        time.sleep(0.2)
```

Dibuja el camino una vez que se encuentra la solución.

### Ejecución del Algoritmo

```python
def algoritmo_astar(dibujar_func, grid, inicio, fin):
    conjunto_abierto = PriorityQueue()
    conjunto_abierto.put((0, 0, inicio))
    nodos_abiertos = {inicio}
    nodos_cerrados = set()
    
    inicio.g = 0
    inicio.h = heuristica(inicio.get_pos(), fin.get_pos())
    inicio.f = inicio.g + inicio.h
    
    while not conjunto_abierto.empty():
        _, _, nodo_actual = conjunto_abierto.get()
        nodos_abiertos.remove(nodo_actual)
        
        if nodo_actual == fin:
            reconstruir_camino(nodo_actual, dibujar_func)
            return True
        
        for vecino, costo in nodo_actual.vecinos:
            g_tentativa = nodo_actual.g + costo
            if g_tentativa < vecino.g:
                vecino.previo = nodo_actual
                vecino.g = g_tentativa
                vecino.h = heuristica(vecino.get_pos(), fin.get_pos())
                vecino.f = vecino.g + vecino.h
                if vecino not in nodos_abiertos:
                    conjunto_abierto.put((vecino.f, 0, vecino))
                    nodos_abiertos.add(vecino)
    return False
```

- Se usa una cola de prioridad para seleccionar el nodo con menor `f` ya que es lo que buscamos en el algoritmo, movernos por el nodo con menor `f`.
- Se expande el nodo actual y se actualizan los valores de `g`, `h`, y `f` dependiendo del nodo calculado y el nodo en el que se esta actualmente.
- Si el nodo final se alcanza, se reconstruye el camino sombreando dichos nodos de un color azul.

---

## 5. Interfaz Gráfica y Lógica del Juego

La función `main()` maneja eventos del usuario y ejecuta el algoritmo cuando se presiona `R`.

```python
if event.key == pygame.K_r and inicio and fin:
    algoritmo_astar(lambda: dibujar(VENTANA, grid, FILAS, ANCHO_VENTANA), grid, inicio, fin)
```

Se actualizan los vecinos antes de ejecutar la búsqueda.