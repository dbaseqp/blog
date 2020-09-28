---
layout: post
title: WhitehatCTF - Web01
excerpt: "I did WhitehatCTF with DiceGang this weekend and focused on the webs. We did pretty well and placed second in the quals right after perfect blue. I thought a couple of them were pretty cool but would have been much better with source. In my opinion, sourceless web ctf challenges should rarely exist."
categories: [Writeups, WhitehatCTF'19]
tags: [web]
---

I did WhitehatCTF with DiceGang this weekend and focused on the webs. We did pretty well and placed second in the quals right after perfect blue. I thought a couple of them were pretty cool but would have been much better with source. In my opinion, sourceless web ctf challenges should rarely exist.

#### Web01
```
My VietNam.

http://15.165.80.50/
```

The site is very simple and the home page doesn't look important. There is just a register and login button. After making an account, we see that we have a todolist and can add more "todo" items to this list. While making test todo items, the url seemed vulnerable to LFI. `http://15.165.80.50/?page=todo`. To verify this was PHP, I just visited `http://15.165.80.50/index.php?page=todo` and it worked.

We tried with the standard `?page=../../../../../etc/passwd` and it didn't work. After trying different payloads, we found that `?page=..././..././..././..././..././etc/passwd` worked. Seems like it just replaced `../` in our send string,  so we can just bypass it by placing one `../` inside of another resulting in `..././`.

Now that we had LFI, we needed to find flag. After looking around and noticing that `http://15.165.80.50/?page=..././..././..././..././..././proc/self/cmdline` worked, we eventually landed on `http://15.165.80.50/?page=..././..././..././..././..././proc/1/cmdline` which displayed the flag
