---
layout: post
title: Fine-tuning en reconocimiento de imágenes mediante Deep Learning
published: false
---

En esta receta vamos a ver como hacer *fine tuning* de una red pre-entrenada para el reconocimiento de objetos. Nuestro objetivo será adaptarla a un problema diferente.

Como ejemplo vamos a usar la red pre-entrenada InceptionResNetV2 con [ImageNet](http://www.image-net.org/) mediante [Keras](https://keras.io/). La vamos a ajustar para clasificar flores, algo para lo cual el clasificador inicial no está preparado.


## Descripción del problema
Partiremos de un conjunto de datos de imágenes de flores. Puedes descargarlo [aquí](http://download.tensorflow.org/example_images/flower_photos.tgz).

Este tipo de imágenes no es detectado por las redes entrenadas con ImageNet. Usemos como ejemplo la siguiente imagen:

![flower]({{ site.baseurl }}/images/flower.jpeg)

Partiendo del ejemplo usado en la receta [“Deep Learning con redes pre-entrenadas en ImageNet”](http://http://blog.neuron4.com/DL_con_redes_preentrenadas/) podemos ver que predice la red.

Nos encontramos con lo siguiente:

```python
$ python inception_resnet_v2.py
...
Prediction: (u'n04286575', u'spotlight', 0.9960238)
```

Como es lógico la red no predice la flor (diente de león), pues no se encuentra entre la lista de clases de ImageNet usadas para entrenar.

NOTA: Si realizas la prueba, no olvides modificar la ruta de la imagen sobre la que realizar la predicción.


## ¿Por qué usar fine-tuning?
El motivo principal para usar *fine-tuning* es el ahorro en recursos de computación y tiempo, pues evitamos la mayor parte del entrenamiento.

Si disponemos de acceso a una red pre-entrenada que resuelva un problema similar al que queremos resolver, podemos usar esa red como punto de partida. Simplemente la adaptamos a nuestro problema y continuamos entrenando con nuestros datos. La red ya ha aprendido a extraer las características universales necesarias, por lo que la mayor parte del trabajo ya está hecho.

## Técnicas de fine-tuning
Si disponemos de una red pre-entrenada para un problema similar y disponemos de muchos datos de entrenamiento para nuestro problema concreto, estamos ante el caso ideal. Simplemente usaremos la red pre-entrenada como punto de partida y continuaremos el entrenamiento con nuestros datos.

Si disponemos de una red pre-entrenada para un problema similar pero tenemos pocos datos de entrenamiento debemos proceder diferente. Entrenar una red con pocos datos nos llevará fácilmente a una situación de overfitting, donde la red aprende a clasificar los datos de entrenamiento pero no los de test (las redes neuronales suelen necesitar muchos datos para poder aprender a generalizar). En este caso entrenaremos solo las últimas capas de la red, congelando el resto. La red ya está entrenada para un problema similar, por lo que la usaremos principalmente como extractor de características.

Si, por otra parte, la red de partida no es todo lo similar a nuestro problema que nos gustaría, la situación se complica. Si disponemos de muchos datos de entrenamiento, la mejor solución suele ser entrenar la red desde cero.

Pero si disponemos de pocos datos de entrenamiento la opción anterior no suele se viable debido al overfitting, a menos que podamos usar con éxito alguna técnica de [data augmentation](https://keras.io/preprocessing/image/#imagedatagenerator). Ante esta situación, una opción suele ser congelar únicamente las primeras capas de la red. Estas capas han aprendido a extraer características universales por lo que pueden ser aprovechadas.

Como veremos más adelante, no suele ser fácil determinar si una red pre-entrenada es adecuada para nuestro problema. Por lo tanto, puede ser necesario hacer algunas pruebas hasta dar con los parámetros adecuados.


## Redes pre-entrenadas en Keras
Keras dispone de diferentes redes pre-entrenadas. En esta receta vamos a usar la red Inception Resnet V2. Remito al lector a una receta anterior en la que ya escribí cómo usar una red pre-entrenada de este tipo: [“Deep Learning con redes pre-entrenadas en ImageNet”](http://http://blog.neuron4.com/DL_con_redes_preentrenadas/).


## Preparación de los datos
Como decía anteriormente vamos a clasificar flores. Descarga y descomprime el conjunto de imágenes si no lo has hecho ya.

Las imágenes vienen en diferentes directorios, por lo que podemos usar el nombre del directorio como nombre de clase. Las podemos leer en Python con la siguiente función:

```python
from keras.preprocessing import image
from keras.applications.inception_resnet_v2 \
    import preprocess_input
import numpy as np
import glob
def load_data(path, pattern):
    class_names={}
    class_id=0
    x = []
    y = []
    for d in glob.glob(os.path.join(path, '*')):
        clname = os.path.basename(d)
        for f in glob.glob(os.path.join(d, pattern)): 
            if not clname in class_names:
                class_names[clname]=class_id 
                class_id += 1
            img = image.load_img(f, target_size=(224, 224))
            npi = image.img_to_array(img)     
            npi = preprocess_input(npi)
            for i in range(4):
                npi=np.rot90(npi, i)
                x.append(npi)
                y.append(class_names[clname])
    return np.array(x), np.array(y), class_names
```

La función asigna un identificador numérico a cada clase. Además, cada imagen leída es preparada y procesada para poder ser enviada a la red.

Además, cada imagen se rota 90 grados para obtener 4 imágenes de cada una. Para entrenar una red neuronal compleja, cuantas más imágenes mejor.

Podemos leer las imágenes y obtener los datos de la siguiente manera:

```python
>>> x, y, class_names = load_data('flower_photos', '*.jpg')
>>> print x.shape, y.shape, len(class_names)
(3670, 224, 224, 3) (3670,) 5
```


Tenemos 3670 imágenes de flores de 5 tipos diferentes.

Las separaremos en dos partes: un conjunto de entrenamiento y un conjunto de test para ver que tal funciona nuestro clasificador.

```python
>>> x_train, x_test, y_train, y_test = \
        train_test_split(x, y, test_size=0.1)
```


A continuación, preparamos las etiquetas para que Keras las entienda, como ya expliqué en: "[Introducción al Deep Learning con Keras](http://blog.neuron4.com/introduccion_al_DL/)".

```python
>>> from keras.utils import np_utils
>>> y_train = np_utils.to_categorical(y_train, len(class_names))
>>> y_test = np_utils.to_categorical(y_test, len(class_names))
>>> print x_train.shape, y_train.shape, x_test.shape, y_test.shape
(3303, 224, 224, 3) (3303, 5) (367, 224, 224, 3) (367, 5)
```

Ya tenemos los datos preparados.


## Carga de la red pre-entrenada
No entraré en demasiados detalles sobre como cargar una red pre-entrenada en Keras, dado que ya lo hice en: “Deep Learning con redes pre-entrenadas en ImageNet”.

Cargamos la red con los pesos de ImageNet:

```python
model = InceptionResNetV2(weights=’imagenet’, include_top=False)
```

A destacar el parámetro “include_top=False”. Este parámetro le indica a la red que no debe incluir la última capa de la red, destinada a realizar la predicción final. Esta capa está preparada para realizar las predicciones de todas las clases de ImageNet. Como nosotros queremos predecir únicamente nuestras 5 clases, queremos sustituir esta capa por una capa personalizada.


## Modificación de las última capa
La red cargada ya no dispone de la última capa. Por lo que vamos a añadirle una capa “softmax” de 5 clases. Esto nos va a permitir que la red haga la predicción de las clases que nos interesan y no de todas las que lleva de serie.

```python
x = model.output 
x = GlobalAveragePooling2D()(x) 
predictions = Dense(5, activation=’softmax’)(x) 
model = Model(input=model.input, output=predictions)
```

Necesitamos usar una capa “GlobalAveragePooling2D” para adaptar la salida de la capa anterior a la capa “softmax”.

## Congelado de capas
El siguiente paso consiste en decidir que capas congelar. No tenemos muchos datos (3303 imágenes), pero tampoco es que sean muy pocas. Es habitual tener que enfrentarse a problemas con menos de 500 imágenes. Por lo que es posible que la mejor opción no consista en congelar todas las capas.

Tendremos que hacer varias pruebas hasta encontrar un numero de capas a congelar adecuado. Para tomar la decisión de si un modelo es apropiado o no, consultaremos los datos de validación, no los de test. Los datos de test deben usarse solo al final para ver si el modelo funciona. Tomar decisiones de diseño en base a los resultados en el conjunto de test es un error grave, pues nos lleva a construir una solución que solo funciona con nuestros datos.

Por lo tanto, usamos el parámetro “validation_split” para indicarle a Keras que separe los datos de entrenamiento en una parte dedicada a entrenar la red y otra parte, el conjunto de validación, dedicada a validar los resultados (no incluida en el entrenamiento). Dejamos el conjunto de test para el final.

A continuación vemos como congelar capas (trainable=False) y como empezar a entrenar generando un conjunto de validación usando el 10% de las imágenes.

```python
LAYERS_TO_FREEZE=700
for layer in model.layers[:LAYERS_TO_FREEZE]:
    layer.trainable = False
model.compile(optimizer="adam", \
    loss='categorical_crossentropy', metrics=['accuracy'])
model.fit(x_train, y_train, batch_size=128, epochs=2, verbose=1, validation_split=0.1)
```

NOTA: Para ver los resultados que presento a continuación recomiendo ir directamente al final y usar el código de la receta completa. Simplemente, es necesario modificar el valor de la variable LAYERS_TO_FREEZE para cada experimento.


Empezaremos congelando la mayor parte de las capas. El modelo final tiene 782 capas.

```python
LAYERS_TO_FREEZE=700
```

Los resultados después de dos epoch son los siguientes:

```python
loss: 0.3671 - acc: 0.8755 - val_loss: 0.8164 - val_acc: 0.8404
loss: 0.1274 - acc: 0.9583 - val_loss: 0.6713 - val_acc: 0.8238
```


En el primer epoch ya vemos síntomas de overfitting. Sin embargo, los resultados del 84% son aceptables. Por lo que el modelo después de entrenar un epoch podría ser un buen candidato para nuestra solución final.

Veamos que ocurre si congelamos más capas.

```python
LAYERS_TO_FREEZE=750
```

Los resultados después de dos epoch son los siguientes:

```python
loss: 0.4001 - acc: 0.8556 - val_loss: 1.0808 - val_acc: 0.7738
```

Los resultados parecen empeorar congelando más capas. En este caso vemos síntomas graves de oferfitting.

Probemos pues congelando menos capas. Cuantas menos capas congeladas, más tiempo tarda el entrenamiento.

```python
LAYERS_TO_FREEZE=500
```

Los resultados empeoran:

```python
loss: 0.3714 - acc: 0.8810 - val_loss: 5.4157 - val_acc: 0.5673
loss: 0.2537 - acc: 0.9361 - val_loss: 3.4444 - val_acc: 0.7224
```

Parece que el valor óptimo podría estar cerca de los 700. Podría hacerse una búsqueda automatizada, pero aquí nos quedaremos con el primer modelo. Puedes experimentar con diferentes parámetros, si lo haces, agradecería comentases tus resultados en la sección de comentarios.

Es hora de probar que resultados obtenemos con el conjunto de test. Usaremos el modelo obtenido después de entrenar un epoch con 700 capas congeladas.

Los resultados obtenidos son:

```python
Testing set accuracy: 0.865122616129
```


Es más que probable que los resultados puedan mejorarse usando técnicas de *data augmentation* un poco más sofisticadas. Pero esto lo dejamos para otra receta.

Para esta receta, el punto importante con el que quedarse es el tiempo dedicado al entrenamiento. Como se puede ver, a medida que se reduce el número de capas congeladas, el tiempo de entrenamiento crece. Por lo que es muy interesante conseguir una red que funcione con muchas capas congeladas, como la que presentamos en la receta. Si se congelan pocas o ninguna ninguna capa, el tiempo de entrenamiento de cada epoch se dispara. Y si lo que se pretende es entrenar la red desde cero, ya es necesario acceder a grandes recursos (Clusters, GPUs, etc).

## La receta
Finalmente os dejo la receta.

```python
from keras.preprocessing.image import ImageDataGenerator
from keras.optimizers import SGD
from keras.models import Model
from keras.utils import np_utils
from keras.preprocessing import image
from keras.applications.inception_resnet_v2 import InceptionResNetV2
from keras.applications.inception_resnet_v2 import preprocess_input, decode_predictions
from keras.layers import Dense, GlobalAveragePooling2D
from sklearn.model_selection import train_test_split
import numpy as np
import os
import glob

def load_data(path, pattern):
    class_names={}
    class_id=0
    x = []
    y = []

    for d in glob.glob(os.path.join(path, '*')):
        clname = os.path.basename(d)
        for f in glob.glob(os.path.join(d, pattern)):
            if not clname in class_names:
                class_names[clname]=class_id
                class_id += 1

            img = image.load_img(f, target_size=(224, 224))
            npi = image.img_to_array(img)
            npi = preprocess_input(npi)
            for i in range(4):
                npi=np.rot90(npi, i)
                x.append(npi)
                y.append(class_names[clname])

    return np.array(x), np.array(y), class_names

x, y, class_names = load_data('flower_photos', '*.jpg')
num_classes = len(class_names)

x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=0.1)
y_train = np_utils.to_categorical(y_train, num_classes)
y_test = np_utils.to_categorical(y_test, num_classes)

model = InceptionResNetV2(weights='imagenet', include_top=False)

x = model.output
x = GlobalAveragePooling2D()(x)
predictions = Dense(num_classes, activation='softmax')(x) 
model = Model(input=model.input, output=predictions)

LAYERS_TO_FREEZE=700
for layer in model.layers[:LAYERS_TO_FREEZE]:
    layer.trainable = False

model.compile(optimizer="adam", loss='categorical_crossentropy', metrics=['accuracy'])

model.fit(x_train, y_train, batch_size=128, epochs=1, verbose=1, validation_split=0.1)
score = model.evaluate(x_test, y_test, verbose=0)
print 'Testing set accuracy:', score[1] 
```
