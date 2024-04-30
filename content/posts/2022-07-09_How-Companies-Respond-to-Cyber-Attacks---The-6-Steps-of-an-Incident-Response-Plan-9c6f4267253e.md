+++
title = 'How Companies Respond to Cyber Attacks | The 6 Steps of an Incident Response Plan'
date = 2022-07-09
draft = false
showtoc=true
+++
![](https://cdn-images-1.medium.com/max/800/1*NYCBo5F_MGeRvxzrEm1hsg.png)

Introduction
============

This article contains information that I have gathered as I’ve done research on incident response. This aims to be actionable for red teamers to know what to look out for and for blue teamers to aid in the creation of an effective incident response plan.

Key Roles
=========

* the CISO ensures cyberattacks are promptly investigated.
* coordinating efforts of incident response during a cyberattack.
* investigating which data may have been stolen.
* containing and securing compromised systems to prevent further damage.
* working with stakeholders to determine incident response plan.

* this role involves ensuring all steps of a data exposure management plan are completed.
* reviewing applicable privacy laws with the General counsel and creating plans of action to ensure that the laws are adhered to.

* the Incident Response Coordinator will update the CISO and members about new events as they take place.
* provide guidance and directing effort in information gathering and documentation.

* gathering and documenting findings of security systems.
* documenting procedural information and appropriate data for Incident Management.
* providing expert opinions in data and technology.

* The ERT consists of officials with the authority to make key decisions in Incident Response.
* May consist of CISO, privacy officer, General Counsel, Representative(s) from the Office of the President, and head of the organization where the cyber incident occurred.

![Image outlining steps in incident response](https://cdn-images-1.medium.com/max/800/1*b6BEiI2FjVfUf6e-zduodQ.png)
**1. Preparation**
==================

* Make sure there are people on the incident response team
* Make sure the team is trained and has access to the services that they would need in the case of an incident
* Have drills simulating an attack
* Create a plan/playbook for the Security Operations team containing steps and actions to take when compromised.

**2. Identification**
=====================

* Find ways to detect deviations in the regular functioning of an application

Software for this can include:

* IDS/IPS
* Security Incident Event Manager (SIEM)
* Endpoint detection & response (EDR)

Alerts will typically be addressed by a Security Operations Center (SOC) analyst who will assess whether there is a threat or a false alarm.

Suspicious behavior might include:

* Unusual behavior from privileged attacks
* Unauthorized users trying to access servers and data
* Anomalies in outbound network traffic
* Traffic sent to or from unknown locations
* Excessive consumption of computing resources. Can be a sign of cryptojacking or running a heavy program
* Changes in configuration
* Hidden files
* Unexpected changes like users getting locked out of their accounts or memberships changing
* Suspicious registry entries

Identify where the attacker initially got in.

Identify what programs were uploaded, which files were changed, which new processes are running, and if any new registry keys were created.

**3. Containment**

* Prevent any further damage from occurring by containing it.
* **Short-term** containment is isolating the affected devices from the rest of the network.
* **Long-term** containment may involve taking an image of the device and performing hard-disk forensics. For this you would want a Digital forensics analyst. This can be necessary when a deeper analysis is required.

**4. Eradication**

* Undoing the changes that the adversary made
* Installing patches
* Disarming malware
* Disabling compromised accounts

**5. Recovery**

* Return to normal service in the business.
* If clean backups are available, these can be used to restore service
* Compromised devices will need rebuilding to ensure recovery
* Affected devices may need additional monitoring

**6. Lessons learned**

* This is the final stage of incident response.
* Have a post-incident review (PIR), or a meeting where representatives from each team discuss what went well and what could be improved. This is where the incident response plan can be refined.


---

References
==========

* <https://docs.microsoft.com/en-us/security/compass/incident-response-planning>
* <https://www.varonis.com/blog/incident-response-plan>
* <https://security.uconn.edu/incident-response-plan/>