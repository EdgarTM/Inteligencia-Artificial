# Implementacion del modo auto en el juego de Pygames

**Enlace al codigo proporcionado como base:** [DropBox](https://www.dropbox.com/scl/fo/tf9swvr39olsxnluw162j/h?dl=0&e=1&st=4761c79w)

## 1. Sistema Central de Disparos

### Lógica de Disparo Horizontal

```python
def disparar_bala():
    global bala_disparada, velocidad_bala
    if not bala_disparada:  # Solo si no hay bala activa
        velocidad_bala = random.randint(-20, -18)  # Velocidad aleatoria
        bala_disparada = True  # Bandera de control
```

Este fragmento implementa un sistema de disparo con las siguientes características:

- **Control de estado único**: `bala_disparada` actúa como semáforo para evitar múltiples balas simultáneas
- **Velocidad variable**: `random.randint(-20, -18)` genera velocidades entre -20 y -18 píxeles por frame, creando variabilidad en la dificultad
- **Dirección fija**: Los valores negativos indican movimiento hacia la izquierda (hacia el jugador)
- **Activación inmediata**: Una vez llamada, la función cambia el estado global del juego

**Mecánica del sistema**: La bala se mueve horizontalmente a través de la pantalla, y solo se puede disparar una nueva cuando la anterior haya sido procesada (colisión o salida de pantalla).

### Reinicio de Bala

```python
def reset_bala():
    global bala, bala_disparada
    bala.x = w - 50  # Posición inicial en el extremo derecho
    bala_disparada = False  # Permite nuevo disparo
```

Esta función es crucial para el ciclo de vida de las balas:

- **Reposicionamiento**: `w - 50` coloca la bala 50 píxeles antes del borde derecho de la pantalla
- **Reset de estado**: `bala_disparada = False` libera el sistema para permitir un nuevo disparo
- **Preparación para siguiente ciclo**: La bala queda lista para ser disparada nuevamente

**Cuándo se ejecuta**: Esta función se llama cuando la bala sale de la pantalla por el lado izquierdo o cuando colisiona con el jugador, preparando el sistema para el siguiente disparo.

## 2. Física de Movimiento del Personaje

### Sistema de Salto con Gravedad

```python
def manejar_salto():
    global jugador, salto, salto_altura, gravedad, en_suelo

    if salto:
        jugador.y -= salto_altura  # Movimiento vertical
        salto_altura -= gravedad  # Aplicación de gravedad

        # Condición de aterrizaje
        if jugador.y >= posicion_original_y:
            jugador.y = posicion_original_y  # Ajuste preciso
            salto = False
            salto_altura = 15  # Reset altura salto
            en_suelo = True  # Flag para nuevo salto
```

Este sistema implementa una física de salto realista con los siguientes componentes:

**Física del movimiento**:
- `jugador.y -= salto_altura`: Mueve el jugador hacia arriba (coordenadas Y negativas)
- `salto_altura -= gravedad`: Reduce la velocidad vertical aplicando gravedad en cada frame
- La gravedad actúa como aceleración constante hacia abajo

**Mecánica del salto**:
1. **Fase ascendente**: `salto_altura` comienza en 15 y disminuye gradualmente
2. **Punto máximo**: Cuando `salto_altura` llega a 0, el jugador alcanza la altura máxima
3. **Fase descendente**: `salto_altura` se vuelve negativa, moviendo al jugador hacia abajo
4. **Aterrizaje**: Al llegar a `posicion_original_y`, se resetean todas las variables

**Control de estado**: `en_suelo` previene saltos dobles y asegura que solo se pueda saltar desde el suelo.

### Movimiento Lateral Complejo

```python
def manejar_desplazamiento():
    global jugador, izquierda, desplazamiento, velocidad_dezplazamiento
    
    if izquierda:
        if ida and jugador.x > posicion_original_x - desplazamiento:
            jugador.x -= velocidad_dezplazamiento
            if jugador.x <= posicion_original_x - desplazamiento:
                ida = False
                vuelta = True
        elif vuelta and jugador.x < posicion_original_x:
            jugador.x += velocidad_dezplazamiento
            if jugador.x >= posicion_original_x:
                jugador.x = posicion_original_x
                izquierda = False
                sin_movimiento = True
```

Este sistema usa tres estados:

- **ida**: Movimiento hacia izquierda hasta límite - El jugador se desplaza desde su posición original hacia la izquierda hasta alcanzar `posicion_original_x - desplazamiento`
- **vuelta**: Regreso a posición original - Una vez alcanzado el límite izquierdo, el jugador regresa gradualmente a su posición inicial
- **sin_movimiento**: Estado de reposo - El jugador permanece en su posición original hasta que se active otro movimiento

**Mecánica detallada del movimiento**:

**Fase IDA**:
```python
if ida and jugador.x > posicion_original_x - desplazamiento:
    jugador.x -= velocidad_dezplazamiento  # Mover hacia izquierda
```
- Se ejecuta mientras el jugador no haya alcanzado el límite izquierdo
- `velocidad_dezplazamiento` controla qué tan rápido se mueve (típicamente 2-5 píxeles por frame)
- Al llegar al límite, se activa automáticamente la fase de vuelta

**Fase VUELTA**:
```python
elif vuelta and jugador.x < posicion_original_x:
    jugador.x += velocidad_dezplazamiento  # Mover hacia derecha
```
- Se ejecuta hasta que el jugador regrese a su posición original
- Usa la misma velocidad pero en dirección opuesta
- Garantiza retorno preciso a la posición inicial

**Transición de estados**:
```
sin_movimiento → [trigger] → izquierda=True, ida=True
ida → [límite alcanzado] → ida=False, vuelta=True  
vuelta → [posición original] → izquierda=False, sin_movimiento=True
```

**Uso estratégico**: Este movimiento permite esquivar proyectiles que vienen en línea recta, creando una ventana temporal donde el jugador no está en la trayectoria de impacto.

## 3. Sistema de Machine Learning Avanzado

### Preprocesamiento Inteligente de Datos

```python
def crear_caracteristicas_expandidas(distancia_bala, velocidad_bala, distancia_bala2):
    try:
        # Cálculo de tiempo estimado de colisión
        tiempo_colision_bala = distancia_bala / abs(velocidad_bala) if velocidad_bala != 0 else 1000
        
        # Características no lineales
        features = [
            distancia_bala,
            np.log1p(distancia_bala),  # Transformación logarítmica
            distancia_bala * velocidad_bala,  # Interacción entre variables
            distancia_bala2**2  # Término cuadrático
        ]
        return features
    except Exception as e:
        print(f"Error creando características: {e}")
        return [distancia_bala, velocidad_bala, distancia_bala2, 100]  # Valores por defecto
```

Esta función implementa **feature engineering** avanzado para mejorar el rendimiento del modelo de ML:

**Características generadas**:
1. **`distancia_bala`**: Distancia directa entre jugador y proyectil
2. **`np.log1p(distancia_bala)`**: Transformación logarítmica que comprime valores grandes y amplifica diferencias en distancias cortas (críticas para decisiones de salto)
3. **`distancia_bala * velocidad_bala`**: Feature de interacción que captura la urgencia de la amenaza (distancia × velocidad = tiempo hasta impacto)
4. **`distancia_bala2**2`**: Término cuadrático que permite al modelo aprender patrones no lineales

**Ventajas del preprocesamiento**:
- **Mejor representación**: Las características no lineales ayudan al modelo a captar patrones complejos
- **Escalas diferentes**: `log1p` maneja mejor las variaciones en distancias cortas vs largas
- **Robustez**: El bloque try-except previene fallos por división por cero o valores inválidos

**Tiempo de colisión**: `tiempo_colision_bala` calcula cuántos frames faltan para el impacto, información crucial para timing de saltos.

### Implementación de Red Neuronal Adaptativa

```python
def enRedNeural():
    global nnNetwork
    # Configuración dinámica según cantidad de datos
    if n_samples < 10:
        hidden_layers = (8, 4)
        max_iter = 200
    elif n_samples < 50:
        hidden_layers = (16, 8)
        max_iter = 300
    
    mlp = MLPClassifier(
        hidden_layer_sizes=hidden_layers,
        activation='relu',
        early_stopping=True if n_samples > 50 else False,
        max_iter=max_iter
    )
    
    # Pipeline completo con escalado
    nnNetwork = Pipeline([
        ('scaler', RobustScaler()),
        ('mlp', mlp)
    ])
    nnNetwork.fit(X, y)
```

Esta implementación utiliza un **diseño adaptativo** que ajusta la arquitectura según la cantidad de datos disponibles:

**Configuración dinámica de arquitectura**:
- **Pocos datos (< 10 samples)**: Red pequeña `(8, 4)` para evitar overfitting
- **Datos moderados (< 50 samples)**: Red mediana `(16, 8)` con más capacidad
- **Muchos datos (≥ 50 samples)**: Configuración más compleja con early stopping

**Componentes clave**:
1. **MLPClassifier**: Red neuronal multicapa con activación ReLU
2. **RobustScaler**: Escalado robusto que maneja outliers mejor que StandardScaler
3. **Pipeline**: Encapsula preprocesamiento y modelo en una sola entidad
4. **Early Stopping**: Previene overfitting deteniéndose cuando la validación no mejora

**Ventajas del diseño**:
- **Escalabilidad**: Se adapta automáticamente al volumen de datos
- **Robustez**: El escalado robusto maneja datos atípicos
- **Eficiencia**: Early stopping evita entrenamientos innecesarios
- **Simplicidad**: Pipeline unifica todo el flujo de procesamiento

**Proceso de entrenamiento**: La red aprende a mapear estados del juego (posiciones, velocidades) a acciones óptimas (saltar/no saltar, moverse/no moverse).

## 4. Sistema de Predicción en Tiempo Real

### Lógica de Predicción con Respaldo

```python
def predecirConRedNeuronal(entrada_basica):
    if nnNetwork is None:
        # Lógica de emergencia basada en reglas simples
        distancia_bala = entrada_basica[0]
        return distancia_bala < 100, False  # Salta si bala cerca
    
    try:
        prediccion = nnNetwork.predict([entrada_basica])
        return prediccion[0][0] == 1, prediccion[0][1] == 1
    except Exception as e:
        print(f"Error en predicción: {e}")
        return False, False  # Predicción conservadora
```

Esta función implementa un **sistema de respaldo robusto** con múltiples niveles de seguridad:

**Sistema de respaldo (Fallback)**:
- **Nivel 1**: Si no hay modelo entrenado (`nnNetwork is None`), usa lógica simple basada en reglas
- **Regla de emergencia**: Salta cuando la bala está a menos de 100 píxeles de distancia
- **Comportamiento conservador**: Solo activa salto, nunca movimiento lateral (más seguro)

**Predicción con modelo entrenado**:
- **Input**: `entrada_basica` contiene las características del estado actual del juego
- **Output**: Tupla de dos booleanos `(saltar, mover_izquierda)`
- **Decodificación**: `prediccion[0][0] == 1` convierte la salida numérica a acción booleana

**Manejo de errores**:
- **Try-catch**: Captura cualquier error durante la predicción (modelo corrupto, datos inválidos, etc.)
- **Respuesta conservadora**: En caso de error, devuelve `(False, False)` - no hacer nada es más seguro que una acción errónea
- **Logging**: Imprime errores para debugging sin romper el juego

**Flujo de decisión**:
1. ¿Existe modelo? → No: usar regla simple
2. ¿Predicción exitosa? → No: modo conservador
3. ¿Output válido? → Sí: ejecutar acción predicha

Esta arquitectura asegura que el juego nunca se cuelgue por fallos del ML, manteniendo siempre una funcionalidad básica.

## 5. Gestión de Estados del Juego

### Sistema Completo de Reinicio

```python
def reiniciar_juego():
    global jugador, bala, bala_disparada, salto, en_suelo
    
    # Reset posiciones
    jugador.x, jugador.y = posicion_original_x, posicion_original_y
    bala.x, bala.y = w - 50, h - 90
    
    # Reset estados
    bala_disparada = False
    salto = False
    en_suelo = True
    
    # Guardado automático de datos si es modo manual
    if not modo_auto and datos_modelo:
        guardar_modelos_automaticamente()
        datos_modelo.clear()
```

Esta función orchestrata un **reinicio completo y consistente** del estado del juego:

**Reinicio de posiciones**:
- **Jugador**: Vuelve a `(posicion_original_x, posicion_original_y)` - posición inicial segura
- **Bala**: Se coloca en `(w - 50, h - 90)` - fuera de pantalla, lista para próximo disparo

**Reinicio de estados físicos**:
- **`bala_disparada = False`**: Libera el sistema de disparo para nuevas balas
- **`salto = False`**: Cancela cualquier salto en progreso
- **`en_suelo = True`**: Confirma que el jugador está en posición de salto válida

**Gestión inteligente de datos**:
- **Condición**: Solo si está en modo manual (`not modo_auto`) Y hay datos recopilados
- **Persistencia**: `guardar_modelos_automaticamente()` salva el aprendizaje acumulado
- **Limpieza**: `datos_modelo.clear()` libera memoria para la siguiente sesión

**Cuándo se ejecuta**:
- Después de una colisión (game over)
- Al cambiar entre modos manual/automático
- Al reiniciar manualmente el juego

**Consistencia del estado**: Esta función garantiza que no queden estados inconsistentes que puedan causar bugs (ej: jugador saltando + en_suelo = True).

## 6. Loop Principal del Juego con Control Dual

### Estructura Principal

```python
def main():
    while correr:
        # 1. Manejo de eventos
        for evento in pygame.event.get():
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE and en_suelo:
                    salto = True
                    en_suelo = False
        
        # 2. Lógica de modo automático
        if modo_auto:
            entrada = [abs(jugador.x - bala.x), velocidad_bala]
            saltar, mover_izquierda = predecirConRedNeuronal(entrada)
            
            if saltar and not salto and en_suelo:
                salto = True
                en_suelo = False
        
        # 3. Actualización física
        if salto:
            manejar_salto()
        
        # 4. Renderizado
        update()
        pygame.display.flip()
        reloj.tick(30)  # 30 FPS
```

Este es el **corazón del juego** que implementa el loop principal con arquitectura modular:

**1. Manejo de eventos (Input del usuario)**:
- **Captura de teclas**: `pygame.event.get()` procesa todos los eventos pendientes
- **Control de salto**: `pygame.K_SPACE` activa salto solo si `en_suelo = True`
- **Prevención de saltos múltiples**: La condición `and en_suelo` evita saltos en el aire

**2. Lógica de modo automático (IA)**:
- **Entrada del modelo**: `[abs(jugador.x - bala.x), velocidad_bala]` representa el estado actual
- **Predicción**: La IA decide las acciones basándose en el estado
- **Ejecución condicionada**: Solo ejecuta si las condiciones físicas lo permiten (mismo sistema que control manual)

**3. Actualización física (Game Logic)**:
- **Procesamiento de salto**: `manejar_salto()` aplica física de movimiento vertical
- **Otros sistemas**: Aquí se procesarían movimientos laterales, colisiones, etc.

**4. Renderizado (Graphics)**:
- **`update()`**: Dibuja todos los elementos en pantalla
- **`pygame.display.flip()`**: Actualiza la pantalla con los cambios
- **`reloj.tick(30)`**: Mantiene framerate constante a 30 FPS

**Arquitectura híbrida**:
- **Dual input**: Acepta tanto input humano como decisiones de IA
- **Física unificada**: Ambos modos usan la misma física, garantizando consistencia
- **Separación de responsabilidades**: Cada fase del loop tiene una función específica

**Game Loop pattern**: Este patrón estándar (Input → Logic → Render → Repeat) asegura un flujo predecible y mantenible.

## Estructuras Clave del Sistema

### 1. Máquina de Estados del Jugador

```python
# Variables de estado globales
salto = False
en_suelo = True
izquierda = False
sin_movimiento = True
ida = False
vuelta = False
```

Esta estructura implementa una **máquina de estados finitos** para controlar el comportamiento del jugador:

**Estados de movimiento vertical**:
- **`salto`**: Indica si el jugador está en proceso de salto
- **`en_suelo`**: Flag de seguridad que previene saltos múltiples

**Estados de movimiento horizontal**:
- **`izquierda`**: Estado maestro del movimiento lateral
- **`sin_movimiento`**: Estado de reposo (posición original)
- **`ida`**: Sub-estado de movimiento hacia la izquierda
- **`vuelta`**: Sub-estado de regreso a posición original

**Transiciones de estado**:
```
sin_movimiento → izquierda → ida → vuelta → sin_movimiento
      ↑                                        ↓
      ←─────────────── (ciclo completo) ───────←
```

**Ventajas del diseño**:
- **Exclusividad**: Solo un estado puede estar activo por categoría
- **Predictibilidad**: Las transiciones están claramente definidas
- **Debugging**: Fácil identificar el estado actual del jugador
- **Escalabilidad**: Sencillo agregar nuevos estados (ej: agacharse, correr)

### 2. Estructura de Datos para ML

```python
datos_modelo = []  # Lista de diccionarios

# Cada entrada contiene:
{
    'input': [
        distancia_bala, 
        velocidad_bala,
        distancia_bala2
    ],
    'output': [
        1,  # 1 si saltó, 0 si no
        0   # 1 si movió izquierda, 0 si no
    ]
}
```

Esta estructura implementa un **dataset supervisado** para entrenar los modelos de machine learning:

**Formato de datos**:
- **`datos_modelo`**: Lista que almacena todas las experiencias de juego
- **Cada entrada**: Diccionario con pares input-output para aprendizaje supervisado

**Features de entrada (input)**:
1. **`distancia_bala`**: Distancia horizontal entre jugador y proyectil
2. **`velocidad_bala`**: Velocidad actual del proyectil (información temporal)
3. **`distancia_bala2`**: Posible segunda bala o proyectil alternativo

**Labels de salida (output)**:
- **Salto**: `[1, 0]` = saltó, `[0, 0]` = no saltó
- **Movimiento**: `[0, 1]` = se movió izquierda, `[0, 0]` = no se movió
- **Combinado**: `[1, 1]` = saltó Y se movió

**Proceso de recolección**:
1. **Captura de estado**: Se registra la situación antes de la acción del jugador
2. **Registro de acción**: Se guarda qué hizo el jugador en esa situación
3. **Almacenamiento**: Se añade el par (estado, acción) al dataset
4. **Entrenamiento**: Los modelos aprenden a mapear estados → acciones óptimas

**Ventajas del formato**:
- **Supervisado**: Permite aprendizaje basado en ejemplos humanos
- **Multi-output**: Un modelo puede predecir múltiples acciones simultáneamente
- **Escalable**: Fácil agregar nuevas features o acciones
- **Interpretable**: Los datos son legibles y debuggeables

### 3. Gestión de Modelos

```python
modelo_estado = {
    'neural': 'no_entrenado',
    'tree': 'no_entrenado',
    'knn': 'no_entrenado'
}

modelos_entrenando = {
    'neural': False,
    'tree': False,
    'knn': False
}
```

Este sistema implementa **gestión de múltiples modelos de ML** con control de estado avanzado:

**Estados de los modelos (`modelo_estado`)**:
- **`'no_entrenado'`**: Modelo sin inicializar o datos insuficientes
- **`'entrenado'`**: Modelo listo para hacer predicciones
- **`'error'`**: Modelo que falló durante entrenamiento
- **`'actualizando'`**: Modelo en proceso de reentrenamiento

**Control de procesos (`modelos_entrenando`)**:
- **Prevención de concurrencia**: Evita entrenar el mismo modelo múltiples veces simultáneamente
- **Gestión de recursos**: Controla qué modelos están usando CPU
- **Estado de UI**: Permite mostrar indicadores de "entrenando..." en la interfaz

**Tipos de modelos soportados**:
1. **`'neural'`**: Red neuronal (MLPClassifier) - Mejor para patrones complejos
2. **`'tree'`**: Árbol de decisión - Rápido y interpretable
3. **`'knn'`**: K-Nearest Neighbors - Bueno con pocos datos

**Workflow de entrenamiento**:
```python
# Pseudo-code del proceso
if not modelos_entrenando['neural'] and datos_suficientes:
    modelos_entrenando['neural'] = True
    modelo_estado['neural'] = 'actualizando'
    
    # Entrenamiento en hilo separado
    entrenar_modelo_neural()
    
    modelo_estado['neural'] = 'entrenado'
    modelos_entrenando['neural'] = False
```

## Resumen de la Arquitectura

Este sistema muestra una arquitectura robusta que integra múltiples componentes avanzados:

### Componentes Principales

**Gameplay tradicional** con controles manuales:
- Sistema de input responsivo con pygame
- Física realista de salto con gravedad
- Mecánicas de disparo con variabilidad aleatoria
- Estados de juego consistentes y predecibles

**Sistema de aprendizaje automático** adaptable:
- Múltiples algoritmos (Neural Networks, Decision Trees, KNN)
- Feature engineering con transformaciones no lineales
- Pipeline de preprocesamiento robusto
- Entrenamiento adaptativo según volumen de datos
- Sistema de respaldo con reglas heurísticas

**Gestión de errores** en tiempo real:
- Try-catch en componentes críticos
- Fallback a comportamientos seguros
- Logging para debugging sin interrumpir gameplay
- Validación de estados antes de acciones críticas

**Física de movimiento** realista:
- Gravedad simulada con aceleración constante
- Movimientos laterales con estados finitos
- Colisiones y límites de pantalla
- Sincronización precisa a 30 FPS

**Persistencia de datos** y modelos:
- Guardado automático de experiencias de aprendizaje
- Serialización de modelos entrenados
- Gestión de memoria con limpieza de datos
- Carga/guardado asíncrono para no bloquear gameplay

### Patrones de Diseño Implementados

**Máquina de estados** para el movimiento:
- Estados mutuamente excluyentes
- Transiciones claramente definidas
- Fácil debugging y extensión
- Prevención de estados inconsistentes

**Pipeline** para preprocesamiento:
- Escalado automático de features
- Transformaciones reproducibles
- Encapsulación de todo el flujo ML
- Fácil intercambio de componentes

**Strategy Pattern** para los modelos de ML:
- Interfaz unificada para diferentes algoritmos
- Selección dinámica del mejor modelo
- Fácil adición de nuevos algoritmos
- Comparación de rendimiento entre modelos

**Observer Pattern** para los eventos de pygame:
- Desacoplamiento entre input y lógica de juego
- Manejo centralizado de eventos
- Extensibilidad para nuevos tipos de input
- Separación clara de responsabilidades

## 7. Código completo Final y aprobado

```python
import pygame
import random
import numpy as np
import os
import glob
from joblib import dump, load
import pandas as pd
from sklearn.neural_network import MLPClassifier
from sklearn.utils import compute_sample_weight
from sklearn.tree import DecisionTreeClassifier
from sklearn.neighbors import KNeighborsClassifier
from sklearn.pipeline import Pipeline
from sklearn.multioutput import MultiOutputClassifier
from sklearn.preprocessing import StandardScaler, RobustScaler, PowerTransformer

# Inicializar Pygame
pygame.init()

# Dimensiones de la pantalla
w, h = 1000, 400
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
posicion_original_x = 50
posicion_original_y = h - 100

# Variables de salto
salto = False
en_suelo = True
salto_altura = 15  # Velocidad inicial de salto
gravedad = 2

# Variables de desplazamiento lateral
izquierda = False
ida = False
vuelta = False
sin_movimiento = True
desplazamiento = 46
velocidad_dezplazamiento = 6

# Variables de pausa y menú
pausa = False
fuente = pygame.font.SysFont('Arial', 24)
menu_activo = True
modo_auto = False  # Indica si el modo de juego es automático

# Lista para guardar los datos de velocidad, distancia y salto (target)
datos_modelo = []

# Variables Extras
titulo = pygame.font.SysFont('Arial', 36)

# Variables para los modelos 
nnNetwork = None
decisionTree = None
knnModel = None
modelo = 0

# Cargar imágenes manteniendo los assets originales
jugador_frames = [
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-1.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-2.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-3.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-4.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-5.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-6.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-7.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-8.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-9.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-10.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-11.png').convert_alpha(), (32, 48)),
    pygame.transform.scale(pygame.image.load('assets/minita/fila-1-columna-12.png').convert_alpha(), (32, 48))
]

bala_img = pygame.transform.scale(pygame.image.load('assets/sprites/purple_ball.png').convert_alpha(), (17, 17))
bala2_img = pygame.transform.rotate(
    pygame.transform.scale(
        pygame.image.load('assets/sprites/purple_ball.png').convert_alpha(), 
        (17, 17)
    ), 
    90
)

fondo_img = pygame.transform.scale(pygame.image.load('assets/game/fondo2.png').convert(), (w, h))
nave_img = pygame.transform.scale(pygame.image.load('assets/game/ufo.png').convert_alpha(), (64, 64))

# Crear los rectángulos
jugador = pygame.Rect(posicion_original_x, posicion_original_y, 32, 48)
bala = pygame.Rect(w - 50, h - 90, 17, 17)
bala2 = pygame.Rect(55, 0, 17, 17)
nave = pygame.Rect(w - 100, h - 100, 64, 64)

# Variables para la bala
velocidad_bala = -18
bala_disparada = False

# Variables para la bala2
velocidad_bala2 = 15
bala2_disparada = False

# Animación
current_frame = 0
frame_speed = 2
frame_count = 0

# Música y sonidos             
sonido_muerte = pygame.mixer.Sound('assets/audio/game_over.wav')    

# Variables globales para el estado de entrenamiento
modelos_entrenando = {
    'neural': False,
    'tree': False, 
    'knn': False
}

modelo_estado = {
    'neural': 'no_entrenado',  # 'no_entrenado', 'entrenando', 'listo', 'error'
    'tree': 'no_entrenado',
    'knn': 'no_entrenado'
}

# Funciones del juego
def disparar_bala():
    global bala_disparada, velocidad_bala
    if not bala_disparada:
        velocidad_bala = random.randint(-20, -18)
        bala_disparada = True

def disparar_bala2():
    global bala2_disparada, velocidad_bala2
    if not bala2_disparada:
        bala2_disparada = True

def reset_bala():
    global bala, bala_disparada
    bala.x = w - 50
    bala_disparada = False

def reset_bala2():
    global bala2, bala2_disparada
    bala2.y = 0
    bala2_disparada = False

def manejar_salto():
    global jugador, salto, salto_altura, gravedad, en_suelo

    if salto:
        jugador.y -= salto_altura
        salto_altura -= gravedad

        if jugador.y >= posicion_original_y:
            jugador.y = posicion_original_y
            salto = False
            salto_altura = 15
            en_suelo = True

def manejar_desplazamiento():
    global jugador, izquierda, desplazamiento, velocidad_dezplazamiento, posicion_original_x, sin_movimiento, ida, vuelta
    
    if izquierda:
        if ida and jugador.x > posicion_original_x - desplazamiento:
            jugador.x -= velocidad_dezplazamiento
            if jugador.x <= posicion_original_x - desplazamiento:
                ida = False
                vuelta = True
        elif vuelta and jugador.x < posicion_original_x:
            jugador.x += velocidad_dezplazamiento
            if jugador.x >= posicion_original_x:
                jugador.x = posicion_original_x
                izquierda = False
                sin_movimiento = True
                ida = False
                vuelta = False

def pausa_juego():
    global pausa
    pausa = not pausa

def limpiar_archivos(patron):
    """Elimina archivos que coincidan con el patrón especificado"""
    for archivo in glob.glob(patron):
        try:
            os.remove(archivo)
            print(f"Eliminado archivo antiguo: {archivo}")
        except Exception as e:
            print(f"Error eliminando {archivo}: {e}")

def borrar_archivos_modelos():
    """Borra todos los archivos de modelos y datos existentes usando la función general"""
    print("Borrando archivos de modelos anteriores...")
    
    # Limpiar archivos de modelos y datos
    limpiar_archivos('Data_Entrenmiento.csv')
    limpiar_archivos('*.joblib')  # Todos los modelos guardados
    limpiar_archivos('*.dot')     # Archivos de gráficos de árboles
    limpiar_archivos('*.pdf')     # Archivos PDF generados
    limpiar_archivos('*.png')     # Imágenes de gráficos
    
    # Limpiar datos existentes en memoria
    global datos_modelo
    datos_modelo = []
    print("Preparado para nuevos datos en modo manual")

def crear_caracteristicas_expandidas(distancia_bala, velocidad_bala, distancia_bala2):
    """Función centralizada para crear características expandidas"""
    try:
        # Características básicas
        tiempo_colision_bala = distancia_bala / abs(velocidad_bala) if velocidad_bala != 0 else 1000
        tiempo_colision_bala2 = distancia_bala2 / 15  # Velocidad constante de bala2
        
        # Crear vector de características expandidas
        features = [
            distancia_bala,
            velocidad_bala,
            distancia_bala2,
            tiempo_colision_bala,
            tiempo_colision_bala2,
            np.log1p(distancia_bala + 1e-6),  # Evitar log(0)
            np.sqrt(distancia_bala2 + 1e-6),   # Evitar sqrt negativo
            distancia_bala * velocidad_bala,
            distancia_bala2**2
        ]
        
        return features
    except Exception as e:
        print(f"Error creando características: {e}")
        # Devolver características por defecto
        return [distancia_bala, velocidad_bala, distancia_bala2, 100, 100, 1, 1, -100, 100]

def preprocesar_datos(datos):
    X = []
    y = []
    for dato in datos:
        # Asegurarnos que tenemos los datos mínimos necesarios
        if len(dato['input']) < 3:
            continue  # Saltar datos incompletos
            
        distancia_bala = dato['input'][0]
        velocidad_bala = dato['input'][1]
        distancia_bala2 = dato['input'][2]
        
        # Usar la función centralizada para crear características
        features = crear_caracteristicas_expandidas(distancia_bala, velocidad_bala, distancia_bala2)
        
        X.append(features)
        y.append(dato['output'])
    
    return np.array(X), np.array(y)

def mostrar_menu():
    global menu_activo, modo_auto, nnNetwork
    pantalla.fill(NEGRO)
    
    textos = [
        "Faser",
        "M: Modo Manual",
        "A: Modo Automático con Diferentes Modelos",
        "Q: Salir del Juego"
    ]
    
    y_offset = h // 2 - 100
    for texto in textos:
        if texto == "Faser":
            texto_renderizado = titulo.render(texto, True, BLANCO)
        else:
            texto_renderizado = fuente.render(texto, True, BLANCO)
        texto_ancho = texto_renderizado.get_width()
        texto_x = (w - texto_ancho) // 2
        pantalla.blit(texto_renderizado, (texto_x, y_offset))
        if texto == "Faser":
            y_offset += 60
        else:
            y_offset += 40
    
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
                    seleccionar_modelo()
                elif evento.key == pygame.K_m:
                    # Borrar archivos existentes al seleccionar modo manual
                    borrar_archivos_modelos()
                    modo_auto = False
                    menu_activo = False
                elif evento.key == pygame.K_q:
                    pygame.quit()
                    exit()

def reiniciar_juego():
    global menu_activo, jugador, bala, nave, bala_disparada, salto, en_suelo, datos_modelo, bala2, bala2_disparada, izquierda, sin_movimiento, ida, vuelta
    
    jugador.x, jugador.y = posicion_original_x, posicion_original_y
    bala.x, bala.y = w - 50, h - 90
    nave.x, nave.y = w - 100, h - 100
    bala2.x, bala2.y = 55, 0
    bala_disparada = False
    bala2_disparada = False
    salto = False
    en_suelo = True
    izquierda = False
    sin_movimiento = True
    ida = False
    vuelta = False
    
    sonido_muerte.play()
    
    # Guardar datos automáticamente si estamos en modo manual y hay datos
    if not modo_auto and datos_modelo:
        guardar_modelos_automaticamente()
        datos_modelo.clear()
    
    menu_activo = True
    mostrar_menu()

def guardar_modelos_automaticamente():
    """Versión mejorada que maneja el estado de entrenamiento"""
    if len(datos_modelo) < 2:
        print(f"Se necesitan al menos 2 muestras para entrenar: {len(datos_modelo)}")
        return
        
    print(f"Guardando y entrenando modelos con {len(datos_modelo)} muestras...")
    
    # Guardar CSV
    datos_csv = {
        'Distancia_Bala': [dato['input'][0] for dato in datos_modelo],
        'Velocidad_Bala': [dato['input'][1] for dato in datos_modelo],
        'Distancia_Bala2': [dato['input'][2] for dato in datos_modelo],
        'Salto': [dato['output'][0] for dato in datos_modelo],
        'Izquierda': [dato['output'][1] for dato in datos_modelo]
    }
    
    df = pd.DataFrame(datos_csv)
    df.to_csv('Data_Entrenmiento.csv', index=False)
    
    # Resetear estados
    for clave in modelo_estado:
        modelo_estado[clave] = 'no_entrenado'
        modelos_entrenando[clave] = False
    
    # Entrenar modelos (esto actualiza los estados automáticamente)
    print("Entrenando Red Neuronal...")
    enRedNeural()
    print("Entrenando Árbol de Decisiones...")
    decision_tree()
    print("Entrenando KNN...")
    kNearestNeighbor()
    
    print("Todos los modelos han sido procesados.")

def guardar_datos():
    global jugador, bala, bala2, velocidad_bala, velocidad_bala2, salto, izquierda
    
    distancia_bala = abs(jugador.x - bala.x)
    distancia_bala2 = abs(jugador.y - bala2.y)
    
    salto_hecho = 1 if salto else 0
    izquierda_hecho = 1 if izquierda else 0
    
    if distancia_bala > 800:
        salto_hecho = 0
    
    if distancia_bala2 > 300:
        izquierda_hecho = 0
    
    datos_modelo.append({
        'input': [
            distancia_bala, 
            velocidad_bala,
            distancia_bala2
        ],
        'output': [
            salto_hecho,
            izquierda_hecho
        ]
    })

def seleccionar_modelo():
    global modelo, menu_activo
    pantalla.fill(NEGRO)
    
    # Verificar estado de los modelos
    verificar_estado_modelos()
    
    textos = [
        "Seleccione un modelo:",
        "1: Red Neuronal",
        "2: Árbol de Decisiones", 
        "3: KNN",
        "",
        "Estado de los modelos:"
    ]
    
    y_offset = h // 2 - 150
    for texto in textos:
        if texto == "Seleccione un modelo:":
            texto_renderizado = titulo.render(texto, True, BLANCO)
        else:
            texto_renderizado = fuente.render(texto, True, BLANCO)
        texto_ancho = texto_renderizado.get_width()
        texto_x = (w - texto_ancho) // 2
        pantalla.blit(texto_renderizado, (texto_x, y_offset))
        if texto == "Seleccione un modelo:":
            y_offset += 60
        else:
            y_offset += 30
    
    # Mostrar estado de entrenamiento
    mostrar_estado_entrenamiento_menu(y_offset)
    
    pygame.display.flip()

    seleccionando = True
    while seleccionando:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                exit()
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_1:
                    if modelo_estado['neural'] == 'listo':
                        modelo = 1
                        seleccionando = False
                        menu_activo = False
                    else:
                        print("Red neuronal no está lista. Entrenando...")
                        enRedNeural()
                elif evento.key == pygame.K_2:
                    if modelo_estado['tree'] == 'listo':
                        modelo = 2
                        seleccionando = False
                        menu_activo = False
                    else:
                        print("Árbol de decisiones no está listo. Entrenando...")
                        decision_tree()
                elif evento.key == pygame.K_3:
                    if modelo_estado['knn'] == 'listo':
                        modelo = 3
                        seleccionando = False
                        menu_activo = False
                    else:
                        print("KNN no está listo. Entrenando...")
                        kNearestNeighbor()
        
        # Actualizar pantalla mientras se entrena
        if any(modelos_entrenando.values()):
            pantalla.fill(NEGRO)
            for i, texto in enumerate(textos):
                if texto == "Seleccione un modelo:":
                    texto_renderizado = titulo.render(texto, True, BLANCO)
                else:
                    texto_renderizado = fuente.render(texto, True, BLANCO)
                texto_ancho = texto_renderizado.get_width()
                texto_x = (w - texto_ancho) // 2
                y_pos = h // 2 - 150 + (60 if i == 0 else 30 * i)
                pantalla.blit(texto_renderizado, (texto_x, y_pos))
            
            mostrar_estado_entrenamiento_menu(y_offset)
            pygame.display.flip()

def enRedNeural():
    global nnNetwork, modelo_estado, modelos_entrenando
    model_filename = 'RedNeuronal_mejorada.joblib'
    
    # Marcar como entrenando
    modelos_entrenando['neural'] = True
    modelo_estado['neural'] = 'entrenando'
    
    if os.path.exists(model_filename):
        try:
            nnNetwork = load(model_filename)
            modelo_estado['neural'] = 'listo'
            modelos_entrenando['neural'] = False
            print("Red neuronal mejorada cargada desde archivo.")
            return
        except Exception as e:
            print(f"Error cargando modelo guardado: {e}")
            os.remove(model_filename)  # Eliminar archivo corrupto

    if not datos_modelo or len(datos_modelo) < 2:
        print("Se necesitan al menos 2 muestras para entrenar la red neuronal.")
        nnNetwork = None
        modelo_estado['neural'] = 'error'
        modelos_entrenando['neural'] = False
        return

    try:
        X, y = preprocesar_datos(datos_modelo)
        n_samples = len(X)
        print(f"Entrenando red neuronal con {n_samples} muestras...")
        
        if len(X) == 0:
            modelo_estado['neural'] = 'error'
            modelos_entrenando['neural'] = False
            nnNetwork = None
            return
        
        # Configuración adaptativa más agresiva para muchas muestras
        if n_samples < 10:
            hidden_layers = (8, 4)
            max_iter = 200
            alpha = 0.01
        elif n_samples < 50:
            hidden_layers = (16, 8)
            max_iter = 300
            alpha = 0.005
        elif n_samples < 200:
            hidden_layers = (32, 16)
            max_iter = 400
            alpha = 0.002
        else:
            # Para muchas muestras - configuración más eficiente
            hidden_layers = (64, 32, 16)
            max_iter = 500  # Reducido de 1000
            alpha = 0.001
        
        mlp = MLPClassifier(
            hidden_layer_sizes=hidden_layers,
            activation='relu',
            solver='adam',
            alpha=alpha,
            learning_rate='adaptive',
            learning_rate_init=0.01,
            max_iter=max_iter,
            early_stopping=True if n_samples > 50 else False,
            validation_fraction=0.1 if n_samples > 50 else 0.2,
            n_iter_no_change=10,  # Parar antes si no mejora
            random_state=42,
            verbose=False,
            warm_start=False
        )
        
        scaler = RobustScaler()
        X_scaled = scaler.fit_transform(X)
        
        print("Iniciando entrenamiento de red neuronal...")
        mlp.fit(X_scaled, y)
        print(f"Red neuronal entrenada. Iteraciones: {mlp.n_iter_}")
        
        nnNetwork = Pipeline([
            ('scaler', scaler),
            ('mlp', mlp)
        ])
        
        # Guardar modelo de forma segura
        try:
            dump(nnNetwork, model_filename)
            modelo_estado['neural'] = 'listo'
            print("Red neuronal entrenada y guardada exitosamente.")
        except Exception as e:
            print(f"Error guardando modelo: {e}")
            modelo_estado['neural'] = 'listo'  # Funciona pero no se guardó
            
    except Exception as e:
        print(f"Error entrenando red neuronal: {str(e)}")
        modelo_estado['neural'] = 'error'
        nnNetwork = None
    
    finally:
        modelos_entrenando['neural'] = False

def predecirConRedNeuronal(entrada_basica):
    global nnNetwork
    
    if nnNetwork is None:
        enRedNeural()
        if nnNetwork is None:
            # Devolver predicción por defecto conservadora
            return False, False

    try:
        # Convertir entrada básica a características expandidas
        distancia_bala, velocidad_bala, distancia_bala2 = entrada_basica
        features_expandidas = crear_caracteristicas_expandidas(distancia_bala, velocidad_bala, distancia_bala2)
        
        # Hacer predicción
        clase_predicha = nnNetwork.predict([features_expandidas])
        
        # Verificar formato de salida
        if len(clase_predicha) > 0 and len(clase_predicha[0]) >= 2:
            saltar = clase_predicha[0][0]
            izquierda = clase_predicha[0][1]
            return saltar == 1, izquierda == 1
        else:
            print("Error en formato de predicción")
            return False, False
            
    except Exception as e:
        print(f"Error en predicción Red Neuronal: {str(e)}")
        # Lógica de respaldo simple
        if entrada_basica[0] < 100:  # Si la bala está cerca
            return True, False  # Saltar
        if entrada_basica[2] < 50:   # Si bala2 está cerca
            return False, True  # Moverse izquierda
        return False, False

def decision_tree():
    global decisionTree, modelo_estado, modelos_entrenando
    model_filename = 'DecisionTree_optimizado.joblib'
    
    modelos_entrenando['tree'] = True
    modelo_estado['tree'] = 'entrenando'
    
    if os.path.exists(model_filename):
        try:
            decisionTree = load(model_filename)
            modelo_estado['tree'] = 'listo'
            modelos_entrenando['tree'] = False
            print("Árbol de decisiones optimizado cargado desde archivo.")
            return
        except Exception as e:
            print(f"Error cargando árbol: {e}")
            os.remove(model_filename)

    if not datos_modelo or len(datos_modelo) < 2:
        print("Se necesitan al menos 2 muestras para entrenar el árbol de decisiones.")
        decisionTree = None
        modelo_estado['tree'] = 'error'
        modelos_entrenando['tree'] = False
        return

    try:
        X, y = preprocesar_datos(datos_modelo)
        n_samples = len(X)
        print(f"Entrenando árbol de decisiones con {n_samples} muestras...")
        
        # Configuración más eficiente para muchas muestras
        if n_samples < 10:
            max_depth = 3
            min_samples_split = 2
            min_samples_leaf = 1
        elif n_samples < 100:
            max_depth = 8
            min_samples_split = 4
            min_samples_leaf = 2
        else:
            # Para muchas muestras - limitar profundidad para evitar overfitting
            max_depth = 15
            min_samples_split = 10
            min_samples_leaf = 5
        
        base_tree = DecisionTreeClassifier(
            max_depth=max_depth,
            min_samples_split=min_samples_split,
            min_samples_leaf=min_samples_leaf,
            criterion='entropy',
            random_state=42,
            max_features='sqrt',
            class_weight='balanced'
        )
        
        decisionTree = Pipeline([
            ('scaler', StandardScaler()),
            ('tree', MultiOutputClassifier(base_tree))
        ])
        
        decisionTree.fit(X, y)
        
        try:
            dump(decisionTree, model_filename)
            print("Árbol de decisiones entrenado y guardado exitosamente.")
        except Exception as e:
            print(f"Error guardando árbol: {e}")
        
        modelo_estado['tree'] = 'listo'
        
    except Exception as e:
        print(f"Error entrenando árbol de decisiones: {str(e)}")
        modelo_estado['tree'] = 'error'
        decisionTree = None
    
    finally:
        modelos_entrenando['tree'] = False

def predecirConArbolDecisiones(entrada_basica):
    global decisionTree
    if decisionTree is None:
        decision_tree()
        if decisionTree is None:
            # Lógica de respaldo mejorada
            distancia_bala, velocidad_bala, distancia_bala2 = entrada_basica
            
            # Reglas heurísticas más inteligentes
            saltar = False
            mover_izquierda = False
            
            # Lógica para salto basada en distancia y velocidad
            tiempo_impacto = distancia_bala / abs(velocidad_bala) if velocidad_bala != 0 else float('inf')
            if tiempo_impacto < 20 and distancia_bala < 150:
                saltar = True
            
            # Lógica para movimiento lateral
            if distancia_bala2 < 80 and distancia_bala > 200:  # Solo moverse si no hay conflicto con la bala horizontal
                mover_izquierda = True
            
            return saltar, mover_izquierda
    
    try:
        # Intentar usar características expandidas primero
        distancia_bala, velocidad_bala, distancia_bala2 = entrada_basica
        
        # Verificar si el modelo espera características expandidas
        if hasattr(decisionTree, 'named_steps'):
            # Es un Pipeline con características expandidas
            features_expandidas = crear_caracteristicas_expandidas(distancia_bala, velocidad_bala, distancia_bala2)
            prediccion = decisionTree.predict([features_expandidas])[0]
        else:
            # Es un modelo simple con características básicas
            prediccion = decisionTree.predict([entrada_basica])[0]
        
        return prediccion[0] == 1, prediccion[1] == 1
        
    except Exception as e:
        print(f"Error en predicción Árbol de Decisiones: {str(e)}")
        # Usar lógica heurística como respaldo
        distancia_bala, velocidad_bala, distancia_bala2 = entrada_basica
        saltar = distancia_bala < 100
        mover_izquierda = distancia_bala2 < 60
        return saltar, mover_izquierda

def kNearestNeighbor():
    global knnModel, modelo_estado, modelos_entrenando
    model_filename = 'KNN_optimizado.joblib'
    
    modelos_entrenando['knn'] = True
    modelo_estado['knn'] = 'entrenando'
    
    if os.path.exists(model_filename):
        try:
            knnModel = load(model_filename)
            modelo_estado['knn'] = 'listo'
            modelos_entrenando['knn'] = False
            print("Modelo KNN optimizado cargado desde archivo.")
            return
        except Exception as e:
            print(f"Error cargando KNN: {e}")
            os.remove(model_filename)

    if not datos_modelo or len(datos_modelo) < 2:
        print("Se necesitan al menos 2 muestras para entrenar KNN.")
        knnModel = None
        modelo_estado['knn'] = 'error'
        modelos_entrenando['knn'] = False
        return

    try:
        X, y = preprocesar_datos(datos_modelo)
        n_samples = len(X)
        print(f"Entrenando KNN con {n_samples} muestras...")
        
        # Configuración más eficiente para muchas muestras
        if n_samples < 5:
            n_neighbors = max(1, n_samples - 1)
            algorithm = 'brute'
        elif n_samples < 50:
            n_neighbors = min(5, max(3, n_samples // 3))
            algorithm = 'ball_tree'
        else:
            # Para muchas muestras - limitar vecinos para eficiencia
            n_neighbors = min(15, max(5, int(np.sqrt(n_samples))))
            algorithm = 'auto'
        
        knn_classifier = KNeighborsClassifier(
            n_neighbors=n_neighbors,
            weights='distance',
            algorithm=algorithm,
            leaf_size=min(50, max(10, n_samples // 10)),
            metric='euclidean',
            n_jobs=-1  # Usar todos los cores disponibles
        )
        
        scaler = RobustScaler() if n_samples > 10 else StandardScaler()
        
        knnModel = Pipeline([
            ('scaler', scaler),
            ('knn', MultiOutputClassifier(knn_classifier))
        ])
        
        knnModel.fit(X, y)
        
        try:
            dump(knnModel, model_filename)
            print(f"Modelo KNN entrenado y guardado con {n_neighbors} vecinos.")
        except Exception as e:
            print(f"Error guardando KNN: {e}")
        
        modelo_estado['knn'] = 'listo'
        
    except Exception as e:
        print(f"Error entrenando KNN: {str(e)}")
        modelo_estado['knn'] = 'error'
        knnModel = None
    
    finally:
        modelos_entrenando['knn'] = False

def predecirConKNN(entrada_basica):
    global knnModel
    if knnModel is None:
        kNearestNeighbor()
        if knnModel is None:
            # Lógica de respaldo más sofisticada
            distancia_bala, velocidad_bala, distancia_bala2 = entrada_basica
            
            # Usar distancias normalizadas para tomar decisiones
            urgencia_bala = max(0, (200 - distancia_bala) / 200) if distancia_bala < 200 else 0
            urgencia_bala2 = max(0, (100 - distancia_bala2) / 100) if distancia_bala2 < 100 else 0
            
            # Decisiones basadas en urgencia
            saltar = urgencia_bala > 0.7
            mover_izquierda = urgencia_bala2 > 0.6 and urgencia_bala < 0.3  # No moverse si también necesita saltar
            
            return saltar, mover_izquierda
    
    try:
        # Verificar tipo de modelo y usar características apropiadas
        distancia_bala, velocidad_bala, distancia_bala2 = entrada_basica
        
        if hasattr(knnModel, 'named_steps'):
            # Pipeline con características expandidas
            features_expandidas = crear_caracteristicas_expandidas(distancia_bala, velocidad_bala, distancia_bala2)
            entrada = np.array(features_expandidas).reshape(1, -1)
        else:
            # Modelo simple
            entrada = np.array(entrada_basica).reshape(1, -1)
        
        prediccion = knnModel.predict(entrada)[0]
        return prediccion[0] == 1, prediccion[1] == 1
        
    except Exception as e:
        print(f"Error en predicción KNN: {str(e)}")
        # Respaldo con lógica simple pero efectiva
        distancia_bala, velocidad_bala, distancia_bala2 = entrada_basica
        
        # Lógica de proximidad con umbrales dinámicos
        umbral_salto = 120 if abs(velocidad_bala) > 15 else 80
        saltar = distancia_bala < umbral_salto
        mover_izquierda = distancia_bala2 < 70 and distancia_bala > 150
        
        return saltar, mover_izquierda

def evaluar_modelos():
    """Función para evaluar el rendimiento de los modelos entrenados"""
    if not datos_modelo or len(datos_modelo) < 5:
        print("No hay suficientes datos para evaluación.")
        return
    
    X, y = preprocesar_datos(datos_modelo)
    
    modelos = [
        ('Red Neuronal', nnNetwork),
        ('Árbol de Decisiones', decisionTree), 
        ('KNN', knnModel)
    ]
    
    print("\n=== EVALUACIÓN DE MODELOS ===")
    for nombre, modelo in modelos:
        if modelo is not None:
            try:
                # Evaluación simple en datos de entrenamiento
                if hasattr(modelo, 'score'):
                    score = modelo.score(X, y)
                    print(f"{nombre}: Precisión = {score:.3f}")
                else:
                    print(f"{nombre}: Modelo cargado correctamente")
            except Exception as e:
                print(f"{nombre}: Error en evaluación - {str(e)}")
        else:
            print(f"{nombre}: Modelo no disponible")
    print("===========================\n")

def update():
    global bala, velocidad_bala, current_frame, frame_count, bala2
    
    pantalla.blit(fondo_img, (0, 0))

    frame_count += 1
    if frame_count >= frame_speed:
        current_frame = (current_frame + 1) % len(jugador_frames)
        frame_count = 0

    if bala_disparada:
        bala.x += velocidad_bala
        
    if bala2_disparada:
        bala2.y += velocidad_bala2

    if bala.x < 0:
        reset_bala()

    if bala2.y >= h-20:
        reset_bala2()

    pantalla.blit(jugador_frames[current_frame], (jugador.x, jugador.y))
    pantalla.blit(bala_img, (bala.x, bala.y))
    pantalla.blit(bala2_img, (bala2.x, bala2.y))
    pantalla.blit(nave_img, (nave.x, nave.y))
    
    if jugador.colliderect(bala) or jugador.colliderect(bala2):
        reiniciar_juego()

def mostrar_estado_entrenamiento():
    """Muestra el estado actual del entrenamiento en pantalla"""
    y_pos = 10
    estados = {
        'no_entrenado': 'No entrenado',
        'entrenando': 'Entrenando...',
        'listo': 'Listo',
        'error': 'Error'
    }
    
    colores = {
        'no_entrenado': (128, 128, 128),  # Gris
        'entrenando': (255, 255, 0),      # Amarillo
        'listo': (0, 255, 0),             # Verde
        'error': (255, 0, 0)              # Rojo
    }
    
    fuente_pequena = pygame.font.SysFont('Arial', 16)
    
    texto_neural = fuente_pequena.render(f"Red Neuronal: {estados[modelo_estado['neural']]}", True, colores[modelo_estado['neural']])
    texto_tree = fuente_pequena.render(f"Árbol Decisiones: {estados[modelo_estado['tree']]}", True, colores[modelo_estado['tree']])
    texto_knn = fuente_pequena.render(f"KNN: {estados[modelo_estado['knn']]}", True, colores[modelo_estado['knn']])
    
    pantalla.blit(texto_neural, (10, y_pos))
    pantalla.blit(texto_tree, (10, y_pos + 20))
    pantalla.blit(texto_knn, (10, y_pos + 40))

def mostrar_estado_entrenamiento_menu(y_offset):
    """Muestra el estado de entrenamiento en el menú de selección"""
    estados = {
        'no_entrenado': 'No entrenado',
        'entrenando': 'Entrenando...',
        'listo': 'Listo ✓',
        'error': 'Error ✗'
    }
    
    colores = {
        'no_entrenado': (128, 128, 128),
        'entrenando': (255, 255, 0),
        'listo': (0, 255, 0),
        'error': (255, 0, 0)
    }
    
    fuente_pequena = pygame.font.SysFont('Arial', 18)
    
    modelos_nombres = [
        ('Red Neuronal', 'neural'),
        ('Árbol Decisiones', 'tree'),
        ('KNN', 'knn')
    ]
    
    for i, (nombre, clave) in enumerate(modelos_nombres):
        estado_texto = f"{nombre}: {estados[modelo_estado[clave]]}"
        color = colores[modelo_estado[clave]]
        texto_renderizado = fuente_pequena.render(estado_texto, True, color)
        texto_ancho = texto_renderizado.get_width()
        texto_x = (w - texto_ancho) // 2
        pantalla.blit(texto_renderizado, (texto_x, y_offset + i * 25))

def verificar_estado_modelos():
    """Verifica el estado actual de todos los modelos"""
    global nnNetwork, decisionTree, knnModel
    
    # Verificar red neuronal
    if nnNetwork is not None and not modelos_entrenando['neural']:
        modelo_estado['neural'] = 'listo'
    elif not modelos_entrenando['neural']:
        modelo_estado['neural'] = 'no_entrenado'
    
    # Verificar árbol de decisiones
    if decisionTree is not None and not modelos_entrenando['tree']:
        modelo_estado['tree'] = 'listo'
    elif not modelos_entrenando['tree']:
        modelo_estado['tree'] = 'no_entrenado'
    
    # Verificar KNN
    if knnModel is not None and not modelos_entrenando['knn']:
        modelo_estado['knn'] = 'listo'
    elif not modelos_entrenando['knn']:
        modelo_estado['knn'] = 'no_entrenado'

def main():
    global salto, en_suelo, bala_disparada, bala2_disparada, modelo, izquierda, sin_movimiento, ida
    reloj = pygame.time.Clock()
    mostrar_menu()
    correr = True

    while correr:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                correr = False
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE and en_suelo and not pausa:
                    salto = True
                    en_suelo = False
                elif evento.key == pygame.K_LEFT and sin_movimiento and not pausa:
                    ida = True
                    izquierda = True
                    sin_movimiento = False
                elif evento.key == pygame.K_p:
                    pausa_juego()
                elif evento.key == pygame.K_q:
                    pygame.quit()
                    exit()

        if not pausa:
            if not modo_auto:
                if salto:
                    manejar_salto()
                if izquierda:
                    manejar_desplazamiento()
                guardar_datos()
            
            if modo_auto:
                entrada = [abs(jugador.x - bala.x), velocidad_bala, abs(jugador.y - bala2.y)]
                
                match modelo:
                    case 1:
                        saltar, mover_izquierda = predecirConRedNeuronal(entrada)
                        
                        if saltar and not salto and en_suelo:
                            salto = True
                            en_suelo = False
                        
                        if mover_izquierda and sin_movimiento:
                            izquierda = True
                            ida = True
                            sin_movimiento = False

                    case 2:
                        saltar, mover_izquierda = predecirConArbolDecisiones(entrada)

                        if saltar and not salto and en_suelo:
                            salto = True
                            en_suelo = False
                        
                        if mover_izquierda and sin_movimiento:
                            izquierda = True
                            ida = True
                            sin_movimiento = False

                    case 3:
                        saltar, mover_izquierda = predecirConKNN(entrada)
                        
                        if saltar and not salto and en_suelo:
                            salto = True
                            en_suelo = False
                        
                        if mover_izquierda and sin_movimiento:
                            izquierda = True
                            ida = True
                            sin_movimiento = False

                if salto:
                    manejar_salto()
                if izquierda:
                    manejar_desplazamiento()

            if not bala_disparada:
                disparar_bala()

            if not bala2_disparada:
                disparar_bala2()
            update()
        
        pygame.display.flip()
        reloj.tick(30)

    pygame.quit()

if __name__ == "__main__":
    main()
```