* [What is Burp Suite?](https://securitynoodle.github.io/RedTeam/BurpSuite/#what-is-burp-suite)
* [Setting up Burp Suite](https://securitynoodle.github.io/RedTeam/BurpSuite/#setting-up-burp-suite)
* [My Writeups That Used Burp Suite](https://securitynoodle.github.io/RedTeam/BurpSuite/#my-writeups-that-used-burp-suite)
* [Sources](https://securitynoodle.github.io/RedTeam/BurpSuite/#sources)

## What is Burp Suite?
Burp Suite (also referred to as just "Burp") is a popular penetration testing tool which is often used for gathering information about web applications. This tool is meant to be used hands-on as the user controls the actions that are performed, mainly the ability to pass HTTP requests between the Burp tools in order to accomplish specific tasks that are relevant to the penetration test. There are several sub tools within Burp such as Scanner, Intruder, Decoder, and Comparer. However one of particular interest is the Repeater tool which is used to manually modify and HTTP requests.

## Setting up Burp Suite
##### This is set up in the Kali Linux 2020 operating system
1. A JRE warning might popup saying `Your JRE appears to be version X from Debian. Burp has not been fully tested on this platform and you may experience problems`. Just click `Ok`

2. Accept Terms and Conditions

3. You may have be brought up with an update is available screen, either close or update

4. Select `Temporary Project`, Click `Next`

5. Click `Start Burp`

6. Open FireFox ESR

7. On the top right, click on the menu dropdown ☰ and click on `Preferences`

8. Scroll to the bottom, under `Network Settings`, click `Settings`

9. Select `Manual proxy configuration`

10. Under Manual proxy configuration, on the `HTTP Proxy box`, type in `127.0.0.1`, this is the local host. 

11. Change port to `8080`, checkmark the `Use this proxy server for all protocols`. 

12. Click `Ok`

13. Go to `https://burp`

14. You may need to accept the certificate, allow the certificate

15. On the top right, click `CA Certificate` and download to desired location

16. Go back to FireFox preferences menu (from step 7).

17. On the left hand side, click on `Privacy & Security`

18. Scroll to the bottom and click on `View Certificates`.

19. Click `Import...`

20. Import the certificate downloaded from step 15

21. A popup will ask if you trust PortSwigger to identify websites/identify email users, checkmark both and click `Ok`. 

22. Click `Ok`. 

## My Writeups That Used Burp Suite
* HTB-Celestial in progress

## Sources
* [How to use Burp Suite for penetration testing by PortSwigger](https://portswigger.net/burp/documentation/desktop/penetration-testing)
* [Burp Suite Tutorial - Web Application Penetration Testing (Part 1) by PentestGeek](https://www.pentestgeek.com/web-applications/burp-suite-tutorial-1)
* [Quick and Dirty BurpSuite Tutorial (2019 Update) by InfosecInstitute](https://resources.infosecinstitute.com/topic/burpsuite-tutorial/)
