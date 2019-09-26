---
title: "Using Firebase in Flutter Web"
slug: using-firebase-in-flutter-web
description: null
date: 2019-09-26T12:21:59+07:00
type: blog
draft: false
categories:
- General
tags:
- Flutter Web
thumbnail: "fun_with_web_devlog_04.png"
description: "Initialize Firebase in a Flutter web application and incorporate Firebase Authentication"
series:
-
---
{{< youtube zo2dVYFHXcA >}}
As with anything in the tech space — take note of this article’s date (September 2019). Landscapes are constantly changing, what is said below may no longer be relevant or there may be easier alternatives.

Using Firebase in Flutter web is fairly straightforward, however, not as simple as it could be. At the time of this writing, there is no universal Flutter library for Dart or universal Firebase package that extends to Android, iOS, and Web (maybe even desktop) when building Flutter applications.

See [here](https://www.reddit.com/r/FlutterDev/comments/d51o4w/were_the_flutter_team_at_google_ask_us_anything/f0jx6xs/?utm_source=share&utm_medium=web2x) for a discussion on the topic.

This article will only discuss the web implementation and how to setup Firebase for Flutter web.

If you’ve never used Firebase before, then you will be better served reading the [Firebase docs](https://firebase.google.com/docs), or watching one of the endless numbers of videos regarding Firebase. I suggest the actual [Firebase channel](https://www.youtube.com/user/Firebase) or [Fireship](https://www.youtube.com/channel/UCsBjURrPoezykLs9EqgamOA).

## Initializing ##

The package that you will need to use is the standard web implementation of Firebase for Dart. Find it here:

https://pub.dev/packages/firebase

The package’s Pub page gives all the information you need to get set up and started. But as a summary, let’s go through the implementation to get Firebase Authentication going.

**NOTE**: If you are using Firebase Hosting then watch the video linked above for an alternative implementation.

First, you’ll need to add Firebase as a dependency in you pubsec.yaml

```
dependencies:
  firebase: ^5.0.0
```

Next, you’ll need to include the Firebase JavaScript libraries in your .html file. For a Flutter project, the default HTML file is located at `web/index.html`.

{{< highlight javascript >}}
<script src="https://www.gstatic.com/firebasejs/6.6.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/6.6.0/firebase-auth.js"></script>
{{< /highlight >}}

_Your versions may be different. You can find the latest versions on the Firebase website. Additionally, you need to import the Firebase libraries that you will be using — for example, Auth, Database, Firestore. For a full list see the Firebase website._

It makes sense to initialize Firebase as soon as your application starts. So head over to your Flutter project's main.dart file and import:

```
import 'package:firebase/firebase.dart';
```

And initialize your Firebase project in main.dart:

{{< highlight dart >}}
void main() {
  initializeApp(
    apiKey: "YourApiKey",
    authDomain: "YourAuthDomain",
    databaseURL: "YourDatabaseUrl",
    projectId: "YourProjectId",
    storageBucket: "YourStorageBucket");
}
{{< /highlight >}}

You can find the properties for your Firebase project in the Firebase console, within the Project Overview/Settings section of your project (in the companion video, linked above, I walk you through the process).

Alternatively, you can also initialize your app within `JavaScript` in the `index.html` file.

{{< highlight html >}}
<body>
  <!-- Previously loaded Firebase SDKs -->

  <script>
    // TODO: Replace the following with your app's Firebase project configuration
    var firebaseConfig = {
      // ...
    };
    // Initialize Firebase
    firebase.initializeApp(firebaseConfig);
  </script>
</body>
{{< /highlight >}}
## Firebase Authentication ##

Now that Firebase is initialized we can easily use the Firebase relevant functionality.

You can get an instance of Firebase Authentication by first importing `firebase.dart` and access the `Auth` object through the `auth()` method, and the `GoogleAuthProvider` object through the `GoogleAuthProvider()` method. See below:

{{< highlight dart >}}
import 'package:firebase/firebase.dart';
import 'package:meta/meta.dart';

@immutable
class UserRepository {
  UserRepository({Auth firebaseAuth, GoogleAuthProvider googleSignin})
      : _firebaseAuth = firebaseAuth ?? auth(),
        _googleSignIn = googleSignin ?? GoogleAuthProvider();

  final Auth _firebaseAuth;
  final GoogleAuthProvider _googleSignIn;

  Future<UserCredential> signInWithGoogle() async {
    try {
      return await _firebaseAuth.signInWithPopup(_googleSignIn);
    } catch (e) {
      print('Error in sign in with google: $e');
      throw '$e';
    }
  }

  Future<UserCredential> signInWithCredentials(
      String email, String password) async {
    try {
      return await _firebaseAuth.signInWithEmailAndPassword(email, password);
    } catch (e) {
      print('Error in sign in with credentials: $e');
      // return e;
      throw '$e';
    }
  }

  Future<UserCredential> signUp({String email, String password}) async {
    try {
      return await _firebaseAuth.createUserWithEmailAndPassword(
        email,
        password,
      );
    } catch (e) {
      print('Error siging in with credentials: $e');
      throw '$e';
      // throw Error('Error signing up with credentials: $e');
      // return e;
    }
  }

  Future<dynamic> signOut() async {
    try {
      return Future.wait([
        _firebaseAuth.signOut(),
      ]);
    } catch (e) {
      print ('Error signin out: $e');
      // return e;
      throw '$e';
    }
  }

  Future<bool> isSignedIn() async {
    final currentUser = _firebaseAuth.currentUser;
    return currentUser != null;
  }

  Future<String> getUser() async {
    return (_firebaseAuth.currentUser).email;
  }
}
{{< /highlight >}}

And here is an example to log in through email and password:

{{< highlight dart >}}
Future<UserCredential> signInWithCredentials(
      String email, String password) async {
    try {
      return await _firebaseAuth.signInWithEmailAndPassword(email, password);
    } catch (e) {
      print('Error in sign in with credentials: $e');
      throw '$e';
    }
}
{{< /highlight >}}

That’s it! You are set up and have an example method. For more examples, you can take a look at the [Fun with Flutter](https://funwith.app) web application’s source code:

https://github.com/funwithflutter/fun-with-flutter-website

There is also a link on the [Firebase Pub page](https://pub.dev/packages/firebase) that point to additional examples.

https://github.com/FirebaseExtended/firebase-dart/tree/master/example

## Conclusion ##

I hope you found this useful. It is quite easy to do once you know how. If you have better solutions let me know in the comments!

If you want to explore a website made entirely in Flutter, as well as read more Flutter articles, go take a look at the [Fun with Flutter website](https://funwith.app). It’s still under active development, and Flutter web is still new, so don’t judge too harshly. Ideally, visit it on Chrome on a computer for best performance.