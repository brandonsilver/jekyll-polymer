---
layout: post 
title: Major site overhaul
tags:
- website
- jekyll
- email
- security
- ssl
---

This past spring break I decided that I *really* needed to dust off my website
and start using it for what I originally intended: documenting the neat projects
I work on, writing about the topics that interest me, and serving as a hub for
my overall Internet presence.

<!--more-->

The improvements I've made should hopefully put
some momentum behind that. In short, I made some layout changes to the site to
make it more readable, aquired a Class 2 SSL Certificate from StartSSL, and
am switching my personal e-mail address to <brandon@brandonsilver.com> (with
a new GPG key to go with it).

### Layout changes ###

I've made a few changes to the CSS of the site. Major updates include making the font sizes larger and increasing the width of the content area to match. I also switched the twitter widgets out due to warnings I was getting in the JavaScript console that my old ones were losing support.

I've also added [archive functionality](/archives.html) so that only the five most recent posts are displayed on the front page, with all of the posts available on the archives page. I did this by adding some [liquid templating logic](https://github.com/shopify/liquid/wiki/liquid-for-designers) to the index:

<img src="/images/2013/03/31/liquid_logic_post_list.png" alt="screenshot of the logic used to list the five most recent posts">

You can see the rest of my changes at this blog's repository on [Github](https://github.com/brandonsilver/jekyll-blog). 

### Security/trust improvements: SSL ###

I also went to the trouble to get a Class 2 SSL Certificate from [StartSSL](http://www.startssl.com/). You can access my website through SSL using [https://www.brandonsilver.com/](https://www.brandonsilver.com/), <strike>but your browser may not indicate the connection is fully secure due to the Twitter and Disqus widgets that come from outside of the encrypted connection. Otherwise I would have began redirecting everything to the SSL-enabled URL by default (this is one of a couple of reasons I'm considering dropping all of the outside-hosted stuff from my website, I'll post about that in more detail later).</strike> **UPDATE** *I've since fixed this issue, and the website now goes through SSL by default.*

As far as StartSSL goes, the application process for my Class 2 Certificate was very simple and easy to follow. For the Class 2 Certificate, they require two forms of picture ID and then contact you via phone to ask further questions to verify your identity. After successful verification, they then provide you with the certificates you need, including wild card certificates. StartSSL also provides **free** Class 1 Certificates if you aren't interested in going through the extra verification or paying the (very, *very* low price as far as most certificates go) $59.90 required for the more extensive certificate.

### Change in primary email, GnuPG key ###

I'm switching my primary personal email address to <brandon@brandonsilver.com> since I haven't done anything with my old Silver Imaging website in quite a while and have no plans to use it in the future. You can look up my new PGP public key on most public key servers.

### Improvements still on the roadmap ###

I plan on adding a "bread crumb" style navigation system to the area above each post which will make page navigation more clear. I'm also considering getting rid of the Disqus and Twitter widgets since they seem to be adding a bunch of tracking cookies not related to the function of my website, which in turn means a compromise of privacy for those who visit pages containing those elements on my website. I'll expand on this issue in more detail at a later date.
