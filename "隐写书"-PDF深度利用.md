## "steganography" - obfuscating PDF exploits in depth

Shortly after [last week's discovery](https://blog.edgespot.io/2019/01/an-interesting-obfuscation-method.html) of a PDF exploit which used the method of this.getPageNumWords() & this.getPageNthWord() for obfuscation, we found another, but much more powerful exploit obfuscation technique in PDF exploits. This technique uses a so-called "steganography" method to hide malicious Javascript code in images embedded in PDF files, it is so powerful as it could bypass almost all AV engines.

The sample was detected as "exploit CVE-2013-3346" by our EdgeLogic engine, same as the previous one.

* https://blog.edgespot.io/2019/01/an-interesting-obfuscation-method.html

![](https://4.bp.blogspot.com/-Kr-NkDqfyho/XEdDnFNT1EI/AAAAAAAAAEo/d9VFG0l_qDwAB_vhf50p7AHCZjNncLPuQCLcBGAs/s1600/edgespot-detection.png)

The sample was first submitted to VirusTotal on 2017-10-10, with a filename "oral-b oxyjet spec.pdf".

![](https://lh4.googleusercontent.com/ORMvSEL5-R-yLYe8ow9YzjUkUWBfXSTxa8d55dxZhoot91KnVhLvyvjd0nBUbPJk9sH433KsJDgdSZnu52NWm-9mbf5uPQv-gMGalUX918rX7HSAJyFj3OQq1zpNsUqrKVlc_Qrf)

Only 1 AV engine has detected this exploit last week (however, as of writing, the detection is increased to 5/57).
* https://www.virustotal.com/#/file/ebc5617447c58c88d52be6218384158ccf96ec7d7755179a31d209a95cd81a69/detection

![](https://3.bp.blogspot.com/-O2dvXaoaRIw/XEdTFYN-N3I/AAAAAAAAAGI/wdCOW241LCQxQgP99qbzggvgoSMLmVRSwCEwYBhgL/s1600/24.png)

After opened, the PDF disguised as an IRS document looked normal.

![](https://4.bp.blogspot.com/-_zJ5pKPgv3I/XEdH34_gUOI/AAAAAAAAAE4/KYsV8Wi3phs4FWS3xJ8yZnI9jZXnNgAhwCLcBGAs/s1600/16.png)

Two layers of obfuscation were used in this sample. The first layer is what we have previously disclosed - the method of "this.getPageNumWords()" and "this.getPageNthWord()". The exploit uses "this.getPageNumWords()" and "this.getPageNthWord()" to read and execute the Javascript hidden as "content". The related code can be found in PDF stream-64.

![](https://lh6.googleusercontent.com/Qa4otHEzSjZlj4B65CmnfgutxzaTfn4EugYFlSf0BaMQdyntnVpxr7qzgwjAdzY3Ue97axGjscZtt2dumd7bKlutVi1aDi9ElBSPm17xJkgmIPM902ailGHvnOGRjtfpy_ADT_-_)

The second layer is new and it's our focus in this blog. The "Javascript content" is stored in stream-119, let's see what it looks like.

![](https://lh6.googleusercontent.com/daXGdDM5pyT4_kjmoaPsX9jnXZRbq9fIF22cHznr97dqymfQ8TLJ1KpnsK7LswND3Tfo-cVqXG_VyxOD_amxM2Pi_bpUFUzG1xLPJLq_-EIzLDWS4PBHGZzcb4Aw0aXZEpipBv0X)

After beautifying the Javascript, it shows as following:

![](https://1.bp.blogspot.com/-ux7d0FWJFqM/XEdNBe3NpMI/AAAAAAAAAFE/JaVx-Zq4P0kRFc7E9C1RRkGya_6hFrrkwCLcBGAs/s1600/22.png)

In order to figure out what the Javascript have done, we need to learn these two PDF JS APIs, the this.getIcon() and the util.iconStreamFromIcon() at first. Following is an extract from Adobe's reference.

![](https://2.bp.blogspot.com/-n78eqA8dplg/XEdNwvK8GCI/AAAAAAAAAFM/l4fI0jyNbbUZilcpU2zsNfyHL5holqNgQCLcBGAs/s1600/4.png)

![](https://2.bp.blogspot.com/-2-4DtAFE_2c/XEdN0UrwVrI/AAAAAAAAAFc/3q4W0pMMGkwmnhuyXBe-8yCkqUAvWEwYQCLcBGAs/s1600/5.png)

According to the API references, these two APIs, working together, are to read the stream of an image named as "icon" stored in the PDF file.

 By examining the above Javascript code, we figured out that the code’s function is to read and decode the "message" hidden in the icon’s stream. Once it read the "message" successfully , it will execute the "message" as Javascript code, via "eval(msg)".

 The icon stream named "icon"  in the object-131 could be saved as a "jpg" file and viewed in image viewer without problem. As shown below:
 
 ![](https://lh3.googleusercontent.com/IPN1eis6eIjQjZsiQR4MRlkGGbw1Zb8P324LrOzw6LFIagc_KB4bsyY8xlc1T1TfSeofYKOkxTbOiXJihanQ9NG2Ky1Ya2CDxjphMhHmwSJJ3ZMl744Xz3DnVGLqDLnXZkMRwF0U)
 
 The malicious data is hidden in the image while the image is still viewable
 
 
 Nevertheless, there’s no suspicious data can be found inside the icon file, since the malicious code data is heavily obfuscated.

 What does the final executed Javascript look like? Here is a piece of the real code, after successful de-obfuscating.
 
 https://lh4.googleusercontent.com/Iun-DdCJrtuagzxaB1eYLCX5_Ecu0MCTTV-P3cBxUGlxJKdVSIqFsnCTZFMym2HzpUIvKqpoEDK8gEt6WMmfxWBdgJCqHIRgTC25dDjKOMoxcCstabRGkRsIWMq9BNb6xzd0VqNR
 
 
 Therefore, we confirmed the exploited vulnerability is CVE-2013-3346.

 Furthermore, we deduce that this sample and the previous one were from the same author, for following reasons.
Both of them exploit the same vulnerability (CVE-2013-3346)
The similarity of the Javascript code in these two exploits.
After some googling, we found that the attacker likely copied a project/technique called "steganography.js", which is open sourced here. The project was developed working on browsers. We believe the person behind the PDF samples made their innovation as they successfully leveraged the technique in PDF format.  We could not find any information mentioning such technique in PDF exploits before, so we believe this is the first time that the "steganography" technique is used to hide PDF exploits.


Conclusion
We were impressed by this technique, which is perfect for malicious code obfuscation for PDF exploits. By using this technique, all streams look normal, all images are viewable, everything looks legitimate. This can probably explain why almost all AV engines missed it.

 In this blog we researched into the truly advanced "steganography" technique used for obfuscating PDF exploits, which is a demonstration of the power of our EdgeLogic engine as we are able to beat this obfuscation technique, among many others.

 Just like the previous one, the "steganography" technique could not only be used to obfuscate this exploit (CVE-2013-3346) but also can be applied to many other PDF exploits including zero-days. We ask security defenders to pay close attention to it.

Follow us @EdgeSpot_io.
