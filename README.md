# Trabajo-Final-Entornos-Inteligentes



## Trabajo 15: Detección de episodios de bloqueo de la marcha mediante vídeo



Sistema de visión por computador y análisis temporal diseñado para detectar y localizar en el tiempo episodios de bloqueo de la marcha (*Freezing of Gait*, FoG) en un entorno doméstico asistido ("casa que cuida").



---



### 1. Descripción del Proyecto



El objetivo principal es identificar bloqueos de la marcha a partir de grabaciones de vídeo convencionales, diferenciándolos de detenciones voluntarias normales y evaluando la robustez del sistema ante oclusiones parciales, cambios de perspectiva y variabilidad entre personas.



El procesamiento se divide en cuatro etapas:



1. **Estimación de pose 2D:** Extracción de coordenadas articulares del cuerpo (caderas, rodillas, tobillos y eje del tronco) mediante **YOLOv8-Pose**. Trabajar sobre el esqueleto en lugar de píxeles crudos reduce el coste computacional y preserva la privacidad de la persona en el hogar.

2. **Ingeniería de características biomecánicas:** Normalización espacial de los puntos clave respecto a la longitud del torso (invarianza frente a la distancia y ángulo de cámara), cálculo de velocidades articulares, longitud de zancada y extracción del *Freeze Index* espectral para distinguir bloqueos con temblor de paradas en reposo.

3. **Clasificación temporal:** Entrenamiento de una red neuronal secuencial en **PyTorch** (BiGRU / TCN) sobre ventanas deslizantes para clasificar cada intervalo en marcha normal, parada voluntaria o episodio de bloqueo.

4. **Asistencia contextual en el hogar:** Lógica de respuesta escalonada no intrusiva:

   - Bloqueos breves (< 2 s): registro silencioso para seguimiento clínico.

   - Bloqueos prolongados (> 3 s): activación de estímulos de apoyo locales (señal visual en el suelo o pulso auditivo rítmico) para ayudar a reanudar la marcha antes de escalar una alerta al cuidador.



---



### 2. Datasets Utilizados



El proyecto combina dos conjuntos de datos públicos en vídeo RGB:



* **GAVD (*Gait Abnormality in Video Dataset*):** Secuencias clínicas reales de marcha parkinsoniana y marcha normal de control con anotaciones de *bounding boxes* y rangos de fotogramas.

* **UR Fall Detection Dataset:** Secuencias grabadas con cámara fija en interiores domésticos que incluyen actividades cotidianas (caminar, detenerse de pie, sentarse, agacharse) y caídas. Se emplea para entrenar la clase negativa (detenciones y movimientos normales en casa) y validar el comportamiento en entorno interior.



---



### 3. Estructura del Repositorio



```text

Trabajo-Final-Entornos-Inteligentes/

├── 01_descarga_datasets-v2.ipynb   # Descarga, compilación y verificación de datos

├── requirements.txt                # Dependencias del entorno Python

├── README.txt                      # Documentación principal del proyecto

└── data/

    ├── raw/

    │   ├── gavd/

    │   │   ├── annotations/        # CSVs originales con anotaciones de GAVD

    │   │   └── clips/              # Vídeos recortados (.mp4)

    │   └── ur_fall/

    │       └── clips/              # Secuencias compiladas (.mp4)

    └── metadata/

        ├── metadata_gavd.csv       # Índice unificado de GAVD

        └── metadata_urfall.csv     # Índice unificado de UR Fall

```



---



### 4. Instalación y Requisitos



Se recomienda utilizar **Python 3.10 o superior** dentro de un entorno virtual aislado.



#### 1. Crear y activar el entorno virtual



En Windows (PowerShell):

```bash

python -m venv .venv

.venv\Scripts\Activate.ps1

```



En Linux / macOS:

```bash

python3 -m venv .venv

source .venv/bin/activate

```



#### 2. Instalar dependencias



```bash

pip install --upgrade pip

pip install -r requirements.txt

```




---



### 5. Flujo de Ejecución



1. **Preparación de datos:** Abrir y ejecutar `01_descarga_datasets-v2.ipynb`. El cuaderno comprueba si los vídeos ya están descargados y genera los índices en `data/metadata/`.

2. **Extracción de pose y características:** Procesar los vídeos registrados con YOLOv8-Pose para generar las series temporales biomecánicas normalizadas.

3. **Entrenamiento y evaluación:** Entrenar el modelo temporal en PyTorch y evaluar su rendimiento frente a paradas normales, cambios de perspectiva y oclusiones.

4. **Demostración:** Ejecutar la inferencia sobre secuencias de prueba para visualizar la detección temporal de los episodios y la respuesta contextual.