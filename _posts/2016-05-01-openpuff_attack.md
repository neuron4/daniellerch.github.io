---
layout: post
title: ATS Attack to OpenPuff Image Steganography.
comments: true
---

## ATS Attack to OpenPuff Image Steganography. 

To hide information in the spatial domain (that is, in the bitmap that represents the image) is usual to modify the least significant bit of some pixels. This technique has evolved significantly over the years, from the now insecure LSB replacement [[1](#5-references),[2](#5-references)] to the modern techniques based on adaptive schemes. In this article we will see different techniques to hide information inside images with practical examples in Python. 

[ Updated on 2017-05-01 ]

<br>

1. [OpenPuff](#1-openpuff)
2. [Preparing data sets](#2-preparing-data-sets)
3. [The Attack](#3-the-attack)


<br>

### 1. OpenPuff

OpenPuff is a tool for hiding information in audio, images and video. We can read this in its website:

*"OpenPuff is a professional steganography tool, with unique features you won't find among any other free or commercial 
software. OpenPuff is 100% free and suitable for highly sensitive data covert transmission"*.

This tool has been some time in my list of Image Steganography Tools without security classification, because its hidding algorithm is secret. So I decided to perform a basic analysis to update the list.

### 2. Preparing Data Sets

I used the ATS method, presented in the following paper:

*"Unsupervised steganalysis based on artificial training sets, Daniel Lerch-Hostalot, 
David Megías. D. Engineering Applications of Artificial Intelligence, April 2016.[10.1016/j.engappai.2015.12.013]."*

The ATS method uses three sets: the set A (the original images), the set B and the set C.

In this basic example we use a set of images formed by cover and stego images and we want to recognise them.

The set A is formed by the original set of images. We do not know what images are cover or stego.

The set B is formed by the images of A after embedding a new message. The set C is formed by the images of B after embedding another message.

This multiple insertion is the base of ATS and it is used to detect cover and stego images using the the sets A an C as Artificial Training Sets (ATS) to train a classifier and classifying the set B. See the original paper for details.

Preparing a test (set A):

For the experiment we have prepared a set with 50 images from the BOSS database, a typical image database in the field of Steganalysis. Half of the images are used as cover, without modification. The other half is embedded using OpenPuff. We have used three random passwords for OpenPuff and we have prepared a random data file to be hidden:

$ head -c 65k </dev/urandom >dataA

We have hidden the information using a bitrate of 20%. The size of 65k is the maximum that OpenPuff allows to hide into the set of 25 selected images.

Preparing the set B and C:

The sets B an C are prepared by the steganalyst so we can not use the same password neither the same data. So we have chosen three different passwords and we have generated another file with random data to prepare the set B.

$ head -c 130k </dev/urandom >dataB

In this case we hide information into all the images of the set B so we more images and we can hide more information.

The procedure to hide information is the same as before.

To prepare the set C we do the same operation, this time we hide information into the images of the set B.


### 3. The Attack

A complete example of the ATS attack and the needed software can be found [here](https://github.com/daniellerch/papers_code/blob/master/ATS/README.md).

Once we have the sets A, B and C we can apply the ATS method. 

```bash
$ ./ATS_SVM_FS.py
./ATS_SVM_FS.py A B C labels.txt
Accuracy:  0.86
```

The accuracy of the method is 86%. 22 images were classified properly as stego and 21 images were properly classified as cover. 7 images were misclassified.





