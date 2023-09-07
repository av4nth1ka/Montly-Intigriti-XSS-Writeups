+ we have three dropdown options.
    - milk
    - cookies
    - alert
 
+ as we need xss, we first go with alert. After choosing alert, we have a prompt to write a message. Whatever we write in the prompt, we get it as a pop up
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/8ff40c40-2998-42af-a289-a4743ddb3c1e)
+ but we can see that we cant execute our payloads. Whatever we give is surrounded by backticks. WHat does backticks in javascript do?
  https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/a5202e3d-b28d-4ab0-b7a6-56399205640d)
Template literals are sometimes informally called template strings, because they are used most commonly for string interpolation (to create strings by doing substitution of placeholders). However, a tagged template literal may not result in a string; it can be used with a custom tag function to perform whatever operations you want on the different parts of the template literal.
+ Take a look at ${expression} So in ALERT choice we can use dollar sign and curly quotes so this means that we can execute JavaScript.
+ for example if we give: ${1+1} the output will be 2
+ but we can get alert when we give document.domain. Bc we will get a error in our Console ‘uncaught reference error document is not defined’ and the reason is bc in JavaScript everything is sensitive so uppercase document is not same as lowercase document. So this is impossible to bypass.

+ Next let us try with cookies option:
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/35514f00-e092-4aaa-8d5f-780615d8b5a6)
+ The values for the eggs and chocolate GET parameters, along with the URL itself and a value called price are being reflected on the page.
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/f4bcaa80-c2f9-4ea0-9f39-e139deaba855)
+ First of all it’s gonna create a function cookie spawn and then it’s gonna do some interesting stuff. So it will set a cookie, cookie shop, it will create a function create which is gonna call cookie spawn with eggs, chocolates and with full URL and other part of URL. All of this is happening in script element. So our user input is being used in a script tag in a function so this is is really dangerous
+ So, lets try giving all special characters in the get parameter and see which one is allowed and which one is not allowed.
+ `https://challenge-0622.intigriti.io/challenge/index.php?choice=cookie&eggs=eggs&chocolate=chocolate~!@$%^&*()_-+=%3C%3E?:%22}{[]\/`
+ And the output was:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/ad7f8215-7934-4f6b-8e3e-254fcf86ad3f)
+ we can see the script tags & quotes are all not allowed.
+ we understood that our input is put straight into the function, so lets try giving some characters which disturbs the functioning of the string.
+ Lets try giving backslash, https://challenge-0622.intigriti.io/challenge/index.php?choice=cookie&\
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/b996f77e-cbee-4826-b853-bd71deb4d9d8)
+ So, we have successfully escaped the single quote for the third argument of cookie_spawn and this can semi-arbitrarily inject JS via the fourth argument. If we want to achieve XSS, we first need to add two forward slashes, at the end of the url because the 4th arguement is messing with the backslash, we need to escape it. so the function becomes:
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/7e5f4bce-ec81-461f-9a0a-4037e6e045ef)
+ Now, we have another error. the remaining part of the fourth arguement is functioning as a ternary operator. so lets try giving as follows:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/650000b2-b3c8-4e0e-90aa-6eccef9065b5)
+ now we have another error. as we escaped using backslashes, we the closing bracket go comment out. so lets add another ) bracket.
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/30c7a5d0-2dfb-48dd-847e-5b05a2df3320)
  Now, its said challenge not defined. When i tried giving my payload:
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/8770b020-6cd3-4e15-9f15-2984a6990996)
  it does work because, /challenge/index.php?choice=cookie:1 this part errors out. So, lets try closing the brace before the xss payload.
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/47ef53e5-2553-4e4f-9e2d-74e7e04e83b6)
  But agains we get an error, about the closing curly brace because there is another closing brace left. So, lets try correcting the remaining brackets as follows:
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/c2aae8f9-c587-4d6d-9f11-ae136ff83983)
and yes we got an xss. lets try getting the xss through the url.
+ so giving everything extrac gave in the console in the url as well, the final payload would be this:
+ `https://challenge-0622.intigriti.io/challenge/index.php?choice=cookie&:1);}alert(document.domain);a={//\`
  And we got the xss!

  








