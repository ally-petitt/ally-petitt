+++
title = 'How I became a hacker before I finished high school [Repost]'
date = 2024-05-07
draft = false
showtoc=true
tags= ["Personal", "OSCP"]
+++

*Author's note: This article was initially published on [Synack's README](https://readme.synack.com/how-i-became-a-hacker-before-i-finished-high-school). They have great content and I recommend that you browse their articles if you are interested cybersecurity.*

![Picture of a maze from above from Unsplash](https://22524429.fs1.hubspotusercontent-na1.net/hubfs/22524429/victor-_qXjdWm8YEo-unsplash.jpg)

> Editor's note: This post from Ally Petitt describes her journey towards earning the vaunted OSCP at 16 > and being an active part of the Synack Red Team at 17. Check out Ally's blog for more of her write-ups > on vulnerabilities she's discovered, hacking techniques and more.

## Life before coding
I was 3 years old when I first declared I was going to be a doctor, my mother told me. When I was 8, I had finally done enough chores around the house to earn $20, and I deposited the total into a college fund. I was 9 years old when I began secretly staying up past my bedtime to read the human anatomy textbook from my father’s bookshelf. When I was 10 I set my sights on getting admitted to Harvard Medical School and becoming a neurosurgeon. This was also the year that I attempted surgery on my arm with a pencil to see what the different layers of skin looked like. As expected from a fifth grader, this was done without anesthesia and, yes, it got infected. When I was 13, however, I survived a global pandemic and that all changed, setting me on a path that would ultimately see me earn my OffSec Certified Professional (OSCP) certification and get my first CVEs as an ethical hacker. 

With COVID-19 came remote school, which meant the schoolwork that took approximately seven hours on campus could now be completed in 40 minutes from my own room. Without the increased time commitment of fixed class periods, I was now free to explore education on my own, without a syllabus or any restrictions. 

I began to ask questions that I had never allowed myself the freedom to ask previously. Becoming a doctor is a prestigious profession. But when I researched the burnout, the relentless schedules, rigorous schooling, student debt and stress that these essential workers experience, I realized a doctor’s lifestyle would not be a good fit for me. 

Doubts about a career as a doctor led to more questions. I used the internet to learn subjects that my middle school didn’t offer: economics, physics, investing, astronomy, trigonometry, neuroscience and Russian. I noticed a trend that I was drawn towards career options that included themes of learning, making connections and investigation. I considered becoming a theoretical physicist, an investigative journalist and many more professions. While enticing, there were aspects of each that made me wary of committing to them.

This brings me to December 2020, the month that I ran an experiment to test the coding ability I could attain in only 20 hours of learning. I wanted to see what I was capable of and I did not expect how humbling it would be. I recall that my first five or six hours were spent attempting to troubleshoot my Python installation. The next three were me trying to figure out why my “Hello World” program was not running in VSCode. (It turned out that I had forgotten the “.py” file extension.) By the end of the 20 hours, however, I had gotten the basics of coding down and was able to create simple projects such as mad-libs. I decided to continue for another week and completed a terminal-based tic-tac-toe game. This was a career trajectory I could see myself in.

## Life after coding to OSCP
I built a humble portfolio of coding projects and within three months landed an internship as a software engineer. Navigating this new environment with my new team was a challenge. I was tasked with writing a Java API to take input from a machine learning backend and utilize it to spin up EC2 spot instances (or their equivalent) across three different cloud providers. I took the responsibility of independently learning Java so that I could better contribute to the project. It was also at this time that I began to share resources and techniques with my teammates who were struggling. I began to assist in delegating tasks and working with everyone’s schedule. Because of this, I was promoted to team lead two months after joining.

Fast-forward 1.5 years, I had purchased and built my own PC and served in three internships where I authored programs to decrease website load time by 2 seconds, parse 2,212 PDFs, save 70% of company expenses on cloud computing, scrape and preserve five critical websites, and I designed and developed a company website from scratch. 

Despite the seemingly rapid success, I had received hundreds of rejections on the basis of my youth, which I tracked on a spreadsheet. I became extremely discouraged due to the limited opportunities available at 16 and considered giving up. 

After deliberate thought, I decided that if I was going to give up, it would be for a reason other than self-doubt or fear. I shifted my focus to what I could do instead of what I couldn’t. I forged ahead, reaching a total of 985 GitHub commits by the end of 2022.

I continued to seek out experience and opportunities in software engineering. During my third software engineering internship, I developed a deeper interest in how wireless communication worked, and whether knowing how to code really meant that I could hack into a router like I used to think (spoiler: it does not).

My cybersecurity journey began by reviewing entry-level TryHackMe modules and learning the basics of networking. It had occurred to me that the OSCP would be a far-reaching certification to obtain, especially with its prestige and notoriety, however, with the limitations that my age had already offered me, I was skeptical in believing that I could obtain it. After further investigation, however, I found that achieving the OSCP at 16 had been done at least once before! 

With the money I had saved from that internship, I purchased an exam voucher for the OSCP and began my lengthy process of consistent and deliberate preparation over a period of approximately seven months. This story can be found on my personal blog.

Earning the OSCP meant more to me than the opportunities it afforded me. It was a symbol of my relentless desire to follow through with the goals that I set for myself and persevere through crippling levels of self-doubt. The OSCP, however, was just the beginning of my journey into infosec.

## Experience (the city, Synack, CVE and data breach)
I learned of an opportunity to intern at the City of Bakersfield over the summer for clerical positions, however, I was interested in a more technical pathway. I found that the city had a job opening for a security analyst. I applied with a flicker of hope that I could help to fill that role during my internship. In the interview process, I explained that I would be willing to take on some of the responsibilities of the analyst as an intern. Shortly thereafter, a thick packet about me was placed on the desk of my future supervisor and he decided to give me an opportunity.

Despite having obtained the OSCP at such an early age, my imposter syndrome was still present. I had come to believe that once I achieved more success, I would have less imposter syndrome, however, in practice, I found that it only became more severe. Now, with the opportunity to work for the city, I was face-to-face with my own self-doubts. 

Ultimately, I took my responsibilities one step at a time. I fell back on my methodology. First, scanning the hundreds of hosts within scope. Then, looking for low-hanging fruits and subsequently narrowing my focus to more granular details, taking thorough notes as I went. 

Upon my first external penetration test at the city, I was manually testing Click2Gov  when I noticed an ID being sent in one of the POST requests that I intercepted in my HTTP proxy. I created a second account and tested that this ID could be modified to be that of another user to make changes to another user’s account. My hypothesis was proven correct, and thus, I had discovered CVE-2023-40362. Additionally, I discovered an insecure direct object reference (IDOR) and multiple cross-site scripting (XSS) vulnerabilities that were more complex to exploit. Following these discoveries, I corresponded with Central Square in order to responsibly disclose the issues so they could be patched. Months later, after a patch was shipped, I publicized the CVE. The vendor then added a reference to the CVE correlating the vulnerability I found to a Click2Gov data breach affecting 300,000 people that led to a class-action lawsuit. I was absolutely shocked that something I found would be associated with such big news.

After this assessment, I did an internal security audit on our printers, where I made the revolutionary discovery that adding a “test.txt” file to the printer’s FTP server would cause that file to print. My co-workers were very confused as to why the word “test” was repeatedly printing when they went to collect their papers until I realized what was happening. Additionally, I helped to extract the password hashes of our municipal enterprise domain containing thousands of accounts, cracking as many as I could to demonstrate the strength of our passwords. Finally, I audited our Active Directory domain, mapping out potential privilege escalation paths that could be used by attackers. These four assessments were completed in the span of only six weeks.

After my term at the city concluded, I continued to perform independent research, leading to the discovery of CVE-2023-43154 and CVE-2023-44792, which were authentication bypass and SQL injection vulnerabilities, respectively. In December 2023, I discovered CVE-2024-27630, CVE-2024-27631 and CVE-2024-27632 in Savane, the web-based software hosting system used by the Savannah bug tracker. It coordinates bug-related information in 405 official GNU projects that have been estimated to be used by millions of people worldwide.

By now, I was beginning to gain more confidence in my abilities as an offensive security professional. I began to apply to more job openings, starting with Synack, where I was grateful to have the opportunity to join the Synack Red Team (SRT) of security researchers.

### SQLi WAF bypass
Being surrounded by so many knowledgeable security researchers really allowed me to grow my own skills. Two weeks into working for the SRT, I discovered an intriguing authentication bypass flaw. Recently, I discovered a SQL injection vulnerability – however, my SQLi payloads were being monitored by a reputable firewall. I attempted different techniques and bypasses for several days before eventually running out of ideas. 

Luckily, I didn’t have to go it alone: I partnered with SRT members Moey and NukeDX in order to construct an initial payload to validate the SQL injection. From there, Moey – who happens to have also joined the SRT at a young age – and I tested the firewall for hours. We tested payloads in online SQL editor sandboxes and read through documentation to find different ways to accomplish data exfiltration. While we were unable to find a payload to extract all the data, we crafted a payload to extract part of it by turning our traditional SQLi into a boolean attack. From there, we brute-forced our way into demonstrating impact with the vulnerability.

## Conclusion
As a current senior in high school, balancing school life with my professional aspirations has been a persistent challenge, however, it is an immensely rewarding journey. Ultimately, my success thus far is a product of the individuals who saw potential in me and gave me a chance or simply a listening ear. In my continuous mission to learn and contribute to this industry, I plan to research more open-source projects, firmware, and otherwise intriguing cyber assets in addition to obtaining more technical certifications in my transition from high school to a career in security research. Although this scar on my arm from my self-imposed “surgery” is fading, its symbolism remains eternal as it represents the insatiable curiosity and unwavering perseverance that has led me to discover and thrive in a field that I am proud to be a part of. 