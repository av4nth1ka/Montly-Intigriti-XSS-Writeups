Author: https://twitter.com/TheRealBrenu
link: https://challenge-0122-challenge.intigriti.io/

+ Whatever we give as input will be reflected
+ when we try `<h1>heyyyy</h1>`
  it go parsed and shown in the box
+ lets try using the script tag:
`<script>alert()</script>`
We didnt get an alert, which means there is some sanitisation process going on
+ When we reload the page,
  ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/46257d2d-699d-4676-b3bc-77e15e38d618)
We can see a file named main.02a05519.js, looking into the code, we can see a weird comment,
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/0f3abfa8-5f20-47b6-9695-b09ef4618eff)
```
+ Sourcemaps: There is a common practice of compressing and combining files and reducing their size and making it more efficient for the web. It compresses the source codes to a single line of code. But for the debugging processes, it would be difficult to pinpoint the source of an issue. That's where source maps come in—they map your compiled code back to the original code.
```
+ Here we can see that the code is in a single line, which is really really long. we cant spent time reading it and understand it to move on further.
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/3eaa3cfe-1c0f-40e1-a37c-2040aca4ed94)
+ So, as mentioned in the comment, lets see what we get here
https://challenge-0122-challenge.intigriti.io/static/js/main.02a05519.js.map
+ Looking at the sources tab, we can see there two pages,
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/ff5b95c8-0222-486a-9a8b-d8d9c2800c3c)
+ Looking through the lines of code, we found this line:
`<div id="viewer-container" dangerouslySetInnerHTML={I0x12(I0x2)}></div>`
+ Other things we notice in the code is the strange functions which have window.atob(identifiers['']) etc
+ atob() takes the base64 string and convert back into ascii.
+ Also, in the main.02a05519.js file, we have the following set of identifiers.
```
const identifiers = {
            I0x1: "UmVzdWx0",
            I0x2: "cGF5bG9hZEZyb21Vcmw=",
            I0x3: "cXVlcnlSZXN1bHQ=",
            I0x4: "bG9jYXRpb24=",
            I0x5: "c2VhcmNo",
            I0x6: "Z2V0",
            I0x7: "cGF5bG9hZA==",
            I0x8: "cmVzdWx0",
            I0x9: "X19odG1s",
            I0xA: "PGgxIHN0eWxlPSdjb2xvcjogIzAwYmZhNSc+Tm90aGluZyBoZXJlITwvaDE+",
            I0xB: "aGFuZGxlQXR0cmlidXRlcw==",
            I0xC: "ZWxlbWVudA==",
            I0xD: "Y2hpbGQ=",
            I0xE: "Y2hpbGRyZW4=",
            I0xF: "YXR0cmlidXRlcw==",
            I0x10: "Z2V0QXR0cmlidXRl",
            I0x11: "ZGF0YS1kZWJ1Zw==",
            I0x12: "c2FuaXRpemVIVE1M",
            I0x13: "aHRtbE9iag==",
            I0x14: "dGVtcGxhdGU=",
            I0x15: "c2FuaXRpemU=",
            I0x16: "Y3JlYXRlRWxlbWVudA==",
            I0x17: "aW5uZXJIVE1M",
            I0x18: "YXBwZW5kQ2hpbGQ=",
            I0x19: "Z2V0RWxlbWVudHNCeVRhZ05hbWU=",
            I0x1A: "Y29udGVudA==",
            I0x1B: "cmVtb3ZlQ2hpbGQ=",
            I0x1C: "SG9tZQ==",
            I0x1D: "c2V0UGF5bG9hZA==",
            I0x1E: "ZWRpdG9yUmVm",
            I0x1F: "bmF2aWdhdGU=",
            I0x20: "aGFuZGxlU3VibWl0",
            I0x21: "ZXZlbnQ=",
            I0x22: "cHJldmVudERlZmF1bHQ=",
            I0x23: "L3Jlc3VsdD9wYXlsb2FkPQ==",
            I0x24: "dmFsdWU=",
            I0x25: "a2V5",
            I0x26: "VGFi",
            I0x27: "c2hpZnRLZXk=",
            I0x28: "c2V0UmFuZ2VUZXh0",
            I0x29: "ICAgIA==",
            I0x2A: "c2VsZWN0aW9uU3RhcnQ=",
            I0x2B: "ZW5k",
            I0x2C: "bGluZVN0YXJ0",
            I0x2D: "c3RhcnQ=",
            I0x2E: "bGVuZ3Ro",
            I0x2F: "c2xpY2U=",
            I0x30: "c2V0U2VsZWN0aW9uUmFuZ2U=",
            I0x31: "Cg==",
            I0x32: "Ym9keQ==",
            I0x33: "dGFyZ2V0",
            I0x34: "Y3VycmVudA=="
        };
```
So, in the l0x1 files, it is converting the above given base64 identifiers into the ascii format
+ So, lets try decoding one fo the base64 string as follows:
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/266fff54-6dda-4e87-9d82-96bf0a1504be)
![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/e237a268-1d18-434e-aa1b-0927e8f31f3f)
+ So, decoding each of them manually is difficult, so we can write a loop as follows:
  - first copy the index.js code and paste it in console as follows
    ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/a3cd3ce2-52dd-47e8-a6cb-9042066d2d35)
    ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/d8b6e5c7-817b-4930-b13a-fd436b8004fa)
  - now i need to replace very identifier with the base64 decoded form. for that we can write a looop.
    ```
    for(let id of Object.keys(identifiers)) {
    text=text.replaceAll(`window.atob(identifiers["${id}"])`, `"${atob(identifiers[id])}"`)
    }
    ```
    ![image](https://github.com/av4nth1ka/Intigriti-XSS-challenges/assets/80388135/3858a085-73bb-4559-87dc-110927cca698)

The prettier version of the above code:
```

import { useState } from "react";
import DOMPurify from "dompurify";
import "../../App.css";

function I0x1({ identifiers }) {
  const [I0x2, _] = useState(() => {
    const I0x3 = new URLSearchParams(
      window["location"]["search"]
    )["get"]("payload");

    if (I0x3) {
      const I0x8 = {};
      I0x8["__html"] = I0x3;

      return I0x8;
    }

    const I0x8 = {};
    I0x8["__html"] = "<h1 style='color: #00bfa5'>Nothing here!</h1>";

    return I0x8;
  });

  function I0xB(I0xC) {
    for (const I0xD of I0xC["children"]) {
      if (
        "data-debug" in
        I0xD["attributes"]
      ) {
        new Function(
          I0xD["getAttribute"](
            "data-debug"
          )
        )();
      }

      I0xB(I0xD);
    }
  }

  function I0x12(I0x13) {
    I0x13["__html"] = DOMPurify[
      "sanitize"
    ](I0x13["__html"]);

    let I0x14 = document["createElement"](
      "template"
    );
    I0x14["innerHTML"] =
      I0x13["__html"];
    document["body"][
      "appendChild"
    ](I0x14);

    I0x14 = document["getElementsByTagName"](
      "template"
    )[0];
    I0xB(I0x14["content"]);

    document["body"][
      "removeChild"
    ](I0x14);

    return I0x13;
  }

  return (
    <div className="App">
      <h1>Here is the result!</h1>
      <div id="viewer-container" dangerouslySetInnerHTML={I0x12(I0x2)}></div>
    </div>
  );
}

export default I0x1;
```


