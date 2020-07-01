---
title: "Flutter Web Updates Sept 2019"
slug: flutter-web-updates-sept-2019
author: "Gordon"
description: null
date: 2019-09-18T12:11:37+07:00
type: blog
draft: false
tags:
- flutter-web
thumbnail: "fun_with_web_devlog_02.png"
description: "Updates to Flutter Web - September 2019"
---
{{< youtube gtJpyd3-H4M >}}

As you may have heard by now, the separate Flutter-Web branch has been merged into the main Flutter repository. This means you no longer need to do special imports for your Flutter-Web projects. It also means you can import packages from the pub store into your Web application as you normally would.

![Flutter web old repo](/pictures/flutter_web_updates_sept_2019/flutter_web_old_repo.png)

Don't get too excited though, at the time of writing, this is only available on the Flutter dev and master channel - so you'd need to switch to those channels and live a bit dangerously to give Flutter Web a test drive. You will also need to enable Flutter Web.

```
flutter config --enable-web
```

For more info on your current channel and how to upgrade and change channels see [Flutter Channels](https://flutter.dev/docs/development/tools/sdk/upgrading).

For a full overview and up to date information on enabling Flutter Web I suggest you visit the Flutter Web [getting started page](https://flutter.dev/docs/get-started/web).

The process of converting my [Flutter Web application](https://funwith.app) to the main Flutter repository was fairly smooth. It involved changing some of the imports in the `pubsec.yaml` file, and changing or removing all of the Flutter-Web specific dependencies. It now resembles the normal `pubsec.yaml` dependencies we have grown to love. If you want to see an example you can take a look at the FunWith site's `pubsec.yaml` [file](https://github.com/funwithflutter/fun-with-flutter-website/blob/master/pubspec.yaml).

Running
```
flutter create your_project_name
```
Should by default support builds for iOS, Android and now Web. You can verify by noting the additional `Web` directory that is created in your main project directory.

You can launch your debug app on Chrome running:
```
flutter run -d chrome
```

All of the above options are also available in Visual Studio Code and I would assume Android Studio (I don't use it). Meaning when you run your app from your IDE it should bring up the browser as an option to launch. Running the app in debug mode will require Chrome. However, when you build you are basically generating static HTML and JavaScript files, meaning it can be hosted as you would normally host a static website and it will run on any device/platform/browser. Performance may vary though - at the time of writing I observed better performance on Chrome.

Over the past couple of days that I converted the [FunWith](https://funwith.app) app to the main repository, added Firebase Authentication, changed code, tried adding packages, made use of native HTML elements, and published a release build, I encountered a lot of hiccups. Some that were platform specific, some that required creative work arounds, some that were [fixed](https://github.com/flutter/flutter/issues/34858) by the Flutter Web team, and obviously one or two of my own.

Point is, proceed with caution. That all said, Flutter Web is SUPER AWESOME.

Over the last couple of months I've seen a lot of improvements in the performance of my app (see the YouTube video above where I run through the different iterations of my app up until this point). Especially now that you can create release builds of your Flutter Web application running:
```
flutter build web --release
```

In addition to that, the browser now recognises your mouse when it hovers over elements, and you can use the scroll wheel on your mouse to scroll in Flutter Web.

I'll be making more posts on Flutter Web as I add features, and I'll also be creating a couple of blog posts and videos explaining my current implementation, how to incorporate Firebase, and some of the work arounds I used to add certain functionality (links, tab-throughs). Note that depending on when you read this the environment might be completely different - as Flutter Web is maturing and adding more features things may be different in the future.

If you want to see a website made in Flutter Web, feel free to check out the [Fun with Flutter](https://funwith.app) website. The code is also open source, if you are interested you can find it [here](https://github.com/funwithflutter/fun-with-flutter-website).

Last but not least, you now create an account on the site (it uses Firebase as a backend). If you sign up the site will reveal to you the Flutter YouTube channel that I currently enjoy the most! Early bird signees will also be rewarded with discounts on future material and other goodies. Depending on when you read this there might already be some other cool stuff on the site. So go check it out anyway :)