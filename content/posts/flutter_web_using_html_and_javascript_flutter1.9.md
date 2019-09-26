---
title: "Flutter Web: Using HTML and JavaScript - Flutter 1.9"
slug: flutter-web-using-html-and-javascript-flutter1
description: null
date: 2019-09-21T12:33:39+07:00
type: blog
draft: true
categories:
- General
tags:
- Flutter Web
series:
-
---
The date of publication is important. If you're reading this and it's not September 2019 then there might be a better solution to the problem I'll be describing below.

If it is September 2019, and/or  the problem below does affect you, then welcome, enjoy, and bask in the information that will be shared with you.

Don't care about the background info and just want a solution? Scroll to the end.

## Le Problème ##

Take a look at the Github issue below for a detailed overview of the problem. Scroll to the bottom of the issue - there might even be a solution to the problem we'll be discussing.

https://github.com/flutter/flutter/issues/35588

Still here? Guess people are still working on the issue.

### Here's my take on the problem ###

Let's begin with a quick overview of Packages and Plugins:

> **Dart packages**: General packages written in Dart, for example the [path](https://pub.dev/packages/path) package. Some of these might contain Flutter specific functionality and thus have a dependency on the Flutter framework, restricting their use to Flutter only, for example the [fluro](https://pub.dev/packages/fluro) package.

> **Plugin packages**: A specialized Dart package which contain an API written in Dart code combined with a platform-specific implementation for Android (using Java or Kotlin), and/or for iOS (using ObjC or Swift). A concrete example is the [battery](https://pub.dev/packages/battery) plugin package.

The above quotes are taken from the [Developing Packages](https://flutter.dev/docs/development/packages-and-plugins/developing-packages) page on the [Flutter.dev](https://flutter.dev) website.

A plugin is essentially a way to perform platform specific functionality from dart code (using widgets) - think fingerprint reader, storage, shared preferences and camera.

We can make plugins for Android, using `Java` or `Kotlin`.

We can make plugins for iOS, using `Objective-C` or `Swift`.

What about Web, using `HTML` and `JavaScript`? At the time of writing, no luck there yet. 

Well that's a lie, technically you can make web plugins, however I'm not sure if there is consensus on the correct way to do this. You can make use of relative file imports to import the relevant platform code - see this [Github issue](https://github.com/flutter/flutter/issues/35588) for more information, or alternatively give this [Medium post](https://medium.com/@rody.davis.jr/how-to-build-a-native-cross-platform-project-with-flutter-372b9e4b504f) a read.

But point remains - how do I use `HTML` and `JavaScript` in my Flutter web app, today!

> "All I want to do is create an external link" - As said by me, myself and I.

Dart as a language targeted the web way before Flutter became a thing. For more information on using Dart for the web, see https://webdev.dartlang.org. It can also be used in conjunction with [Angular](https://angulardart.dev/).

Point is, there are Dart libraries available to target and build for the web.

An example of such a library is [dart:html](https://api.dart.dev/stable/2.5.0/dart-html/dart-html-library.html):

> HTML elements and other resources for web-based applications that need to interact with the browser and the DOM (Document Object Model).

> This library includes DOM element types, CSS styling, local storage, media, speech, events, and more. To get started, check out the [Element](https://api.dart.dev/stable/2.5.0/dart-html/Element-class.html) class, the base class for many of the HTML DOM types.

With the recent merge of Flutter-Web into the main Flutter repository it is no longer possible (at the time of writing) to directly do one of the following imports:

```
import dart:html;
```

or

```
import dart:js;
```

These libraries are platform specific, i.e. the Web/Browsers.

They cannot be used within an Android or iOS application. As such, if you're using Flutter Web on the main Flutter channel, in a project targeting Web, Android and iOS - then your IDE will likely throw this error:

```
Target of URI doesn't exist
```

![Target URI doesn't exist](/pictures/flutter_web_html_and_javascript_support/dart_html_error.png)

The error you see above is caused by the Flutter Analyzer SDK, see this comment below from the [Github issue](https://github.com/flutter/flutter/issues/35588):

> Flutter has two copies of the Dart SDK. One - used by flutter tools - has the dart:html library and the standard Dart web compilers. That allows the flutter tool to build flutter for web apps. That copy lives in //bin/cache/dart-sdk/

> The other copy of the Dart SDK is used to analyze user code against. That copy lives in //bin/cache/pkg/sky_engine/lib/. The libraries there are defined in the flutter/engine repo in https://github.com/flutter/engine/blob/master/sky/packages/sky_engine/lib/_embedder.yaml.

## The Solution ##

> We had a meeting with the Dart Analyzer team and are moving forward with adding `dart:html` and `dart:js` to the Flutter Analyzer SDK

While writing this article this was the last comment for the [Github issue](https://github.com/flutter/flutter/issues/35588). Meaning that, depending on when you're reading this, it might no longer be an issue.

So try and import `dart:html` or `dart:js`. If it works, great! You wasted your time reading this article. Don't feel bad, I wasted my time writing it.

Doesn't work?

Also great!

You can use the [universal_html](https://pub.dev/packages/universal_html) package in the meantime:

> Cross-platform dart:html that works in the browser, Dart VM, and Flutter. Typical use cases are:
> * Cross-platform application development (e.g. Flutter mobile and web versions).
> * Web crawling and scraping

### HTML Examples ###

For `HTML`, import like this:

```
import 'package:universal_html/prefer_universal/html.dart' as html;
```

Simple example, let's open a link.

```
html.window.open('https://youtube.com', 'youtube');
```

Let's pop a window:

```
html.window.alert('Hello, world!');
```

### JavaScript Example ###

For `JavaScript`, import like this:

```
import 'package:universal_html/prefer_universal/js.dart' as js;
```

Let's pop a window:

```
js.context.callMethod('alert', ['Hello, world!']);
```

> The top-level getter [context](https://api.dartlang.org/stable/2.5.0/dart-js/context.html) provides a [JsObject](https://api.dartlang.org/stable/2.5.0/dart-js/JsObject-class.html) that represents the global object in JavaScript, usually window.

For more complex usage, see the [Dart JS library](https://api.dartlang.org/stable/2.5.0/dart-js/dart-js-library.html) and [Dart HTML library](https://api.dart.dev/stable/2.5.0/dart-html/dart-html-library.html).

## Conclusion ##

Again, depending on when you're reading this, you might not need [universal_html](https://pub.dev/packages/universal_html). You might not even need `HTML` and JavaScript access, depending on the ecosystem of web related Flutter plugins.

`HTML` and `JavaScript` access will most likely only concern plugin creators. Just like you normally wouldn't directly use Kotlin or Swift when creating a Flutter application, you would ideally not directly use JavaScript and HTML when developing for the web in Flutter. Instead these functionalities will be exposed through the plugins we know and love, or through new plugins that you can easily incorporate.

But that's for the future. For now, if you want to open a link, for example, then you need access to `HTML` and JavaScript.