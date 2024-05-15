Pseudonym generator: https://challenge-1023.intigriti.io/challenge


Analysis:
+ "<%- Outputs the unescaped value into the template".(header.ejs)
+ the page title is being read from the title variable.
+ There are 2 endpoints where title is returned as part of the response (404 and /) where a function getTitle() is invoked as part of the logic.
+ The getTitle() function takes a path variable and consists of 4 steps:

- Decoding the url encoding
- Splitting the path on each /
- Setting the path variable to the last element of the resulting split
- Returning a sanitized string via DOMPurify


+ attack entry point would be somehow bypassing or tricking DOMPurify to not sanitize our input, however we need to take into consideration that the path is split based on / so we would need to find a way to bypass this
+ Our goal is to escape the title tag by somehow entering `</title>` in the url without DOMPurify sanitizing our input.
+ Things to note:
    + DOMPurify does not sanitize any input inside HTML attributes.(This solves our problem where we need to close the `</title>` tag and escape it.)
    + Insecure Puppeteer Setup
The Puppeteer instance, part of the bot.js script, launched a Chromium browser with explicitly disabled security features, namely through the flags --disable-web-security and --no-sandbox. These settings are highly insecure as they disable the same-origin policy and sandboxing, respectively. The same-origin policy is a critical security mechanism that prevents documents or scripts loaded from one origin from interacting with resources from another origin. Disabling it effectively allows any loaded document to request resources from any origin, which is a dangerous setting in a browser controlled by Puppeteer.

The --no-sandbox flag also introduces significant vulnerabilities. Sandboxing is a security feature that executes processes in a restricted environment to limit the potential damage from a malicious exploit. By disabling sandboxing, the browser process runs with higher privileges, potentially allowing a malicious script to perform actions on the host system that would otherwise be restricted.

    + Chrome DevTools Protocol Access
The Chrome DevTools Protocol (CDP) provides extensive control over a Chrome browser instance, including the ability to inspect, debug, and manipulate web pages and their environment in real-time. The challenge's setup did not secure the CDP adequately, as it was possible to brute-force the debugging port. The CDP was accessible because the Puppeteer instance ran with a known or predictable debugging port, which could be accessed without proper authentication or origin checks.

In a secure configuration, WebSocket connections to the CDP should only be allowed from trusted origins, and the debugging port should not be exposed unnecessarily. However, in this challenge, neither of these precautions was taken, leaving the CDP wide open for exploitation. This oversight allowed the execution of arbitrary commands in the browser context, which could be used to bypass security restrictions and access local resources.

