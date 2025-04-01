# Avance del ejercicio de modificacion del codigo de reconocimiento de caracteristicas importantes del rostro

```python
import cv2
import mediapipe as mp
import numpy as np

# Inicializar MediaPipe Face Mesh
mp_face_mesh = mp.solutions.face_mesh
face_mesh = mp_face_mesh.FaceMesh(static_image_mode=False, max_num_faces=2, 
                                  min_detection_confidence=0.5, min_tracking_confidence=0.5)

# Captura de video
cap = cv2.VideoCapture(0)

# Definir puntos de interés 
selected_points = {
    "contorno": [10, 338, 297, 332, 284, 251, 389, 356, 454, 323, 361, 288, 397, 365, 379, 378, 400, 377, 152, 148, 176, 149, 150, 136, 172, 58, 132, 93, 234, 127, 162, 21, 54, 103, 67, 109, 123, 352],
    "ceja_izq": [70, 63, 105, 66, 107, 55, 65, 52, 53, 46],
    "ceja_der": [336, 296, 334, 293, 300, 276, 283, 282, 295, 285],
    "ojos": [33, 7, 163, 144, 145, 153, 154, 155, 133, 173, 157, 158, 159, 160, 161, 246, 362, 398, 384, 385, 386, 387, 388, 466, 263, 249, 390, 373, 374, 380, 381, 382],
    # "nariz": [1, 2, 98, 327, 326, 6, 197, 195, 5, 4, 45, 220, 115, 48, 219, 64, 240, 44, 275, 244, 233, 232, 231, 230, 229, 228, 31, 35, 143, 156, 30, 29, 27, 28, 56, 190],
    "boca": [61, 146, 91, 181, 84, 17, 314, 405, 321, 375, 291, 185, 40, 39, 37, 0, 267, 269, 270, 409, 78, 95, 88, 178, 87, 14, 317, 402, 318, 324, 308, 191, 80, 81, 82, 13, 312, 311, 310, 415, 308]
}

# Colores para diferentes grupos de puntos
colors = {
    "contorno": (0, 255, 0),     # Verde
    "ceja_izq": (255, 0, 0),     # Azul
    "ceja_der": (0, 0, 255),     # Rojo
    "ojos": (255, 255, 0),       # Cian
    # "nariz": (255, 0, 255),      # Magenta
    "boca": (0, 255, 255)        # Amarillo
}

def distancia(p1, p2):
    """Calcula la distancia euclidiana entre dos puntos."""
    return np.linalg.norm(np.array(p1) - np.array(p2))

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    frame = cv2.flip(frame, 1)  # Espejo para mayor naturalidad
    rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = face_mesh.process(rgb_frame)

    if results.multi_face_landmarks:
        for face_landmarks in results.multi_face_landmarks:
            puntos = {}
            
            # Dibujar todos los puntos seleccionados por grupo
            for grupo, indices in selected_points.items():
                for idx in indices:
                    x = int(face_landmarks.landmark[idx].x * frame.shape[1])
                    y = int(face_landmarks.landmark[idx].y * frame.shape[0])
                    puntos[idx] = (x, y)
                    cv2.circle(frame, (x, y), 2, colors[grupo], -1)
            
            # Ejemplo: Calcular distancia entre puntos de los ojos (opcional)
            if 33 in puntos and 133 in puntos:  # Puntos de los ojos izquierdo y derecho
                d_ojos = distancia(puntos[33], puntos[133])
                cv2.line(frame, puntos[33], puntos[133], (0, 255, 0), 1)
                cv2.putText(frame, f"D ojos: {int(d_ojos)}", (puntos[33][0], puntos[33][1] - 10), 
                            cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 1)
            
            # Calcular distancia entre los puntos 10 y 152
            if 10 in puntos and 152 in puntos:
                d_10_152 = distancia(puntos[10], puntos[152])
                cv2.line(frame, puntos[10], puntos[152], (255, 255, 255), 1)
                cv2.putText(frame, f"D: {int(d_10_152)}", 
                            (puntos[10][0], puntos[10][1] - 30),  # Posición arriba del punto 10
                            cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1)
                
            if 123 in puntos and 352 in puntos:  # Verifica que los puntos existan
                d_media = distancia(puntos[123], puntos[352])
                cv2.line(frame, puntos[123], puntos[352], (255, 255, 255), 1)
                cv2.putText(frame, f"D: {int(d_media)}", 
                            (puntos[123][0], puntos[123][1] - 10),  # Ajusta posición Y si hay superposición
                            cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1)

    cv2.imshow('PuntosFacialesMediaPipe', frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```