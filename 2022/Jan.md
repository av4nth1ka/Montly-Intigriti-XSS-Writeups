Author: https://twitter.com/TheRealBrenu
link: https://challenge-0122-challenge.intigriti.io/

+ Whatever we give as input will be reflected
+ when we try `<h1>heyyyy</h1>`
  it go parsed and shown in the box
+ lets try using the script tag:
`<script>alert()</script>`
We didnt get an alert, which means there is some sanitisation process going on
+ When we reload the page,
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/46257d2d-699d-4676-b3bc-77e15e38d618)
We can see a file named main.02a05519.js, looking into the code, we can see a weird comment,
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/0f3abfa8-5f20-47b6-9695-b09ef4618eff)
```
+ Sourcemaps: There is a common practice of compressing and combining files and reducing their size and making it more efficient for the web. It compresses the source codes to a single line of code. But for the debugging processes, it would be difficult to pinpoint the source of an issue. That's where source maps come in—they map your compiled code back to the original code.
```
+ Here we can see that the code is in a single line, which is really really long. we cant spent time reading it and understand it to move on further.
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/3eaa3cfe-1c0f-40e1-a37c-2040aca4ed94)
+ So, as mentioned in the comment, lets see what we get here
https://challenge-0122-challenge.intigriti.io/static/js/main.02a05519.js.map
+ Looking at the sources tab, we can see there two pages,
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/ff5b95c8-0222-486a-9a8b-d8d9c2800c3c)

