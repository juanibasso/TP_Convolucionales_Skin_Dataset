# TP4 - Redes convolucionales para clasificar lesiones de piel (ISIC)

Clasificación de imágenes dermatoscópicas de **9 clases** de lesiones de piel (dataset ISIC, 878 imágenes originales) con una MLP de referencia y una CNN. El trabajo incluye análisis exploratorio, eliminación de duplicados, split estratificado, comparación MLP vs. CNN, decisiones de arquitectura, data augmentation hecha a mano y evaluación final en test.

Todo el análisis está en un único notebook: `tp-convolucionales-en-skin-dataset.ipynb` (entregado **con los outputs de la corrida**).

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `tp-convolucionales-en-skin-dataset.ipynb` | Notebook ejecutado, con código, outputs y análisis |
| `requirements.txt` | Dependencias de Python |
| `README.md` | Este archivo |
| `.gitignore` | Evita subir el dataset y los modelos entrenados |

**El dataset y los modelos entrenados no se incluyen en el repositorio.**

## Cómo reproducirlo

### 1. Conseguir el dataset
Se necesita la carpeta `data/Split_smol`, con `train/` y `val/`, y una subcarpeta por clase dentro de cada una (9 clases). Colocala en `data/Split_smol` junto al notebook, o apuntá a ella con la constante `DATA_DIR`.

### 2. Opción A: Kaggle (como se ejecutó originalmente)
1. Subir el dataset como un Dataset de Kaggle y agregarlo al notebook como input.
2. Importar el notebook y ajustar `DATA_DIR` a la ruta del input. En la corrida entregada fue:
   `/kaggle/input/datasets/juanibasso/dataset-skin/data/Split_smol`
3. Activar GPU (opcional) y ejecutar todas las celdas con **Run All**.

### 2. Opción B: local
```bash
git clone https://github.com/cselmo/CNN-en-Skin-Dataset.git
cd CNN-en-Skin-Dataset
python -m venv .venv
source .venv/bin/activate        # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
# colocar el dataset en data/Split_smol y ajustar DATA_DIR en la sección 0 si hace falta
jupyter notebook tp-convolucionales-en-skin-dataset.ipynb
```
Ejecutar las celdas de arriba hacia abajo. Funciona en CPU (el enunciado estima ~20-25 min) y en GPU (la corrida entregada tardó mucho menos).

## Constantes (sección 0)

| Constante | Valor | Nota |
|---|---|---|
| `DATA_DIR` | ruta de Kaggle | **Único valor modificado** respecto del notebook base |
| `SIZES` | `[32, 64, 96]` | por defecto |
| `SEEDS` | `[0, 1, 2]` | por defecto |
| `MAX_EPOCHS` / `PATIENCE` | `60` / `10` | por defecto |
| `BATCH` | `32` | por defecto |
| `AUG_COPIES` | `2` | por defecto |
| `SEED_SPLIT` | `42` | por defecto (no se cambia) |
| `SEEDS_FINAL` | `[0, 1, 2, 3, 4]` | por defecto |

Entorno de la corrida entregada: Kaggle, GPU (`cuda`), Python 3.13, PyTorch 2.11.0.

## Metodología

- Se eliminan duplicados exactos por hash MD5 (878 → 844 imágenes) y se verifica con un hash perceptual que no queden casi-duplicados.
- Split estratificado 60/20/20 con semilla fija: **506 train / 169 val / 169 test**.
- Normalización por canal con media y desvío calculados **solo con train**.
- La data augmentation se aplica **solo a train y después del split**, con transformaciones escritas a mano (flips, rotaciones de 90°, brillo/contraste).
- Val y test nunca se aumentan. Todas las decisiones (arquitectura, augmentation) se toman con validación.
- El **test se usa una sola vez**, al final (sección 8).
- Las comparaciones reportan media ± desvío de 3 semillas, y una diferencia solo se considera real si supera ese ruido y el error estándar de la validación/test.

## Resultados principales

Configuración final (elegida con validación): CNN de 4 bloques, 16 filtros, kernel 3, 64 px, con augmentation `flips+rot90`.

| Modelo | Parámetros | Test accuracy | Test F1 macro |
|---|---|---|---|
| CNN final | 361 113 | 0.679 ± 0.043 | 0.672 ± 0.045 |
| MLP 64 px | 6 358 793 | 0.483 ± 0.043 | 0.454 ± 0.057 |

(5 semillas; error estándar binomial del test ≈ 0.036). El recall de melanoma de la CNN (semilla 0) es bajo: **0.25** (5 de 20). El análisis completo está en las celdas ✏️ del notebook.

## Reproducibilidad

Los resultados pueden variar levemente entre ejecuciones, incluso con las mismas semillas, por el no determinismo de las operaciones en GPU. Por ejemplo, la CNN base dio una val_acc de 0.613, 0.637 y 0.621 en tres celdas distintas de la misma corrida. Por eso las diferencias pequeñas entre configuraciones (menores a ~0.04) no se consideran concluyentes.

## Limitaciones

Dataset chico (169 imágenes en val y en test, 13-20 por clase), una sola partición, 3-5 semillas, y posibles atajos del dataset (la resolución y el color medio se asocian con la clase). No hay ID de paciente, por lo que no se puede garantizar que imágenes de una misma lesión no queden en particiones distintas.

## Uso de IA

Declarado en la última celda del notebook.
