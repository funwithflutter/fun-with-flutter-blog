---
title: "Making a Blog With Flutter and Hugo"
slug: making-a-blog-with-flutter-and-hugo
author: "Gordon"
date: 2019-07-09T11:00:55+07:00
type: blog
draft: false
categories:
- General
tags:
- Flutter Web
- Hugo
thumbnail: "fun_with_web_devlog_01.png"
description: "Using Hugo to generate a static API and making a website with Flutter."
# series: 
# -
---
{{< youtube 3VTTrGZrYS0 >}}

Lets begin with a quick description for Flutter Web and Hugo, and then provide an overview and motivation for using these two technologies together.

## Overview ##

> [Flutter Web](https://flutter.dev/web):
is a code-compatible implementation of Flutter that is rendered using standards-based web technologies: HTML,CSS and JavaScript. With Flutter for web, you can compile existing Flutter code written in Dart into a client experience that can be embedded in the browser and deployed to any web server. You can use all the features of Flutter, and you don’t need a browser plug-in.

> [Hugo](https://gohugo.io/):
is one of the most popular open-source static site generators. With its amazing speed and flexibility, Hugo makes building websites fun again. Hugo is written in Go (aka Golang).

## Motivation ##

The FunWith [website](https://funwith.app/) and [blog](https://fun-with-blog.firebaseapp.com/) are companions for the Fun with Flutter YouTube [channel](https://www.youtube.com/funwithflutter). So using Flutter Web seemed like an obvious choice as it serves the dual purpose of function and learning (for myself as well as others).

The use of Hugo came more from need than desire. Flutter Web is still in early stages and as a result I wanted to ensure that the content I produce has a safe place to live; come in Hugo. While at the same time I wanted to expose that content to the Flutter web application and make use of Flutter as much as I could; come in Hugo as an API.

![Why not both meme](/memes/meme_why_not_both.png)

Lastly, I did not want to make unnecessary work for myself. Frameworks like Hugo, Ghost, Jekyll, etc, are made in such a way that they are easily configurable, fast, and most important it's easy to produce content on.

My vision was that I could make a normal blog post, save it on the FunWith blog, and auto-magically the FunWith app should be updated to reflect the change and expose the new blog content.

Motivation and expectations set - let’s jump into the code.

## The Code ##

The main reason you're here. Both the [blog](https://github.com/funwithflutter/fun-with-flutter-blog) and [site](https://github.com/funwithflutter/fun-with-flutter-website) are open source so take a look if you want!

The [Hugo docs](https://gohugo.io/getting-started/quick-start/) provide a better explanation than I could to get setup and started using Hugo.

I did not do anything special for the blog, but let me give you a quick overview of how Hugo works. In Hugo you can define layout files that style different sections of your site. For example, you could have layout files that create the home page, site sections, and individual posts. Once you run Hugo it proceeds to take your markdown files (and their respective folders and sub-folders) and generates static `HTML` pages based on these defined layouts. Hugo will also generate an `index.html` file for the landing (home) page.

If you use a theme for Hugo there is technically nothing you need to do on your end except to make the markdown files (blog posts) and then run Hugo to generate the static `HTML` pages.

I was not too concerned about the look of the FunWith blog as it mostly serves to statically serve the content, while the Flutter site will take that content and make it pretty. So I didn’t spend any time customizing the blog - except for some time learning the ropes of Hugo and choosing a theme. The theme that I'm using at the time of writing is [Slick](https://themes.gohugo.io/slick/).

Next, we need a way to get the blog content into our Flutter web application. This is relatively simple to do, provided you understand some Hugo templating and functionality.

#### Generating JSON ####

This is all possible owing to Hugo's ability to output [different file types](https://gohugo.io/templates/output-formats/). For example, instead of outputting `HTML` pages we can set it up to output `json` files - or both `json` and `HTML` as we will be doing. Similar to how we would create layout files to customize our theme, we can also define layouts that will build static `json` files when running Hugo.

The first thing to do is define the layout types in the Hugo configuration file. Below is an example of such a configuration.

`Config.toml`
    
{{< highlight json >}}
[outputs]
    home = ["json","html"]
    page = ["html"]
    section = ["html"]
{{< /highlight >}}

As you can see we are stating which section of the site require what output types. Only the home page is marked to output `json` and `HTML`. That means upon running Hugo it will produce an `index.html` and `index.json` file in accordance to the `HTML` and `json` index layout files we provide.

The `index.html` page will be built using the theme's `HTML` layout file. However, we need to tell Hugo what layout file to use for the home page’s `json` output, as this is not specified by the theme we are using.

In our site’s directory we must create a new layout file within the `layouts` folder. We’ll need to call this file `index.json.json` - as this is the file name that Hugo will be looking for to build `index.json`.

As an example, let's give this file the following content:

`index.json.json - example file `  

{{< highlight json >}}
{
    "api": {
        "version": 1.0,
        "description": "doing some tests"
    }
}
{{< /highlight >}}

Building Hugo will generate an `index.json` file with this exact content, as is. What we want is for Hugo to generate `json` content based on our blog data (markdown files). For that we’ll need to make use of [Hugo’s templating](https://gohugo.io/templates/introduction/).


Below is an example of how I generate some of the `json` data for the FunWith site.

`index.json.json - layout`

    {{- $.Scratch.Add "allPages" slice -}}
    {{- range .Data.Pages.ByDate.Reverse -}}
        {{- $.Scratch.Add "allPages" (dict "uri" .Permalink "title" .Title "content" .Plain ) -}}
    {{- end -}}

    {{- dict "pages" ( $.Scratch.Get "allPages") | jsonify -}}

This is an example `index.json.json` layout file that will iterate over all available posts on the blog (in reverse date) and then create a dictionary of the URI, the title, and the content of the posts. It will then take that dictionary and pipe (pass in as a parameter) it to the `jsonify` function to generate `json` data.

The above layout will produce output similar to the below (depending on the content of the blog/posts).

`index.json - output`

{{< highlight json >}}
{
    "pages": [
        {
            "content": "Choclate is good!\n",
            "title": "Chock Stick",
            "uri": "https://fun-with-blog.firebaseapp.com/posts/chock-stick/"
        },
        {
            "content": "Building, working, playing. Blah.\n",
            "title": "Test Post",
            "uri": "https://fun-with-blog.firebaseapp.com/posts/test-post/"
        },
        {
            "content": "First page. Doing some testing.\n",
            "title": "First Post",
            "uri": "https://fun-with-blog.firebaseapp.com/posts/first-post/"
        }
    ]
}
{{< /highlight >}}

If you’re not familiar with Hugo, and Go, the code above might seem very strange. This article will not discuss Hugo templating as there are a number of better resources for that. I would suggest starting with the Hugo documentation and if you’re interested in a more complete example of using Hugo to generate an API then you can take a look at the following links:

https://forestry.io/blog/build-a-json-api-with-hugo/

https://forestry.io/blog/hugo-json-api-part-2/

That is basically that. We now have a static `json` file that we can host and query to retrieve the blog content that we need.

For example, from the Flutter web application I make an HTTP request to the [index.json](https://fun-with-blog.firebaseapp.com/index.json) file hosted on the FunWith [blog](https://fun-with-blog.firebaseapp.com/). Once the `json` is retrieve it can be parsed into objects and used within the application as desired. Future blog posts will go into more detail regarding this.

## What’s Next ##

We have the tools and the know how to expand this as much as we want. We can define as many `json` output files as desired for as many sections and posts as needed. These `json` files can be fine grained to output the exact content that we want.

At the time of writing this blog I’m redirecting users from the FunWith app to the FunWith blog to read the actual blog posts. For example, a user can see what blog posts there are on the FunWith site, and upon clicking a post they are taken to the FunWith blog to do the actual reading.

This is not ideal and in the future this will change. Either by parsing the markdown directly with Flutter, or alternatively making use of an iFrame or WebView to render the blog content in site.

Well that’s all! I don’t expect a lot of you read up until here. If you did, thanks and I hope you enjoyed this post. I will be making more blogs related to the creation of the FunWith site and how the site expands and changes.

Until next time.

-Gordon