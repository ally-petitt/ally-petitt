+++
title = 'How Did an 18-Year-Old Hack Uber?'
date = 2022-09-24
draft = false
+++

![](https://cdn-images-1.medium.com/max/800/0*p8HCs81YRFvhgaR3.jpg)
An 18-year-old hacker gained admin access to Uber on September 15, 2022. These are the steps that the hacker took:

1. The hacker obtained an Uber employee’s phone number.
2. He directed the employee to a phishing site that looked like an Uber login page. The employee logged in and the hacker gained his credentials.
3. The hacker tried to get around the MFA by doing a Multi-Factor Authentication Fatigue attack. This attack consists of spamming MFA requests to the employee until he gets annoyed enough to allow the login attempt to go through.
4. He continued the attack for over an hour before changing tactics. He contacted the employee on WhatsApp claiming to be on the IT team. He said that in order for the spamming to stop, he must accept the request.
5. The employee accepted the request and the hacker gained access to the network.
6. The hacker found network shares with PowerShell scripts that contained hard-coded admin credentials.
7. The hacker used the username and password of the admin to gain access to Amazon Web Services (AWS), GSuite, DA, DUO, OneLogin, Uber security dashboards, and their financial data.
8. He continued to vandalize reports on Uber’s HackerOne bug bounty program before declaring his presence in Uber’s Slack workspace.

![](https://cdn-images-1.medium.com/max/800/0*d0qrDND0kRrSzFmQ)
The hacker leaked screenshots of the Google Admin panel, Avengers dashboard, security dashboards, and their AWS IAM summary.

![Google Admin panel](https://cdn-images-1.medium.com/max/800/1*Nwaj3JZDBIbVPP9y2y9aYQ.png)
![](https://cdn-images-1.medium.com/max/800/0*aBZc_OpaYL5iYHKQ.jpg)
Aftermath
=========

Uber’s stock prices dropped by 6% following the day of the breach, lowering its valuation by over $2 billion. The stock prices did eventually recover. Uber responded with a tweet on their PR Twitter account.

![Picture of tweet from @Uber_Comms responding to security incident](https://cdn-images-1.medium.com/max/800/1*16105fZhVW_EBRmOWDI_hg.png)
Uber followed up by stating that there was no evidence of sensitive user data being stolen, they have notified authorities, and their internal and external applications are operational.

![](https://cdn-images-1.medium.com/max/800/1*oeZ-TBZHlybgDJMZbpdCUg.png)
They also created a “Security Updates” page on their [Uber Newsroom](https://www.uber.com/newsroom/security-update) where they make posts about the developments in their investigation.

Shifting blame
==============

Uber tried to shift the blame towards Lapsus$, an international hacker group that has conducted cyberattacks against large companies and government organizations such as Ubisoft, Nivida, Samsung, and Brazil’s Ministry of Health. It is speculated that Uber is attempting to make their hack less embarrassing by claiming it was done by a more elite organization.

Most Embarrassing Hack Ever?
============================

This hack could have been completely avoided had better cybersecurity practices been instilled in Uber. Hard-coded credentials are among the [2022 CWE top 25](https://cwe.mitre.org/top25/archive/2022/2022_cwe_top25.html#cwe_top_25) most dangerous software weaknesses. Any cybersecurity professional should be aware of the risk that is involved in having hard-coded credentials in an accessible share. In addition, the employee should not have had access to these sensitive network shares. This goes against the fundamental cybersecurity [principle of least privilege](https://www.cisa.gov/uscert/bsi/articles/knowledge/principles/least-privilege). This hack is a learning opportunity of how important the fundamentals are when establishing cybersecurity in an organization.