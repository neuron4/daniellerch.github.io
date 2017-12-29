---
layout: post
title: Steganography and Steganalysis in Images III - LSB Matching and Matrix Embedding
---

## Steganography and Steganalysis in Images III

### LSB Matching and Matrix Embedding

As we saw before, [LSB replacement is not a secure technique](http://pages.daniellerch.me/stego_lsbr). Nevertheless, there is a very simple modification to the insertion technique that makes the operation simmetrical. If we do this, the method becomes very difficult to detect.

Instead of replacing the LSB of the pixel the right thing to do is to increase or to decrease randomly by 1. The effect on the LSB is the same but the operation does not introduce so evident anomalies. 

There is not easy statistical attack to detect this operation and, consequently, the security of LSB matching is significantly better than that of LSB replacement. Actually, the only way to deal with steganalysis of LSB matching based techniques is through machine learning.

1. [LSB matching](#1-lsb-matching)

2. [Matrix embedding](#2-matrix-embedding)



<br>
#### 1. LSB Matching

Hiding information using LSB matching is very easy. If the value of LSB is the same we want to hide, we do nothing. If not, we increase or decrease by 1 randomly. 

Let's suppose whe have the following fixel values in an image:

| 160 | 60 | 53 | 128 | 111 | 43 | 84 | 125 |

If we obtain its binary code, that is:

| 10100000 | 00111100 | 00110101 | 10000000 | 
| 01101111 | 00101011 | 01010100 | 01111101 |

Let's suppose now we want to hide the A letter in ASCII code. This, in the binary code, is the number 01000001. So we need to add +1 or -1 randomly if the LSB does not match the value of the pixel we want to hide. A possible result is:


| (+0) 1010000**0** | (+1) 0011110**1** | (-1) 0011010**0** | (+0) 1000000**0** | 
| (-1) 0110111**0** | (+1) 00101**100** | (+0) 0101010**0** | (+0) 0111110**1** | 

<br>



In this example we are going to use the F16 image:

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
As usual there is no difference for the human eye between the cover and the stego images. 

The presented program has some limitations. The first one is we are hiding information in a sequential manner and consequently we hide all the information in the beginning of the image. Second, we are hiding information in all the pixels. This is very disruptive but can be easily improved. For example, we can choose part of the pixels where we want to hide information using a key known only by the sender and the receiver. We can do this using a low bitrate because this is harder to detect than a higher bitrate. But his unfortunately decreases the capacity. we can deal with this problem thanks to different techniques that try to minimize distortion.


<br>
#### 2. Matrix embedding

To minimize the distortion introduced hiding data a common approach is to use matrix embedding to hide the same data modifying less pixels of the image. It can seem a little bit strange at the beginning, but this is possible with a simple trick. 

Let's suppose you want to hide two bits. Using LSB matching as we shown before we have to modify the pixel 50% of the time, because the other 50% of the time the value of the LSB is already the same we want to hide. This means the effectiveness of our method is 1/2.

Suppose now we use LSB matching by hiding two bits in groups of three pixels:

| P1 | P2 | P3 |

And that we use the following for the first bit we want to hide:

$$ M_1 = LSB(P_1) \oplus LSB(P_2) 1$$

And the following formula for the second bit we want to hide:

$$ M_2 = LSB(P_2) \oplus LSB(P_3) 1$$

Note this method is very easy to apply. If $$M_1$$ and $$M_2$$ match the bits we want to hide we do nothing. If none of $$M_1$$ and $$M_2$$ match the bits, we have to change the value of $$LSB(P_2)$$. If $$M_1$$ match but $$M_2$$ does not match we change the value of $$LSB(P_3)$$ and if $$M_2$$ match but $$M_1$$ does not match we change the value of $$LSB(P_1)$$. With this metodology we hide two bits and we only have to modify one. 

Let's see an example. We have the following pixels:

| 10010100 | 10010101 | 10010111 |

If we want to hide $$00$$ one possible result is:

| (+1) 1001010**1** | 10010101 | 10010111 |


If we want to hide $$01$$ one possible result is:

| 10010100 | (-1) 1001010**0** | 10010111 |


If we want to hide $$10$$, we do not need to change anything:

| 10010100 | 10010101 | 10010111 |


And finally, if we want to hide $$11$$ one possible result is:

| 10010100 | 10010101 | (-1) 1001011**0** |

<br>
This simple idea can be generalized using binary [Hamming codes](https://en.wikipedia.org/wiki/Hamming_code).

Let's suppose we want to hide $$p$$ bits in a block of pixels by modifying only one bit. The first we need is a $$M$$ matrix that contains all non zero binary vectors with $$p$$ elements in its columns. For example, if we want to hide information in blocks of 3 pixels, as before, one possible $$M$$ matrix is:

$$ M=\begin{pmatrix} 0001111\\0110011\\1010101 \end{pmatrix} $$

We can use this python code to generate an $$H$$ matrix:

```python
import numpy
import sys

def prepare_M(n_bits):
    M=[]
    l=len(bin(2**n_bits-1)[2:])
    for i in range(1, 2**n_bits):
        string=bin(i)[2:].zfill(l)
        V=[]
        for c in string:
            V.append(int(c))
        M.append(V)
    M=numpy.array(M).T
    return M
 ```

For $$p=3$$ we get the matrix:

```bash
>>> prepare_M(3)
array([[0, 0, 0, 1, 1, 1, 1],
       [0, 1, 1, 0, 0, 1, 1],
       [1, 0, 1, 0, 1, 0, 1]])
```


The number of pixels we need in each block is $$2^p-1$$. Following the example, if $$p=3$$ we need blocks of 7 pixels. 

Now, our formula to calculate the message is:

$$ m = Mc $$

where $$c$$ is the vector of bits in the cover image. Lets suppose we want to hide the message m=(1,1,0) in the following pixels:

| 11011010 | 11011011 | 11011011 | 11011010 | 11011011 | 11011010 | 11011010 |

We only want the LSBs:

| 0 | 1 | 1 | 0 | 1 | 1 | 0 | 0 |

So, in the example the value of c is:

$$ c=(0,1,1,0,1,1,0,0) $$

If we apply the formula:

$$ m = Mc = (1, 0, 0) $$

This is not the message we want to hide, so we want to hide m=(1,1,0). We need to find how to modify $$c$$, that is, we need to find the stego version in which $$ m = Mc = (1, 1, 0) $$.

Ee need to find the column of M that is different. We can do this with a simple substraction $$ m-Mc $$. After that, we only have to change the value of the corresponding pixel.

Following our example:

$$ m-Mc = (0, 1, 0) $$

The column:

$$ \begin{pmatrix} 0\\1\\0 \end{pmatrix} $$

Corresponds to the second column of $$M$$:

$$ M=\begin{pmatrix} 0001111\\0110011\\1010101 \end{pmatrix} $$

That means we have to change the second pixel of $$c$$ to obtain the stego block $$s$$:

$$ c=(0,1,1,0,1,1,0,0) $$

$$ s=(0,0,1,0,1,1,0,0) $$

And in this case, our previous formula works:

$$ m = Ms = (1, 1, 0) $$

This time, we get the message we want to hide. So our stego pixels are:

| 11011010 | 11011010 | 11011011 | 11011010 | 11011011 | 11011010 | 11011010 |



As a summary, we have hidden three bits in a block of seven pixels by modifying only one bit.

These operation can be easily automated in Python using the following functions to hide and to unhide data:

```python
def ME_hide_block(M, c, m):
    r=m-M.dot(c)
    r=r%2

    idx=0
    found=False
    for i in M.T:
        if numpy.array_equal(i, r):
            found=True
            break
        idx+=1

    # the block does not need to be modified
    if not found:
        return c

    s=numpy.array(c)
    if s[idx]==0: s[idx]=1
    else: s[idx]=0

    return s
    
 def ME_unhide_block(M, s):
    m=M.dot(s)
    m=m%2
    return m
 ```

Let's suppose now we want to hide 4 bits in blocs of $$2^4-1=15$$ pixels. For example with m=(1, 1, 0, 0) and c=(0, 1, 1, 0, 1, 0, 0, 1, 1, 0, 1, 1, 1, 0, 0). We only have to do this:

```python
n_bits=4
M=prepare_M(n_bits)
m=numpy.array([1, 1, 0, 0])
c=numpy.array([0, 1, 1, 0, 1, 0, 0, 1, 1, 0, 1, 1, 1, 0, 0])
s=ME_hide_block(M, c, m)
print s
m_recovered=ME_unhide_block(M, s)
print m_recovered
```

We can use blocks of different sizes but if the number of bit we want to hide in each block is too high the number of pixels we need en each block could be prohibitive (remember we need $$2^p-1$$ pixels). As a consequence, a high undetectability suppose a very low capacity because the big size of the blocks.


