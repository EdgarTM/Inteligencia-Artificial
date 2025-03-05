```python
import pygame
from queue import PriorityQueue
import math

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

    def es_pared(self):
        return self.color == NEGRO

    def es_inicio(self):
        return self.color == NARANJA

    def es_fin(self):
        return self.color == PURPURA
        
    def es_abierto(self):
        return self.color == VERDE
        
    def es_cerrado(self):
        return self.color == ROJO
        
    def es_camino(self):
        return self.color == AZUL

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
        
    def hacer_abierto(self):
        self.color = VERDE
    
    def hacer_cerrado(self):
        self.color = ROJO
        
    def hacer_camino(self):
        self.color = AZUL
        
    def actualizar_vecinos(self, grid):
        self.vecinos = []
        
        # Verificar los 8 vecinos (4 cardinales y 4 diagonales)
        # Para cada vecino, guardamos el nodo y su costo de movimiento (10 para ortogonal, 14 para diagonal)
        
        # Abajo (costo 10)
        if self.fila < self.total_filas - 1 and not grid[self.fila + 1][self.col].es_pared():
            self.vecinos.append((grid[self.fila + 1][self.col], 10))
            
        # Arriba (costo 10)
        if self.fila > 0 and not grid[self.fila - 1][self.col].es_pared():
            self.vecinos.append((grid[self.fila - 1][self.col], 10))
            
        # Derecha (costo 10)
        if self.col < self.total_filas - 1 and not grid[self.fila][self.col + 1].es_pared():
            self.vecinos.append((grid[self.fila][self.col + 1], 10))
            
        # Izquierda (costo 10)
        if self.col > 0 and not grid[self.fila][self.col - 1].es_pared():
            self.vecinos.append((grid[self.fila][self.col - 1], 10))

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
        nodo_actual.hacer_camino()
        dibujar_func()

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
                    
            # Presionar ESPACIO para comenzar el algoritmo
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and inicio and fin and not comenzado:
                    for fila in grid:
                        for nodo in fila:
                            nodo.actualizar_vecinos(grid)
                            
                    lambda_dibujar = lambda: dibujar(ventana, grid, FILAS, ancho)
                    
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
