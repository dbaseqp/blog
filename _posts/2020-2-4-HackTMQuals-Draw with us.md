---
layout: post
title: HackTMQuals - Draw with us
excerpt: "HackTM challeneges were pretty unique and challenging. I didn't spend much time on the ctf and only was able to solve the first web challenge which exploited unicode and objects in javascript to get the flag."
categories: [Writeups, HackTMQuals'19]
tags: [web]
---

HackTM challeneges were pretty unique and challenging. I didn't spend much time on the ctf and only was able to solve the first web challenge which exploited unicode and objects in javascript to get the flag.

#### Draw with us
```
Come draw with us!  
  
http://167.172.165.153:60000/  
  
Author: stackola

File: stripped.js
```

We are given a URL which links to a frontend that allows us to paint a canvas after logging in. We are also provided with a javascript file that we assume is the nodejs code running behind the api.

When looking at stripped.js we see there there is a route `/flag` with the following code

```javascript
app.get("/flag", (req, res) => {
	// Get the flag
	// Only for root  
	if (req.user.id == 0) {
		res.send(ok({ flag: flag }));
	} else { res.send(err("Unauthorized")); } });
```

It is clear that our goal is to become the user with id 0 which is initialized into the users array. In the route `/init` we see that if we can get the values of `p` and `q`  then we can get a token which validates us as the admin user. 

```javascript
let target = md5(config.n.toString()); 
let pwHash = md5( bigInt(String(p)).multiply(String(q)).toString());
if (pwHash == target && clearPIN === _clearPIN) {
	// Clear the board
	board = new  Array(config.height).fill(0).map(() =>  new Array(config.width).fill(config.backgroundColor));
	boardString = boardToStrings(); io.emit("board", { board: boardString });
}
```

We see that the global config object is defined as the following

```javascript
const config = { port: process.env.PORT || 8081, width: 120, height: 80, usersOnline: 0, message: "Hello there!", p: p, n: n, adminUsername: "hacktm", whitelist: ["/", "/login", "/init"], backgroundColor: 0x888888, version: Number.MIN_VALUE };
```

It is clear our goal is to get the values `p` and `n` from the config object, then solve for q using the equation `q = n / p`. How can we obtain these values? There is a route called `/serverInfo` which lets us view items in the config object. All we have to do is add the items `p` and `n` to our users rights property. There is also conveniently a route named `/updateUser` which lets us change our user's rights and the color we paint.

```javascript
app.post("/updateUser", (req, res) => { 
	// Update user color and rights
	// Only for admin
	// POST
	// {  // color: 0xDEDBEE,
	// rights: ["height", "width", "usersOnline"]
	// }
	let uid = req.user.id;
	let user = users[uid];
	if (!user || !isAdmin(user)) { res.json(err("You're not an admin!")); return; }
	let color = parseInt(req.body.color); users[uid].color = (color || 0x0) & 0xffffff;
	let rights = req.body.rights || [];
	if (rights.length > 0 && checkRights(rights)) {
		users[uid].rights = user.rights.concat(rights).filter(onlyUnique);
	}
	res.json(ok({ user: users[uid] }));
});
```

This function poses two challenges. The first is the function `isAdmin` and the second is the function `checkRights`

```javascript
function isAdmin(u) {
	return u.username.toLowerCase() == config.adminUsername.toLowerCase();
}
```
```javascript
function checkRights(arr) {
	let blacklist = ["p", "n", "port"];
	for (let i = 0; i < arr.length; i++) {
		const element = arr[i];
		if (blacklist.includes(element)) { return  false; }
	}
	return  true;
}
```

In order to bypass isAdmin, it seems like we need to login as `hacktm` however it uses an uppercase check in order to make sure we are not admin which is done with the function isValidUser.

```javascript
function isValidUser(u) {
	return ( u.username.length >= 3 && u.username.toUpperCase() !== config.adminUsername.toUpperCase() );
}
```

My first observation is that one is a toUpperCase check and one is a toLowerCase check. This leads me to think that by using unicode characters I can bypass the login check. Sure enough, there was a unicode character that when .toLowerCase() is used on it, returns k. Thus, if I signed in with the user `hacKtm`, I would have an account which was admin. The next step was to add `p` and `n` to my account's rights. However, this was disallowed by the checkRights function I showed above. After testing, I found that if I send an objeect instead of a string. It would bypass the check. Thus, if I send `{rights: [['p']]}` and then `{rights: [['n']]}`, I will be able to add these to my own rights. 

Now, all that was left was the call `/serverInfo` with my token and it would give me the `p` and `n` stored in the config object. After finding `q` using the formula listed above, all I had to do was send this to the route `/init`, granting me the token of the user with id 0. The final step was to call `/flag` with this new token and we successfully get the flag.
