---
layout: post
title: Steganography and Steganalysis in Images I - Naive techniques
---

## Steganography and Steganalysis in Images I

### Naive techniques
This is the first of several articles about steganalysis in images. In this article we are going to deal with some techniques too naive to be taken seriously but still being used frequently.


1. [Append one file to another](#1-append-one-file-to-another)
   
2. [Writing text with similar colors](#2-writing-text-with-similar-colors)
   
3. [Using the alpha channel](#3-using-the-alpha-channel)


<br>
#### 1. Append one file to another

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




<br>
#### 2. Writing text with similar colors

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
#### 3. Using the alpha channel

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


