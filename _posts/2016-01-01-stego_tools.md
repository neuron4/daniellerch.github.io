---
layout: post
title: Steganalysis
---

## Analysis of Image Steganography Tools and Algorithms

Here you can find a list of image steganography tools and their kwnown attacks. I maintain this list to be updated about the status of the tools available and the level of security they provide. If you know another tool that should be in the list or a missing attack, please contact with [me](http://daniellerch.me). 

[ *Updated on 2016-12-09* ]

<br>

### State of the art Methods

- **ASO (Adaptive Steganography by Oracle)** <br>
status: *state-of-the-art* <br>
notes: no implementation available<br>
[papers
[#1](http://hal-lirmm.ccsd.cnrs.fr/lirmm-00838993/file/ASO_soumis.pdf)
[#2](http://www.eurasip.org/Proceedings/Eusipco/Eusipco2012/Conference/papers/1569585231.pdf)]

- **HUGO (Highly Undetectable steGO)** <br>
status: *state-of-the-art*<br>
[[code](http://dde.binghamton.edu/download/stego_algorithms/download/HUGO_bounding_linux_make_v10.tar.gz)]
[[paper](https://hal.archives-ouvertes.fr/hal-00541353/document)]
[Attacks
[#1](http://www.ws.binghamton.edu/fridrich/Research/TIFS2012-SRM.pdf)
[#2](http://www.ws.binghamton.edu/fridrich/Research/color-04.pdf)
[#3](http://www.sciencedirect.com/science/article/pii/S0952197616000026)]

- **MiPOD** <br>
status: *state-of-the-art* <br>
[[code](http://dde.binghamton.edu/download/stego_algorithms/download/MiPOD_matlab.zip)]
[[paper](http://www.ws.binghamton.edu/fridrich/Research/MiPOD.pdf)]
[Attacks
[#1](http://www.ws.binghamton.edu/fridrich/Research/TIFS_ECSP.pdf)]

- **UNIWARD** <br>
status: *state-of-the-art* <br>
[[S-UNIWARD code](http://dde.binghamton.edu/download/stego_algorithms/download/S-UNIWARD_linux_make_v10.tar.gz)]
[[J-UNIWARD code](http://dde.binghamton.edu/download/stego_algorithms/download/J-UNIWARD_linux_make_v11.tar.gz)]
[[paper](http://dde.binghamton.edu/vholub/pdf/EURASIP14_Universal_Distortion_Function_for_Steganography_in_an_Arbitrary_Domain.pdf)]
[Attacks 
[#1](http://www.ws.binghamton.edu/fridrich/Research/Improving-SCA-Features-final.pdf)
[#2](http://www.ws.binghamton.edu/fridrich/Research/Explicit_transforms.pdf)
[#3](http://www.ws.binghamton.edu/fridrich/Research/SCA_JPEG_final1.pdf)]

- **WOW** <br>
status: *state-of-the-art* <br>
[[code](http://dde.binghamton.edu/download/stego_algorithms/download/WOW_linux_make_v10.tar.gz)]
[[papers](http://dde.binghamton.edu/vholub/pdf/WIFS12_Designing_Steganographic_Distortion_Using_Directional_Filters.pdf)]
[Attacks
[#1](http://www.ws.binghamton.edu/fridrich/Research/TIFS2012-SRM.pdf)
[#2](http://www.ws.binghamton.edu/fridrich/Research/color-04.pdf)
[#3](http://www.sciencedirect.com/science/article/pii/S0952197616000026)]


### Detectable or broken steganography

- **F5** <br>
status: *detectable* <br>
[[code](http://code.google.com/p/f5-steganography/)]
[[paper](http://f5-steganography.googlecode.com/files/F5%20Steganography.pdf)]
[Attacks 
[#1](http://ws2.binghamton.edu/fridrich/Research/f5.pdf) 
[#2](http://openaccess.uoc.edu/webapps/o2/bitstream/10609/40841/1/Patterns_O2.pdf)]

- **JPHIDE and JPSEEK** <br>
status: *detectable* <br>
[[site](http://linux01.gwdg.de/~alatham/stego.html)]
[Attacks
[#1](http://openaccess.uoc.edu/webapps/o2/bitstream/10609/40841/1/Patterns_O2.pdf)]

- **JSTEG** <br>
status: *detectable* <br>
[[site](http://zooid.org/~paul/crypto/jsteg/)]
[Attacks
[#1](http://users.ece.cmu.edu/~adrian/487-s06/westfeld-pfitzmann-ihw99.pdf)]

- **OpenPuff** <br>
status: *detectable* <br>
notes: It uses a secret hiding algorithm.<br>
[[site](http://embeddedsw.net/OpenPuff_Steganography_Home.html)]
[Attacks
[#1](http://blog.daniellerch.me/p/openpuff.html)]

- **OpenStego** <br>
status: *detectable* <br>
notes: It saves non hidden data that allows an easy detection. <br>
[[site](http://www.openstego.info/)]
[Attacks
[#1](http://0xword.com/es/libros/64-esteganografia-y-estegoanalisis.html)]

- **Perturbed Quantization** <br>
status: *detectable* <br>
[[site](http://dde.binghamton.edu/download/pq/)]
[[paper](http://dde.binghamton.edu/download/pq/Fri05pq.pdf)]
[Attacks
[#1](http://openaccess.uoc.edu/webapps/o2/bitstream/10609/40841/1/Patterns_O2.pdf)]

- **Silenteye** <br>
status: *detectable* <br>
notes: The program uses a two-bit LSB replacement.<br>
[[site](http://silenteye.v1kings.io/)]
[Attacks
[#1](http://www.ws.binghamton.edu/fridrich/Research/acm_2001_03.pdf)
[#2](http://www.ece.mcmaster.ca/~sorina/papers/LSBfinalTSP.pdf)]

- **Steghide** <br>
status: *detectable* <br>
[[site](http://steghide.sourceforge.net/)]
[Attacks
[#1](http://openaccess.uoc.edu/webapps/o2/bitstream/10609/40841/1/Patterns_O2.pdf)]

- **StegoBlue** <br>
status: *detectable* <br>
notes: The program uses +1 embedding.<br>
[[site](https://github.com/oni49/stegoBlue)]
[Attacks
[#1](http://www.ws.binghamton.edu/fridrich/Research/TIFS2012-SRM.pdf)
[#2](http://www.sciencedirect.com/science/article/pii/S0952197616000026)]



