+++
title = "Goodbye World - Why and How I Migrated Away From Medium"
date = 2024-04-30
showtoc = true
draft = true
tags= ["Self-hosting", "Tutorial"]
+++

## Introduction
Hey everyone, as you may have noticed, this is my first post exclusive to this personal site! I first began my technical blog on Medium, but have now decided to take the step into posting on my custom-made site. For those who would like to follow suit, I can share how I created this site.

There are probably 100 guides on how to setup a site with Hugo, the framework that I used. This post is meant for 

## Why I'm leaving

* Autonomy
* Disagreement with Medium's philosophy
* Professionalism

### Why Hugo?
I decided to use the Hugo static site generator. I used to build websites before I pivoted to cybersecurity, but my days about worrying about whether my project looks good on a folding phone are behind me. I wanted something that was quick, customizable, and statically hosted. Hugo happens to be the first solution to this that I was recommended by a connection and it is open source. My experience setting up this site has been great, so I think they were 100% right.

## How I Transferred Previous Blog Posts
I had about 26 blogs on my Medium account, so transferring each manually would have taken an immense amount of time. I used scripts to automate this process. The first was a tool developed by Oliver Ernst called [medium-to-markdown](https://github.com/smrfeld/medium-to-markdown).

The second was a tool was developed by myself: https://github.com/ally-petitt/medium-to-hugo-post. Due to my specific use case, this was really a script that was intended to be chained with `medium-to-markdown.py`. As per the README instructions on `medium-to-markdown`, I requested an export of my data. The program was then able to convert my posts into markdown.

My overall conversion process was as follows:

```
python3 ./medium-to-markdown/run.py convert --posts-dir ./export/posts --output-dir ./output
python3 ./medium-to-hugo-post/medium-to-hugo-post.py --md-dir ./previous_blogs/md --posts-dir ./content/posts
```

### Limitations
The script that I created was not perfect. It relies on generalized regular expressions to parse image

### Blog Decisions
