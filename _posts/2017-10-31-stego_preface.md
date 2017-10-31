## Steganography and Steganalysis in Images 

### Preface
You can find a lot of tools on the Internet to hide information inside an image. Unfortunately, most of them are like 
hiding a safe behind a picture: it is not exactly the safest option. In this series of articles I will try to shed some light on it.

But first, let me give you a brief introduction to the field using **the prisoners' problem**. The prisoners' problem was first introduced by G. J. Simmons in 1983 [[1](#references)] and tells the history of Alice and Bob, two prisoners working on an escape plan. They are allowed to communicate, but their communications pass through the warden, Eve. Eve will attempt to find any hidden communication between Alice and Bob, and if she finds one, she will cut the communication channel. 

The prisoners problem is an example of the need of techniques for sending information in a cover manner. This is frequently used for presenting Data Hiding. Data hiding is a collection of techniques to embed secret data into digital media such that its existence becomes undetectable. These techniques can be used in different application scenarios, such as secret communications (Steganography) or copyright protection (Watermarking), among others. 

The criteria for deciding whether a method is adequate or not is its detectability. In steganography if a method is detectable we consider it broken. Imagine you use a cryptographic algorithm to perform your online financial transactions that can be decrypted by an attacker: this is unacceptable. Therefore, the same can be applied to steganography: its main objective is undetectability so if a steganographic method is detectable it is useless. 

The techniques used to detect hidden information are known as steganalysis. Although they are usually used to detect hidden information, in some cases we can also extract the message. This is an interesting result but this is not so important because there exists other tools to keep the message secret: cryptography. Actually it is usual to encrypt a message before hiding it. Steganography wants to keep secret the existence of the communication. So, if the communication is detectable the steganographic method is not acceptable. 

<br>

### References: 

[1] G. J. Simmons. “The prisoner’s problem and the subliminal channel”. In D. Chaum, editor, Advances in Criptology, CRYPTO 83, pages 51-67, Santa Barbara, CA, August 22-24, 1983. New York: Plenum Press.

