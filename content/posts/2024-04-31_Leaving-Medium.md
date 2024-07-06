+++
title = "Goodbye World - Migrating Away From Medium | Graduation & Next Steps"
date = 2024-05-03
showtoc = true
draft = false
tags= ["Self-hosting", "Tutorial", "Personal"]
+++


![Medium logo crossed out](/images/Medium-crossed.png)
## Introduction
Hello everyone and welcome to the first exclusive post on my new personal website! I am very excited to be here and I hope that you are as well. The picture above is a bit dramatic, but communicates the message that I have decided to move forward. Those who have been following me will know that I first began my technical blog on Medium, which was an approachable outlet for sharing my knowledge as I first entered the field.

I will graduate from high school this month, which marks the beginning of a new chapter in my life. In the spirit of this transitional period of my life, I decided to make the shift from posting my technical articles on Medium to this personal website. 

The technical details of how I migrated my blogs from Medium to here will be explained further down in this article.

### Appreciation
I want to first express my gratitude for Medium and the supportive readers that the platform has allowed me to connect with. Medium is where I got my start in writing technical blogs and has been an instrumental outlet in sharing my knowledge as I was learning it.


## Why I'm Leaving Medium

With that in mind, my decision to leave Medium is multi-faceted. 

Not only am I ascending into adulthood, I am maturing into a professional career doing work that I am extremely passionate about. I wanted that increased seriousness to be reflected in the articles that I write, which is more aptly served by a personal website than Medium (even more so with a custom domain, but that is a work in progress).

Additionally, this website offers a more authentic experience. I have purposefully omitted analytics for the benefit of my readers' privacy. As a result, these posts are more like a conversation with interested readers than a numbers game.

Finally, creating my own website affords me autonomy. My readers are no longer berated by banners and pop-ups to subscribe to get a Medium membership so that they can view other content on the site with the vast majority of it being pay-walled. I can provide my readers with a site that is free to access without distractions- something that I have come to appreciate in my own internet browsing. 

I also understand that I have readers that prefer stay updated with their authors through RSS feed aggregators. For you, I added an [RSS feed](https://ally-petitt.com/index.xml) to the site :).

## Next Steps
![Picture of a graduation](https://cdn.vanderbilt.edu/vu-wpfsx/wp-content/uploads/sites/7/2024/04/09234832/commencement-topper-Image-2024-scaled.jpeg)

I am excited about the future. I used to be petrified about life after graduation, but I have learned to embrace the unknown. There is so much to learn, attempt, and experience and I am at this exciting point in my life where it is all novel. There are certain topics in cybersecurity that I wish I could re-learn again for the first time because I remember the potent feelings awe and inspiration of the moment I first read about them. Here's to a future of continued discovery.

Regarding this blog, as I continue to perform security research and study new techniques, I plan to publish writeups or otherwise educational content. Writing will not be my main focus, so I will prioritize the articles I write based on what I find very interesting or if I think my audience would benefit from them. 

## Creating This Website
I was recommended the static site generator, Hugo, by a colleague who I was discussing website-building with. After setting up a Hugo app, I found appreciation in the ease of setup and its customizability and decided to build this website to completion with it. 

Since there are already a plethora of tutorials for Hugo, I will omit the initial app setup (those that are interested can view Hugo's [QuickStart Guide](https://gohugo.io/getting-started/quick-start/)). Instead, I will focus on the technical challenges that were specific to this use case of converting a personal Medium blog to a Hugo-compatible markdown post.

### How I Transferred Previous Blog Posts
I had 26 published blogs on my Medium account, so transferring each manually would have taken an immense amount of time. Consequently, I used scripts to automate this process. The first was a tool developed by Oliver Ernst called [medium-to-markdown](https://github.com/smrfeld/medium-to-markdown). The second was a tool was developed by myself: [medium-to-hugo-post](https://github.com/ally-petitt/medium-to-hugo-post). 

My workflow involved chaining both of these scripts together such that the output directory of `medium-to-markdown` was the input directory of `medium-to-hugo-post` with `/md` appended. As per the README instructions on `medium-to-markdown`, I requested an export of my data from Medium, which I then fed into `medium-to-markdown`. This converted my posts into markdown with the exception of image tags and their captions. 

```
python3 ./medium-to-markdown/run.py convert --posts-dir ./export/posts --output-dir ./output
```

Following the markdown conversion, I noticed that the new markdown files required additional modifications to properly format on Hugo. To remedy this, I created `medium-to-hugo-post` to add a "[Front Matter](https://gohugo.io/content-management/front-matter/)" to each post and finalize the conversion of images and their subtitles to markdown.

```
python3 ./medium-to-hugo-post/medium-to-hugo-post.py --md-dir ./output/md --posts-dir ./content/posts
```

I outline these steps and the limitations of these tools in more detail in the medium-to-hugo-post [README.md](https://github.com/ally-petitt/medium-to-hugo-post/blob/main/README.md) file.

### Conclusion
Migrating to this website is a symbol of a new beginning. I could not be more enthusiastic about this transformative period of my life as I transition into a full-time career doing the work that I love. Thank you to everyone who has supported me throughout this journey, and I can't wait to see what the future has in store!
