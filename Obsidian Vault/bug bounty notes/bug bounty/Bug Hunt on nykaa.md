# Nykaa’s Responsible Disclosure Policy

Nykaa takes the security of our systems and data privacy very seriously. We constantly strive to make our systems safe for our customers to use. However, if in the rare case a security researcher or member of the general public discovers a security vulnerability in our systems and responsibly shares the details with us, we appreciate their contribution and work closely with them to address any reported issue with urgency. Further, we are happy to acknowledge your contributions publicly.

# Process to report an issue

	Email your findings to [application.security@nykaa.com](https://www.nykaa.com/responsible-disclosure/application.security@nykaa.com) with the subject: "Vulnerability Report <Vulnerability Name>". Please include your contact information along with your mobile number. Note: Reports will not be accepted if the email subject does not include "Vulnerability Report".
	

# Terms and Guidelines

- No user/customer data is modified, deleted or misused without prior explicit permission
- The finding of vulnerabilities should not cause any disruption of services and thus a deprecated user experience for any user
- You shall not expose the findings on any medium - including but not limited to social media, research papers and blogs (personal or otherwise)
- Any and all information and/or finding(s) regarding the vulnerability shall be kept confidential between you and Nykaa and not disclosed to any third party by you at any time
- Exploiting vulnerability for personal gains will lead us to take strict legal action against you
- In case of an inadvertent privacy breach, ensure that you notify us with immediate effect
- You shall allow us time to close the vulnerabilities identified
- Please remember that law of the land is always withheld and while conducting your research, you shall refrain from violating applicable laws and regulation, including but not limited to applicable information technology and data privacy laws
- Assist in mitigation of the vulnerability if required
- You hereby agree to the above mentioned Responsible Disclosure Guidelines and any deviation therefrom will entitle us to take appropriate legal action against you

# SCOPE OF THE PROGRAM

### Targets in scope

- *.nykaa.com
- *.nykaaman.com
- *.nykaafashion.com
- *.nykdbynykaa.com
- *.superstore.in
- *.twentydresses.com
- *.pipabella.com
- *.gloot.co.in
- *.kicaactive.com
- *.lbb.in
- *.lbbshop.in
- *.iykykclub.com
- *.gajragang.com
- Nykaa Beauty mobile app ( [Android](https://play.google.com/store/apps/details?id=com.fsn.nykaa&hl=en_IN&gl=US) | [iOS](https://apps.apple.com/in/app/nykaa-makeup-beauty-shopping/id1022363908) )
- NykaaMan mobile app ( [Android](https://play.google.com/store/apps/details?id=com.fsn.nykaa.man) | [iOS](https://apps.apple.com/in/app/nykaa-man-mens-shopping-app/id1423089781) )
- NykaaFashion mobile app ( [Android](https://play.google.com/store/apps/details?id=com.fsn.nds) | [iOS](https://apps.apple.com/in/app/nykaa-fashion-shopping-app/id1439872423))
- SuperStore mobile app ( [Android](https://play.google.com/store/apps/details?id=com.fsn.nykaa.superstore) )
- Disha mobile app ( [Android](https://play.google.com/store/apps/details?id=com.fsn.nykaa.disha) )
- LBB mobile app ( [Android](https://play.google.com/store/apps/details?id=littleblackbook.com.littleblackbook.lbbdapp.lbb&hl=en_IN&gl=US) | [iOS](https://apps.apple.com/in/app/lbb-lifestyle-discovery-app/id1041376594) )

Out of Scope Targets

All external services/software not managed or controlled by Nykaa are considered out of scope / ineligible for recognition.  
Vendor Endpoints  
Delivery Endpoints  
3rd Party Applications

# OUT OF SCOPE VULNERABILITIES

WEB

1. Vulnerabilities that do not demonstrate security impact will be considered out of scope for this program.
2. Vulnerabilities regarding SPF/DMARC/DKIM records without verifiable proof of spoofing 
3. Best practice concerns like non-session cookies not marked secure and HTTP only, SSL/TLS configuration, missing security headers, etc.
4. Vulnerabilities reported by automated tools and scanners without additional proof of concept
5. End of Life Browsers / Old Browser versions (e.g. Internet Explorer 6)
6. Denial of Service(DoS) and Distributed Denial of Service(DDoS) attacks
7. Exploits that need physical access to the victim’s device
8. Host header injection
9. Unauthenticated/logout/login CSRF
10. Previously known vulnerable libraries without a working Proof of Concept
11. Any kind of spoofing attacks or any attacks that lead to phishing (e.g. Email spoofing, Capturing login credentials with fake login page)
12. Self XSS
13. Bugs requiring exceedingly unlikely user interaction example Social engineering attacks, both against users or Nykaa employees
14. Third-party API key disclosures without any impact or which are supposed to be open/public. Specifically, exposed Google Map API keys and keys in Android XML files. 
15. OPTIONS / TRACE HTTP methods enabled
16. Known public files or directories disclosure (e.g. robots.txt, CSS/images, etc)
17. Presence of application or web browser ‘autocomplete’ or ‘save password’ functionality
18. Any kind of vulnerabilities that require installation of software like web browser add-ons, etc. in the victim's machine
19. Brute force on forms (e.g. Newsletter / ContactUs page)
20. Missing best practices in Content Security Policy.
21. Missing SSL, CAA headers
22. Functional, UI, and UX bugs and spelling mistakes.

Android/IOS

1. Exploits that are reproducible only on rooted/jailbroken devices
2. Absence of certificate pinning
3. Bypassing root/jailbroken detection
4. Snapshot/Pasteboard/Clipboard data leakage
5. Lack of obfuscation
6. Irrelevant activities/intents exported
7. Lack of Exploit mitigations i.e., PIE, ARC, or Stack Canaries in the IOS app
8. Lack of binary protection control

Acknowledgements

We are not part of a cash/bug bounty program but are happy to issue a certificate of recognition to individuals who report security issues responsibly and help us make Nykaa systems more secure
# Enumeration for idor and BAC

## Subdomain enumeration for in scope domains

## Account creation
user1 ---> Firfox
user2 ---> chrome

#user1 cythom12345@proton.me no passwd auth by otp
#user2 cybus12345@proton.me no passwd auth by otp

## untested things
## urls which have sensitive data 

```
https://www.nykaa.com/app-api/index.php  
https://www.nykaa.com/app-api/index.php/customer/check_existence  
https://www.nykaa.com/app-api/index.php/customer/send_otp  
https://www.nykaa.com/app-api/index.php/customer/suggest_mobile_number  
https://www.nykaa.com/app-api/index.php/products/load_wishlist  
https://www.nykaa.com/app-api/index.php/react/navigation?forDevice=desktop  
https://www.nykaa.com/app-api/index.php/store_api/v1/getAllStores  
https://www.nykaa.com/app-api/index.php/user  
https://www.nykaa.com/app-api/index.php/user/update_profile/  
https://www.nykaa.com/app-api/index.php/user/user_details  
https://www.nykaa.com/customer/account/ajaxLogin/  
https://www.nykaa.com/fe-api/omsApis/v1/orderList?email=cythom12345%40proton.me&offset=4&domain=NYKAA  
https://www.nykaa.com/fe-api/vault/system/info/getVaultParamsWithUserDetails  
https://www.nykaa.com/gateway-api/user/otp/email-verification/v1/send  
https://www.nykaa.com/ReactConfig.php  
https://www.nykaa.com/ro5UZCPLnfJB3PD2D-3Yw59scQo/bYaur2u9ki7Gf0umiL/YhJzb2w0Ag/NDV6K3/spEGIB  
https://www.nykaa.com/ro5UZCPLnfJB3PD2D-3Yw59scQo/cDaur2u9ki7G/Sx16b2w0Ag/Rh5XC1/YyBmB5  
https://www.nykaa.com/ro5UZCPLnfJB3PD2D-3Yw59scQo/cDaur2u9ki7G/Sx16b2w0Ag/Rh5XC1/YyBmB5?v=cd43f730-9d39-79a2-d2ee-31190df91737
```


## sensetive perameters

```
{"setPath":{},"userId":"7251765824460563","sessionId":"1511121536923142","sessionProperties":{"time":1785647721511,"id":"1511121536923142","initial_pageview_info":{"time":1785647721511,"id":"173994328729144","title":"Buy Cosmetics Products & Beauty Products Online in India at Best Price | Nykaa","url":{"domain":"www.nykaa.com","path":"/","query":"","hash":""}},"search_keyword":"","referrer":"","utm":{"source":"","medium":"","term":"","content":"","campaign":""}}}; _cs_id=5ae6b997-533d-ae92-fc66-35968eecd565.1785216087.2.1785647722.1785647722.1780035253.1819380087740.1.x; _hp5_let.5185273=1785647725197; PHPSESSID=L5uLsmaYmY9rNcd1785647738865; PRIVE_TIER=null; pro=false; _gcl_au=
```

## cookie 

```
ecoded form:-

Newrelic: eyJ2IjpbMCwxXSwiZCI6eyJ0eSI6IkJyb3dzZXIiLCJhYyI6IjEyMjUxNTkiLCJhcCI6IjEwMDE0NzQ3MTciLCJpZCI6IjU5YzU3YjQxNzIyNTgwNjIiLCJ0ciI6IjQzZGUzYTRjYWRiNWRiNTM1M2Q2OWMxMTlhZjhlODAwIiwidGkiOjE3ODU2NDk2NjgzMTF

decoded form:-
Newrelic: 
{"v":[0,1],"d":{"ty":"Browser","ac":"1225159","ap":"1001474717","id":"59c57b4172258062","tr":"43de3a4cadb5db5353d69c119af8e800","ti":1785649668311}}
```

## userids:-

cybus: `"id":"106795870"`
cythom: ``

## How the Url works

```
"Nykaa","url":{"domain":"www.nykaa.com","path":"/","query":"","hash":""}},"search_keyword":"","referrer":"","utm":{"source":"","medium":"","term":"","content":"","campaign":""}}}; 

```

# Table of high impact features


| Features         | impact | go deeper or not |
| ---------------- | ------ | ---------------- |
| orders           |        |                  |
| profile          |        |                  |
| wallet           |        |                  |
| wishlist         |        |                  |
| giftcard         |        |                  |
| bag              |        |                  |
| my coupons       |        |                  |
| my saved payment |        |                  |
# Features

---
## orders



---

## profile




---

## wallet




---


## wishlist



---

## giftcard




---

## bag



---

## my coupan




---

## my saved payments




---
