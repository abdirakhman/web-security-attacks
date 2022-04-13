# The same origin policy
The same-origin policy is a critical security mechanism that restricts how a document or script loaded by one origin can interact with a resource from another origin.
In other words, suppose you have many tabs in your browser. And of them is malicious website and one of them is bank website. Same origin policy restricts malicious website to access bank website.   

## Definition of an origin
TL;DR <br>
Origin = Domain name + Protocol + Port

Two URLs have the same origin if the protocol, port (if specified), and host are the same for both. You may see this referenced as the "scheme/host/port tuple", or just "tuple". (A "tuple" is a set of items that together comprise a whole — a generic form for double/triple/quadruple/quintuple/etc.) <br>
[More about same origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)

## Cookies
Suppose we have following URL: [scheme://domain:port/path?params](). Then Same Original Policy (SOP) for cookies will be ([scheme], domain, path). [] - means optional. 
## Setting/deleting cookies by server
Cookies are set by the server using Set-cookie header. 

```
Set-cookie: NAME=VALUE ;
domain = (when to send) ;
path = (when to send)
secure = (only send over SSL);
expires = (when expires) ;
HttpOnly (later)
```
Note1: if expires = NULL, then cookies are for this session only. <br>
Note2: Default scope is domain and path of setting URL <br>
Note3: To delete cookie we can set “expires” to date in past <br>
The domain and path fields are called scope of the cookie. 
## Scope setting rules
We can set cookie for any domain-suffix URL-hostname, except TLD. In other words, a page can set a cookie for its own domain or any parent domain, as long as the parent domain is not a public suffix. For example, suppose host="login.site.com". Then we can set cookies to "login.site.com" and ".site.com". However, we can not set cookies to "user.site.com", "othersite.com" or ".com". 
## Identification of cookies
They are identified by (name, domain, path)
Suppose we have two cookies:
```
cookie 1
name = userid
value = test
domain = login.site.com
path = /
secure
```
and
```
cookie 2
name = userid
value = test123
domain = .site.com
path = /
secure
```
We can see that they are distinct as their domain are not same.
## Reading cookies on server
When browser access the server it sends all the cookies in URL scope: 
* cookie-domain is domain-suffix of URL-domain, and
* cookie-path is prefix of URL-path, and
* [protocol=HTTPS if cookie is “secure”]

Example:
So suppose you have two cookies which are both set by "login.site.com".
```
cookie 1
name = userid
value = u1
domain = login.site.com
path = /
secure
```
```
cookie 2
name = userid
value = u2
domain = .site.com
path = /
non-secure
```
And if you access 
| Website | cookies sent |
| ------- | ------------ |
| http://checkout.site.com/ | cookie: userid=u2 |
| http://login.site.com/ | cookie: userid=u2 |
| https://login.site.com/ | cookie: userid=u1; userid=u2 (arbitrary order) |

## HttpOnly Cookies
A cookie with the HttpOnly attribute is inaccessible to the JavaScript Document.cookie API; it's only sent to the server. For example, cookies that persist in server-side sessions don't need to be available to JavaScript and should have the HttpOnly attribute. This precaution helps mitigate cross-site scripting (XSS) attacks.

## Potential cookie problems
When cookies are sent, server can not see the cookie attributes (e.g. secure, HttpOnly) and it can not see which domain set the cookie. In other words server sees only `Cookie: NAME=VALUE`.
### Problem 1.
Suppose Alice logins at "login.site.com". The server will send session id cookies for ".site.com".
Then if Alice visits "evil.site.com" it will overwrite ".site.com" session-id cookie with session-id of user “badguy”.
And if Alice visits "course.site.com" to submit homework,
"course.site.com" thinks it is talking to “badguy”.
Problem: course.site.com expects session-id from
login.site.com; It is hard to tell that session-id cookie was overwritten.
### Problem 2.
Suppose Alice logs in at "https://accounts.google.com/". It will set some cookies with secure tag. What will happen if Alice visits accidentally "http://www.google.com/"? The network attacker can easily append secure cookie to response, thus overwriting secure cookie. 
Problem: network attacker can re-write HTTPS cookies! HTTPS cookie value cannot be trusted

### Problem 3.
Even though "x.com/A" doesn't see cookies of "x.com/B" it can be accessed using DOM SOP. Since DOM SOP is not tied to path "x.com/A" can easily manipulate "x.com/B". 
By writing following code "x.com/A" can access cookies from "x.com/B".
```html
<iframe src=“x.com/B"></iframe>
alert(frames[0].document.cookie);
```
Path separation is done for efficiency not security:
"x.com/A" is only sent the cookies it needs.

### Problem 4.
Cookies have no integrity. User can easily change and delete cookies if needed. In order to fix this problem, HMAC can be used. 