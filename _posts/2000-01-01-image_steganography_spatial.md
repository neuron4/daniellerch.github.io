---
layout: post
title: Practical Image Steganography in the Spatial Domain.
comments: true
---

## Practical Image Steganography in the Spatial Domain. 

To hide information in the spatial domain (that is, in the bitmap that represents the image) is usual to modify the least significant bit of some pixels. This technique has evolved significantly over the years, from the now insecure LSB replacement [[1](#5-references),[2](#5-references)] to the modern techniques based on adaptive schemes. In this article we will see different techniques to hide information inside images with practical examples in Python. 

[ Updated on 2017-05-01 ]

<br>

1. [Basic information hiding](#1-basic-information-hiding)

2. [Matrix Embedding](#2-matrix-embedding)

3. [Wet Paper Codes](#3-wet-paper-codes)

4. [Other methods](#4-other-methods)

5. [References](#5-references)

<br>

### 1. Basic information hiding:

The most basic technique to hide information in the spatial domain is the replacement of the least significant bit (LSB) of the pixel by a bit of the message. By this way we can hide at most one bit per pixel. When we hide information in all the pixels we say we are using a bitrate of 100% or bit rate 1. This modifies significantly the statistics of the image and, in consequence, this is easy to detect. As far as the main objective of steganography is undetectability it is preferable to use lower bitrates. 

By the other side, the replacement of the LSB is an asymmetrical operation. When we replace the LSB of an even pixel this produces the same effect of adding one when we replace by one or does not produce any effect when we replace by zero. Similarly, when we replace the LSB of an odd pixel this produces the same effect of subtracting one when we replace by zero or does not produce any effect when we replace by one. If we plot an histogram of the pixel intensities we can see the effects of the embedding. This is because the bars that represent the number of pixels of an even value grow, and the bars that represent the number of pixels of an odd value grow. Therefore, if we hide enough data, for example using bitrate 1, the bars tend to have the same height. This is how the Histogram Attack works [[3](#5-references)] and it is the basis for subsequent attacks that knocked out LSB replacement [[1](#5-references),[2](#5-references)]. So, we are not going to use LSB replacement. Instead we are going to use LSB matching. LSB matching is very similar to LSB replacement, our target is to change the value of the LSB. But in LSB matching, instead of replacing the LSB we change it adding or substracting one. The only difference is that we are using an operation with carriage that is symmetrical. These small variation makes LSB matching much harder to detect.

Let's consider the Baboon's image in grayscale:
![Baboon]({{ site.baseurl }}/images/baboon.png)

We extract a block of 3x3 pixels from the upper left corner.

| 160 | 60 | 53 |
| 128 | 111 | 43 |
| 84 | 125 | 51 |

And we obtain its binary code:

| 10100000 | 00111100 | 00110101 |
| 10000000 | 01101111 | 00101011 |
| 01010100 | 01111101 | 00110011 |


Let's suppose that we want to hide the A letter in ASCII code. That is, we want to hide the binary number 01000001. 



### 2. Matrix Embedding

asdas
asdas
asdas

### 3. Wet Paper Codes 

asdas
asdas
asdas

### 4. Generative Schemes based on Deep Learning

asdas
asdas
asdas

### 5. References

[1]. Reliable Detection of LSB Steganography in Color and Grayscale Images. Jessica Fridrich, Miroslav Goljan and Rui Du.
Proc. of the ACM Workshop on Multimedia and Security, Ottawa, Canada, October 5, 2001, pp. 27-30. 

[2]. Detection of LSB steganography via sample pair analysis. S. Dumitrescu, X. Wu and Z. Wang. IEEE Transactions on Signal Processing, 51 (7), 1995-2007.

[3]. Attacks on Steganographic Systems. A. Westfeld and A. Pfitzmann. Lecture Notes in Computer Science, vol.1768, Springer-Verlag, Berlin, 2000, pp. 61−75. 


### xxx

bla bla bla
asd
sa
$$
\sum_{i=1}^{\inf}\fraq{1}{i}
$$

```python
import numpy as np
a=np.array()
print "holas"
```
| P | Q | P * Q |
| - | - | - |
| T | T | T |
| T | F | F |
| F | T | F |
| F | F | F |
