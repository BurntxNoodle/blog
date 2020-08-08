---
layout: post
title: HackTheBox Resolute Writeup
---

Resolute is a Windows box that features two users where the first user is used to get initial access (Melanie) and while as Melanie, one is supposed to find plaintext credentials into the second user’s account: Ryan. It is discovered that Ryan is under the Contractors group which is under the DnsAdmins group. This information is used to escalate into Administrator access via dll injection. The key components of gaining unauthorized administrator access:

* Default/Reuse of new accounts that are created
* Plaintext credentials of other users are found under the “PSTranscripts” directory
* Incorrect group settings/policies

Read the full writeup [here](https://burntxnoodle.github.io/writeups/HTB-Resolute/).
