# Red Neuronal para Juego 5 en Línea Sin Gravedad  


##  Consideraciones del Modelo  
Para definir la red neuronal, se toman en cuenta las siguientes características:  

- **Dimensiones del Tablero**: 20*20.  
- **Datos de entrada** :
  - Turno del jugador actual.  
  - Estado de las casillas (vacías, ocupadas por el jugador 1 o 2).  
  - Número de casillas restantes.  
  - Posición de las fichas que se encuentren ya colocadas en el tablero.  

---

## Estructura de la Red Neuronal  

### Tipo de Red Neuronal  
Se utilizará una **Red Neuronal Feedforward (FFNN) con una arquitectura perceptron Multicapa**.  

### Número de Entradas  
El tablero tiene **20x20 = 400** posiciones, pero las entradas incluyen información adicional:  

- **400** → Representan cada celda del tablero:  
  - `0`: Celda vacía  
  - `1`: Ficha del jugador 1  
  - `2`: Ficha del jugador 2  
- **1** → Turno del jugador actual (`0` o `1`)  
- **1** → Casillas disponibles (`0-400`)    

### Función de Activación  
Se usará la función **ReLU** en las capas ocultas para mejorar el aprendizaje y **Softmax** en la capa de salida para la clasificación de jugadas óptimas.  

---

## Capas y Parámetros  

1. **Capa de entrada**:  
   - Estado de cada una de las casillas del juego
   - Turno del jugador (1 o 2)
   - Fichas puestas en el tablero 

2. **Capas ocultas**:  
   - Conformadas por la funcion de activacion ReLU para realizar los calculos internos de los datos ingresados.  

3. **Capa de salida**:  
   - Estado de las celdas (0, 1, 2), para así seleccionar la mejor probabilidad de una jugada con ventaja o buen posicionamiento en el tablero dependiendo del turno del jugador.  
