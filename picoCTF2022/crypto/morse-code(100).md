**Description**

Author: Will Hong

Morse code is well known. Can you decrypt this? Download the file here. Wrap your answer with picoCTF{}, put underscores in place of pauses, and use all lowercase.

Hints
1) Audacity is a really good program to analyze morse code audio.

**Solve**
I'm sure you have all heard of morse code. The implementation of using short signals and long signals to make up the English alphabet.
Like the hint says, open the audo in audacity to view the file.!

![Image](https://user-images.githubusercontent.com/59877252/160716768-6f0c0303-70c1-437e-aa20-ba880f288a62.png)

As you can see, there are a series of long beeps, short beeps, and pauses translate these out to a dot, a dash, and a space respectively.
This transforms into:

..-  --...  ....  ----.  -----  -..  .--  ..---  -----  ..-  ----.  ....  --...
U    7      H     9      0      D    W    2      0      U    9      H     7
Use a morse code chart to decode.
![Image](https://2rdrtx4bt29lo91s31mjhkji-wpengine.netdna-ssl.com/wp-content/uploads/2019/03/Morse-code-chart-dit-dah-dot-dash-communication-shtf-survival-prepping-1.jpg)

The code decodes to:
U7H90DW20U9H7

put spaces in between the large pauses in the sound

This then gets wrapped in the challenge wrapper to get
picoCTF{u7h_90d_w20u9h7}
