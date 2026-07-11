# 🤠 Here is Hackerone link:
	https://hackerone.com/bose_vdp
## Overview

Last updated on February 11, 2026. [View changes](https://hackerone.com/bose_vdp/policy_versions) 

# Program Rules

- Please provide detailed reports with reproducible steps. If the report is not detailed enough to reproduce the issue, the issue may not be marked as triaged.
- Submit one vulnerability per report unless you need to chain vulnerabilities to provide impact.
- When duplicates occur, we only triage the first report received (provided that it can be fully reproduced).
- Only interact with accounts you own or with the explicit permission of the account holder.

# Test Plan

Session Layer: HTTP Headers

Researchers should add headers to requests such as:

```
X-HackerOne-Research: [H1 username]
```

## ☢️ Target basic recon info on github:
	https://github.com/sudo-su-ayan-ali/docs/tree/main/bug_hunting/bose

## 🧑🏻‍🤝‍🧑🏾 users details:
- #user1= cythom12345@proton.me #passwd1= Cythom@12345 -> chrome
- #user2= cybus12345@proton.me #passwd2= Cybus@12345 -> firefox

## 🗺️ Mapping all the areas:
- Mapping using **user1** for maximum manual mapping for the site.

- on `/account` :-
	- personal details :-
		- name, phone, email, password, sign uped date, birthdate
	- Addresess :- 
		- delivery address
	- Marketing & communications
		- Review your subscription status for email and mobile app notifications. If you’d like to receive Bose communications related to new products, promotions, and more, please subscribe below. If your current status is “Not chosen,” you will not receive these communications and do not need to unsubscribe.
- on /orders :-
- types of id or other parameters:

| parameters | value                                                                                           | type                              | reason for used in request                                                                          |
| ---------- | ----------------------------------------------------------------------------------------------- | --------------------------------- | --------------------------------------------------------------------------------------------------- |
| dwsid      | dwsid=z6SZe0tKxXO7ItDY4c-2ebK8SBh0uaxm-yHlr08yhx4keWUyxJKw7SBjgS6TTg9YFEoic4gvVT1ctBvi2KqP1A==; | **visitor session ID**            | **Salesforce Commerce Cloud** (formerly Demandware) that stores your current **visitor session ID** |
| _cfuvid    | _cfuvid=S2wjtAfqD0TH2CAJIjR6mhFUL07IfDkf6t5moiDrqeg-1779877836303-0.0.1.1-604800000             | security cookie for rate limiting | **cookie to distinguish between users from the same IP address.**                                   |
|            |                                                                                                 |                                   |                                                                                                     |
|            |                                                                                                 |                                   |                                                                                                     |
