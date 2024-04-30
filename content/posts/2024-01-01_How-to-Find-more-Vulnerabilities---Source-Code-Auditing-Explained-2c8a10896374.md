+++
title = 'How to Find more Vulnerabilities — Source Code Auditing Explained'
date = 2024-01-01
draft = false
showtoc=true
tags= ["Tutorial", "Security Research", "Static Analysis"]
+++

![](https://cdn-images-1.medium.com/max/800/0*jDmBerRWYf3eZh2J)
*https://images.pexels.com/photos/374559/pexels-photo-374559.jpeg?auto=compress&amp;cs=tinysrgb&amp;w=1260&amp;h=750&amp;dpr=1*

Introduction
============

Whitebox penetration testing can be intimidating. Complex web applications may contain hundreds of thousands of lines of code and deciphering the connection between the various web components and their numerous implementations is challenging. A powerful, yet simple technique to approach the code review of such an application is to break it into manageable pieces.

In this article, I will be outlining a methodology that can be used to break down large web applications, such as Content Management Systems (CMSs) into manageable components that can be systematically analyzed for vulnerabilities or logic errors. This is the approach that I used to discover CVE-2023–43154 and it is the approach taught in the AWAE, OffSec’s prerequisite course to the OffSec Web Expert (OSWE) certification.

Note that this article has an emphasis on web security, but the methodology can be applied to other fields of research as well.

Let’s begin.

Methodology
===========

At a high level, this methodology begins with gaining familiarity with the application as a whole. This phase includes

* Identifying the technology stack — programming language, templating engine, database, etc..
* Mapping out the application to gain a high-level overview of the structure of the project — commands like `tree -L 3` may be used.
* Reading documentation
* Understanding common use cases of the application
* Exploring the application through its interface

These steps help to develop a holistic understanding of the application. This enables you to better relate your subsequent findings to how the application is used, aiding in a more accurate assessment of impact and the likelihood of a successful exploit.

Then, the approach can branch into either of two directions: bottom-up or top-down. To understand what this means, it is first important to define what is meant by a “source” and a “sink”. From there, it will become more clear what is meant by the aforementioned approaches.

Sources and Sinks
-----------------

Vulnerabilities in web applications commonly arise from the manner in which user input is handled. The entry point for user input is referred to as a source. An example of this would be input from a form, a query parameter in the URL, or a cookie. These are sources of user input that are then interpreted and handled by the web application.

A sink, on the contrary, is where the user input is actually used. This is where the vulnerability occurs. In the case of a SQL injection, the source may be a query parameter that has contents concatenated with a SQL query.

To further illustrate this point, I will provide an example. The vulnerable PHP code below has a source, `comment` , and a sink, `<?php echo $_GET[‘comment’]; ?>`. The lack of sanitization on the PHP code as it echos the user input results in an XSS vulnerability.


```
<!-- Source: https://example.com/?comment=<script>alert(1)</script> -->  
<div id="comment">  
 <!-- Sink -->  
 <?php echo $\_GET['comment']; ?>   
</div>
```
For a more visual explanation, this [LiveOverflow](https://youtu.be/ZaOtY4i5w_U?si=zXfJBsNlhp4h-CLa) video provides an excellent demonstration of the topic.

Bottom-Up Approach
------------------

The bottom-up approach refers to first locating sinks and tracing the code “upwards” to the source. For instance, I may search the code base for all occurrences of the sink `system()`, and from there follow the value that is passed into the function to search for any user input that may be implemented into that parameter.

Regular expressions are a common technique to speed up the process of finding sinks. Patterns such as `^.*unserialize\(.+\)` may be used to locate specific sinks (in this case, the `unserialize()` function) with more accuracy and can be used to narrow down the search effort. It is important to note, however, that this method has limitations. The regex results are only as strong as the pattern used and even well-established regular expressions can miss vulnerabilities and sinks.

This method is generally more tedious since not all occurrences of a sink will be used in the context of user-controlled data. Therefore, there can be more code to parse through in order to find a bug. Despite its repetitiveness, this approach has the advantage of increasing the likelihood of finding **higher severity and less accessible vulnerabilities** that are very difficult, if not impossible, to discover without access to the source code.

Top-Down Approach
-----------------

On the contrary, the top-down approach consists of identifying sources and following their implementation in the code until a sink is found. For instance, I may submit the string `“test”` into a `EditProfile` functionality on a website. A top-down approach would consist of finding where the `“test”` string is received in the code and following the logic used on it until I locate its sink, or implementation, in a SQL query.

Regular expressions can be used here as well. A basic example is `^.*?query` to find query parameters. Regular expressions will typically become more complex depending on the specific context and the vulnerabilities that they are searching for.

This approach excels in finding vulnerabilities that are **more accessible, but typically less severe**. To clarify, high severity and high accessibility vulnerabilities can be discovered through either approach. One approach will simply increase your likelihood of finding a particular kind of vulnerability and the one that is chosen will depend on the individual priorities of the code review.

Functionality-Based Approach
----------------------------

A different perspective on code review is breaking the application down into sections based on functionality. Instead of starting broad with all of the application’s sources or sinks and then narrowing down into a vulnerable one, this approach involves starting narrow with one functionality and learning how it works on a deeper level in order to search for vulnerabilities in that particular segment of the application.

This approach is particularly useful when prioritizing certain types of vulnerabilities based on factors such as required authentication level. In this instance, one can curate a list of functionalities that can be used without authentication and analyze those for vulnerabilities. Prioritization is a useful tool when auditing large code bases to increase the chances of finding higher severity vulnerabilities or medium-low severity vulnerabilities that can more elegantly be chained together to result in a higher-severity attack.

![](https://cdn-images-1.medium.com/max/800/0*UPqLe8hb4dLd2h30)
Tooling
=======

I previously mentioned that searching through the code with regular expressions can be used to quickly identify sources and sinks. In addition to this, tools such as Semgrep can also be used to grep for certain patterns in the code. Semgrep looks at the code as more than a text file. It develops and abstract syntax tree (AST) in order to better understand the semantics. This can lead to its recognition of a certain vulnerable pattern in code that a regex pattern would miss. On the other hand, both are only as strong as the way they were written and they can lead to many false positives.

Static Application Security Testing (SAST) solutions can also aide in identifying vulnerabilities. [SpectralOps](https://spectralops.io/), Checkmarx, and Veracode are all examples of SAST tools used by organizations. I have not tried any of these at the time of writing, so I can not recommend one with certainty. SAST tools are generally meant for use by developers in the CI/CD pipeline, but they can also be used by researchers.

Conclusion
==========

The approach to static analysis that you use will impact the types of vulnerabilities that you are more likely and their likelihood of being exploitable. I encourage you to experiment with different approaches to static analysis and decide for yourself which methods work best for your goals. A mixed approach is quite common and even encouraged for best results.

Happy hacking!