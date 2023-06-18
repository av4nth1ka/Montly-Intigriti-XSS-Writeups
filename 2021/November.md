# November XSS Challenge 2021

Description: Find a way to execute arbitrary javascript on the iFramed page and win Intigriti swag.<br>
Link: https://challenge-1121.intigriti.io/challenge/index.php?s= <br>
Author: https://twitter.com/IvarsVids <br>

# Analysis & Solution

+ This page basically has the details of top 10 owasp vulnerabilities.
+ We have a search functionality, when we give, ?s=security, we will some results
+ we can also see that what ever we search is getting printed in the page, when we try some simple xss, it wont work.
+ While looking at the source code, we can see that they search payload is also reflecting the title of the page, but not in the search area, which means it is only reflecting in the client side, which brings the possibility of client side template injection.
```
<p>You searched for v-{{search}}</p>
<ul class="tilesWrap">
  <li v-for="item in owasp">
    <h2>v-{{item.target}}</h2>
    <h3>v-{{item.title}}</h3>
```
+ This application is using vuejs. So, the application is using some templating to put the correct value there ans templates in vue are not vulnerable to xss.
+ In title the templating is not used, the string is just echoed in there.But we dont get an xss, because we still in the title, lets try closing the title tag. 
payload: ?s=</title><script>alert(1)<%2Fscript>
+ ![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/f5c759c3-7121-4273-89a5-f3fe5b6a770a)
+ The script still didnt get executed. Looking at the console we can see the following error.
![image](https://github.com/Avanthikaanand/Intigriti-XSS-challenges/assets/80388135/7522f8b3-d45b-42fd-9210-befa0269aa9c)
+ So, there is content security policy passed on as a header.
 ```
 content-security-policy
	base-uri 'self'; default-src 'self'; script-src 'unsafe-eval' 'nonce-1c20499376a893b7542be51a5998ef1d' 'strict-dynamic'; object-src 'none'; style-src 'sha256-dpZAgKnDDhzFfwKbmWwkl1IEwmNIKxUv+uw+QP89W3Q='
 ```
 + 


