---
layout: post
title:  "SOC CTF 2024 Steganography Writeup"
summary: "My solutions to all the problems in the Steganography Section of the in-house CTF for USU"
author: BreydenSummers
date: '2024-04-02 22:07:15 +0000'
category: CTF
thumbnail: /assets/img/posts/SOC-CTF-2024/header.png
keywords: CTF, Utah State University, Steganography
permalink: /blog/SOC-CTF-2024-Steganography-Writeup/
---

## Got any Grapes? 1 and 2
![notsuspiciousfile](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/d99c4557-1b45-4bff-ac40-ff8dcf2a5dde) <br> 
Here is the image for the first problem. To solve this one we need to use steghide to extract the text to the file.
```bash
steghide extract -sf notsuspiciousfile.jpg 
```
Once we do that we can view the text of the output file to get our flag: SOC_CTF{TH3N_H3_WADDL3D_AWAY}. For the next one it seems we need to crack this one. We can use the tool stegseek with the rockyou database to do this:
```bash
stegseek definitely_not_suspicious_file.jpg ./PATH/TO/rockyou.txt 
```
Once we do that we get this as our cracked passwords: 1q2w3e4r5t. We can take a look at the output file to get our flag: SOC_CTF{WED0NTSELLFRUIT}. 

## Duck Vision
To solve this one we can upload it to this [website](https://www.aperisolve.com/) (this is a great place to start for a lot of these problems.) 
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/e6f8760b-02dc-4a1f-ad36-3abaa8a43010)
From here we can see that hidden in the image is the flag. We can just copy that out and submit it. SOC_CTF{NOONEEXPECTSTHEDUCKINQUISITION}.

## inSpector Quack
For this challenge, a good place to start is with a program called [audacity](https://www.audacityteam.org/). This tool lets you inspect audio. To solve this challenge we can notice that there is a emphasis on Spector in the title. So we can take a look at the spectrogram for the audio. <br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/c0fd5ab6-6e32-4d7e-9f8a-189c8ad28667) <br>
Once we do that we get this:
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/178a9fa2-ad1c-4c96-a884-ca571f0bb0e8)<br>

## BEEP BEEP

We can open this one in audacity as well:<br>
![image](https://github.com/BreydenSummers/BreydenSummers.github.io/assets/40399657/32d6bd27-d07c-4780-93de-33d881a1b7ae)<br>
We can see that it reliably turns off and on completely throughout the entire recording. It is pretty safe to assume that the data is in binary form and is encoded into the audio. Wav files store data by giving "speed" and then the "height" at given interverls for the audio file. But if you are not familiar with a wav file this challenge is great for the use of a llm for generating code to manipulate the WAV. Here is what I came up with:
```python
import wave
import math
import struct

def audio_to_binary(filename):
    wavfile = wave.open(filename, 'r')
    length = wavfile.getnframes()
    framerate = wavfile.getframerate()
    duration_per_bit = 1  # Assuming each bit was encoded with a 1-second duration

    frames_per_bit = int(framerate * duration_per_bit)
    num_bits = int(length / frames_per_bit)

    binary_str = ''

    for i in range(num_bits):
        frames = wavfile.readframes(frames_per_bit)
        amplitude_sum = 0
        for j in range(frames_per_bit):
            frame = frames[2*j:2*j+2]  # Assuming 16-bit audio
            amplitude = struct.unpack('<h', frame)[0]
            amplitude_sum += abs(amplitude)

        average_amplitude = amplitude_sum / frames_per_bit
        # Determine if this segment represents a '1' or '0' based on amplitude.
        # This threshold might need adjustment depending on how '1's and '0's were encoded.
        threshold = 32767 / 2  # Half of max volume for 16-bit audio
        binary_str += '1' if average_amplitude > threshold else '0'

    wavfile.close()
    return binary_str

def binary_to_text(binary_str):
    text = ''
    for i in range(0, len(binary_str), 8):
        byte = binary_str[i:i+8]
        text += chr(int(byte, 2))
    return text

def main():
    filename = "TEST.wav"
    binary_str = audio_to_binary(filename)
    text = binary_to_text(binary_str)
    print(f"Decoded text: {text}")

if __name__ == "__main__":
    main()
```
This takes in the wav file named TEST.wav and then gets the binary out of it and then converts it to an ascii string. Depending on the file you might have to change the frames and duration per bit. As well as if the audio is 16 bit or not. After running this code you should get this as the decoded flag: SOC_CTF{BEEP_BEEP_I_AM_A_DUCK}.
