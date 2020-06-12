+++
categories = []
date = ""
description = ""
draft = true
slug = ""
tags = []
thumbnail = ""
title = "Provider with StateNotifier"
type = ""

+++
## Overview

In this tutorial we will take a look at the [State Notifier](https://pub.dev/packages/state_notifier) and [Flutter State Notifier](https://pub.dev/packages/flutter_state_notifier) packages, and how they can be used with [Provider](https://pub.dev/packages/provider).

As a demonstration we will create a counter application that also needs access to an external dependency to store the counter value to a "fake" local storage.

A simple example, but it serves to illustrate the simplicity of `StateNotifier` as well as the benefits it provides.

Prefer to watch instead of read? Here's the video:

INSERT VIDEO

## Before we begin

**Note**: This tutorial assumes you are familiar with Provider and understand what a [ChangeNotifier](https://api.flutter.dev/flutter/foundation/ChangeNotifier-class.html "ChangeNotifier") and [ValueNotifier](https://api.flutter.dev/flutter/foundation/ValueNotifier-class.html "ValueNotifier") is.

If you are not familiar with `Provider` and `ChangeNotifier` then I recommend you first watch my video on `Provider` basics.

INSERT VIDEO

But as a quick summary a `ValueNotifier` is a `ChangeNotifier` that holds a single value.

> When value is replaced with something that is not equal to the old value as evaluated by the equality operator ==, this class notifies its listeners.

```dart
final counter = ValueNotifier(0);
counter.addListener(_myCallback);
counter.value = 10;  // Calls _myCallback.
counter.value += 1;  // Also calls _myCallback.
counter.removeListener(_myCallback);
  
counter.value += 1;  // Doesn't call anything.
counter.dispose();
```

Flutter also provides widget to easily consume a `ValueNotifier` :

So what is the stateNotifier package?

StateNotifier re-implements ValueNotifier outside of Flutter.

To quote the documentation:

> Extracting ValueNotifier outside of Flutter in a separate package has two purposes:

* It allows Dart packages with no dependency on Flutter to use these classes. **This means that we can use them on AngularDart for example.**
* It allows solving some common problems with the original ChangeNotifier/ValueNotifier and/or their combination with provider.

This second bullet is what we are really interested in. The bit that says "their combination with provider".

By using state_notifier instead of the original ValueNotifier, we get a lot of benefits:

* A significant simplification of the integration with [**provider**](https://pub.dev/packages/provider)
* Simplified testing/mocking
* Improved performances on `addListener` & `notifyListeners` equivalents.
* Extra safety through small API changes

In this tutorial, we will also use the flutter_state_notifier package which add extra Flutter bindings to StateNotifier. This package is what allows us to integrate StateNotifier with Provider and with our widgets.

With the summary out of the way. Let's start the coding example.