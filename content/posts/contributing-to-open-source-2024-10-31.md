---
title: "Contributing to Open Source: A beginner's experience"
date: 2024-10-31T14:57:04+01:00
draft: false
toc: false
images:
tags: 
  - dev-notes
---
# A Beginner's Experience to Open Source: My Personal Experience

When I first started my tech career, a lot of people told me, "Contribute to open source!" While I get why it's a valuable experience, I couldn't shake the feeling that it just didn't make sense for a beginner. I kept asking myself: How does a novice even start contributing to an open-source codebase? 

Over the past 12 months, I’ve gained some experience and a bit more perspective. Here’s my opinion as a beginner (take it as such): You don’t force it.
# My First Open Source Contribution

My first open source contribution was to DHH's project Omakub. If you don’t know who DHH is, check out his [blog](https://world.hey.com/dhh), as he shares some great insights.

Being a Linux user, I was already somewhat familiar with things like bash scripting and basic system workings. So when DHH announced Omakub to encourage developers to use Linux as a daily driver, I was intrigued. His idea was simple: if your work runs on Linux, then you should be familiar with it as a daily tool.

For my contribution, I issued a pull request to Omakub that installs the latest version of Node.js. The code itself was a bit quirky, as I later found out, but it was accepted, and it counts as a contribution!
A Contribution to My Blog Theme

My next contribution was to the Hugo theme I use for this blog’s site. I wanted something simple but didn't want to compromise on aesthetics. I found the Hermit v2 theme, which fit my style perfectly—it’s Nord-themed (spoiler alert: I’m a sucker for the Nord color scheme).

While it was mostly what I needed, it didn’t have everything out of the box. For one post, I wanted to add a flow chart to show the backend workflow for Frontpages, a news aggregator site I built. So I was able to implement the feature with some research into Hugo’s templating syntax and submitted a pull request. It was accepted. As it turned out, the maintainer of the codebase had intended to add the feature but it had slipped his mind.

# When Open Source Gets Complicated

My most recent contribution was a bit more challenging. Linux has its learning curve, and I’m not just talking about command-line basics. I’m a developer, so I’m already comfortable in the terminal. With tools like Helix and Zellij as my editor and terminal multiplexer, I spend a lot of time there, stringing together commands and key sequences.

But Linux customization can get complex, especially with a setup like Sway, a minimalistic window manager where you have to build everything yourself. This deep-level configuration gives you a tailored environment, but it also leads you down some rabbit holes.

For example, KDE, a fully-featured desktop environment (a good alternative in the Linux world if you want to avoid the "rabbit holes"), offers an app called KDE Connect. This app seamlessly syncs your phone and PC over Wi-Fi, making it easy to pass clipboard content, transfer files, and even send a webpage from your desktop browser to your phone with one click.

Before KDE Connect, I used Pocket to save pages for cross-device access, which worked but wasn’t as smooth. With KDE Connect, integration feels truly seamless—no extra steps, no delays, just a fully synced experience across devices.

I wanted a similar KDE Connect feature on Sway, since this was a great feature to have, I installed an open-source browser extension meant to support it for folks with a similar linux setup as mine only to find out that it no longer worked. After combing through numerous issues reported on the repo, it became clear that the project wasn’t actively maintained.

I decided to see if I could get it working again. It seemed that only a few small changes were needed, as the whole thing had likely broken due to an API name change to some properties being checked as the original developer had noted some years ago. After a quick look through the project, I could see that I was familiar with the core components: the extension was written in JavaScript, and its background script interfaced with a binary (written in Go) that needed to be installed on the host PC. The only challenge? The KDE Connect cli codebase itself was in C++, a language I wasn’t experienced with.

As it turned out, reading code in an unfamiliar syntax wasn’t as big a hurdle as I’d thought. Even in a new language, it’s often possible to get a general sense of the logic. Eventually, I tracked down a crucial change in a seven-year-old commit all thanks to a well-defined commit message (pro tip: always use clear, concise commit messages—they’re invaluable for collaborations).


From there, I worked my way up the commit history noting down changes  that could be relevant in related files.  With all this, I was able to identify any additional modifications that would be needed. Initially, I felt overwhelmed by the large codebase, especially in a language I wasn’t familiar with, full of foreign concepts. But after about four hours of digging, I was able to pinpoint the exact changes required in the compiled binary that served as a bridge between kdeconnect cli and the open source extension.

After testing, I now have a fully working browser extension and a pull request awaiting review. Even if the project is no longer actively maintained, I’ve created a fork on GitHub with the fix, complete with references to the relevant issues. By using the GitHub `fixes #issue-number` tag, I linked the pull request directly to the issue, so it automatically closes if the pull request is accepted and also notify those who were interested in this particular fix. Despite the project being broken for around seven years, interest in the solution hasn’t waned—the latest issue was created less than a month ago. This fork can now serve as a base for anyone looking to build on the original project.

# Final Thoughts

If there’s a trend in my contributions, it’s this: I contributed to things I already use, am familiar with, and, in most cases, are written in languages I know well. My advice? Don’t push yourself to get into open source just for the sake of it. As you improve in skills and curiosity, you’ll naturally find projects where you can make a difference, no matter how small.

# Links to Resources
 - DHH's blog: [www.world.hey.com/dhh](https://world.hey.com/dhh)
