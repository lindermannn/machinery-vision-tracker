# 🏗️ BermGuard AI — Visión computacional para maquinaria y pretiles

**Desarrollado en seis días · Dos métodos de visión computacional · Del diseño a la entrega integrada**

Pipeline de visión computacional para monitorear maquinaria minera y estimar la altura de pretiles. Esta presentación destaca el **Método 2**, con capturas de sus salidas sobre video sintético generado por IA.

![Camión y bulldozer identificados, proximidad estimada y perfil del pretil](docs/images/method2-video02-seeded-05s.png)

*Video 02, segundo 5, corrida con máscaras aprobadas del pretil. Los metros y el veredicto del overlay son estimaciones bajo supuestos, no topografía ni certificación de seguridad.*

---

## ✨ Qué hace

- Detecta maquinaria con YOLO11n-seg ajustado con anotaciones propias.
- Asocia detecciones entre cuadros mediante tracking con asignación global.
- Delimita el pretil a partir de máscaras aprobadas y propagación temporal.
- Estima altura y proximidad con geometría de cámara, referencias e intervalos.
- Expone los casos sin evidencia como N/D, sin confundirlos con condiciones seguras.
- Genera video anotado, series CSV, gráficos y metadata de la corrida.

## 🎬 El Método 2 en acción

| Maquinaria y pretil — segundo 3 | Escena nocturna — segundo 1 |
|---|---|
| ![Bulldozer y límites del pretil de día](docs/images/method2-video02-seeded-03s.png) | ![Camión y referencia del pretil de noche](docs/images/method2-video02-seeded-01s.png) |

Capturas del mismo video, sin retoques al dibujo original. La escena nocturna corresponde a una corrida con máscaras externas aprobadas: no demuestra reconocimiento automático del pretil en oscuridad.

## 🖼️ El overlay, función por función

### Lo que identifica la cámara

| Elemento | Qué muestra |
|---|---|
| Caja y clase | Detección de maquinaria |
| ID | Identidad asignada por el tracker, no identidad verificada manualmente |
| Líneas naranja y azul | Límites del pretil en la corrida con máscaras aprobadas |

### Lo que estima la geometría

| Elemento | Qué muestra |
|---|---|
| Altura e intervalo | Estimación del pretil e incertidumbre declarada |
| Línea entre vehículos | Proximidad estimada para ese par |
| Rojo / ámbar / verde | Umbrales del ejercicio: menos de 10 m / hasta 20 m / más de 20 m |
| Plomo / N/D | Evidencia o geometría insuficiente; no significa seguridad |

### Lo que queda registrado

| Artefacto | Qué permite revisar |
|---|---|
| Serie de altura | Variación de estimaciones durante el clip |
| Matriz de distancias | Menor distancia conservadora disponible por par de IDs |
| Distribución espacial | Recorridos observados en coordenadas de imagen |

## ⚙️ Cómo funciona

```text
Video → YOLO11n-seg ajustado → vehículos → tracking → geometría por escena
                                                            ↓
Semilla aprobada → SAM 2.1 / propagación → pretil → fusión y diagnóstico
                                                            ↓
                              Video con overlay · CSV · gráficos · metadata
```

| Componente | Función |
|---|---|
| YOLO11n-seg | Detector especializado ajustado con anotaciones propias |
| SAM 2.1 Hiera Small | Segmentación promptable y propagación temporal; requiere semilla |
| Tracking | Asignación húngara, movimiento y ciclo de vida ante detecciones faltantes |
| Geometría | Referencias dimensionales, intervalos y compuertas de validez por escena |
| Diagnóstico | Separación entre observación, estimación y ausencia de evidencia |
| Validación | Fixtures, revisión visual, contratos, Docker offline y hashes |

También se implementó un Método 1 independiente: extracción fotométrica del pretil con memoria diurna y Grounding DINO Tiny para vehículos.

## 📐 De píxeles a metros: altura del pretil

El Método 2 no convierte todos los píxeles con una escala global fija. El reporte final y el módulo de altura documentan este procedimiento:

1. **Delimitar el pretil:** partir de una máscara aprobada y de su propagación temporal. En cada columna válida se obtiene el borde superior (cresta) y el inferior (pie).
2. **Medir el grosor visible:** `fila_pie − fila_cresta + 1`, en píxeles. Se excluyen columnas bloqueadas por maquinaria cuando se proporciona su máscara dinámica y se toma la mediana del grosor y de la fila del pie.
3. **Ajustar la cámara por escena:** usar el tamaño aparente de maquinaria y las dimensiones de clase declaradas. El reporte utiliza camión de 6,6 × 13,7 × 8,29 m y bulldozer de 4,5 × 10,9 × 6,7 m, con incertidumbres de 12 % y 15 %, respectivamente. Son referencias de clase, no dimensiones medidas de cada vehículo sintético.
4. **Convertir a metros:** aplicar el horizonte y la altura de cámara estimados para ese segmento:

```text
altura_pretil_m = grosor_px × altura_cámara_m / (fila_pie_px − horizonte_px)
```

El FOV supuesto se cancela en esta expresión de altura; sí afecta las distancias entre vehículos. En la corrida con máscaras del video 02, el metadata registra horizonte **274,49 px** y altura de cámara estimada **10,63 m**. No son parámetros levantados en terreno.

5. **Publicar incertidumbre y estado:** calcular los extremos usando los límites del horizonte y la incertidumbre de altura de cámara. Las compuertas rechazan soporte insuficiente, saltos de área o grosor y geometría inválida, en vez de dibujar una medición engañosa.

### Ejemplo de la misma corrida mostrada

La primera fila del CSV registra **42 px** de grosor y **3,134 m**, con intervalo **2,489–3,876 m**. Al final, la fila 239 registra **37 px** y **2,685 m**, con intervalo **2,137–3,315 m**. Son estimaciones por cuadro, no cambios físicos comprobados del pretil.

El código compara el intervalo con un umbral del ejercicio basado en una rueda de referencia de 3,596 m: **1,798 m** para el criterio de media rueda o **2,397 m** para dos tercios. Devuelve `cumple` si el límite inferior supera el umbral, `por_debajo` si el superior queda por debajo y `no_concluyente` si el intervalo lo cruza. El criterio normativo seleccionado es un supuesto del proyecto; el overlay no certifica cumplimiento legal en terreno.

**No mezclar versiones:** los 3,60 m e intervalo 2,74–4,23 m del video 02 en la sección 5.4 del benchmark corresponden a la corrida del sprint 18, no a esta salida integrada con máscaras. Los ejemplos de arriba proceden del CSV que acompaña estas imágenes.

## 📈 Resultados gráficos del video 02

Son los archivos originales de la misma corrida con máscaras aprobadas; no se recalcularon ni se retocaron.

### Altura visible del pretil en el tiempo

![Serie de altura estimada del pretil del video 02](docs/images/video02-berm_height_over_time.png)

Cada punto representa una altura estimada disponible en su instante. Permite inspeccionar variación y continuidad durante el clip. El gráfico muestra valores nominales, **no la banda de incertidumbre**: los límites están en `berm_height_series.csv`. La variación visual no demuestra un derrumbe ni exactitud topográfica.

### Proximidad y distribución espacial

| Distancias mínimas conservadoras | Recorridos observados en imagen |
|---|---|
| ![Matriz de distancias mínimas entre IDs](docs/images/video02-minimum_distance_matrix.png) | ![Trayectorias de puntos de contacto en píxeles](docs/images/video02-vehicle_spatial_distribution.png) |

La **matriz** resume la menor distancia conservadora disponible entre pares de IDs durante el video. Las celdas blancas no indican seguridad: incluyen la diagonal y pares sin una distancia válida disponible. No representa distancias simultáneas de todos los vehículos.

La **distribución espacial** dibuja el punto de contacto observado de cada track en coordenadas de imagen, **en píxeles, no metros**. Los IDs identifican seguimientos del algoritmo; sin identidades anotadas no equivalen necesariamente a siete máquinas distintas.

## ⏱️ Desarrollo en seis días

Entre el **6 y el 11 de septiembre de 2026** se desarrollaron dos enfoques: un Método 1 con extracción clásica y Grounding DINO Tiny, y un Método 2 con YOLO11n-seg y SAM2. El trabajo incluyó anotación asistida, entrenamiento, tracking, geometría, visualización, pruebas y empaquetado. Esta presentación se centra en el Método 2.

| Fecha | Hitos documentados |
|---|---|
| 6 de septiembre | Definición del alcance, contratos de ejecución y primer pipeline clásico sobre los cuatro videos |
| 7 de septiembre | Comparación inicial de enfoques, integración de Grounding DINO y revisión visual que detectó problemas pese a las pruebas en verde |
| 8 de septiembre | Memoria diurna del pretil, estados de evidencia y verificación del paquete del Método 1 desde su extracción |
| 9 de septiembre | Refinamiento del Método 1 y rediseño del Método 2 con SAM2, YOLO-seg y desarrollo por sprints |
| 10 de septiembre | Anotación asistida con revisión humana en Label Studio; mejoras de altura y calibración del Método 1 |
| 11 de septiembre | Entrenamientos de YOLO11n-seg, integración de tracking y geometría del Método 2, corrección de la evaluación de máscaras, reporte y entrega |

El plazo describe el desarrollo de la prueba técnica, no una validación de producción. La reorganización de archivos y esta presentación del portfolio se realizaron posteriormente.

## 🛠️ Anotación y decisiones de ingeniería

Según la documentación de la entrega:

- Definí la arquitectura del Método 2 y organicé el desarrollo en sprints.
- Construí y revisé el flujo de anotación asistida: SAM2 propone; Label Studio permite aceptar, corregir o dibujar.
- Distinguí máscaras humanas de propuestas aprobadas, lo que obligó a corregir la evaluación nocturna.
- Propuse dimensiones de maquinaria como referencias geométricas.
- Rechacé resultados visualmente incorrectos aunque las pruebas unitarias pasaran.
- Integré IA y MCP con revisión humana, trazabilidad y comprobación de artefactos.

## 🚀 Instalación y ejecución

El código de ambos métodos está disponible bajo **AGPL-3.0**. Los videos completos,
pesos y bases de anotación no se incluyen; utiliza material propio autorizado.

```bash
python -m venv .venv
# Activa el entorno según tu sistema operativo.
python -m pip install -r requirements.txt
python main.py --help
```

### Método 2 — YOLO-seg y geometría

Necesita un checkpoint de segmentación compatible con las clases de maquinaria:

```bash
python main.py --method 2 --input examples/videos --output outputs/method2 --weights models/yoloseg_maquinaria_train_v2.pt --device cpu
```

Crea la carpeta de entrada y coloca tus clips localmente. Para GPU usa `--device 0`
con una instalación de PyTorch compatible. Las máscaras externas del pretil son
opcionales mediante `--berm-mask-root`; sin ellas queda en N/D. El runner no
ejecuta SAM2 automáticamente.

### Método 1 — Grounding DINO y pretil clásico

El script siguiente descarga el detector público y verifica su revisión y hash;
la descarga requiere conexión y espacio para sus pesos:

```bash
python scripts/fetch_semantic_detector.py --destination models/grounding-dino-tiny
python main.py --method 1 --input examples/videos --output outputs/method1 --detector-model-dir models/grounding-dino-tiny
```

### Pruebas

```bash
python -m unittest discover -s tests
python scripts/validate_publication.py
```

Las pruebas incluyen fixtures generados durante la ejecución y no necesitan los
videos originales. El segundo comando revisa higiene de publicación, no precisión
del detector. El Dockerfile permite construir el entorno; no incluye modelos.

## 🧪 Training your own segmentation model

El detector de maquinaria se ajustó con anotaciones aprobadas en Label Studio. SAM2 ayudó a generar propuestas de máscaras para revisión humana; **no se entrenó SAM2 como detector automático de pretiles**.

Hubo dos entrenamientos de YOLO11n-seg. El Método 2 usa `train_v2`, inicializado desde el checkpoint `best.pt` de `train_v1`, en lugar de entrenar desde cero.

Este comando expresa los parámetros principales registrados en `train_v2/args.yaml`, con rutas relativas de ejemplo; requiere disponer del checkpoint anterior y de un dataset de segmentación propio. No reproduce por sí solo todo el entorno original.

```bash
yolo segment train model=weights/train_v1/best.pt data=dataset/dataset.yaml epochs=30 patience=10 imgsz=960 batch=4 seed=42 device=0 workers=2 optimizer=auto deterministic=True project=runs/segment name=train_v2
```

| Parámetro documentado | Valor |
|---|---|
| Imágenes de entrenamiento / validación | 70 / 16 |
| Clases | Camion_Mina, Bulldozer, Cargador_Frontal, Excavadora, Otra_Maquinaria |
| Épocas / paciencia | 30 / 10 |
| Tamaño de entrada / lote | 960 px / 4 |
| Semilla | 42 |
| Backbone congelado | No: `freeze: null` |
| Optimizador | Automático |

A diferencia del ejemplo de ajedrez, aquí no se documenta un backbone congelado ni una política explícita de learning rate reducido. Para adaptar el detector a otra faena, se necesita un dataset propio revisado y una validación separada por video o cámara: el experimento original comparte videos entre entrenamiento y validación y **no demuestra generalización**.

## 💡 Aprendizajes

- **Pasar pruebas no basta:** resultados con tests en verde fueron rechazados por revisión visual. Los contratos de código y la inspección del video detectan problemas distintos.
- **Una propuesta aprobada no es una etiqueta independiente:** distinguir máscaras dibujadas a mano de las generadas por SAM2 evitó evaluar el modelo contra sus propias predicciones y obligó a corregir la evaluación nocturna.
- **Segmentar no es reconocer:** SAM2 sigue lo que se le indica, pero no identifica automáticamente el pretil de una escena nueva. La semilla es una dependencia que debe quedar explícita.
- **La memoria necesita límites:** conservar una referencia diurna puede ayudar de noche, pero trasladarla después de un cambio de escena produce geometría incorrecta. Hay que registrar su origen y cuándo deja de ser válida.
- **Los metros requieren supuestos defendibles:** referencias dimensionales, perspectiva e incertidumbre importan más que una conversión global de píxeles. Sin calibración externa, el resultado sigue siendo una estimación.
- **N/D también es un resultado útil:** cuando la geometría falla, declarar incertidumbre es preferible a mostrar un verde que sugiera seguridad sin evidencia.
- **El tracking no se valida contando IDs:** más identificadores pueden representar fragmentación, no más vehículos. Se necesitan trayectorias anotadas para medir su calidad.
- **Un dataset pequeño puede dar métricas engañosas:** repartir cuadros de los mismos videos entre entrenamiento y validación no demuestra rendimiento en otra cámara o faena.
- **El material sintético condiciona los resultados:** los clips comprimen el tiempo, tienen oclusiones rápidas, cambios abruptos de luz y camiones que se transforman, aparecen de la nada o cambian de apariencia entre cuadros. Esas discontinuidades pueden romper un seguimiento sin corresponder a un movimiento físico real; no deben atribuirse automáticamente al detector o al tracker.
- **La IA acelera, la evidencia decide:** anotación asistida, agentes y MCP ayudaron a construir y revisar el sistema; las decisiones finales se contrastaron con código, imágenes, pruebas y procedencia.

## ⚠️ Límites del experimento

Cuatro videos sintéticos, aproximadamente 40 segundos y 1.022 cuadros. De 42 máscaras aprobadas del pretil, sólo 11 se dibujaron a mano. Las propuestas del modelo no se consideran automáticamente verdad de terreno.

**Condiciones del material:** videos generados por IA con acción y transiciones aceleradas, oclusiones rápidas y *morphing* de maquinaria, incluyendo apariciones repentinas. El corpus documentado tiene 24–30 FPS nominales, pero pocos cuadros para describir cambios tan comprimidos: no equivale a observar esos movimientos a velocidad natural. La oscuridad y la compresión también reducen la evidencia visual. Estos resultados describen el comportamiento sobre ese material, no una validación con cámaras reales en faena.

- La validación de YOLO comparte videos con el entrenamiento: no demuestra generalización.
- No hay verdad de terreno de identidades ni distancias medidas.
- SAM2 requiere semilla; un video nuevo sin ella deja el pretil en N/D.
- La memoria nocturna falla en algunos casos documentados y no está validada de forma general.
- Los metros dependen de referencias y FOV supuesto, sin calibración externa.
- Las capturas seleccionadas explican la salida; no son una evaluación completa ni evidencia de producción.

## 📚 Documentación

- [Plan de publicación](PUBLICATION_PLAN.md)
- [Ingeniería asistida por IA](AI_ASSISTED_ENGINEERING.md)
- [Mapa de procedencia](docs/PROVENANCE_MAP.md)
- [Limitaciones y honestidad experimental](docs/LIMITATIONS.md)
- [Auditoría antes del primer commit](docs/PRE_PUBLISH_AUDIT.md)

## 🙏 Built with

[Ultralytics YOLO11](https://github.com/ultralytics/ultralytics) · [SAM 2.1](https://github.com/facebookresearch/sam2) · [PyTorch](https://pytorch.org/) · [OpenCV](https://opencv.org/) · [SciPy](https://scipy.org/) (asignación húngara) · [NumPy](https://numpy.org/) · [Label Studio](https://labelstud.io/) · [Docker](https://www.docker.com/)
