link: https://github.com/wisec/domxsswiki/wiki
link: https://domgo.at/cxss/example/1
## Sources
- source = **user-controlled input**
- This is the "entry point" of the payload.
	`document.laction`
	`document.URL`
	`document.referrer`
	`window.name`
	`location.search(query string)`
	`location.hash(#fragment)`
	input:
		`Form (<input>,<textarea>)`
		`Cookies(document.cookie)`
		`Http headers`
- EX:
```code

# in xss lab:
  https://target.com/page?search=payload
--> `search` = SOURCE

```

---
## Sinks
- sink = where data is inserted into the DOM or executed
- This is the "execution point"
- **Dangerous sinks (critical for XSS):**
	- `innerHTML`
	- `outerHTML`
	- `document.write()`
	- `eval()`
	-  `setTimeout()` / `setInterval()` (string-based)
	- `Function()`
	-  `insertAdjacentHTML()`
- ex:
```javascript
document.getElementById("output").innerHTML = userInput;
// If userInput comes from Source --> xss
```

