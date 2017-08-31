[ [Index - Steganography in Images](http://pages.daniellerch.me/stego_index) ]<br><br>


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

