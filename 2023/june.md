Protocapture: https://challenge-0623.intigriti.io/

```
<script src="./static/jquery-2.2.4.js"></script>
<script src="./static/jquery-deparam.js"></script>
```
+  the deparam lib is vulnerable to a very known Prototype pollution
poc: https://github.com/BlackFan/client-side-prototype-pollution/blob/master/pp/jquery-deparam.md
+ On the challenge site, passing `?__proto__[test]=hello` as parameter, sets the test proprety of Object.prototype to the string hello.
+ Note: window.name converts all stored values to their string representations using the toString method.
Apparently, setting something to a variable called name automatically calls toString.
+ `?name[test]=test&__proto__[toString]=function(){alert(1)}` Unfortunately, this doesn’t fire. The reason is that we are not reassigning the alert function to toString, we are literally reassinging the string "function(){alert(1)}"
+ I was trying to use the name variable as a gadget, but was unsuccessful. At some point, I found out that google reCaptcha is a known gadget for prototype pollution! PoC: https://github.com/BlackFan/client-side-prototype-pollution/blob/master/gadgets/recaptcha.md

+ We have to set window.recaptcha to true.
+ There has to exist an HTML element with a class attribute set to g-recaptcha and data-sitekey set to anything we like
+ It is very clear that somehow we have to bypass the if(window.captcha) check. The only way seems to have document.domain set to localhost
+ an object (window in this case) get’s a propriety from Object.prototype only if it’s not defined lower in it’s inheritance chain 
+  In other words, to pollute window.recaptcha, we have to make sure that it’s not locally defined in the code. To do so, it is enough to have document.domain set to literally anything diffrent that challenge-0623.intigriti.io
+ We can append a dot at the end of our domain
+ The reason is, the dot is the Top Level Domain for ALL domains, but we don’t have to put it in the URL, as browsers understand anyway.
With that in place our plan is:
+ make window.recaptcha undefined by visiting challenge-0623.intigriti.io. instead of challenge-0623.intigriti.io
+ pollute window.recaptcha using the usual deparam vulnerability (`add __proto__[recaptcha]=true` in the URL parameters)
+ Note: The sanitizer is created using the following line:

`new Sanitizer({})`
as opposed to
`new Sanitizer()`
+ The Sanitizer constructor accepts a config object. If this object is empty, it behaves as it it’s not there at all, and defaults to the standard configuration.
+ at the end we don’t need an HTML element. The sitekey variable just needs to be defined. As it’s starts as undefined I guess, we can pollute it. So passing `__proto__[sitekey]=something` will be enough to fire to execute the reCaptcha API correctly, and Fire the XSS!
+ final payload: `https://challenge-0623.intigriti.io./challenge/?name=%3Cdiv%20class%3D%22g-recaptcha%22/%3E&__proto__[recaptcha]=true&__proto__[srcdoc][]=%3Cscript%3Ealert(document.cookie)%3C/script%3E&__proto__[sitekey]=test`


