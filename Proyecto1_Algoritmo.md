# Implementación del algoritmo A*
**Elaborado por:** Edgar Tapia Martinez (21120262)

Para la realizacion de este proyecto, se tomo como referencia los
apuntes y el codigo base proporcionados por el profesor en su página, asi como diversos ejercicios realizados en clase que funjieron un papel importante para la comprensión de la forma de trabjar del algoritmo.

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


## 6. Código completo con la implementación del algoritmo

**Se incluyen comentarios para explicar los procesos en cada parte del código presentado**

```
import pygame
from queue import PriorityQueue
import math
import time

# Configuraciones iniciales
ANCHO_VENTANA = 600
VENTANA = pygame.display.set_mode((ANCHO_VENTANA, ANCHO_VENTANA))
pygame.display.set_caption("Visualización de A*")
pygame.font.init()  # Inicializar el módulo de fuentes

# Colores (RGB)
BLANCO = (255, 255, 255)
NEGRO = (0, 0, 0)
GRIS = (128, 128, 128)
VERDE = (0, 255, 0)
ROJO = (255, 0, 0)
NARANJA = (255, 165, 0)
PURPURA = (128, 0, 128)
AZUL = (0, 0, 255)
AZUL_TENUE = (173, 216, 230)  # Azul claro/tenue
TURQUESA = (64, 224, 208)
AMARILLO = (255, 255, 0)

class Nodo:
    def __init__(self, fila, col, ancho, total_filas):
        self.fila = fila
        self.col = col
        self.x = fila * ancho
        self.y = col * ancho
        self.color = BLANCO
        self.ancho = ancho
        self.total_filas = total_filas
        
        # Valores para A*
        self.g = float("inf")  # Costo desde el inicio
        self.h = 0  # Heurística (distancia estimada al final)
        self.f = float("inf")  # f = g + h
        self.vecinos = []
        self.previo = None
        
    def get_pos(self):
        return self.fila, self.col
        
    def get_centro(self):
        return self.x + self.ancho // 2, self.y + self.ancho // 2

    def es_pared(self):
        return self.color == NEGRO

    def es_inicio(self):
        return self.color == NARANJA

    def es_fin(self):
        return self.color == PURPURA
        
    def es_actual(self):
        return self.color == ROJO
        
    def es_camino(self):
        return self.color == AZUL_TENUE

    def restablecer(self):
        self.color = BLANCO
        self.g = float("inf")
        self.h = 0
        self.f = float("inf")
        self.previo = None

    def hacer_inicio(self):
        self.color = NARANJA

    def hacer_pared(self):
        self.color = NEGRO

    def hacer_fin(self):
        self.color = PURPURA
        
    def hacer_actual(self):
        self.color = ROJO
    
    def hacer_blanco(self):
        self.color = BLANCO
        
    def hacer_camino(self):
        self.color = AZUL_TENUE
        
    def actualizar_vecinos(self, grid):
        self.vecinos = []
        
        # Verificar los 8 vecinos (4 cardinales y 4 diagonales)
        # Para cada vecino, guardamos el nodo y su costo de movimiento (10 para ortogonal, 14 para diagonal)
        
        # Definir los 8 posibles movimientos y sus costos
        direcciones = [
            # Cardinales (costo 10)
            (1, 0, 10),   # Abajo
            (-1, 0, 10),  # Arriba
            (0, 1, 10),   # Derecha
            (0, -1, 10),  # Izquierda
            
            # Diagonales (costo 14)
            (1, 1, 14),   # Abajo-Derecha
            (1, -1, 14),  # Abajo-Izquierda
            (-1, 1, 14),  # Arriba-Derecha
            (-1, -1, 14)  # Arriba-Izquierda
        ]
        
        for df, dc, costo in direcciones:
            nueva_fila = self.fila + df
            nueva_col = self.col + dc
            
            # Verificar si la nueva posición está dentro de los límites
            if (0 <= nueva_fila < self.total_filas and 
                0 <= nueva_col < self.total_filas and 
                not grid[nueva_fila][nueva_col].es_pared()):
                
                # Para diagonales, necesitamos verificar que no haya paredes en el camino
                if costo == 14:
                    # Verificar que no haya obstáculos en los nodos adyacentes a la diagonal
                    if (grid[self.fila][nueva_col].es_pared() or 
                        grid[nueva_fila][self.col].es_pared()):
                        continue
                
                self.vecinos.append((grid[nueva_fila][nueva_col], costo))

    def dibujar(self, ventana):
        pygame.draw.rect(ventana, self.color, (self.x, self.y, self.ancho, self.ancho))
        
        # Mostrar los valores de f, g, h en cada nodo
        if not self.es_pared() and not self.es_inicio() and not self.es_fin():
            font = pygame.font.SysFont('Arial', 10)
            
            # Solo mostrar valores finitos
            if self.f != float("inf"):
                texto_f = font.render(f"f:{int(self.f)}", True, NEGRO)
                ventana.blit(texto_f, (self.x + 2, self.y + 2))
            
            if self.g != float("inf"):
                texto_g = font.render(f"g:{int(self.g)}", True, NEGRO)
                ventana.blit(texto_g, (self.x + 2, self.y + self.ancho//3))
            
            if self.h > 0:
                texto_h = font.render(f"h:{int(self.h)}", True, NEGRO)
                ventana.blit(texto_h, (self.x + 2, self.y + 2*self.ancho//3))

def heuristica(p1, p2):
    # Distancia Manhattan multiplicada por 10 para mantener consistencia con los costos
    x1, y1 = p1
    x2, y2 = p2
    return 10 * (abs(x1 - x2) + abs(y1 - y2))
    
def reconstruir_camino(nodo_actual, dibujar_func):
    while nodo_actual.previo:
        nodo_actual = nodo_actual.previo
        if not nodo_actual.es_inicio():  # No colorear el nodo inicial
            nodo_actual.hacer_camino()
        dibujar_func()
        time.sleep(0.2)  # Delay corto para visualizar el camino

def algoritmo_astar(dibujar_func, grid, inicio, fin):
    contador = 0
    conjunto_abierto = PriorityQueue()
    conjunto_abierto.put((0, contador, inicio))
    nodos_abiertos = {inicio}
    nodos_cerrados = set()
    nodo_actual_temporal = None
    
    inicio.g = 0
    inicio.h = heuristica(inicio.get_pos(), fin.get_pos())
    inicio.f = inicio.g + inicio.h
    
    while not conjunto_abierto.empty():
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                
        # Restablecer el color del nodo anterior si existe
        if nodo_actual_temporal and nodo_actual_temporal != inicio and nodo_actual_temporal != fin:
            nodo_actual_temporal.hacer_blanco()
                
        # Obtener el nodo con la menor f
        _, _, nodo_actual = conjunto_abierto.get()
        nodos_abiertos.remove(nodo_actual)
        
        # Guardar referencia al nodo actual
        nodo_actual_temporal = nodo_actual
        
        # Colorear el nodo actual de rojo (solo si no es inicio ni fin)
        if nodo_actual != inicio and nodo_actual != fin:
            nodo_actual.hacer_actual()
            dibujar_func()
            time.sleep(0.5)  # Delay de 1 segundo para visualizar
        
        # Si encontramos el final
        if nodo_actual == fin:
            reconstruir_camino(nodo_actual, dibujar_func)
            fin.hacer_fin()
            inicio.hacer_inicio()
            return True
            
        # Examinar todos los vecinos
        for vecino, costo in nodo_actual.vecinos:
            # Calcular la nueva g usando el costo específico
            g_tentativa = nodo_actual.g + costo
            
            # Si encontramos un camino mejor
            if g_tentativa < vecino.g:
                vecino.previo = nodo_actual
                vecino.g = g_tentativa
                vecino.h = heuristica(vecino.get_pos(), fin.get_pos())
                vecino.f = vecino.g + vecino.h
                
                if vecino not in nodos_abiertos and vecino not in nodos_cerrados:
                    contador += 1
                    conjunto_abierto.put((vecino.f, contador, vecino))
                    nodos_abiertos.add(vecino)
        
        # Añadir nodo actual a cerrados
        nodos_cerrados.add(nodo_actual)
    
    return False

def crear_grid(filas, ancho):
    grid = []
    ancho_nodo = ancho // filas
    for i in range(filas):
        grid.append([])
        for j in range(filas):
            nodo = Nodo(i, j, ancho_nodo, filas)
            grid[i].append(nodo)
    return grid

def dibujar_grid(ventana, filas, ancho):
    ancho_nodo = ancho // filas
    for i in range(filas):
        pygame.draw.line(ventana, GRIS, (0, i * ancho_nodo), (ancho, i * ancho_nodo))
        for j in range(filas):
            pygame.draw.line(ventana, GRIS, (j * ancho_nodo, 0), (j * ancho_nodo, ancho))

def dibujar(ventana, grid, filas, ancho):
    ventana.fill(BLANCO)
    for fila in grid:
        for nodo in fila:
            nodo.dibujar(ventana)

    dibujar_grid(ventana, filas, ancho)
    pygame.display.update()

def obtener_click_pos(pos, filas, ancho):
    ancho_nodo = ancho // filas
    y, x = pos
    fila = y // ancho_nodo
    col = x // ancho_nodo
    return fila, col

def main(ventana, ancho):
    FILAS = 10
    grid = crear_grid(FILAS, ancho)

    inicio = None
    fin = None

    corriendo = True
    comenzado = False

    while corriendo:
        dibujar(ventana, grid, FILAS, ancho)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                corriendo = False

            # Si el algoritmo ya comenzó, no permitir más cambios
            if comenzado:
                continue

            if pygame.mouse.get_pressed()[0]:  # Click izquierdo
                pos = pygame.mouse.get_pos()
                fila, col = obtener_click_pos(pos, FILAS, ancho)
                nodo = grid[fila][col]
                if not inicio and nodo != fin:
                    inicio = nodo
                    inicio.hacer_inicio()

                elif not fin and nodo != inicio:
                    fin = nodo
                    fin.hacer_fin()

                elif nodo != fin and nodo != inicio:
                    nodo.hacer_pared()

            elif pygame.mouse.get_pressed()[2]:  # Click derecho
                pos = pygame.mouse.get_pos()
                fila, col = obtener_click_pos(pos, FILAS, ancho)
                nodo = grid[fila][col]
                nodo.restablecer()
                if nodo == inicio:
                    inicio = None
                elif nodo == fin:
                    fin = None
                    
            # Presionar R para comenzar el algoritmo
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r and inicio and fin and not comenzado:
                    # Actualizar los vecinos de todos los nodos
                    for fila in grid:
                        for nodo in fila:
                            nodo.actualizar_vecinos(grid)
                            
                    # Función lambda para pasar dibujar como una función
                    lambda_dibujar = lambda: dibujar(ventana, grid, FILAS, ancho)
                    
                    # Ejecutar el algoritmo A*
                    algoritmo_astar(lambda_dibujar, grid, inicio, fin)
                    comenzado = True
                    
                # Presionar C para limpiar el tablero
                if event.key == pygame.K_c:
                    inicio = None
                    fin = None
                    grid = crear_grid(FILAS, ancho)
                    comenzado = False

    pygame.quit()

main(VENTANA, ANCHO_VENTANA)
```