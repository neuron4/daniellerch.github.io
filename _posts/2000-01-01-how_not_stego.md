---
layout: post
title: How not to hide information inside an image
comments: true
---

## How Not to Hide Information Inside an Image. 

You can find a lot of tools on the Internet to hide information inside an image. Unfortunately, most of them are like 
hiding a safe behind a picture: it is not exactly the safest option. In this article I will try to show which steganographic techniques you should not use. 

The criteria for deciding whether a method is adequate is its detectability. In steganography and stegaganalysis if a method is detectable we consider it broken. Imagine you use a cryptographic algorithm to perform your online financial transactions that can be decrypted by an attacking part. Nowadays this is unacceptable. So, the same can be applied to steganography.


<br>

1. [Naive techniques](#1-naive-techniques)

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

One of these techniques is to hide a file at the end of other file. Some image formats allow this operation without breaking things. For example the GIF image format. If we hide a ZIP file at the end of a GIF file, we can view the image without noticing any different.

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




### 2. Sequential LSB replacement and the histogram attack

PENDING...

### 3. Random LSB replacement and the SPA attack

PENDING...

### 4. LSB Matching and Machine Learning

PENDING...

### 5. Minimizing distortion and adaptive algorithms

PENDING...

### 6. Dealing with CSM

PENDING...

### 7. So, what can I do?

PENDING...

### 5. References

PENDING...


