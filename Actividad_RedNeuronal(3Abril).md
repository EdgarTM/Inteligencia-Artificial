# Modelado de una Red Neuronal para Identificación de Emociones
(Actividad realiada el día jueves 03 de Abril)

Para nosotros poder identificar las emociones usando las landmarks de MediaPipe, se podrían tomar en cuenta las siguientes especificaciones.

## 1. Tipo de Neurona
Para este problema, se utilizarán **neuronas perceptrón multicapa** (MLP - Multi-Layer Perceptron) debido a su capacidad de aprendizaje no lineal. Estas neuronas se agruparán en capas densamente conectadas para mejorar la capacidad de clasificación de emociones.

## 2. Estructura de la Red Neuronal
La arquitectura de la red neuronal estará conformada por:

- **Capa de entrada**: Recibe los valores numéricos de los landmarks faciales extraídos con MediaPipe.
- **Capas ocultas**: Utilizan neuronas densamente conectadas con funciones de activación no lineales para aprender representaciones complejas.
- **Capa de salida**: Genera la clasificación final de la emoción detectada.

### Estructura propuesta:
1. **Capa de entrada**:
   - Número de neuronas: 468 puntos x 3 coordenadas (X, Y, Z) = **1404 neuronas**.
   - También se podría considerar reducir el número de entradas a unicamente tomar en cuenta los puntos importantes que nosotros consideremos, reduciendo asi las entradas.
   
2. **Capas ocultas**:
   - Primera capa oculta: 512 neuronas (activación ReLU)
   - Segunda capa oculta: 256 neuronas (activación ReLU)
   - Tercera capa oculta: 128 neuronas (activación ReLU)
   
3. **Capa de salida**:
   - Número de neuronas: 6 (para 6 emociones básicas: felicidad, tristeza, enojo, sorpresa, miedo, y neutral)
   - Función de activación: Softmax

## 3. Patrones Utilizados
Los datos de entrenamiento estarán conformados por vectores de landmarks faciales etiquetados con la emoción correspondiente. Se utilizará una base de datos con rostros etiquetados en distintas emociones, asegurando diversidad en expresiones, edades y géneros para una mejor generalización de dichas emociones en distintas edades y asi mismo en diferentes situaciones y contextos.

## 4. Función de Activación
Se emplearán las siguientes funciones de activación:

- **ReLU (Rectified Linear Unit)** en las capas ocultas para evitar el problema del desvanecimiento del gradiente y mejorar la eficiencia del aprendizaje.
- **Softmax** en la capa de salida, ya que permite convertir las salidas en probabilidades que representen cada emoción.

## 5. Número Máximo de Entradas
El número máximo de entradas es **1404** (468 landmarks con coordenadas X, Y y Z). Sin embargo, se podría reducir este número utilizando técnicas de reducción de dimensionalidad como PCA o seleccionando solo los landmarks más relevantes.

## 6. Valores Esperados en la Salida
La salida de la red neuronal será un vector de 6 probabilidades, donde cada posición corresponde a una emoción:

| Índice | Emoción   |
|--------|----------|
| 0      | Felicidad |
| 1      | Tristeza  |
| 2      | Enojo     |
| 3      | Sorpresa  |
| 4      | Miedo     |
| 5      | Neutral   |

Cada valor en la salida será una probabilidad entre 0 y 1, y la emoción predicha será la de mayor probabilidad.
