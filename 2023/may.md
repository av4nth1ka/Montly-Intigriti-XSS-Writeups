0523: It’s Fun to Review the E.C.M.A:https://challenge-0523.intigriti.io/



the key takeaways from the analysis of the script are:

+ The payload must contain only alphabets, comma(,), period(.), plus(+), semicolon (;), backslash(\) or parentheses( () )
+ The payload must NOT contain any of the words: alert|prompt|eval|setTimeout|setInterval|Function|location|open|document|script|url|HTML|Element|href|String|Object|Array|Number|atob|call|apply|replace|assign|on|write|import|navigator|navigation|fetch|Symbol|name|this|window|self|top|parent|globalThis|new|proto|construct|xss
+ The size of the payload must be less than 100 characters.

+ A good way to look at problems like this is to think of it not in terms of what you are not allowed to do, but in terms of what you can do.
+ In JavaScript, as with most programming languages, single quotes can be used to represent a string, and the plus (+) operator can be used for string concatenation.
+ This means that while alert does not pass the input validation, 'ale'+'rt' , equivalent to 'alert', will pass it since there is no complete ‘alert’ word in the input! The same can be applied to the worddocument as well.

So, our objective payload transforms from alert(document.domain) to 'ale'+'rt'('docu'+'ment'.domain)
+ the problem here is alert is a function, while 'alert' is a string. Similarly, document is a Document object, while 'document' is a String object. When we attempt to call the function on a string object, we get an error
+ The most common way is to use the eval() function. The eval function evaluates the string and returns the object it reference
eg: eval('ale'+'rt') => alert().
but eval() is also blacklisted and csp is also there.
+ ECMAScript is a JavaScript standard intended to ensure the interoperability of web pages across different web browsers. In the 2015 release of ECMAScript (the 6th version), a lot of new features were added.
The release included two metaprogramming features: The Proxy class and the Reflect object.
In JavaScript, Proxyis an object that wraps around another object and can intercept or modify its behavior. However, the feature of our interest is the Reflect object.
+ The Reflect object in JavaScript has a collection of utility methods that allow you to perform common object-related operations, such as accessing properties, invoking methods, creating objects.
+ Here are the static methods of the Reflect object, linked with the documentation:

Reflect.apply()
Reflect.construct()
Reflect.defineProperty()
Reflect.deleteProperty()
Reflect.get()
Reflect.getOwnPropertyDescriptor()
Reflect.getPrototypeOf()
Reflect.has()
Reflect.isExtensible()
Reflect.ownKeys()
Reflect.preventExtensions()
Reflect.set()
Reflect.setPrototypeOf()
+ The get() method of the Reflect object is semantically equivalent to property access, and hence can be directly applied to our payload:
Reflect.get(window,'ale'+'rt')
+ The words ‘Reflect’ and ‘get’ are also not blacklisted, which means we can use them in our payload directly.Using this, `window[‘alert’]` is equivalent to Reflect.get(window, 'alert')
+ Reflect.get() requires a target object to be passed as the first parameter. This object is a window object, since (as previously mentioned) alert() is a method of the global window object. The document object is also a property of the window object. This means that any payload will require the name of a blacklisted object as a parameter
+ for now, we can use the word ‘window’ as a placeholder in order to obtain the second evolution of our payload:

`Reflect.get(window,'ale'+'rt')(Reflect.get(window,'docu'+'ment').domain)`
+ now we need to figure out how to reference the window object without using the word ‘window’.

+  there are 4 properties which directly return a window object:

- self : Returns a reference to the current window. Does not work for us since the word ‘self’ is not allowed
- parent: Returns a reference to the parent window of the current window, which in this case would be the same window. Does not work again since the word ‘parent’ is not allowed
- top : Returns a reference to the topmost window, which is the current window in the challenge page. Does not work since ‘top’ is not allowed
- window : Self-explanatory
+ There is another property of the window class, which does not return a window object, but rather returns the window as an array-like object.
+ Window frame property: returns the window itself, which is an array like object listing the direct sub-frames of the current window. 
+ And this is the final piece of the puzzle. Substituting window with frame we get our final payload:

`Reflect.get(frames,'ale'+'rt')(Reflect.get(frames,'docu'+'ment').domain)`