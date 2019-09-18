---
title: "The Fun with Flutter website"
slug: funwith-intro
description: null
date: 2019-07-03T21:26:20+07:00
type: blog
draft: false
categories:
- General
tags:
- flutter-web
series:
-
description: The Fun with Flutter website, made entirely in Flutter!
thumbnail: fun_with_web_devlog_00.png
---
Made entirely in Flutter! Well for the most part but it’s still pretty cool!

Care to watch a video instead of reading, here you go:

{{< youtube ZV8A1fCdrFg >}}

This is a short introductory post as I will be making follow up videos and blog posts detailing the creation, struggles and fun I had in making the [FunWith](https://www.funwith.app) website. These will discuss architecture, state management, dependency injection, cute animations, and custom widgets of course. 

They’ll also discuss shortcomings, for example there is no easy way to render a video yet and it took way too much Googling to find a way to make an external link.

For me, however, the most exciting part will be to see the performance improvements in the site as Flutter for Web matures and what we can do once more packages become available.

As a side note, the website is nowhere near complete. It’s even short of my initial vision for version 1.0 - there was still a lot left to do in my Trello board.

But, I set myself a deadline and this is the current state of the website. And [here](https://www.funwith.app) it is.

There are a couple of nice things that I did do however, that I’m proud off.

* I’m using the Bloc pattern for state management which required a bit of a mind shift, but now it’s in a pretty solid state and easy to interact with from the UI.
* There is a nice integration between Hugo for the blog content and the Flutter website. Where all I need to do is write a Hugo post and the website updates automatically. For those of you that don’t know, Hugo is a framework to easily create a static website or blog.
* And there is also a little bit of fancy animation that was easy to do using a neat trick with paddings.

## The Future

So what content can you expect? To be honest - whatever I feel like!

I’ll have a section showcasing flutter packages that I’ve made - go take a look on the site, these packages are interactive (as they're written in Flutter). What better way to showcase a Flutter package than have it be interactive on a website made in Flutter - pretty cool.

There will be blog posts.

There will be a section for tips and tricks or neat resources that I’ve come across. At some point in the future I’d also like to make a section for quality learning material.

Again, whatever I feel like. I’m using this as a learning opportunity for myself that, over time, will hopefully be extended, expanded and refined to be of value to you and other like minded people.

The next blog post and video will be how I use Hugo to generate the blog content and expose the content through a static API to the Flutter website. I took this route as an easy way to overcome some of the limitations for Flutter web, in the future this might be completely different.

Anyway, there’s a lot of code, writing and video editing to do. Everything is open source, so please if you feel like you want to contribute with a blog post, a video, an idea, or even code - you are more than welcome. If you are still learning and feel like you don’t have anything to contribute, then let me tell you you’re wrong. As I said, I’m still learning and figuring out most things as I go along. Why not join me?

Cheers people, catch you in the next video - or blog post I guess :)

Oh, and remember to check out the website for yourself - link in the description!