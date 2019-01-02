---
title: "Goodbye Wordpress, Hello Gatsby"
author: Richard
tags: [Web, Blog-post]
image: ../img/post-headers/gatsby-post-header.jpg
date: 2019-01-02
draft: false
---

I've maintained a Wordpress blog for years now, and I always did like Wordpress. Creating Wordpress themes was one of the first significant software development
projects I did as a kid. However, as my hosting account expired recently and I got hit with renewal fees that were 3x the original cost (this is _extremely common_ across web hosts), I decided to take a second look at how I could maintain a personal site with less technical and financial overhead. 

I've also been thinking a lot about how I'd like to lay out my site. I really wanted to keep blog posts and projects separate. Wordpress could do this, but the off-the-shelf themes didn't exactly support what I wanted to do. Of course, I could extend an existing theme, but I didn't really want to write PHP and deal with Wordpress any more.

Enter Gatsby - the static site generator. As opposed to Wordpress which generates pages on the fly by pulling post data from a database, Gatsby takes markdown files (for pages, posts, etc.) and generates an entirely static site. The content can then be hosted inexpensively, with zero server-side dependencies. The pages are constructed using typical modern web technologies, like React. I've been having a great time with React recently, so I decided to give it a shot. 

## The Old Host
First I had to deal with the old web host. Once my account expired, they locked me out of cpanel, and I also couldn't use an FTP connection. I totally understand the site going down (since I was behind on payment), but I was not thrilled that the admin panel was closed and my data was held hostage by the host. 

Not wanting to pay 3x the original hosting cost, I converted to monthly billing and paid my final month. This let me login to the sites and export the wordpress data. Ten bucks was easily worth it to get away from them. 

## Gatsby Bringup 
Gatsby has starter templates, so I chose [this one](https://github.com/scttcper/gatsby-casper) to serve as a base template for the new site. 

Next I had to install Gatsby, and it was here that I encountered my first issues. After running `npm install -g gatsby`, the `gatsby` command did nothing, as if Gatsby were never installed. 

The solution came from [this Github issue](https://github.com/gatsbyjs/gatsby/issues/4967), in the form of the following commands:

    `npm config set prefix /usr/local`
    `npm i -g gatsby-cli`
    `npm i -g gatsby`

## Adjusting the Theme
Gatsby development was very smooth once the install issue was worked out, so I set about stripping down some elements of the theme that I didn't want, such as the social media hooks and buttons. For the most part, I commented these out since I may want them later.

It didn't take too long to understand how pages and posts are created. The content folder structure is very similar to Wordpress, too. 






