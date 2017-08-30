---
layout: post
title: How not to hide information inside an image
comments: true
---

## How Not to Hide Information Inside an Image. 

You can find a lot of tools on the Internet to hide information inside an image. Unfortunately, most of them are like 
hiding a safe behind a picture: it is not exactly the safest option. In this article I will try to show which steganographic techniques you should not use. 

The criteria for deciding whether a method is adequate or not is its detectability. In steganography and stegaganalysis if a method is detectable we consider it broken. Imagine you use a cryptographic algorithm to perform your online financial transactions that can be decrypted by an attacker: this is unacceptable. So, the same can be applied to steganography.

The main objective of steganalysis is to detect hidden information. If the information is detected we consider the steganographic method broken. Sometimes we can also extract the message but this is not so important because if we want to keep secret the message we have cryptography (Actually is usual to encrypt a message before hiding it). Steganography wants to keep secret the existence of the communication. So, if the communication is detectable the steganographic method is not acceptable. 


<br>

1. [Naive techniques](#1-naive-techniques)

   1.1. [Append one file to another](#11-append-one-file-to-another)
   
   1.2. [Writing text with similar colors](#12-writing-text-with-similar-colors)
   
   1.3. [Using the alpha channel](#13-using-the-alpha-channel)


2. [LSB replacement and the SPA attack](#2-lsb-replacement-and-the-spa-attack])

   2.1. [LSB replacement](#21-lsb-replacement)

   2.2. [The SPA Attack](#22-the-spa-attack)


3. [JPEG images and histogram estimation](#3-jpeg-images-and-histogram-estimation)

   3.1. [Hiding information in DCT coefficients](#31-hiding-information-in-dct-coefficients)

   3.2. [Histogram estimation](#32-histogram-estimation)


4. [LSB Matching and Machine Learning](#4-lsb-matching-and-machine-learning)

   4.1. [LSB Matching](#41-lsb-matching)

   4.2. [Machine learning based steganalysis](#42-machine-learning-based-steganalysis)

   4.3. [The Cover Source Mismatch problem](#43-the-cover-source-mismatch-problem)


5. [Minimizing distortion and adaptive algorithms](#5-minimizing-distortion-and-adaptive-algorithms)

6. [Dealing with CSM](#6-dealing-with-csm)

7. [Tips](#7-tips)

8. [So, what can I do?](#8-so-what-can-i-do)

9. [References](#9-references)

<br>

### 1. Naive techniques

In this section we are going to deal with these techniques too naive to be taken seriously but still being used frequently.

<br>
#### 1.1. Append one file to another

One of these techniques is to hide one file at the end of other file. Some image formats allow this operation without breaking things. For example the GIF image format. If we hide a ZIP file at the end of a GIF file, we can view the image without noticing any different.

We can do this in Linux/Mac with:

```bash
cat file.zip >> file.gif
```

Or in Windows:

```bash
copy /B file.gif+file.zip file.gif
```

See for example a GIF image of Groot:

![groot]({{ site.baseurl }}/images/hns_groot.gif)


And, the same GIF image with a ZIP file at the end:

![groot-stego]({{ site.baseurl }}/images/hns_groot_stego.gif)

Do you see any difference? I'm sure you do not. But it doesn't mean the method is secure. Actually, this is like hiding a safe behind a picture in the real world. 


Obviously, the ZIP file can be extracted. For example, using Linux:

```bash
$ unzip hns_groot_stego.gif
Archive:  hns_groot_stego.gif
warning [hns_groot_stego.gif]:  4099685 extra bytes at beginning or within zipfile
  (attempting to process anyway)
 extracting: hw.txt                  
$ cat hw.txt 
Hello World!
```

The same method can be used using different file formats which could be images or not. For example, you can do this with PNG, JPEG and others.


| Tip #1: Do not hide information by appending a file. |


<br>
#### 1.2. Writing text with similar colors

Other naive technique consist of writing text with a similar color, for example using 1px of difference from the original color. This can't be detected by the human eye.


See for example this image of Bender:

![bender]({{ site.baseurl }}/images/hns_bender.png)


And, the same image with some extra information:

![bender]({{ site.baseurl }}/images/hns_bender_stego.png)

Do you see any difference? I don't think so. But it is not difficult to uncover the secret. 

The following Python code applies a high-pass-filter using [convolution](https://en.wikipedia.org/wiki/Kernel_(image_processing)). Usually, this filter is used to detect edges. This is adequate for our purposes because we want to highlight these parts of the image with a change in the color. 


```python
import numpy as np
from scipy import ndimage, misc

I = misc.imread('hns_bender_stego.png')
kernel = np.array([[[-1, -1, -1],
                    [-1,  8, -1],
                    [-1, -1, -1]],
                   [[-1, -1, -1],
                    [-1,  8, -1],
                    [-1, -1, -1]],
                   [[-1, -1, -1],
                    [-1,  8, -1],
                    [-1, -1, -1]]])

highpass_3x3 = ndimage.convolve(I, kernel)
misc.imsave('hns_bender_stego_broken.png', highpass_3x3)
```

As you can see in the result image a simple filter can detect the hidden message. 

![bender]({{ site.baseurl }}/images/hns_bender_stego_broken.png)


| Tip #2: Do not hide information by drawing shapes, letters or simila ideas. |



<br>
#### 1.3. Using the alpha channel

Other naive technique consist of hiding information into the alpha channel. That is, the channel dedicated to transparency. 

This example image of Homer has a transparent background:

![bender]({{ site.baseurl }}/images/hns_homer.png)

If we read, for example, the data from the upper left corner we can see how the information data is organized:

```python
from scipy import ndimage, misc
I = misc.imread('hns_homer.png')
print I[0,0]
```

<br>
After executing the script we see this:

```bash
[0, 0, 0, 0]
```

Every pixel is represented by four values: RGBA. The first byte corresponds to red color, the second byte to green color, the third byte to blue color and the fourth byte represents the alpha channel (the opacity). Zero opacity means a transparent pixel. If the value was 255 the pixel would not be transparent. 

The upper left corner pixel is transparent, so the value of RGB bytes is ignored. This provides an easy way to hide data. 

The following code reads secret data from file "secret_data.txt" and hide it into an image called "hns_homer_stego.png". Every secret byte is hidden in every pixel with zero opacity. In this way we only overwrite invisible pixels. 

```python
from scipy import ndimage, misc

f=open('secret_data.txt', 'r')
blist = [ord(b) for b in f.read()]

I = misc.imread('hns_homer.png')

idx=0
for i in xrange(I.shape[0]):
    for j in xrange(I.shape[1]):
        for k in xrange(3):
            if idx<len(blist) and I[i][j][3]==0:
                I[i][j][k]=blist[idx]
                idx+=1

misc.imsave('hns_homer_stego.png', I)
```

<br>
As a result, we obtain the following image:

![bender]({{ site.baseurl }}/images/hns_homer_stego.png)


We do not see the message. But again, this is not a secure option. We can unhide the data, simply by removing the transparency. This is a very easy operation that can be done with the following script:

```python
from scipy import ndimage, misc
I = misc.imread('hns_homer_stego.png')
for i in xrange(I.shape[0]):
    for j in xrange(I.shape[1]):
        I[i,j][3]=255

misc.imsave('hns_homer_stego_broken.png', I)
```

<br>
After executing this script we obtain the following image:

![bender]({{ site.baseurl }}/images/hns_homer_stego_broken.png)

This image has a black background. But there is a section at the beginning where we see random colors. This is the result of hiding our secret bytes as a pixels. 

If an attacker performs this operation he/she has enough information to detect and extract the secret data.


| Tip #3: Do not hide information using the alpha channel. |



<br>
### 2. LSB replacement and the SPA attack

#### 2.1 LSB replacement

A basic technique to hide information in the bitmap of the image is to replace the Least Significant Bit (LSB) of the pixel by a bit of the message we whant to hide. By this way we can hide at most one bit per pixel, so the capacity of this method is the eighth part of the number of pixels.

In this example We are going to use the Lena image, a common image in steganography and watermarking:

![lena]({{ site.baseurl }}/images/hns_lena.png)


Let's see how to implement this technique in Python:

```python
import sys
from scipy import ndimage, misc

bits=[]
f=open('secret_data.txt', 'r')
blist = [ord(b) for b in f.read()]
for b in blist:
    for i in xrange(8):
        bits.append((b >> i) & 1)

I = misc.imread('hns_lena.png')

idx=0
for i in xrange(I.shape[0]):
    for j in xrange(I.shape[1]):
        for k in xrange(3):
            if idx<len(bits):
                I[i][j][k]&=0xFE
                I[i][j][k]+=bits[idx]
                idx+=1

misc.imsave('hns_lena_stego.png', I)
```

The first we do is to get secret data from 'secret_data.txt'. Then we split each pixel into bits and we store this in a list. This bits is what we want to hide in the LSB of the pixels.

Finally we get each pixel and remove the LSB. Then we put into the LSB the bit of the message. This is done by these operations:

```python
I[i][j][k]&=0xFE
I[i][j][k]+=bits[idx]
```

As a result, we get the following image:

![lena-stego]({{ site.baseurl }}/images/hns_lena_stego.png)

As usual, there is no difference for the human eye.

But how can we know if there is a hiden message? we will see in the next section. 

But before, I'm sure you want to know how to extract the message. Here you have the Python code:


```python
import sys
from scipy import ndimage, misc

I=misc.imread('hns_lena_stego.png')
f = open('output_secret_data.txt', 'w')

idx=0
bitidx=0
bitval=0
for i in xrange(I.shape[0]):
    for j in xrange(I.shape[1]):
        for k in xrange(3):
            if bitidx==8:
                f.write(chr(bitval))
                bitidx=0
                bitval=0
            bitval |= (I[i, j, k]%2)<<bitidx
            bitidx+=1

f.close()
```

What we do is to extract every pixel reading the LSB. Every time we have 8 bits we save the whole byte into the output file.

<br>
#### 2.2 The SPA Attack

LSB replacement seems a good steganographic technique. An attacker can extract and read the message but this is easy to solve. we only have to encrypt it and if the attacker extracts the message he/she will think this is garbage. Other option is to use a [PRNG](https://en.wikipedia.org/wiki/Pseudorandom_number_generator) to choose which pixels we want to use to hide information. In this case we do not use all the pixels, we are embedding information using a bitrate smaller than one. For example, if we hide information using a 25% of the pixels we say we are using a bitrate 0.25. This reduces capacity increasing security, because less information is more difficult to detect.

So, we have a  mostly secure steganongraphic method. Isn't it? 

No!, it is not. LSB replacement is an asymmetrical operation and it can be detected. To see what it means an asymmetrical operation, let's analyze what is happening when we replace the LSB.

When we replace the LSB of a pixel with an even vallue this produces the same effect of adding one when we replace by one or does not produce any effect when we replace by zero. Similarly, when we replace the LSB of a pixel with an odd value this produces the same effect of subtracting one when we replace by zero or does not produce any effect when we replace by one. 

Think a litle bit about this. When we hide data, the value of the even pixels increases or remains the same and the value of odd pixels decrease or remains the same. This is the asymmetrical operation I said before and this type of operation introduces statistical anomalies into the image. This fact was exploited first by the histogram attack [[1](9-references)] and later by the RS attack [[2](#9-references)] and the SPA attack [[3](#9-references)].

The Sample Pair Analysis (SPA) is detailed in [[3](#9-references)] so we refer the reader to the original paper for a detailed explanation and its corresponding maths. 

The following code implements the SPA attack:

```python
import sys
from scipy import ndimage, misc
from cmath import sqrt

if len(sys.argv) < 2:
    print "%s <img>\n" % sys.argv[0]
    sys.exit(1)

channel_map={0:'R', 1:'G', 2:'B'}

I3d = misc.imread(sys.argv[1])
width, height, channels = I3d.shape

for ch in range(3):
    I = I3d[:,:,ch]

    x=0; y=0; k=0
    for j in range(height):
        for i in range(width-1):
            r = I[i, j]
            s = I[i+1, j]
            if (s%2==0 and r<s) or (s%2==1 and r>s):
                x+=1
            if (s%2==0 and r>s) or (s%2==1 and r<s):
                y+=1
            if round(s/2)==round(r/2):
                k+=1

    if k==0:
        print "ERROR"
        sys.exit(0)

    a=2*k
    b=2*(2*x-width*(height-1))
    c=y-x

    bp=(-b+sqrt(b**2-4*a*c))/(2*a)
    bm=(-b-sqrt(b**2-4*a*c))/(2*a)

    beta=min(bp.real, bm.real)
    if beta > 0.05:
        print channel_map[ch]+": stego", beta
    else:
        print channel_map[ch]+": cover"

```

This SPA implementation provides the estimated embedding rate. If the predicted bit rate is too low we consider the analyzed image as cover. Otherwise we consider it as stego.

Note we analize each channel (R, G and B) separately. So we can detect if there is information only in one channel.

Let's try our program with the cover image:

```bash
$ python spa.py hns_lena.png
R: cover
G: stego
B: stego
```

And now, let's try with the stego image.

```bash
$ python spa.py hns_lena_stego.png 
R: stego 0.0930809062336
G: stego 0.0923858529528
B: stego 0.115466382367
```

That means the program detects aproximately a bitrate of 0.10. This is almost correct.

The SPA attack can detect reliably images embedded with bitrates over 0.05 but it also works fairly well witht lower bitrates (~0.03). These are very low bitrates so we can consider the LSB replacement practically broken. 


| Tip #4: Use LSB replacement only if you hide a few bytes (as in some watermarking applications) |



<br>
### 3. JPEG images and histogram estimation

We can hide information inside a JPEG image by modifying its bitmat as in the last section. But we can also modify its [DCT](https://en.wikipedia.org/wiki/Discrete_cosine_transform) coefficients. These coefficients are used to represent a compressed version of the image, so when we open an image with a viewer, the viewer uncompress the image using the DCT coefficients. The idea to modify a DCT coefficient is also modifying its LSB. 

<br>
#### 3.1. Hiding information in DCT coefficients

To hide information into the DCT coefficients we need a JPEG low level library or some JPEG steganography tool. In this case we are going to use an implementation of [F5](https://link.springer.com/chapter/10.1007/3-540-45496-9_21), a known steganographic algorithm. For this example we have used [this implementation](https://code.google.com/archive/p/f5-steganography/downloads).

Our cover image is the Peppers image in JPEG format:

![peppers]({{ site.baseurl }}/images/hns_peppers.jpg)


To hide data into this file, we use the following command:

```bash
java -jar f5.jar e -e secret_data.txt -p password \
                   -q 100 hns_peppers.jpg hns_peppers_stego.jpg
```

And, as a result, we obtain the following image:

![peppers-stego]({{ site.baseurl }}/images/hns_peppers_stego.jpg)




#### 3.2. Histogram estimation

There are different JPEG steganograohy methods and differents attacks to JPEG steganography. Here we are going to see a simple attack that consists of estimating the histogram of DCT coefficients. This histogram is a graph bar in whitch each bar represents the frequency of each DCT value. So the first step is to extract de DCT coefficients. 

To extract the DCT coefficients we can use [this tool](http://www.daniellerch.me/snippets/stego/dctdump.c). Download it and compile it with the following command:

```bash
$ gcc dctdump.c -o dctdump -ljpeg
```

After that, you can extract the coefficients of the cover image:

```bash
./dctdump hns_peppers.jpg raw > hns_peppers.dct
```

With the list of coefficients we can use a simple Python script to draw the histogram. We will see an example later.

By the moment let me explain what we want to do. The DCT coefficients give a representation of the image. If we draw a histogram using the coefficients of the cover (original) image and we draw other histogram using the coefficients of the image after some type of rotation, we expect both histograms to be similar. We say the second histogram is an estimation of the real histogram.

But if we modify the DCT coefficients in some sort of steganographic embedding operation we are produciong an unatural result. If we rotate now the image we are generation new DCT coefficients and we do not expectet them to be similar. We can see these histograms in the following image:

![histograms]({{ site.baseurl }}/images/hns_histograms.png)

If we compare the first and second histograms we can see they are similar. This says us there is no hiden message. But if we compare the third and fourh histograms we see they are very different. This is the result of modify the DCT coefficients.

<br>
To draw the histograms firs we need to generate new images. We rotate the images 1º and we crop the central part to avoid the edges generated after rotation:

```bash
convert -rotate 1 hns_peppers.jpg hns_peppers_rot.jpg
convert -crop 500x500+12+12 hns_peppers_rot.jpg hns_peppers_rot_crop.jpg
convert -rotate 1 hns_peppers_stego.jpg hns_peppers_stego_rot.jpg
convert -crop 500x500+12+12 hns_peppers_stego_rot.jpg hns_peppers_stego_rot_crop.jpg
```

Then, we generate files with the DCT coefficients:

```bash
./dctdump hns_peppers.jpg raw > hns_peppers.dct
./dctdump hns_peppers_stego.jpg raw > hns_peppers_stego.dct
./dctdump hns_peppers_rot_crop.jpg raw > hns_peppers_rot_crop.dct
./dctdump hns_peppers_stego_rot_crop.jpg raw > hns_peppers_stego_rot_crop.dct
```

Finally, we use the following Python script to draw the histograms:

```python
from scipy import ndimage, misc
import matplotlib
import matplotlib.pyplot as plt
import numpy

def read_values(filename):
    f = open(filename, "r")
    lines = f.read().split('\n')
    values = [int(v) for v in lines if v!='' and v not in ['0', '1', '-1']]
    return values

values = read_values("hns_peppers.dct")
values_stego = read_values("hns_peppers_stego.dct")
values_rot_crop = read_values("hns_peppers_rot_crop.dct")
values_stego_rot_crop = read_values("hns_peppers_stego_rot_crop.dct")

font = {'size': 6}
matplotlib.rc('font', **font)

rng=(-50, 50)
x=plt.subplot(1,4,1)
x.set_ylim(0, 20000)
x.set_title("Cover")
plt.hist(values, bins=256, range=rng, width=1)
x=plt.subplot(1,4,2)
x.set_ylim(0, 20000)
x.set_title("Estim. Cover")
plt.hist(values_rot_crop, bins=256, range=rng, width=1)
x=plt.subplot(1,4,3)
x.set_ylim(0, 20000)
x.set_title("Stego")
plt.hist(values_stego, bins=256, range=rng, width=1)
x=plt.subplot(1,4,4)
x.set_ylim(0, 20000)
x.set_title("Estim. Stego")
plt.hist(values_stego_rot_crop, bins=256, range=rng, width=1)
plt.show()
```

| Tip #5: Use DCT steganography only if you hide a few bytes (as in some watermarking applications) |



<br>
### 4. LSB Matching and Machine Learning

As we saw above, LSB replacement is not a secure technique. Nevertheless, there is a very simple modification to the insertion technique that makes the operation simmetrical. If we do this, the method becomes very difficult to detect.

Instead of replacing the LSB of the pixel the right thing to do is increase or decrease randomly by 1. The effect on the LSB is the same but the operation does not introduce so evident anomalies. 

There is not easy statistical attack to detect this operation and, consequently, the security of LSB matching is significantly better than that of LSB replacement. Accually, the only way to deal with steganalysis of LSB matching based techniques is throught machine learning.

<br>
#### 4.1. LSB Matching

Hiding information using LSB matching is very easy. If the value of LSB is the same we want to hide, we do nothing. If not, we increase or decrease by 1 randomly. 

In this example we use the F16 image:

![f16]({{ site.baseurl }}/images/hns_f16.png)

<br>
See an example program in Python to hide information:


```python
#!/usr/bin/python

import sys
from scipy import ndimage, misc
import random

bits=[]
f=open('secret_data.txt', 'r')
blist = [ord(b) for b in f.read()]
for b in blist:
    for i in xrange(8):
        bits.append((b >> i) & 1)

I = misc.imread('hns_f16.png')

sign=[1,-1]
idx=0
for i in xrange(I.shape[0]):
    for j in xrange(I.shape[1]):
        for k in xrange(3):
            if idx<len(bits):
                if I[i][j][k]%2 != bits[idx]:
                    s=sign[random.randint(0, 1)]
                    if I[i][j][k]==0: s=1
                    if I[i][j][k]==255: s=-1
                    I[i][j][k]+=s
                idx+=1

misc.imsave('hns_f16_stego.png', I)
```

<br>
The result after embedding is this:

![f16-stego]({{ site.baseurl }}/images/hns_f16_stego.png)


<br>
To extract the message we can use the same program we used with LSB replacement:

```python
import sys
from scipy import ndimage, misc

I=misc.imread('hns_f16_stego.png')
f = open('output_secret_data.txt', 'w')

idx=0
bitidx=0
bitval=0
for i in xrange(I.shape[0]):
    for j in xrange(I.shape[1]):
        for k in xrange(3):
            if bitidx==8:
                f.write(chr(bitval))
                bitidx=0
                bitval=0
            bitval |= (I[i, j, k]%2)<<bitidx
            bitidx+=1

f.close()
```

<br>
As usual there is no difference for the human eye between the cover and the stego images. But this time the method is almost secure. Or in any case, much more harder to detect than the methods presented before. Actually, to deal with LSB matching we need heavy machinery. Let's see how to apply [machine learning](https://en.wikipedia.org/wiki/Machine_learning) to steganalysis.


<br>
#### 4.2. Machine learning based steganalysis

Machine learning was applied succesfuly to many applications and steganalysis is not an exception. To apply machine learning to steganalysis the first we need is a training database, that is a database of cover and stego images.  




<br>
#### 4.3. The Cover Source Mismatch problem



PENDING...

<br>
### 5. Minimizing distortion and adaptive algorithms

PENDING...

<br>
### 6. Dealing with CSM

PENDING...

<br>
### 7. Tips

PENDING...

<br>
### 8. So, what can I do?

PENDING...

<br>
### 9. References

[1]. Attacks on Steganographic Systems. A. Westfeld and A. Pfitzmann. Lecture Notes in Computer Science, vol.1768, Springer-Verlag, Berlin, 2000, pp. 61−75. 

[2]. Reliable Detection of LSB Steganography in Color and Grayscale Images. Jessica Fridrich, Miroslav Goljan and Rui Du.
Proc. of the ACM Workshop on Multimedia and Security, Ottawa, Canada, October 5, 2001, pp. 27-30. 

[3]. Detection of LSB steganography via sample pair analysis. S. Dumitrescu, X. Wu and Z. Wang. IEEE Transactions on Signal Processing, 51 (7), 1995-2007.



