+++
title = 'How I Found 3 CVEs in 2 Days'
date = 2024-03-21
draft = false
showtoc=true
tags= ["CVE", "Security Research", "Writeup", "GNU"]
+++

![](https://cdn-images-1.medium.com/max/800/0*wGuRqNuBuFV5u69n.png)
Author: Ally Petitt

Introduction
============

Christmas break is notoriously refreshing for high schoolers like myself, however, unlike most high school students, I got to spend mine doing the most fascinating work in the world: security research.

I had previously used Savannah, a GNU bug tracker, to submit a bug report, so when I noticed that the underlying technology, [Savane](https://git.savannah.nongnu.org/cgit/administration/savane.git/), was open source, I knew I had to put it on my list of research projects. To my surprise, I was able to discover 3 CVEs within the span of 2 days including insecure access control, cross-site request forgery (CSRF), and a bad seed vulnerability in Savane v3.12 and prior versions.

Testing Environment:

* Firefox v103.0 (64-bit)
* Debian 12, stable-slim Docker container
* PHP development server

Note that these aren’t all-encompassing results as my short audit of this application was only over the course of a few days.

Methodology
===========

This audit consisted of both dynamic and static analysis. The majority of my source code analysis methodology can be found [here](https://medium.com/@allypetitt/how-to-find-more-vulnerabilities-source-code-auditing-explained-2c8a10896374). For the sake of brevity, I will only include relevant details for this article. My first step was configuring a local development environment via a [Dockerfile I](https://gist.github.com/ally-petitt/3490c857b4c6eeea860c6e094ce47ce3) created.

With a running instance of Savane, I shifted my focus to source code analysis, pinpointing files of particular interest for security issues including `utils.php`, `upload.php`, and `users.php`. While there were interesting leads, many of them did not pan out how I had hoped, so I began to parse through functionalities with more potential for impact. This led me to authenticated functionalities since any user can create an account and expands the attack surface drastically.

Account Management
==================

I began by assessing account management functionalities such as updating passwords and changing email addresses for Cross-Site Request Forgery (CSRF) or access control vulnerabilities since they happened to be at the forefront of my mind at the time of this audit.

The majority of the critical account management functionalities appeared to be protected by a`form_id` parameter, which contained a randomized string in each request. In effect, `form_id` acted as a CSRF token.

![](https://cdn-images-1.medium.com/max/800/1*Q3BI1WCSQj8kJmb7iCEh4g.png)
Additionally, attempting to change the password required the previous user password to be known, which provided no advantage over attempting to brute-force their password to begin with.

![](https://cdn-images-1.medium.com/max/800/1*IcA_HpVxf3LP9FlM7miVUw.png)
*http://172.17.0.2:7890/my/admin/change.php?item=password*

With these precautions in place, it was evident that further investigation was required.

CVE-2024–27631 — CSRF (CWE-**352)**
-----------------------------------

I suspected that there may be additional critical vulnerabilities without CSRF protection. Specifically, I began to look inside of admin folders in the source code, eventually finding `siteadmin/usergroup.php`, which enabled superusers to edit of any user’s profile.

![](https://cdn-images-1.medium.com/max/800/1*_stvfUY2HtkR9wda-TgZQQ.png)
*Example of /siteadmin/usergroup.php to edit the account of user 60079*

This page contained three unprotected functionalities of interest. The first was a function that can be used to grant a user admin flags, leading to privilege escalation without account takeover.

![](https://cdn-images-1.medium.com/max/800/1*TDquE7S3UnTHXilR4TYrVA.png)
*frontend/php/siteadmin/usergroup.php*

The second and third functionalities could be leveraged for account takeover by either changing the email address of a user’s account:

![](https://cdn-images-1.medium.com/max/800/1*HvHtu0HaoxRLzojL3vjfPQ.png)
*frontend/php/siteadmin/usergroup.php*

Or, changing the password of any user’s account.

![](https://cdn-images-1.medium.com/max/800/1*-N45bZ7BrPdguKZWsQs0cw.png)
*frontend/php/siteadmin/user_changepw.php*

Given these promising leads, I began to develop a Proof-of-Concept (PoC) to demonstrate that the vulnerability existed in practice. (I have found that with static analysis, sometimes a detail in the code can be overlooked that actually mitigates the vulnerability.) The following is what I came up with and successfully tested:


```
<!-- The efficacy of this payload is browser-dependent -->  
<form id="autosubmit" action="http://<savane\_instance>/siteadmin/user\_changepw.php" method="POST">  
  
 <input name="form\_pw" type="hidden" value="Password1!" />  
 <input name="form\_pw2" type="hidden" value="Password1!" />  
 <input name="user\_id" type="hidden" value="<user\_id>" />  
 <input name="update" type="hidden" value="Update" />  
 <input type="submit" value="Submit Request" />  
</form>  
  
<script>  
 document.getElementById("autosubmit").submit();  
</script>
```
Thus, I discovered CVE-2024–27631! A patch was implemented in [this commit](https://git.savannah.nongnu.org/cgit/administration/savane.git/commit/?h=i18n&id=d3962d3feb75467489b869204db98e2dffaaaf09).

CVE-2024–27632 — Bad Seed (CWE-335)
-----------------------------------

While tracing CSRF-related functionalities in the code, I came across the logic for generating and serving the CSRF tokens, or form IDs. The execution flow is as follows:

1. Protected PHP pages contain a call to `form_header()`.
2. The `form_header()` function seeds the current Unix timestamp.
3. A random number is chosen and hashed to create `form_id`.
4. The form ID is added as a hidden element in the form on the protected page.

The definition of `form_header()` can be seen in the image below.

![](https://cdn-images-1.medium.com/max/800/1*Zs5bxl01fMcxsXTfZd4mzw.png)
*frontend/php/include/form.php*

I was particularly interested in the mechanism behind seeding the Pseudo-Random Number Generator ([PRNG](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)), which occured in `utils_srand()`.

![](https://cdn-images-1.medium.com/max/800/1*SIBvRLsqwT0QVwFI1ZczrA.png)
*frontend/php/include/utils.php*

It was apparent that `[mt\_srand()](https://www.php.net/manual/en/function.mt-srand.php)` was used to seed the [Unix timestamp](https://www.php.net/manual/en/function.microtime.php) (learn more about PRNGs [here](https://www.geeksforgeeks.org/pseudo-random-number-generator-prng/)). Since this function is called when a protected page is loaded, the seed is renewed with the current time upon visiting the page. If the timestamp that a victim visited a page is known, can be approximated, or can be otherwise triggered, it is possible to independently generate the exact same form ID token, passing the CSRF check! This would allow for CSRF attacks on arbitrary form submissions, leading to potential account takeover.

Savane v3.13 contains a patch implemented in this [commit](https://git.savannah.nongnu.org/cgit/administration/savane.git/commit/?h=i18n&id=dee5195d18f9ab16c860e8114819083673f66b95).

Group Management Functionalities
================================

Another class of functionalities I began to explore was group management. I tested CRUD (Create, Read, Update, and Delete) functionalities utilized when submitting a bug report to a group. I was unable to exploit file uploads in bug reports (I may address this in a future article if enough people are interested), however, I was vigilant on access control issues when I observed the web traffic for deleting an uploaded attachment.

![](https://cdn-images-1.medium.com/max/800/1*4Pup6AnABk8L9bM8QGeGow.png)
*Deleting a file attachment with ID 40225*

CVE-2024–27630 — IDOR (**CWE-639**)
-----------------------------------

Upon submitting a bug report, uploaded files are deposited in the uploads directory (`/var/lib/savane/trackers_attachments`) with a file name equivalent to the file’s ID.

Naturally, I began to investigate the process of deleting a file, a function only accessible to tracker admins. The following function is responsible for handling file deletion on attachments (`frontend/php/include/trackers/data.php:2417`):


```
function trackers\_data\_delete\_file ($group\_id, $item\_id, $file\_id)  
{  
 global $sys\_trackers\_attachments\_dir;  
 # Make sure the attachment belongs to the group.  
 $res = db\_execute ("  
 SELECT bug\_id from " . ARTIFACT . " WHERE bug\_id = ? AND group\_id = ?",  
 [$item\_id, $group\_id]  
 );  
 if (db\_numrows ($res) <= 0)  
 {  
 # TRANSLATORS: the argument is item id (a number).  
 $msg = sprintf (  
 \_("Item #%s doesn't belong to project"), $item\_id  
 );  
 fb ($msg, 1);  
 return;  
 }  
  
 $result = false;  
 # Delete the attachment.  
 if (unlink ("$sys\_trackers\_attachments\_dir/$file\_id"))   
 $result = db\_execute ("  
 DELETE FROM trackers\_file WHERE item\_id = ? AND file\_id = ?",  
 [$item\_id, $file\_id]  
 );
```
The user can control the `$item_id` and `$file_id` parameters through the URI. Can you spot the vulnerability? It is subtle.

Note that there is input validation ensure the `$file_id` is a number. Otherwise, there would be a directory traversal vulnerability allowing for arbitrary file deletion.

The function first checks if the user is part of the group corresponding to the `$item_id` and proceeds to delete the attachment before running a SQL query updating the database. The `$file_id` is not checked at all! This means that as long as the attacker is an admin of the group referenced in `$item_id`, they could delete any file.

I was quite surprised to have spotted this discrepency, but very pleased. The following steps can be taken to reproduce the vulnerability:

1. Have an account that is an admin of a group with a bugtracker (attacker account).
2. With a separate user account (victim), upload a file attachment in a bug report to a group that the attacker is not an admin of. A sample of the subsequent upload directory is as follows:


```
root@60ae93fe131f:/var/lib/savane/trackers\_attachments# ls  
40226 40227 40230 40231
```
3. Visit the homepage of the group that the attacker is an admin of. Then, visit Bugs > Browse and note a valid Item ID on the leftmost column of the table. This ID will be used in the next step.

4. As the attacker, make a get request to the path `/bugs/index.php?func=delete_file&item_id=<ATTACKER_ITEM_ID>&item_file_id=<FILE_ID_TO_DELETE>`.

* In my case, this looks like <http://172.17.0.2:7890/bugs/index.php?func=delete_file&item_id=50697&item_file_id=40231>.

5. Verify that the victim’s file (from a group the attacker doesn’t have privileges on) has been deleted.


```
root@60ae93fe131f:/var/lib/savane/trackers\_attachments# ls  
40226 40227 40230
```
Due to the incremental file names, it is possible for an attacker to iteratively delete every file attachment on the web server! Since recently uploaded files harbor the highest file ID numbers, an attacker can upload a file, observe the ID, and delete every ID below that number as one would in an Insecure Direct Object Reference (IDOR) vulnerabillity. This vulnerability was patched [here](https://git.savannah.nongnu.org/cgit/administration/savane.git/commit/?h=i18n&id=39180aea8f38425035b4d1e73819b58007ac6e83).

Thanks
------

Special thanks to Ineiev, the maintainer of Savane. He was very receptive to my responsible disclosure and helped take steps to patch it.

Conclusion
==========

Combing through Savane’s source code was one of the highlights of my Christmas break. While discovering 3 CVEs came as a surprise, I am appreciative of the educational value of this experience. I hope that you learned something from this writeup. If you have any questions or comments, feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/ally-petitt/)!