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


2. [Sequential LSB replacement and the histogram attack](#2-sequential-lsb-replacement-and-the-histogram-attack])

3. [Random LSB replacement and the SPA attack](#3-random-lsb-replacement-and-the-spa-attack)

4. [LSB Matching and Machine Learning](#4-lsb-matching-and-machine-learning)

5. [Minimizing distortion and adaptive algorithms](#5-minimizing-distortion-and-adaptive-algorithms)

6. [Dealing with CSM](#6-dealing-with-csm)

7. [So, what can I do?](#7-so-what-can-i-do)

5. [References](#5-references)

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


| Tip #1: Do not hide information by appending a file. It is easy to detect and easy to extract | 


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
blist = list((ord(b) for b in f.read()))

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


We do not se the message. But again, this is not a secure option. We can unhide the data, simply by removing the transparency. This is a very easy operation that can be done with the following script:

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



<br>
### 2. Sequential LSB replacement and the histogram attack

PENDING...

<br>
### 3. Random LSB replacement and the SPA attack

PENDING...

<br>
### 4. LSB Matching and Machine Learning

PENDING...

<br>
### 5. Minimizing distortion and adaptive algorithms

PENDING...

<br>
### 6. Dealing with CSM

PENDING...

<br>
### 7. So, what can I do?

PENDING...

<br>
### 5. References

PENDING...


