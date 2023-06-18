# October XSS Challenge 2021

Link: https://challenge-1021.intigriti.io/ <br>
Description: Find a way to execute arbitrary javascript on this page and win Intigriti swag.<br>
Author: @0xTib3rius: https://twitter.com/0xTib3rius <br>

# Analysis
+ Looking at the source code we can see that it is getting redirected to `challenge/challenge.php`
+ https://challenge-1021.intigriti.io/challenge/challenge.php
+ The description given in the challenge
```
ARE YOU SCARED?
ARE YOU STILL SANE?
NOBODY CAN BREAK THIS!
NOBODY CAN SAVE INTIGRITI
I USE ?html= TO CONVEY THESE MESSAGES
I'LL RELEASE INTIGRITI FROM MY WRATH...
... AFTER YOU POP AN XSS
ELSE, INTIGRITI IS MINE!
SIGNED* 1337Witch69
```
So, there is a ?html get parameter. So, when we tried giving something in the html parameter for example, ?html=hello, hello will be printed in the page. So, yes we can insert some content. What we need in xss. Lets try to get a simple xss.
+ ?html=<script>alert(document.domain)</script>, suprisingly the page is just blank.
+ Now looking at the source code, we can see that there is csp.
```
http-equiv="Content-Security-Policy"
      content="default-src 'none'; script-src 'unsafe-eval' 'strict-dynamic' 'nonce-acc4f1f009b7b4257c495e6edbe035bf'; style-src 'nonce-8d1bbffa377d5e5074bd2926a61907ba'"
```
+ CSP basically says what to load and what not to load on a webpage. Here the default-src is given as none, which means it doesnot allow to load anything. Then script-src defines valid sources of javascript, and it set to unsafe-eval which means Allows unsafe dynamic code evaluation such as JavaScript eval(). The script-src also has an attribute called nounce, which means that script tags which has a attribute nounce with the given value will be allowed to run. That is why our script didnt run. But the nounce is randomly generated. so we can send a script tag with a nounce cuz the with every new request the nounce will be new.
+ Strict-dynamic: The 'strict-dynamic' source expression specifies that the trust explicitly given to a script present in the markup, by accompanying it with a nonce or a hash, shall be propagated to all the scripts loaded by that root script. At the same time, any allowlist or source expressions such as 'self' or 'unsafe-inline' will be ignored, then trust given to one script will be give to another script which was made from the other script.
+ So, lets again loook into the source code:
```
window.addEventListener("DOMContentLoaded", function () {
        e = `)]}'` + new URL(location.href).searchParams.get("xss");
        c = document.getElementById("body").lastElementChild;
        if (c.id === "intigriti") {
          l = c.lastElementChild;
          i = l.innerHTML.trim();
          f = i.substr(i.length - 4);
          e = f + e;
        }
        let s = document.createElement("script");
        s.type = "text/javascript";
        s.appendChild(document.createTextNode(e));
        document.body.appendChild(s);
      });
 ```
 + From the code we can see that a script tag is being created and it will be executed. We can also see that a child gets appended called e. The variable e has some random character string + the value of the get parameter xss. Our payload we give in xss will get executed by the script tag.Lets check whether we can get an alert from the above script tag.
 + ?xss=alert(document.domain), when we give this, it didnt got executed because of the garbage infront of it.
 ![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/24135b81-f23a-4428-888f-8108d36feddf)
+ Again looking at the source code we can see that something is getting prepended with e.
```
if (c.id === "intigriti") {
          l = c.lastElementChild;
          i = l.innerHTML.trim();
          f = i.substr(i.length - 4);
          e = f + e;
```
+ So, our goal is to create a tag with id=intigriti, take its last child element, so we need to create another tag inside the main tag, and then the last four characters of the child element content will be prepended to e. We, need to prepend a single quote('), so try giving the following payload to xss get parameter.
`<tagid="intigrit"><tag2>'abc</tag2></tag>`
+ ![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/79deca18-a84d-4fe0-8977-2e3c1b9fd55c)
We can see that the tag is in body tag with the id=body, and it is not in the last element that is with the id=container.
So, the <tag> is inside the h1 tag. To, get out of the h1 tag, we can prepend a closing tag infront of the payload. Like wise we can also escape from div tag by closing a div tag.
![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/8ad087b3-25dc-4591-a4e3-2691abd86c5d)
+ So, now we in the first level of the children of the body tag, but we need to be the last. 
+ A weird functionality of the browser is that, it closes every tag which is left open. Lets try removing the </tag> from our payload and see what happens
![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/1c4f73e6-f128-4adb-802e-c77566c46a96)
We can see that the browser automatically closed the tag and now <tag> became the last child, and the div tag with container id came inside the <tag>.
+ Now we need to get the quote to the last element. Lets try removing the </tag2> as well.
![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/e4844d71-2839-49cf-8423-f0726872399b)
+ Now we need to create something after the div tag. we know that if we create a tag the browser will create a closing tag.
 So, lets try giving a random tag as follows:
`?xss=alert(document.domain)&html=</h1></div><tag id="intigriti"><tag2><hello>`
![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/21799a5b-c9d5-4ca6-92ec-54be658ef893)
+ Now lets try adding quotes.
![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/832583ad-9713-4807-acda-9f375c880f09)
The result will be as above, but we need the quote in the front so as to execute the alert() statement.
``` 
 ;alert(document.domain)&html=</h1></div><tag id="intigriti"><tag2><hello'aa> 
```
 'Giving the above payload, 
![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/6481eec2-6d73-4dd9-b7b3-4094841ef930)
Hence solved!!
                                                                          
## Reference:
+ https://content-security-policy.com/
+ https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src
                                                                         
