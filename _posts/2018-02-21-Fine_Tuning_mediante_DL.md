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
