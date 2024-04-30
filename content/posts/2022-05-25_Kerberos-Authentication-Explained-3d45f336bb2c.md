+++
title = 'Kerberos Authentication Explained'
date = 2022-05-25
draft = false
showtoc=true
+++

![Kerberos- a three-headed dog](https://cdn-images-1.medium.com/max/800/1*lPQ16gDtSYYccrc2-iExiA.jpeg)
When first learning Kerberos, it can feel like you’re being chased by the three-headed dog. Not to fear, however, because today I’ll be explaining a high-level overview of Kerberos authentication. Kerberos was designed to provide secure authentication to services over a potentially insecure network. It is used by many organizations to implement single sign-on (SSO).

Kerberos Terminology
====================

In order to understand the step-by-step explanation, it is important to have a basic understanding of the various components of Kerberos.

* **Kerberos Realm-** the domain in which Kerberos has the ability to authenticate a user
* **Principal-** A unique identity within a Realm that represents either a user or service
* **Client-** The user that is trying to access a service
* **Service-** A resource provided to a client (eg. a file server, application, etc.)
* **Key Distribution Center (KDC)-** Supplies tickets and generates temporary session keys that allow a user to securely authenticate to a service. It also stores the secret symmetric keys for all of the users and services
* **Authentication Server-** ensures that the client making the request to the service is a known user. It then issues a ticket-granting ticket
* **Ticket Granting Server-** ensures that the user is making a request to a known service and grants service tickets

How Kerberos Works
==================

![](https://cdn-images-1.medium.com/max/800/1*X7pcMLxklUmSlsC8ZvDO1A.png)
1. The user sends an unencrypted message to the Authentication server requesting to access a service
2. The Authentication server validates that the request came from a known user and generates a ticket-granting ticket (a ticket that allows you to be granted a ticket by the Ticket Granting Server)
3. The Ticket Granting Ticket (TGT) is sent to the user alongside a message encrypted with the user’s secret key
4. The user uses their secret key to decrypt the message and generates new messages.
5. The user sends their new messages and the TGT that they received (step 3) to the Ticket Granting Server
6. The Ticket Granting Server decrypts the Ticket Granting Ticket and does validation. Then, it creates a service ticket and a new encrypted message back to the user
7. The user decrypts the message and creates an Authenticator message. The user Authenticator and the service ticket is finally sent to the service.
8. The service decrypts and validates the service ticket and Authenticator message. Then, it sends back its own Authenticator message to the user.

To paraphrase this, Kerberos makes sure that the user is legit, then it makes sure that the service is legit. Finally, a secure connection is created between the user and the service.

Key Benefits of Using Kerberos
==============================

* Passwords are never sent across the network
* Encryption keys are never directly exchanged
* Mutual authentication between the client and the application

Conclusion
==========

I hope that this overview cleared up some of the confusion around Kerberos authentication. Thank you for reading.

More Resources
==============

* [Kerberos Authentication Explained | A deep dive](https://youtu.be/5N242XcKAsM)
* [Kerberos (protocol)](https://en.wikipedia.org/wiki/Kerberos_%28protocol%29)
* [How Does Kerberos Work? The Authentication Protocol Explained](https://www.freecodecamp.org/news/how-does-kerberos-work-authentication-protocol/)