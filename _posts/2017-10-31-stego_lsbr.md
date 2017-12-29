---
layout: post
title: Steganography and Steganalysis in Images II - LSB replacement and the SPA attack
---

## Steganography and Steganalysis in Images II

### LSB replacement and the SPA attack
In this article we are going to deal with LSB replacement, one of the most used techniques in image steganography. Unfortunately this technique was broken many years ago. First we present the technique and after that we show how to detect it.

1. [LSB replacement](#1-lsb-replacement)

2. [The SPA Attack](#2-the-spa-attack)

<br>

#### 1. LSB replacement

A basic technique to hide information in the bitmap of the image is to replace the Least Significant Bit (LSB) of the pixel by a bit of the message we whant to hide. 

Let's suppose whe have the following fixel values in an image:

| 160 | 60 | 53 | 128 | 111 | 43 | 84 | 125 |

If we obtain its binary code, that is:

| 10100000 | 00111100 | 00110101 | 10000000 | 
| 01101111 | 00101011 | 01010100 | 01111101 |

Let's suppose now we want to hide the A letter in ASCII code. This, in binary code, is the number 01000001. So we need to replace the LSB of each pixel whit each one of the bits we want to hide. The result is:


| 1010000**0** | 0011110**1** | 0011010**0** | 1000000**0** | 
| 0110111**0** | 0010101**0** | 0101010**0** | 0111110**1** | 

<br>
By this way we can hide at most one bit per pixel, so the capacity of this method is the eighth part of the number of pixels.

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
#### 2. The SPA Attack

LSB replacement seems a good steganographic technique. An attacker can extract and read the message but this is easy to solve. we only have to encrypt it and if the attacker extracts the message he/she will think this is garbage. Other option is to use a [PRNG](https://en.wikipedia.org/wiki/Pseudorandom_number_generator) to choose which pixels we want to use to hide information. In this case we do not use all the pixels, we are embedding information using a bitrate smaller than one. For example, if we hide information using a 25% of the pixels we say we are using a bitrate 0.25. This reduces capacity increasing security, because less information is more difficult to detect.

So, we have a  mostly secure steganongraphic method. Isn't it? 

No!, it is not. LSB replacement is an asymmetrical operation and it can be detected. To see what it means an asymmetrical operation, let's analyze what is happening when we replace the LSB.

When we replace the LSB of a pixel with an even vallue this produces the same effect of adding one when we replace by one or does not produce any effect when we replace by zero. Similarly, when we replace the LSB of a pixel with an odd value this produces the same effect of subtracting one when we replace by zero or does not produce any effect when we replace by one. 

Think a litle bit about this. When we hide data, the value of the even pixels increases or remains the same and the value of odd pixels decrease or remains the same. This is the asymmetrical operation I said before and this type of operation introduces statistical anomalies into the image. This fact was exploited first by the histogram attack [[1](#references)] and later by the RS attack [[2](#references)] and the SPA attack [[3](#references)].

The Sample Pair Analysis (SPA) is detailed in [[3](#references)] so we refer the reader to the original paper for a detailed explanation and its corresponding maths. 

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


<br>
### References

[1]. Attacks on Steganographic Systems. A. Westfeld and A. Pfitzmann. Lecture Notes in Computer Science, vol.1768, Springer-Verlag, Berlin, 2000, pp. 61−75. 

[2]. Reliable Detection of LSB Steganography in Color and Grayscale Images. Jessica Fridrich, Miroslav Goljan and Rui Du.
Proc. of the ACM Workshop on Multimedia and Security, Ottawa, Canada, October 5, 2001, pp. 27-30. 

[3]. Detection of LSB steganography via sample pair analysis. S. Dumitrescu, X. Wu and Z. Wang. IEEE Transactions on Signal Processing, 51 (7), 1995-2007.




