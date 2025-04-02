# Análisis del Procedimiento para Detección de Gestos Faciales con MediaPipe


Para el proceso de detección de gestos y emociones usando MediaPipe desde mi punto de vista debemos comenzar con el hecho de tener en consideración los aspectos técnicos clave y parámetros de detección, los cuales analisaremos en lo siguiente.

## 1. Componentes Fundamentales

### 1.1 Modelo Facial de MediaPipe
MediaPipe emplea un modelo facial 3D compuesto por 468 puntos de referencia (landmarks) que representan:

- **Contorno facial**: 148 puntos (forma ovalada del rostro)
- **Labios**: 40 puntos (20 superiores + 20 inferiores)
- **Ojos**: 64 puntos (32 por cada ojo)
- **Cejas**: 32 puntos (16 por cada ceja)
- **Nariz**: 12 puntos (eje nasal y contorno)

### 1.2 Mecánica de los Gestos Faciales
Los gestos se identifican mediante:

- Variaciones en la **contracción/relajación muscular**
- Cambios en las **distancias relativas** entre puntos clave
- Patrones de **movimiento temporal** (duración, velocidad)

## 2. Parámetros de Detección

### 2.1 Métricas Clave para Gestos
Se utilizan las siguientes métricas para identificar gestos específicos:

| Gestos        | Puntos Involucrados               | Fórmula de Cálculo               | Umbral    |
|---------------|-----------------------------------|-----------------------------------|-----------|
| **Parpadeo**  | 145, 159, 133, 33 (ojo izquierdo) | `dist(párpados)/dist(esquinas)`   | < 0.25    |
| **Sonrisa**   | 61, 291, 0, 17                    | `dist(comisuras)/dist(labios)`    | > 1.5     |
| **Cejas**     | 70, 63 vs punto de referencia 10  | `Δy(ceja-mentón)`                 | +15%      |

### 2.2 Características de Gestos Básicos

| Gestos           | Región       | Parámetros Clave          | Duración Típica |
|------------------|-------------|---------------------------|-----------------|
| Parpadeo simple  | Ojos        | Ratio < 0.25              | 100-300 ms      |
| Guiño            | Ojo unilateral | Asimetría > 40%         | 300-500 ms      |
| Sonrisa amplia   | Boca        | Ratio > 1.8               | > 500 ms        |

## 3. Detección de Emociones

### 3.1 Parámetros Clave para la Detección de Emociones
La detección de emociones se basa en la combinación de múltiples gestos faciales y sus variaciones en tiempo real. Se consideran los siguientes aspectos:

| Emoción     | Regiones Clave   | Parámetros Analizados                          | Indicadores |
|------------|----------------|--------------------------------|-------------|
| **Felicidad** | Boca, ojos    | Sonrisa amplia, elevación de mejillas | Ratio labios > 1.5, ojos levemente cerrados |
| **Tristeza**  | Boca, cejas   | Comisuras caídas, cejas elevadas en el centro | Ángulo comisuras < -10°, Δy(ceja-centro) > 5% |
| **Enojo**    | Cejas, ojos, boca | Cejas fruncidas, ojos entrecerrados, labios tensos | Δy(cejas) < -10%, labios apretados |

### 3.2 Consideraciones para la Precisión
- Es recomendable usar análisis temporal para distinguir emociones pasajeras de expresiones sostenidas.
- Factores ambientales como iluminación y resolución de la imagen pueden afectar la precisión.
- Se puede mejorar la detección combinando estos parámetros con redes neuronales entrenadas en expresiones faciales.

## Consideraciones Adicionales
- La precisión depende de la calidad del input visual.
- Requiere calibración inicial para adaptarse a diferencias faciales.