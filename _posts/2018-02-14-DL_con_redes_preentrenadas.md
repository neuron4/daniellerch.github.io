---
layout: post
title: Deep Learning con redes pre-entrenadas en ImageNet
published: true
---

En esta receta vamos a ver como usar redes pre-entrenadas usando la base de datos de imágenes [ImageNet](http://www.image-net.org/). Esto nos permite disponer de un sistema de clasificación de imágenes bastante rápido y sin demasiadas complicaciones. Además, este sistema puede ser usado como base para entrenar usando otro tipo de imágenes sin tener que partir de cero.

## ¿Qué es ImageNet?
ImageNet es un proyecto que proporciona una gran base de datos de imágenes con sus correspondientes anotaciones indicando el contenido de las imágenes. Una base de datos de este tipo es imprescindible para la investigación en visión artificial.

Estas imágenes son usadas en las competiciones ILSVRC.

## ¿Que es ILSVRC?
ILSVRC o ImageNet Large Scale Visual Recognition Challenge es una competición anual organizada por el equipo de ImageNet. En ella diferentes equipos de investigación prueban sus algoritmos de reconocimiento visual. En las competiciones se usa un subconjunto de imágenes de ImageNet que contiene 1,2 millones de imágenes de 1000 clases diferentes.

El resultado de esta competición es una buena manera de saber que redes son las mejores en clasificación de imágenes.

## ¿Qué es una red pre-entrenada?
Durante el entrenamiento de una red neuronal se calculan los pesos óptimos para la clasificación, asociados a las conexiones entre neuronas. Esos pesos determinan el funcionamiento de la red. Después de entrenar la red podemos guardar los pesos, así como la arquitectura de la red. De esta manera, al usar la red en el futuro, no tenemos que volver a entrenarla.

Por otra parte, cabe tener en cuenta que el entrenamiento de una red es largo y tedioso, y requiere una gran cantidad de recursos a nivel computacional. Principalmente, cuando hablamos de redes de reconocimiento de imágenes como las que se usan en el ILSVRC.

Por este motivo, una forma de reducir el consumo de recursos es partir de una red pre-entrenada. Así, a parte de la opción de trabajar directamente con una red pre-entrenada capaz de clasificar con una precisión del estado del arte, también podemos usar esta red como base para nuestros modelos. El uso de una red pre-entrenada y su adaptación a unas necesidades concretas se conoce como *fine tuning*.

## ¿Qué red usar?
De las diferentes arquitecturas que existen actualmente en el estado del arte podemos encontrar algunas pre-entrenadas en Keras. De hecho, en la documentación de [Keras](https://keras.io/) podemos encontrar incluso una comparativa entre los [diferentes modelos](https://keras.io/applications/).

En base a la comparativa mencionada, en la receta de hoy vamos a usar la red [InceptionResNetV2](https://keras.io/applications/#inceptionresnetv2) por ser la que mejor precisión ofrece, aún y cuando el tamaño que ocupa el modelo es significativamente grande.

## Carga de una red pre-entrenada
En la documentación de Keras podemos ver los módulos en los que se encuentra cada red pre-entrenada. De estos módulos debemos importar la clase correspondiente a la red que queremos usar y las funciones preprocess_input() y decode_predictions(). Todos los módulos las tienen.

Por ejemplo, para la red ResNet50 haríamos lo siguiente:

```python
from keras.applications.resnet50 import \ 
    ResNet50, preprocess_input, decode_predictions
```
    
De la misma forma, para la red que queremos usar, haremos:

```python
from keras.applications.inception_resnet_v2 import \
    InceptionResNetV2,preprocess_input, decode_predictions
```


## Predicción de una clase
Primero leemos la imagen que queremos clasificar:

```python
from keras.preprocessing import image
img = image.load_img('aguila.jpg', target_size=(224, 224))
```

Para el ejemplo he usado la siguiente imagen:

<img class="image" src="{{ site.baseurl }}/images/eagle.jpeg"/>


A continuación, preparamos la imagen para pasársela a la red.

```python
model = InceptionResNetV2(weights='imagenet')
img = image.load_img(img_path, target_size=(224, 224))
x = image.img_to_array(img)
x = np.expand_dims(x, axis=0)
x = preprocess_input(x)
```

Finalmente, realizamos la predicción:

```python
>>> model.predict(x)
(u'n01614925', u'bald_eagle', 1.0)
```

## La receta

Aquí dejo la receta completa.

```python
from keras.applications.inception_resnet_v2 import InceptionResNetV2
from keras.preprocessing import image
from keras.applications.inception_resnet_v2 import preprocess_input, decode_predictions
import numpy as np

model = InceptionResNetV2(weights='imagenet')

img_path = 'aguila.jpg'
img = image.load_img(img_path, target_size=(224, 224))
x = image.img_to_array(img)
x = np.expand_dims(x, axis=0)
x = preprocess_input(x)

preds = model.predict(x)
print 'Prediction:', decode_predictions(preds, top=1)[0][0]
```







