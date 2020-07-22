---
layout: page
title: Offensive Security
---

This page will contain general information about Offensive Security/Pentesting/Red Teaming, CTFs I have utilized to learn a bit about the field, some networking information, and a general list of resources

<br>

From the [creators of the OSCP certification and Kali Linux](offensive-security.com/why-offsec/#:~:text=An%20Offensive%20Approach&text=The%20best%20tests%20simulate%20the,offense%20is%20the%20best%20defense.), an explanation on why learning the offensive part of information security is important:

> Well-designed mitigation strategies need routine security tests. The best tests simulate the techniques and methods of an intruder. By encouraging students to use the same tools, techniques, and mindset as a hacker, we level the playing field for defenders.

<br>

## Penetration Testing vs Red Teaming

There is a slight difference between what Penetration Testing and Red Teaming both entail, and if this subfield of security is of interest to you - it's benefitial to learn what the difference is.

A great article by [Rapid7](https://blog.rapid7.com/2016/06/23/penetration-testing-vs-red-teaming-the-age-old-debate-of-pirates-vs-ninja-continues/#:~:text=A%20Penetration%20Test%20often%20takes,that%20will%20achieve%20their%20goals.) goes into some detail on what the goals, and techniques each utilizes that differes from one another. Below is a summary of what the Rapid7 blog says.

On Penetration Testing:

> ... Penetration Testing is testing to find as many vulnerabilities and configuration issues as possible in the time allotted, and exploiting those vulnerabilities to determine the risk of the vulnerability. Penetration Testing is designed to find vulnerabilities and assess to ensure they are not false positives. However, Penetration Testing goes further, as the tester attempts to exploit a vulnerability. This can be done numerous ways and, once a vulnerability is exploited, a good tester will not stop. They will continue to find and exploit other vulnerabilities, chaining attacks together, to reach their goal. Each organization is different, so this goal may change, but usually includes access to Personally Identifiable Information (PII), Protected Health Information (PHI), and trade secrets.

TL;DR: Pentesters find as many vulnerabilities as possible and finds mulitple attack vectors to reach their goal (depending on contract). 

On Red Teaming:

> A Red Team Assessment is similar to a penetration test in many ways but is more targeted. The goal of the Red Team Assessment is NOT to find as many vulnerabilities as possible. The goal is to test the organization's detection and response capabilities. The red team will try to get in and access sensitive information in any way possible, as quietly as possible. The Red Team Assessment emulates a malicious actor targeting attacks and looking to avoid detection, similar to an Advanced Persistent Threat (APT). A Red Team Assessment does not look for multiple vulnerabilities but for those vulnerabilities that will achieve their goals. The goals are often the same as the Penetration Test. Methods used during a Red Team Assessment include Social Engineering (Physical and Electronic), Wireless, External, and more. A Red Team Assessment is NOT for everyone though and should be performed by organizations with mature security programs.

TL;DR: Red Teaming is a more careful, quiet approach with the same goals as a Penetration Test, but it's purpose is to also test the organization's detection/response and to act like a Threat Actor similar to an APT.

