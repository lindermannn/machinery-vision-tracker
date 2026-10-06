# Entradas propias

Los videos completos no se distribuyen. Para ejecutar el proyecto, utiliza videos
propios o material autorizado y checkpoints obtenidos por separado.

```bash
python main.py --method 2 --input examples/videos --output outputs/demo --weights models/yoloseg_maquinaria_train_v2.pt --device cpu
```

Crea `examples/videos` localmente y coloca allí tus clips. El checkpoint debe ser
compatible con las clases de maquinaria del proyecto. Sin máscaras externas del
pretil, el Método 2 declara N/D; no ejecuta SAM2 automáticamente.
