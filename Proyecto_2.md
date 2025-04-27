# Implementacion del modo auto en el juego de Pygames

**Enlace al codigo proporcionado como base:** [DropBox](https://www.dropbox.com/scl/fo/tf9swvr39olsxnluw162j/h?dl=0&e=1&st=4761c79w)

### 1. Creación y Entrenamiento del Modelo

```python
# Inicializar el modelo de red neuronal
modelo = MLPClassifier(hidden_layer_sizes=(10, 5), max_iter=1000)

# Entrenar el modelo con datos recopilados
modelo.fit(X, y)

# Guardar el modelo entrenado
joblib.dump(modelo, 'modelo_salto.pkl')
```

El modelo utilizado es un `MLPClassifier` (Multi-Layer Perceptron Classifier) de scikit-learn con dos capas ocultas de 10 y 5 neuronas respectivamente. Una vez entrenado, se guarda utilizando la función `joblib.dump()` en un archivo llamado 'modelo_salto.pkl'.

### 2. Recopilación de Datos para Entrenamiento

```python
# Estructura de datos para almacenar información del juego
datos_modelo = []

# Función para guardar datos en modo manual
def guardar_datos():
    distancia = abs(jugador.x - bala.x)
    salto_hecho = 1 if salto else 0  # 1 si saltó, 0 si no saltó
    # Guardar velocidad de la bala, distancia al jugador y si saltó o no
    datos_modelo.append((velocidad_bala, distancia, salto_hecho))
```

Los datos son recopilados durante el juego en modo manual, registrando:
- La velocidad de la bala
- La distancia entre el jugador y la bala
- Si el jugador saltó o no (1 o 0)

Estos datos servirán para que el modelo aprenda en qué momentos debe saltar.

### 3. Preparación de Datos para Entrenamiento

```python
# Preparar características (X) y etiquetas (y) para entrenamiento
X = np.array([[dato[0], dato[1]] for dato in datos_modelo])  # Características: velocidad y distancia
y = np.array([dato[2] for dato in datos_modelo])  # Etiqueta: saltó o no saltó
```

Antes de entrenar el modelo, se preparan los datos:
- **X**: matriz con las características (velocidad y distancia)
- **y**: vector con las etiquetas (si saltó o no)

### 4. Carga del Modelo

```python
# Función para cargar un modelo previamente entrenado
def cargar_modelo():
    global modelo
    try:
        if os.path.exists('modelo_salto.pkl'):
            modelo = joblib.load('modelo_salto.pkl')
            print("Modelo cargado con éxito")
            return True
        else:
            print("No se encontró un modelo guardado")
            return False
    except Exception as e:
        print(f"Error al cargar el modelo: {e}")
        return False
```

Al iniciar el juego, se intenta cargar un modelo previamente entrenado desde el archivo 'modelo_salto.pkl' utilizando `joblib.load()`. Si el archivo no existe, se informa al usuario y se procede sin modelo.

### 5. Realización de Predicciones con el Modelo

```python
def predecir_salto(velocidad, distancia):
    global modelo
    if modelo is not None:
        # Preparar los datos para la predicción
        X = np.array([[velocidad, distancia]])
        # Hacer la predicción
        prediccion = modelo.predict(X)[0]
        # Devolver True si el modelo predice que debería saltar (1), False en caso contrario (0)
        return prediccion == 1
    return False
```

En modo automático, el modelo cargado toma decisiones de salto basándose en la velocidad de la bala y la distancia al jugador, devolviendo True cuando predice que se debe saltar.

### 6. Flujo de Entrenamiento del Modelo

El entrenamiento ocurre cuando:
1. El juego termina por una colisión
2. Se han recopilado suficientes datos en modo manual
3. Se llama a la función `entrenar_modelo()`:

```python
def entrenar_modelo():
    global datos_modelo, modelo
    if len(datos_modelo) < 10:  # Necesitamos al menos algunos datos para entrenar
        print("No hay suficientes datos para entrenar el modelo.")
        return False
    
    # Preparar los datos para el entrenamiento
    X = np.array([[dato[0], dato[1]] for dato in datos_modelo])  # Características: velocidad y distancia
    y = np.array([dato[2] for dato in datos_modelo])  # Etiqueta: saltó o no saltó
    
    # Inicializar y entrenar el modelo
    modelo = MLPClassifier(hidden_layer_sizes=(10, 5), max_iter=1000)
    modelo.fit(X, y)
    
    # Guardar el modelo entrenado
    joblib.dump(modelo, 'modelo_salto.pkl')
    print(f"Modelo entrenado con {len(datos_modelo)} ejemplos y guardado como 'modelo_salto.pkl'")
    return True
```

### Código Completo

```python
import pygame
import random
import numpy as np
from sklearn.neural_network import MLPClassifier
import joblib
import os

# Inicializar Pygame
pygame.init()

# Dimensiones de la pantalla
w, h = 800, 400
pantalla = pygame.display.set_mode((w, h))
pygame.display.set_caption("Juego: Disparo de Bala, Salto, Nave y Menú")

# Colores
BLANCO = (255, 255, 255)
NEGRO = (0, 0, 0)

# Variables del jugador, bala, nave, fondo, etc.
jugador = None
bala = None
fondo = None
nave = None
menu = None

# Variables de salto
salto = False
salto_altura = 15  # Velocidad inicial de salto
gravedad = 2
en_suelo = True

# Variables de pausa y menú
pausa = False
fuente = pygame.font.SysFont('Arial', 24)
menu_activo = True
modo_auto = False  # Indica si el modo de juego es automático

# Lista para guardar los datos de velocidad, distancia y salto (target)
datos_modelo = []

# Red neuronal
modelo = None

# Cargar las imágenes
jugador_frames = [
    pygame.image.load('assets/minita/fila-1-columna-1.png'),
    pygame.image.load('assets/minita/fila-1-columna-2.png'),
    pygame.image.load('assets/minita/fila-1-columna-3.png'),
    pygame.image.load('assets/minita/fila-1-columna-4.png'),
    pygame.image.load('assets/minita/fila-1-columna-5.png'),
    pygame.image.load('assets/minita/fila-1-columna-6.png'),
    pygame.image.load('assets/minita/fila-1-columna-7.png'),
    pygame.image.load('assets/minita/fila-1-columna-8.png'),
    pygame.image.load('assets/minita/fila-1-columna-9.png'),
    pygame.image.load('assets/minita/fila-1-columna-10.png'),
    pygame.image.load('assets/minita/fila-1-columna-11.png'),
    pygame.image.load('assets/minita/fila-1-columna-12.png'),
]

bala_img = pygame.image.load('assets/sprites/purple_ball.png')
fondo_img = pygame.image.load('assets/game/fondo2.png')
nave_img = pygame.image.load('assets/game/ufo.png')
menu_img = pygame.image.load('assets/game/menu.png')

# Escalar la imagen de fondo para que coincida con el tamaño de la pantalla
fondo_img = pygame.transform.scale(fondo_img, (w, h))

# Crear el rectángulo del jugador y de la bala
jugador = pygame.Rect(50, h - 100, 32, 48)
bala = pygame.Rect(w - 50, h - 90, 16, 16)
nave = pygame.Rect(w - 100, h - 100, 64, 64)
menu_rect = pygame.Rect(w // 2 - 135, h // 2 - 90, 270, 180)  # Tamaño del menú

# Variables para la animación del jugador
current_frame = 0
frame_speed = 2  # Cuántos frames antes de cambiar a la siguiente imagen
frame_count = 0

# Variables para la bala
velocidad_bala = -10  # Velocidad de la bala hacia la izquierda
bala_disparada = False

# Variables para el fondo en movimiento
fondo_x1 = 0
fondo_x2 = w

# Función para disparar la bala
def disparar_bala():
    global bala_disparada, velocidad_bala
    if not bala_disparada:
        velocidad_bala = random.randint(-30, -10)  # Velocidad aleatoria negativa para la bala
        bala_disparada = True

# Función para reiniciar la posición de la bala
def reset_bala():
    global bala, bala_disparada
    bala.x = w - 50  # Reiniciar la posición de la bala
    bala_disparada = False

# Función para manejar el salto
def manejar_salto():
    global jugador, salto, salto_altura, gravedad, en_suelo

    if salto:
        jugador.y -= salto_altura  # Mover al jugador hacia arriba
        salto_altura -= gravedad  # Aplicar gravedad (reduce la velocidad del salto)

        # Si el jugador llega al suelo, detener el salto
        if jugador.y >= h - 100:
            jugador.y = h - 100
            salto = False
            salto_altura = 15  # Restablecer la velocidad de salto
            en_suelo = True

# Función para actualizar el juego
def update():
    global bala, velocidad_bala, current_frame, frame_count, fondo_x1, fondo_x2

    # Mover el fondo
    fondo_x1 -= 1
    fondo_x2 -= 1

    # Si el primer fondo sale de la pantalla, lo movemos detrás del segundo
    if fondo_x1 <= -w:
        fondo_x1 = w

    # Si el segundo fondo sale de la pantalla, lo movemos detrás del primero
    if fondo_x2 <= -w:
        fondo_x2 = w

    # Dibujar los fondos
    pantalla.blit(fondo_img, (fondo_x1, 0))
    pantalla.blit(fondo_img, (fondo_x2, 0))

    # Animación del jugador
    frame_count += 1
    if frame_count >= frame_speed:
        current_frame = (current_frame + 1) % len(jugador_frames)
        frame_count = 0

    # Dibujar el jugador con la animación
    pantalla.blit(jugador_frames[current_frame], (jugador.x, jugador.y))

    # Dibujar la nave
    pantalla.blit(nave_img, (nave.x, nave.y))

    # Mover y dibujar la bala
    if bala_disparada:
        bala.x += velocidad_bala

    # Si la bala sale de la pantalla, reiniciar su posición
    if bala.x < 0:
        reset_bala()

    pantalla.blit(bala_img, (bala.x, bala.y))

    # Colisión entre la bala y el jugador
    if jugador.colliderect(bala):
        print("Colisión detectada!")
        reiniciar_juego()  # Terminar el juego y mostrar el menú

# Función para guardar datos del modelo en modo manual
def guardar_datos():
    global jugador, bala, velocidad_bala, salto
    distancia = abs(jugador.x - bala.x)
    salto_hecho = 1 if salto else 0  # 1 si saltó, 0 si no saltó
    # Guardar velocidad de la bala, distancia al jugador y si saltó o no
    datos_modelo.append((velocidad_bala, distancia, salto_hecho))

# Función para pausar el juego y guardar los datos
def pausa_juego():
    global pausa
    pausa = not pausa
    if pausa:
        print("Juego pausado. Datos registrados hasta ahora:", datos_modelo)
    else:
        print("Juego reanudado.")

# Función para mostrar el menú y seleccionar el modo de juego
def mostrar_menu():
    global menu_activo, modo_auto
    pantalla.fill(NEGRO)
    texto = fuente.render("Presiona 'A' para Auto, 'M' para Manual, o 'Q' para Salir", True, BLANCO)
    pantalla.blit(texto, (w // 4, h // 2))
    pygame.display.flip()

    while menu_activo:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                exit()
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_a:
                    modo_auto = True
                    menu_activo = False
                elif evento.key == pygame.K_m:
                    modo_auto = False
                    menu_activo = False
                elif evento.key == pygame.K_q:
                    print("Juego terminado. Datos recopilados:", datos_modelo)
                    pygame.quit()
                    exit()

# Función para entrenar el modelo con los datos recopilados
def entrenar_modelo():
    global datos_modelo, modelo
    if len(datos_modelo) < 10:  # Necesitamos al menos algunos datos para entrenar
        print("No hay suficientes datos para entrenar el modelo.")
        return False
    
    # Preparar los datos para el entrenamiento
    X = np.array([[dato[0], dato[1]] for dato in datos_modelo])  # Características: velocidad y distancia
    y = np.array([dato[2] for dato in datos_modelo])  # Etiqueta: saltó o no saltó
    
    # Inicializar y entrenar el modelo
    modelo = MLPClassifier(hidden_layer_sizes=(10, 5), max_iter=1000)
    modelo.fit(X, y)
    
    # Guardar el modelo entrenado
    joblib.dump(modelo, 'modelo_salto.pkl')
    print(f"Modelo entrenado con {len(datos_modelo)} ejemplos y guardado como 'modelo_salto.pkl'")
    return True

# Función para cargar un modelo previamente entrenado
def cargar_modelo():
    global modelo
    try:
        if os.path.exists('modelo_salto.pkl'):
            modelo = joblib.load('modelo_salto.pkl')
            print("Modelo cargado con éxito")
            return True
        else:
            print("No se encontró un modelo guardado")
            return False
    except Exception as e:
        print(f"Error al cargar el modelo: {e}")
        return False

# Función para hacer predicciones con el modelo
def predecir_salto(velocidad, distancia):
    global modelo
    if modelo is not None:
        # Preparar los datos para la predicción
        X = np.array([[velocidad, distancia]])
        # Hacer la predicción
        prediccion = modelo.predict(X)[0]
        # Devolver True si el modelo predice que debería saltar (1), False en caso contrario (0)
        return prediccion == 1
    return False

# Función para reiniciar el juego tras la colisión
def reiniciar_juego():
    global menu_activo, jugador, bala, nave, bala_disparada, salto, en_suelo, datos_modelo
    
    # Entrenar el modelo con los datos recopilados si estamos en modo manual
    if not modo_auto and len(datos_modelo) > 0:
        exito = entrenar_modelo()
        if exito:
            print("Modelo entrenado correctamente tras la colisión")
    
    menu_activo = True  # Activar de nuevo el menú
    jugador.x, jugador.y = 50, h - 100  # Reiniciar posición del jugador
    bala.x = w - 50  # Reiniciar posición de la bala
    nave.x, nave.y = w - 100, h - 100  # Reiniciar posición de la nave
    bala_disparada = False
    salto = False
    en_suelo = True
    # Mostrar los datos recopilados hasta el momento
    print("Datos recopilados para el modelo: ", datos_modelo)
    mostrar_menu()  # Mostrar el menú de nuevo para seleccionar modo

def main():
    global salto, en_suelo, bala_disparada, modo_auto

    reloj = pygame.time.Clock()
    
    # Intentar cargar un modelo previamente entrenado
    cargar_modelo()
    
    mostrar_menu()  # Mostrar el menú al inicio
    correr = True

    while correr:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                correr = False
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE and en_suelo and not pausa:  # Detectar la tecla espacio para saltar
                    salto = True
                    en_suelo = False
                if evento.key == pygame.K_p:  # Presiona 'p' para pausar el juego
                    pausa_juego()
                if evento.key == pygame.K_q:  # Presiona 'q' para terminar el juego
                    print("Juego terminado. Datos recopilados:", datos_modelo)
                    pygame.quit()
                    exit()

        if not pausa:
            # Modo manual: el jugador controla el salto
            if not modo_auto:
                if salto:
                    manejar_salto()
                # Guardar los datos si estamos en modo manual
                guardar_datos()
            # Modo automático: la IA controla el salto
            else:
                # Obtener la velocidad de la bala y la distancia al jugador
                distancia = abs(jugador.x - bala.x)
                # Hacer una predicción con el modelo
                if en_suelo and predecir_salto(velocidad_bala, distancia):
                    salto = True
                    en_suelo = False
                    print(f"Salta con velocidad={velocidad_bala}, distancia={distancia}")
                if salto:
                    manejar_salto()

            # Actualizar el juego
            if not bala_disparada:
                disparar_bala()
            update()

        # Actualizar la pantalla
        pygame.display.flip()
        reloj.tick(30)  # Limitar el juego a 30 FPS

    pygame.quit()

if __name__ == "__main__":
    main()

```