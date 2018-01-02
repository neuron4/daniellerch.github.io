---
layout: post
title: Steganalysis of OpenStego
---

## Steganalysis of OpenStego

### Introduction

[OpenStego](https://www.openstego.com) is an open source steganography software written in Java. In this post we are going to analyze its security. Our main objetive is to know if the sofware is detectable because indetectability is its main feature. Secondly we want to know if we can extract the message.

This post is based on OpenStego v0.7.2.

<br>
1. [Using the tool from command line](#1-using-the-tool-from-command-line)

2. [Preparing a testing set](#2-preparing-a-testing-set)

3. [Steganalysis](#3-steganalysis)


<br>
#### 1. Using the tool from command line

OpenStego has a graphical user interface but it can be used also from command line. We are going to use this feature.

To hide information we can use the following command:

```bash
java -jar openstego.jar embed -a randomlsb -mf secret.txt \
                        -cf cover.png -sf stego.png -p mypassword
```

To unhide our data we can use the following command:
```bash
java -jar openstego.jar extract -a randomlsb -sf stego.png \
                        -xd /tmp/ -p mypassword
```

For our purposes we only need the first command.



<br>
#### 2. Preparing a testing set









