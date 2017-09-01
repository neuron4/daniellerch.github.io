[ [Index - Practical Image Steganography and Steganalysis](http://pages.daniellerch.me/stego_index) ]<br><br>



You can find a lot of tools on the Internet to hide information inside an image. Unfortunately, most of them are like 
hiding a safe behind a picture: it is not exactly the safest option. In this article I will try to show which steganographic techniques you should not use. 

The criteria for deciding whether a method is adequate or not is its detectability. In steganography and stegaganalysis if a method is detectable we consider it broken. Imagine you use a cryptographic algorithm to perform your online financial transactions that can be decrypted by an attacker: this is unacceptable. So, the same can be applied to steganography.

The main objective of steganalysis is to detect hidden information. If the information is detected we consider the steganographic method broken. Sometimes we can also extract the message but this is not so important because if we want to keep secret the message we have cryptography (Actually is usual to encrypt a message before hiding it). Steganography wants to keep secret the existence of the communication. So, if the communication is detectable the steganographic method is not acceptable. 




----


The prisoners problem was first introduced by G. J. Simmons in 1983 [1] and tells the history of Alice and Bob, two prisoners working on an escape plan. They are allowed to communicate, but their communications pass through the warden, Eve. Eve will attempt to find any hidden communication between Alice and Bob, and if she finds one, she will cut the communication channel. 

The prisoners problem is an example of the need of techniques for sending information in a cover manner. This is frequently used for presenting Data Hiding.

Data hiding is a collection of techniques to embed secret data into digital media such that its existence becomes undetectable. These techniques can be used in many different application scenarios, such as secret communications (Steganography) or copyright protection (Watermarking), among others. 

References: 

[1] G. J. Simmons. “The prisoner’s problem and the subliminal channel”. In D. Chaum, editor, Advances in Criptology, CRYPTO 83, pages 51-67, Santa Barbara, CA, August 22-24, 1983. New York: Plenum Press.

