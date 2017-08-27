---
layout: post
title: How not to hide information inside an image
comments: true
---

## How Not to Hide Information Inside an Image. 

You can find a lot of tools on  Internet to hide information inside an image. Unfortunately, most of them are like 
hiding a safe behind a picture: it is not exactly the safest option. In this article I will try to show which steganographic techniques you should not use. 



[ Updated on 2017-xx-xx ]

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




### 2. Sequential LSB replacement and the histogram attack

### 3. Random LSB replacement and the SPA attack

### 4. LSB Matching and Machine Learning

### 5. Minimizing distortion and adaptive algorithms

### 6. Dealing with CSM

### 7. So, what can I do?

### 5. References


