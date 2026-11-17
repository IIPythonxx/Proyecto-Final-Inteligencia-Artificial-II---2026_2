# Proyecto-Final-Inteligencia-Artificial-II---2026_2
Repositorio con los archivos correspondientes a nuestro proyecto final de IA2.


# Clasificación de retinopatía diabética en fotos de fondo de ojo (APTOS 2019)

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USUARIO/REPO/blob/main/IA2_DR_APTOS_final.ipynb)

Proyecto final de **Inteligencia Artificial II**, Universidad Industrial de Santander (UIS).
Profesor: Luis Carlos Guayacán.

- Camilo Andrés Herrera Celis (2231530)
- Camilo Andrés Bracho Rueda (2231517)

## Resumen

Clasificamos fotos de fondo de ojo en los 5 niveles de la escala internacional de retinopatía diabética (0 sin retinopatía, 1 leve, 2 moderada, 3 severa y 4 proliferativa). Comparamos con el mismo protocolo:

- **CNNs desde cero:** CNN simple, AlexNet y SqueezeNet 1.1, las dos últimas con BatchNorm.
- **CNNs preentrenadas en ImageNet:** DenseNet121, EfficientNetB0 y MobileNetV2.
- **Transformers:** ViT desde cero, un híbrido DenseNet121 + Transformer, y DeiT-Small y Swin-Tiny preentrenados (KerasHub).
- **Flujo de dos etapas:** una etapa 1 decide sano o enfermo y una DenseNet121 clasifica la severidad de 1 a 4. Probamos dos etapas 1: un autoencoder entrenado solo con ojos sanos y un clasificador binario (EfficientNetB0).
- **Regresión con cortes:** EfficientNetB0 con una sola salida que predice la severidad como un número, y cortes entre clases fijos u optimizados en validación.

Además, para ver en qué se fija cada modelo, programamos varios mapas de calor con `tf.GradientTape`: Grad-CAM, Grad-CAM++, Score-CAM, EigenCAM, LayerCAM, Integrated Gradients, Attention Rollout y el mapa de error del autoencoder.

## Resultados

Test de 550 imágenes (15% del dataset, estratificado), que no se usaron ni para entrenar ni para elegir nada. La métrica principal es el **QWK** (Kappa de Cohen con pesos cuadráticos), la métrica oficial de APTOS.

| Modelo | Tipo | Params (M) | Accuracy | F1 macro | QWK |
|---|---|---:|---:|---:|---:|
| **Swin-Tiny** | transformer preentrenado | 27.5 | 0.793 | **0.650** | **0.906** |
| Híbrido DenseNet + Transformer | CNN preentrenada + transformer | 5.7 | 0.787 | 0.629 | 0.886 |
| Regresión EfficientNetB0 (cortes optimizados) | CNN preentrenada, 1 salida | 4.1 | 0.745 | 0.511 | 0.883 |
| Regresión EfficientNetB0 (cortes fijos) | CNN preentrenada, 1 salida | 4.1 | 0.747 | 0.564 | 0.880 |
| Dos etapas: binario + DenseNet121 | EfficientNetB0 + DenseNet121 | 11.1 | 0.785 | 0.614 | 0.870 |
| DeiT-Small | transformer preentrenado | 21.7 | 0.795 | 0.618 | 0.870 |
| EfficientNetB0 | CNN preentrenada | 4.1 | **0.798** | 0.615 | 0.859 |
| DenseNet121 | CNN preentrenada | 7.0 | 0.760 | 0.589 | 0.842 |
| SqueezeNet 1.1 + BN | CNN desde cero | 0.7 | 0.745 | 0.586 | 0.820 |
| CNN simple | CNN desde cero | 1.2 | 0.769 | 0.513 | 0.795 |
| AlexNet + BN | CNN desde cero | 58.3 | 0.720 | 0.453 | 0.744 |
| MobileNetV2 | CNN preentrenada | 2.3 | 0.724 | 0.424 | 0.723 |
| ViT | transformer desde cero | 2.0 | 0.689 | 0.412 | 0.714 |
| Dos etapas: autoencoder + DenseNet121 | detección de anomalías | 7.6 | 0.504 | 0.347 | 0.273 |
| *Dos etapas con etapa 1 perfecta (referencia)* | | 7.6 | 0.813 | 0.647 | 0.890 |

Entre corridas los resultados varían hasta 0.03 de QWK, así que los primeros puestos (entre 0.87 y 0.91) forman un grupo de cabeza más que un orden estricto.

### Hallazgos principales

- **Swin-Tiny es el mejor modelo** en QWK (0.906) y F1 macro (0.650), y el que mejor acierta la clase 1 (70%).
- **Preentrenar es lo que más pesa:** los transformers preentrenados (Swin 0.906, DeiT 0.870) superan por mucho al ViT desde cero (0.714).
- **Más parámetros no es mejor:** el híbrido logra 0.886 con 5.7 M de parámetros, y SqueezeNet (0.7 M) supera a AlexNet (58 M).
- **Etapa 1 supervisada:** el autoencoder no separó sanos de enfermos (AUC 0.548 en test), porque el disco óptico y los vasos también se reconstruyen mal en ojos sanos. El clasificador binario llega a AUC 0.998 y sube el QWK del flujo de 0.273 a 0.870, cerca de su techo de 0.890.
- **Regresión:** predecir la severidad como un número sube la misma EfficientNetB0 de 0.859 a 0.880. Optimizar los cortes en validación casi no ayudó en test (0.883) y bajó el F1 macro, porque los cortes se ajustaron a la validación.
- **Mapas de calor:** los mejores modelos miran las lesiones (exudados y hemorragias). Algunos marcan el disco óptico o el borde de la foto, que podrían ser atajos. Esto último es una hipótesis que no comprobamos.

## Contenido del repositorio

| Archivo | Qué es |
|---|---|
| `IA2_DR_APTOS_final.ipynb` | Notebook completo, ya ejecutado en Colab con todas sus salidas: datos, modelos, evaluación, mapas de calor y conclusiones |
| `Presentacion_DR_final.html` | Presentación final (12 diapositivas). Es un solo archivo con las imágenes incluidas: se descarga y se abre en el navegador |
| `README.md` | Este archivo |

Para ver la presentación: descargar el `.html` y abrirlo en el navegador. Se avanza con las flechas del teclado y `F` activa la pantalla completa. La tabla se puede ordenar y las matrices de confusión cambian de modelo con los botones.

Si GitHub no muestra el notebook por su tamaño, se puede abrir con el botón de Colab de arriba o en [nbviewer](https://nbviewer.org/).

## Cómo ejecutar el notebook

1. Abrir el notebook en Google Colab y elegir `Entorno de ejecución → Cambiar tipo de entorno → T4 GPU`.
2. Tener un `kaggle.json` (Kaggle → Settings → API → Create New Token) y haber aceptado las reglas de la competencia [APTOS 2019 Blindness Detection](https://www.kaggle.com/competitions/aptos2019-blindness-detection). La celda de la sección 4 lo pide para descargar los datos. Los pesos de DeiT y Swin también se descargan de Kaggle a través de KerasHub.
3. La sección 1 instala `kaggle` y actualiza `keras` y `keras-hub`, porque Swin necesita `keras-hub` 0.30 o más. Si la sesión ya tenía una versión vieja cargada, hay que reiniciar la sesión y volver a correr desde arriba.
4. La sección 3 monta Google Drive. Ahí se guardan pesos, historias de entrenamiento, tiempos, el caché del preprocesamiento y la tabla de resultados (`/content/drive/MyDrive/IA2_DR`).
5. `SOLO_CARGAR` (sección 2):
   - `True`: cada modelo carga sus pesos desde Drive si existen, y solo entrena los que falten.
   - `False`: entrena todo desde cero. En una T4 son unas 3 horas en total; cada modelo tarda entre 10 y 20 minutos.

Los pesos entrenados no están en el repositorio. La primera vez el notebook entrena todo y los deja en Drive; desde ahí, con `SOLO_CARGAR = True`, una ejecución completa toma unos 30 a 40 minutos (la mayor parte es la descarga del dataset y los mapas de calor).

### Entorno usado

- Google Colab con GPU NVIDIA T4 (16 GB) y mixed precision (`float16`).
- TensorFlow 2.21, Keras 3.15 y KerasHub 0.33.
- scikit-learn, OpenCV, pandas, matplotlib y seaborn, que ya vienen en Colab.

## Metodología resumida

- **Datos:** APTOS 2019 (Kaggle), 3.662 imágenes etiquetadas, muy desbalanceadas (49% sanos, solo 5% de clase 3). Partición estratificada 70 / 15 / 15: 2.563 de entrenamiento, 549 de validación y 550 de test.
- **Preprocesamiento:** recorte del borde negro, redimensión a 224×224 y filtro de Ben Graham, que resalta vasos y lesiones. Se calcula una vez y queda en caché.
- **Entrenamiento:** el mismo protocolo para todos los modelos:
  - Augmentation: giros, rotación, zoom, brillo y contraste.
  - Pesos por clase (raíz del inverso de la frecuencia) y label smoothing de 0.05.
  - AdamW con warmup, cosine decay y recorte de gradiente.
  - Se guarda la época con mejor QWK en validación.
- **Selección:** la mejor época, el umbral de la etapa 1 y los cortes de la regresión se eligen en validación. El test se usa solo para reportar.
- **Métricas:** QWK como principal, más accuracy, precision, recall y F1 macro, matrices de confusión normalizadas por fila, y AUC para la decisión sano / enfermo.

## Limitaciones y trabajo futuro

- Una sola partición: con unas 29 imágenes de clase 3 en test, los resultados son ruidosos. La validación cruzada daría números más estables y cortes de regresión menos sobreajustados.
- Imágenes de 224×224: los microaneurismas de la clase 1 ocupan pocos píxeles. Probar 384 o 512.
- Ensamble de los mejores modelos (Swin, híbrido y regresión).
- Enmascarar el fondo de la foto para comprobar si los modelos usan el borde como atajo.
- No es una herramienta clínica: haría falta validarla con fotos de otras clínicas y cámaras.

## Referencias

- APTOS 2019 Blindness Detection, Kaggle: https://www.kaggle.com/competitions/aptos2019-blindness-detection
- Liu et al. (2021). *Swin Transformer: Hierarchical Vision Transformer using Shifted Windows*. ICCV.
- Touvron et al. (2021). *Training data-efficient image transformers & distillation through attention* (DeiT). ICML.
- Dosovitskiy et al. (2021). *An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale* (ViT). ICLR.
- Huang et al. (2017). *Densely Connected Convolutional Networks* (DenseNet). CVPR.
- Tan y Le (2019). *EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks*. ICML.
- Selvaraju et al. (2017). *Grad-CAM: Visual Explanations from Deep Networks via Gradient-based Localization*. ICCV.
- Sundararajan et al. (2017). *Axiomatic Attribution for Deep Networks* (Integrated Gradients). ICML.
- Abnar y Zuidema (2020). *Quantifying Attention Flow in Transformers* (Attention Rollout). ACL.
