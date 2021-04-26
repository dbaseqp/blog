layout: post
title: HackTheBox CTF 2021
excerpt: "My team and I competed in HackTheBox's CTF Cyber Apocolypse and we got 189th place out of 4740 teams (top 3.99%). I focused mainly on crypto and helped my team occasionally in other categories."
categories: [competition]
tags: [hackthebox, ctf, review]
---

## Overview

1. [Thoughts and Review](#Thoughts-and-Review)
2. [Crypto Challenges](#Cypto-Challenges)
3. [Other Challenges](#Other-Challenges)

## Thoughts and Review

#### My Thoughts

To put it simply, I really enjoyed HackTheBox CTF.

For context, I have competed in many other CTFs before, however, I never really got that far in any of them. I would always try to mix up the kinds of challenges that I focused on to increase the breath of my knowledge, but this time I focused almost all of my time in crypto. From my teammates, I was able to work on a few other challenges, but going pretty deep (for a rookie) in crypto felt like a decent acomplishment. Over the 20 or so total hours of researching, I was able to learn a lot about cryptography. For example, I already knew how XOR worked in concept, but I never really understood how it was used in crypto. It took many hours of research to find good resources that explained concepts like AES, multi-pad attacks, and RSA simply enough for a novice to understand. RSA in particular was very difficult to research, as most resources explained things either way too simply (leaving out the parts I needed) or way too in-depth (convoluting the information). I hope that I explained these concepts well enough that others won't have to go through what I did in the below writeups of the challenges. Lastly, I'm really proud of my team's work. We were able to get 189th place worldwide which is the top 3.99%. Among US teams, we were top 40, which really shocked me because this CTF was open to students, hobbyists, and professionals alike. To be top 40 in US as a ragtag group of college students is a feat I'm glad to have made with my team.

#### My Review

##### Variation
HTB CTF had lots of variance for the challenges. The categories were as follows: Web, Pwn, Crypto, Reversing, Forensics, Hardware, & Misc. Each category had a number of challenges that tested different skillsets. Though my team only were able to get deep in a few categories, specifically web and crpyto, the other categories had interesting challenges based on what I could grasp from reading their descriptions. 
##### Difficulty
Unlike other CTFs that I've seen in the past, HTB CTF had a nice progressive skill curve based on my team's experience. However, a few teams did finish all the challenges fairly early. Like I mentioned earlier, this CTF was open to all skill levels so it is possible that only the very best would be able to complete all the challenges, but I think that this CTF is definitely aimed more for the middle tier of CTF competitors. Those who wish for a truly challenging CTF might have found the challenges in their main role's specific category a decent skills test to learn a few things at best or just a time sink at worst.
##### Other Notes
One good thing I really enjoyed was the Docker instances they had to host the challenges. I found them to be a neat functionality that I haven't seen other platforms incorporate so seamlessly. On another note, I found that the challenges themselves were a good learning tool for me, but unfortunately they were take offline as soon as the competition ended. I understand that HTB wanted conserve resources and make the materials private again, but I was hoping for at least 24 hours to save any challenges competitors like myself might have had yet to complete.

##### Rating
Note: the score is not a 100% accurate reflection of the competition, it's just give my feeling a number.

Final score: 9/10

## Crypto Challenges
I did get stuck on a couple of the Crypto challenges, but I was able to complete a handful of them. The challenges were as follows:
* [Nintendo Base64](#Nintendo-Base64)
* [Phasestream 1](#Phasestream-1)
* [Phasestream 2](#Phasestream-2)
* [Phasestream 3](#Phasestream-3)
* [Phasestream 4](#Phasestream-4)



---

### Nintendo Base64

Summary of Information Given:
* ASCII art of a picture showing "nintendo64x8"

Given the name, I assumed it was a base 64 encoded message. I went to base64decode.org and plugged in the message. However, after the first decryption, I didn't see the flag anywhere. That's when it hit me, the ASCII art was probably referencing it was base 64 encoded 8 times, so I continued decoding the message until I got the flag.

#### Flag

`CHTB{3nc0d1ng_n0t_3qu4l_t0_3ncrypt10n}`

---

### Phasestream Series

Each one was similar, but just enough different to make you think about it entirely differently. Each Phasestream challenge revolved around the concept of XOR, and they were extremely difficult if you simply looked for the solution online, so we'll start with that. 

### What is XOR?
XOR, also known as eXclusive OR, is a logical operation that basically means, "either but never both". If you are unfamiliar with XOR, we'll start with XOR in binary representation. Say you have 2 unique bytes, Byte 1 and Byte 2. If you line the two bytes on top of each other, the bits should align on top of each other. Wherever a '1' bit overlaps with another '1' bit, they cancel each other out and result in '0' for that bit. If a '1' bit overlaps a '0' bit, the result is '1'.

Here's an example of how it works.


|       Label       |   Data   |
|:-----------------:|:--------:|
|      Byte 1       | 10001100 |
|      Byte 2       | 00001111 |
| Byte 1 XOR Byte 2 | 10000011 |

Importantly, XOR has 4 important properties: the communative, associative, self-inverse, and identity properties. Taken together, these properties imply the result of XORing any series of values can be reversed into the original value by XORing with a previously used value. If you are still confused about XOR, try reading [this journal](https://accu.org/journals/overload/20/109/lewin_1915/) or researching separately.

Why is this important to cryptography? Because ciphers/encryption algorithms that XOR plaintext with encryption keys to produce ciphertexts can be undone if you have enough information about the results and the algorithm. For example, if you know your `ciphertext = plaintext ⊕ key`, you can use `ciphertext ⊕ key` to get your plaintext back (BTW ⊕ is the symbol for XOR). 

### Phasestream 1

Summary of Information Given:
* Aliens have discovered XOR to encrypt plaintext
* XOR key is 5 bytes
* Encrypted flag: 2e313f2702184c5a0b1e321205550e03261b094d5c171f56011904
* Hint: What is the format of a flag?

First things first, the hint references flag format. For HTB CTF, the format was "CHTB{", which I just so happened to notice was 5 characters (and a character is 1 byte). Since we know the flag was encrypted with XOR, all we really need to do is find the key to decrypt it. As I mentioned in my section about XOR, `ciphertext = plaintext ⊕ key` which implies, `ciphertext ⊕ plaintext = key`.

---
#### A brief intermission if that equation confused you...
If this is confusing to you, think of it like this: 
We know `ciphertext = plaintext ⊕ key` is true by definition.

Our assumption `ciphertext ⊕ plaintext = key` could then be rewritten as `plaintext ⊕ key ⊕ plaintext = key`.

Since XOR is associative and self-inversing, `plaintext ⊕ key ⊕ plaintext = key` -> `plaintext ⊕ plaintext ⊕ key = key` -> `key = key` which is obviously true. Therefore, `ciphertext ⊕ plaintext = key` is true.

This is known as a known-plaintext attack.

---

At this point, my strategy is to simply XOR the encrypted text with the known plaintext string, "CHTB{". Since the known plaintext string is the same length as the key, it should reveal the full encryption key. I went to dcode.fr and input the information I knew. 

![](https://i.imgur.com/zf2A1jg.png)

You'll notice that the there's a lot of gibberish produced, however, the there is a 5-byte chunk of plaintext at the beginning, "mykey". Based on the context, I assumed that it was the true encrpytion key.

![](https://i.imgur.com/u8Tq4SZ.png)

Using "mykey" as the encryption key revealed the flag.
#### Flag
`CHTB{u51ng_kn0wn_pl41nt3xt}`

### Phasestream 2

Summary of Information Given:
* Aliens have discovered "security by obscurity" and threw junk data in with the real flag
* XOR key is 1 bytes
* Flag is in 10,000 line file, output.txt
* Encryption technique is XOR

This challenge is more or less the exact same as the previous, except we have to go through up to 10,000 lines. My solution was to XOR every line in the file based on a list of possible 1-byte keys. My script was based on [laconicwolf's commandline XOR tool](https://github.com/laconicwolf/crypto-tools/blob/master/repeating_key_xor_encrypt_decrypt.py), but you could either make your own or find another XOR tool for this. Its a bash script that runs the Python XOR decryption script and only prints the line that produces the flag format. 

##### My Bash Script Solution
```
#!/bin/bash
iter=0
possibleKeys=(a b c d e f g h i j k l m n o p q r s t u v w x y z A B C D E F G H I J K L M N O P Q R S T U V W X Y Z)
for i in ${possibleKeys[@]}; do
        python3 repeating_key_xor_encrypt_decrypt.py -xd -f output.txt -k $i | grep "CHTB{" && echo "Key: $i"
done
```


I just gave it execute permission and ran it, and it spat out the flag.
#### Flag
`CHTB{n33dl3_1n_4_h4yst4ck}`

### Phasestream 3

Summary of Information Given:
* Aliens have discovered AES in CTR mode encryption, which turns AES into a stream cipher
* We are given the encryption program, phasestream3<area>.py
* The flag is given in output.txt

First, I opened the encryption Python script to see how they implemented AES-CTR encryption and noticed something that shouldn't be there. While the aliens encrypted the flag, a second message was encrypted as a test and was left in as plaintext.

    test="No right of private conversation was enumerated in the Constitution. I don't suppose it occurred to anyone at the time that it could be prevented"

AES-CTR encryption ultimately boils down to, `ciphertext = AES(ctr) ⊕ plaintext` where AES(ctr) is really just acting as the encryption key. Every time the program is run, theoretically, it should produce a new key. However, since the key was reused, we know both given ciphertexts use the same key and we can abuse this to XOR cancel the encryption key without knowing what it actually is. This is known as a many-time-pad attack (or a reused key attack). Since we also know the plaintext, we can combine it with a known-plaintext attack to get the flag.

##### Reused Key Attack Proof:
1. `ciphertext1 ⊕ plaintext ⊕ ciphertext2 = flag`
2. `[AES(ctr) ⊕ plaintext] ⊕ plaintext ⊕ [AES(ctr) ⊕ flag] = flag`
3. `AES(ctr) ⊕ AES(ctr) ⊕ plaintext ⊕ plaintext ⊕ flag = flag`
4. `0 ⊕ 0 ⊕ flag = flag`
5. `flag = flag`

##### In code:

```
from pwn import xor

ciphertext1 = bytes.fromhex("08501b3dbd0fb2f7c87aeb3a224a9d568fa8ad83ff442548b5f4334f0fe1dd6b8f5d5e410be5af2d7ea642b12d8f459f2ab666d4f79a9115dc9cf22ed60e899769fd206c40819bbefe2b5a2ec592a387c6927d866b6343466d5effde0666dd3bb7f657ed651bfcf45fd5b264a36406c6b6dbb1a81272029c5e06da438a0281c19c1e10a0dc47d6ae994557e82663e9f59578")
ciphertext2 = bytes.fromhex("05776f0daf1ae9f6dd26e945390bad7fda889c97ff6036")

test = b"No right of private conversation was enumerated in the Constitution. I don't suppose it occurred to anyone at the time that it could be prevented."

key = xor(ciphertext1, test)
print(xor(key, ciphertext2))
```

#### Flag
`CHTB{r3u53d_k3Y_4TT4cK}`

### Phasestream 4

Summary of Information Given:
* The aliens fixed their mistake and encrypted a different flag
* Same as Phasestream 3 otherwise

This climax of the Phasestream series revisits the core themes of the previous chapters. Phasestream 4 is the exact same except there is no plaintext leftover, but they still include a 2nd ciphertext. This is still vulnerable because this is still a many-time-pad attack, we just have to change the approach.

If you've been following along, `ciphertext1 ⊕ ciphertext2 = plaintext ⊕ flag` because the encryption keys will cancel out (flag is just another plaintext). We can complete this challenge by basically doing the same techniques as Phasestream 3 and 1, in that order. XOR the two ciphertexts together, then XOR the result with what we guess might be in one of the two plaintexts. I used [CameronLonsdale's Multi-pad tool](https://github.com/CameronLonsdale/MTP) to decrypt the message. The tool XORd what it could and then you have to fill in the rest by guessing. I started one side with 'CHTB{' and saw the other side start 'I alo'. I guessed that it was 'I alone' which extended part of the flag. This process of guessing and extending each plaintext is known as crib dragging, and it ultimately resulted in enough of the plaintext message that I could recognized it as a quote.

    "I alone cannot change the world, but I can cast a stone across the water to create ma"

Just XOR the plaintext ⊕ flag with the discovered plaintext, and we'll get the flag.

#### Flag
`CHTB{stream_ciphers_with_reused_keystreams_are_vulnerable_to_known_plaintext_attacks}`

## Other Challenges
Disclaimer: For these challenges, my team solved these in collaboration. My individual participation varied so I can't do a detailed breakdown for each solution.

### Alien Camp

This challenge had us connect to a netcat listener that gave us a quiz of 500 math questions. The twist was the math questions were in emojis (instead of numbers), each question must be solved in under 2 seconds, and each emoji had a different value every time. 

I noticed that the quiz had 2 options when connected to: '1' to see the emoji values, and '2' to take the quiz. Based on the these factors, I assumed all we needed to do to create a script that creates a Python dictionary that uses the hexcodes of each emoji as the key to its value.

##### The script my team made together:
<script src="https://gist.github.com/dbaseqp/397de1a5620d5ddfc1fa94a497df6a68.js"></script>
