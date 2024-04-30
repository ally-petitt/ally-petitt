<img alt="picture of a bug" class="graf-image" data-height="360" data-image-id="1*W1qjqIKKNMK9_QYWXGQzRw.png" data-is-featured="true" data-width="360" src="https://cdn-images-1.medium.com/max/800/1*W1qjqIKKNMK9_QYWXGQzRw.png"/>

Antivirus Evasion: What it is and How to do it
==============================================

How Does Antivirus Software Actually Work?
==========================================

Antivirus software acts as a defense from trojans, viruses, ransomware, spyware, adware, and much more. There are 3 main ways that it detects malware: signature-based detection, heuristic-based detection, and anomaly-based detection.

<img class="graf-image" data-height="223" data-image-id="1*ehROjARifrRnkEITjkFwMw.png" data-width="867" src="https://cdn-images-1.medium.com/max/800/1*ehROjARifrRnkEITjkFwMw.png"/>

Signature-Based Detection
-------------------------

The scanner will search for specific strings in a program and check for them in a database of known viruses. The strings are often the payload of the malicious code. If the signatures match, the activity is flagged for suspicious activity. Many of these databases store over 250,000 different signatures.

Downsides to this approach:

* The database only stores the values of known signatures
* Newer variations of malware may go undetected if their new signature is not stored in the database
* Viruses can be easily and quickly altered to change their signature

<img class="graf-image" data-height="360" data-image-id="1*8jt-Qw_ai_h_GCSJV15fuQ.png" data-width="640" src="https://cdn-images-1.medium.com/max/800/1*8jt-Qw_ai_h_GCSJV15fuQ.png"/>

Anomaly-Based Detection
-----------------------

Instead of referencing a static database that needs to be continuously updated, this type of detection checks the running program for patterns of . This can be referred to as an expert system. Often, these detection systems will utilize a **machine learning model** that was trained using data from the past years of the company using the antivirus software.

<img class="graf-image" data-height="128" data-image-id="1*meCStYsucAG7XXY88Lnj0w.jpeg" data-width="474" src="https://cdn-images-1.medium.com/max/800/1*meCStYsucAG7XXY88Lnj0w.jpeg"/>

Heuristic-Based Detection
-------------------------

This is one of the few methods capable of detecting polymorphic viruses. Heuristic analysis may consist of multiple different methods, including static heuristic analysis. This technique deals with decompiling a program and comparing the source code to the source code of known viruses stored in a database.

Additionally, heuristic analysis may include dynamic heuristics, a method of containing the program in a virtual machine to test what happens when the code is executed. The running program is monitored for suspicious activity such as overwriting files or self-replication.

<img class="graf-image" data-height="168" data-image-id="1*VY6GEFR7NyUkIPcXNT49bg.jpeg" data-width="299" src="https://cdn-images-1.medium.com/max/800/1*VY6GEFR7NyUkIPcXNT49bg.jpeg"/>

How to Evade Antivirus Detection
================================

**On-Disk Evasion**
-------------------

The following are techniques used for on-disk evasion:

* **Obfuscation**- this method involves rewriting the code to appear more confusing and harder to read.
* **Cryptography**- the code will be encrypted with the decryption key stored in a stub. The program will be decrypted in memory.
* **Packing**- the code is condensed into a smaller binary file which results in a different signature on the payload.
* **Encoding**- the payload may be encoded as base64, hexadecimal, or other types of encodings.

In-Memory Evasion
-----------------

The following is a technique used for in-memory evasion:

* Use Windows APIs to inject a payload into a running process
* Payload is executed in the memory of running process in a separate thread

<img class="graf-image" data-height="165" data-image-id="1*H6l629Qydy1pFxjjyoQn9A.jpeg" data-width="306" src="https://cdn-images-1.medium.com/max/800/1*H6l629Qydy1pFxjjyoQn9A.jpeg"/>

Command-Line Tools
------------------

Here is a list of tools that can help craft undetectable payloads to bypass the antivirus software:

* Veil-Evasion
* Shellter
* Invoke-Obfuscation

Going Deeper
============

Here are some resources for those who are looking for a deeper understanding of antivirus evasion.

* [Veil-Evasion Complete Tutorial](https://youtu.be/iz1twCSJZyo)
* [Defense Evasion — Red Team Notes 2.0](https://dmcxblue.gitbook.io/red-team-notes-2-0/red-team-techniques/defense-evasion)
* [Malware Analysis Bootcamp - Extracting Strings](https://youtu.be/V3_vc7BO9lU)
* [ATT&CK Deep Dive: Defense Evasion](https://youtu.be/WmJcbDfy9L4)
* [Windows Red Team Defense Evasion Techniques](https://www.linode.com/docs/guides/windows-red-team-defense-evasion-techniques/)