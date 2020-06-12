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

Flutter also provides a widget to easily consume a `ValueNotifier`, called

[ValueListenableBuilder](https://api.flutter.dev/flutter/widgets/ValueListenableBuilder-class.html):

```dart
final ValueNotifier<int> _counter = ValueNotifier<int>(0);

ValueListenableBuilder(
  builder: (BuildContext context, int value, Widget child) {
    // This builder will only get called when the _counter
    // is updated.
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      children: <Widget>[
        Text('$value'),
        child,
      ],
    );
  },
  valueListenable: _counter,
  // The child parameter is most helpful if the child is
  // expensive to build and does not depend on the value from
  // the notifier.
  child: ChildWidget(),
);
```

Both a `ValueNotifier` and `ChangeNotifier` provide a convenient way to listen to value changes on an object. These classes are used throughout Flutter and they are used extensively with `Provider` when managing state. Using these two classes to manage the state of your application in Flutter is completely fine, and there are entire architectures based on this approach - see [Stacked](https://pub.dev/packages/stacked) made by the FilledStacks community.

However, `ValueNotifier` and `ChangeNotifier` have some limitations and their implementation with `Provider` requires a lot of boilerplate code and is not as simple as it could be.

## What is StateNotifier and why?

`StateNotifier` re-implements `ValueNotifier` outside of Flutter.

To quote the documentation:

> Extracting ValueNotifier outside of Flutter in a separate package has two purposes:
>
> * It allows Dart packages with no dependency on Flutter to use these classes. **This means that we can use them on AngularDart for example.**
> * It allows solving some common problems with the original ChangeNotifier/ValueNotifier and/or their combination with provider.

This second bullet is what we are really interested in. The bit that says "their combination with provider".

By using `StateNotifier` instead of the original `ValueNotifier`, we get a lot of benefits:

* A significant simplification of the integration with [Provider](https://pub.dev/packages/provider)
* Simplified testing/mocking
* Improved performances on `addListener` & `notifyListeners` equivalents.
* Extra safety through small API changes

## Show me the code

We will begin by creating a `Counter` class that extends `StateNotifier` with a type `int`.

```dart
class Counter extends StateNotifier<int>{
  Counter() : super(0);

  void increment() {
    state++;
  }
 
  void decrement() {
    state--;
  }
}
```

The code is pretty clear, we have an `increment` and `decrement` method, which directly change the `state`. This state object is retrieved from `StateNotifier` and is of type `int` . Every time the state changes any subscribers to the `Counter` instance will be notified.

We can use this `Counter` like we would use a normal `ValueNotifier`. But for this tutorial we want to integrate `StateNotifier` with `Provider`. So let's do that.

### Using StateNotifier with Provider

We'll use the [Flutter State Notifier](https://pub.dev/packages/flutter_state_notifier) package which add extra Flutter bindings to `StateNotifier`. This package is what allows us to integrate `StateNotifier` with `Provider` and our other widgets.

#### StateNotifierProvider:

[StateNotifierProvider](https://pub.dev/documentation/flutter_state_notifier/latest/flutter_state_notifier/StateNotifierProvider-class.html) is the equivalent of [ChangeNotifierProvider](https://pub.dev/documentation/provider/latest/provider/ChangeNotifierProvider-class.html) but for [StateNotifier](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html).

Its job is to create a [StateNotifier](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html) and dispose of it when the provider is removed from the widget tree.

It is used like most providers, with a small difference:  
Instead of exposing one value, it exposes two values at the same time:

* The [StateNotifier](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html) instance
* And the `state` of the [StateNotifier](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html)

For example, if we want to instantiate and provide our `Counter`, we can do this.

```dart
void main() => runApp(
      StateNotifierProvider<Counter, int>(
        create: (_) => Counter(),
        child: MyApp(),
      ),
    ); 
```

This allows us to both:

* obtain the [StateNotifier](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html) in the widget tree, by writing `context.read<`Counter`>()`
* obtain and observe the current \[int\], through `context.watch<int>()`

So our `MyApp` will look like this:

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    const fabPadding = EdgeInsets.all(5);

    return MaterialApp(
      title: 'Material App',
      home: Scaffold(
        appBar: AppBar(
          title: Text('State Notifier Demo'),
        ),
        floatingActionButton: Column(
          crossAxisAlignment: CrossAxisAlignment.end,
          mainAxisAlignment: MainAxisAlignment.end,
          children: <Widget>[
            Padding(
              padding: fabPadding,
              child: FloatingActionButton(
                child: Icon(Icons.add),

                ///Increment Counter
                onPressed: () {
                  context.read<Counter>().increment();
                },
              ),
            ),
            Padding(
              padding: fabPadding,
              child: FloatingActionButton(
                  child: Icon(Icons.remove),

                  ///Decrement Counter
                  onPressed: () {
                    context.read<Counter>().decrement();
                  }),
            ),
          ],
        ),
        body: _Body(),
      ),
    );
  }
}

class _Body extends StatelessWidget {
  const _Body({
    Key key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        child: Text(
          context.watch<int>().toString(),
          style: Theme.of(context).textTheme.headline4,
        ),
      ),
    );
  }
}
```

That it. We have a working counter application. Our `_Body` is watching for changes on the `int` state and will rebuild whenever the state changes.

## Service Locators

Now we're getting to the best part. [**StateNotifier**](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html) is easily compatible with [**provider**](https://pub.dev/packages/provider) through an extra mixin: `LocatorMixin`.