# Mapa de procedencia

| Fuente local | Uso en la publicación | Tratamiento |
|---|---|---|
| `02_METODO_1_FINAL` | Método 1 aprobado y congelado | Extraer sólo código y documentación redactada |
| `03_METODO_2_FINAL` | Runner y componentes integrados | Código propio publicado sin pesos ni salidas privadas |
| `00_INDICE_Y_APRENDIZAJES` | Cronología y lecciones | Adaptar rutas locales y revisar confidencialidad |
| `05_EVIDENCIA` | Procedencia y validaciones | Conservar resúmenes, no datos originales |
| `07_EXPERIMENTOS` | Historial de desarrollo | Mantener fuera del repositorio público |
| `08_HERRAMIENTAS` | Label Studio/SAM-2 local | Documentar proceso; no subir entornos ni bases |
| `01_ENTREGA` | Artefacto enviado | No copiar al repositorio público |

La publicación debe tener una procedencia suficiente para explicar el trabajo,
pero no una copia de los datos de la empresa o del proceso de selección.

## Capturas autorizadas del Método 2

El responsable autorizó las capturas el 5 de octubre de 2026 y confirmó que los videos
son generados por IA. Se extrajeron fotogramas de salidas existentes, sin repetir
inferencia ni modificar overlays.

| Imagen en `docs/images/` | Origen relativo a `03_METODO_2_FINAL` | Tiempo |
|---|---|---|
| `method2-video02-seeded-01s.png` | `salida_video02_con_mascaras/video_02/method_2/processed_with_osd.mp4` | 1 s |
| `method2-video02-seeded-03s.png` | misma corrida con máscaras aprobadas | 3 s |
| `method2-video02-seeded-05s.png` | misma corrida con máscaras aprobadas | 5 s |
| `method2-video02-04s.png` | `salidas_4_videos/video_02/method_2/processed_with_osd.mp4` | 4 s |

Sus respectivos `metadata.json` registran 12,5081 FPS con máscaras y 25,7145 FPS
sin máscaras. No se mezclan métricas entre corridas. Son tiempos de procesamiento,
no métricas de precisión.

La narrativa adapta el README de entrega, el reporte de benchmark y los documentos
curados. Para la geometría final se usa el reporte: la tabla del README de entrega
conserva una referencia anterior a escala global que no se reproduce aquí.

## Gráficos y explicación de altura

Los tres gráficos `video02-berm_height_over_time.png`,
`video02-minimum_distance_matrix.png` y `video02-vehicle_spatial_distribution.png`
son copias sin modificación de los archivos homónimos de
`salida_video02_con_mascaras/video_02/method_2/`.

Los ejemplos numéricos de altura proceden de las filas 0 y 239 del
`berm_height_series.csv` de esa corrida. El horizonte y altura de cámara proceden
de su `metadata.json`. La fórmula, medición de banda, intervalo y veredicto se
contrastaron con `metodo2/method2_berm_height/height.py`; las referencias de clase
con la sección 3.2 del reporte de benchmark. La cifra de altura del sprint 18
se identifica como otra corrida, no se atribuye a estos gráficos.

La autorización se limita a estas capturas y gráficos; no publica videos completos, pesos ni
datasets. La selección visual no demuestra generalización ni seguridad operativa.
