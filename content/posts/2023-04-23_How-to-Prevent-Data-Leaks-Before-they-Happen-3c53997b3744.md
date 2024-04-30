+++
title = 'How to Prevent Data Leaks Before they Happen'
date = 2023-04-23
draft = false
+++

Data Loss Prevention

![](https://cdn-images-1.medium.com/max/800/0*65eSMdR110wv_HGg)
What is Data Loss Prevention?
=============================

Data Loss Prevention (DLP) is a strategy for preventing data exfiltration and destruction. Examples of data include financial information, customer data, trade secrets, and other confidential information that could harm a company or its customers if exposed.

Common causes of data loss include:

* **Human error**- accidental deletion of sensitive files, misconfiguring security settings, or being the victim of a social engineering attack.
* **Insider threats**- unauthorized saving and distribution of sensitive files by those with access to corporate systems.
* **Malware**- programs that infect a system and steal or corrupt data.
* **Physical damage**- natural disasters, hardware failures, and power outages can all result in the loss of data.
* **Theft or loss of devices**- data on the device at the time of being stolen/lost may be compromised.

Steps of Data Loss Prevention
=============================

![](https://cdn-images-1.medium.com/max/800/1*3VYpuFQz25D0HYRxMrEyMg.png)
*https://cdn.sketchbubble.com/pub/media/catalog/product/optimized1/7/f/7f3f224484e9abfed723832a1fee08a73cdf27c9b5eb2b177a645fcc3c2c7261/data-loss-prevention-slide2.png*

1. Data Classification
----------------------

Data classification falls under the broad umbrella of data governance. This is essentially the process of determining which data is considered to be sensitive so that it can be protected. It is an important prerequisite step to a successful DLP program.

2. Data Mapping
---------------

This is the process of determining how data is transferred, where data is stored, how it is processed, and who has access to it. The goal of data mapping is to create a comprehensive inventory of an organization’s data assets.

Data is often in one of two states: in transit or at rest. It is typical that an audit will be performed on all the systems and applications that contain data at rest including file servers, cloud storage, databases, email systems, and other data repositories. Once this has been completed, it is common to create a map illustrating the flow of data through an organization. This includes identifying how data is processed, transmitted, and stored, as well as who has access to it at each stage of the process. The information collected through data mapping can be used to develop DLP policies and controls to protect sensitive data.

3. Protect Data
---------------

Data protection involves implementing controls in order to maintain the confidentiality of the information classified in step 1. Common data protection methods include:

* **Encryption**- the robustness and scope of encryption used will be determined by the assets that it is aiming to protect and how sensitive they are.
* **Access Control**- limiting user permissions, enforcing multi-factor authentication (MFA), and potentially monitoring user behavior to detect suspicious activity
* **Security Policy**- social media policy, clear desk policy, acceptable use policy, and end user policy are all potential ways to enforce operational security.
* **Employee Training**- employees should be trained on data handling best practices, strong password policy, and identifying social engineering schemes. Regular awareness campaigns can reinforce the importance of data security.

4. DLP Software Solutions
-------------------------

![](https://cdn-images-1.medium.com/max/800/0*hwsT5tgv1e1AHF6S)
*https://innovative.land/wp-content/uploads/2020/08/Data-Loss-Prevention-DLP.jpg*

Commercial tools can be used to prevent data loss the methods mentioned previously. DLP software can be found from cloud platforms and email providers as native implementations. Commercial solutions include Symantec CloudSOC CASB, Symantec DLP, McAfee Total Protection for DLP, and Digital Guardian to name a few.

As explained by CloudFlare, DLP solutions may detect sensitive data through data fingerprinting, keyword matching in files, pattern matching, file matching, and exact data matching. By comparing the unique identifiers of sensitive data, the software is able to determine whether the sensitive data is being exfiltrated and enact measures to prevent it. For example, the DLP program may block suspicious outgoing emails.

Conclusion
==========

Data loss prevention is an essential element of any organization’s cybersecurity strategy, as it helps protect sensitive data from loss or theft. By implementing a comprehensive DLP program, organizations can reduce the risk of data breaches and maintain the trust of their customers and stakeholders. However, it’s important to remember that DLP requires ongoing monitoring, evaluation, and improvement to ensure its effectiveness over time.

More Reading
============

* <https://www.cloudflare.com/learning/access-management/what-is-dlp/>
* <https://www.youtube.com/watch?v=TPU6VrBa0Fs>
* <https://en.wikipedia.org/wiki/Data_loss>
* <https://www.secureworld.io/industry-news/data-loss-prevention-next-gen-dlp>