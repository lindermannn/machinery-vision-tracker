# Componentes de terceros

El código propio se publica bajo AGPL-3.0. Las dependencias conservan sus licencias
originales; esta publicación no las relicencia. No se incluyen sus pesos ni código
vendorizado.

| Componente | Uso | Licencia / fuente |
|---|---|---|
| Ultralytics YOLO11 | Segmentación de maquinaria | [AGPL-3.0](https://github.com/ultralytics/ultralytics/blob/main/LICENSE) |
| SAM 2.1 | Propuestas y propagación de máscaras en el desarrollo | [Apache-2.0 y avisos asociados](https://github.com/facebookresearch/sam2) |
| Grounding DINO | Detector del Método 1 | [Apache-2.0](https://github.com/IDEA-Research/GroundingDINO/blob/main/LICENSE) |
| Depth Anything V2 Small | Adaptador experimental conservado y tests de assets; no es el Método 2 final | [Apache-2.0 para Small](https://github.com/DepthAnything/Depth-Anything-V2) |
| PyTorch / torchvision | Inferencia | [BSD-3-Clause y avisos asociados](https://github.com/pytorch/pytorch/blob/main/LICENSE) |
| Transformers / Hugging Face Hub | Carga de modelos | [Apache-2.0](https://github.com/huggingface/transformers/blob/main/LICENSE) |
| NumPy / SciPy | Cálculo y asociación húngara | BSD-3-Clause |
| OpenCV | Video y procesamiento de imagen | Apache-2.0; revisar también componentes del wheel |
| Matplotlib | Gráficos | Licencia de Matplotlib y avisos asociados |
| PyYAML | Configuración | MIT |
| Pillow | Imágenes | MIT-CMU |

La versión pública integra los dos runners existentes. La propagación de SAM2
se realizó en el flujo de preparación: el runner del Método 2 recibe máscaras
externas, no ejecuta SAM2 automáticamente. Sin semilla el pretil queda en N/D.

Los checkpoints y datos deben obtenerse por separado con permisos y licencias
apropiados. Las imágenes del README son ejemplos sintéticos autorizados, no un
dataset de entrenamiento distribuido.
