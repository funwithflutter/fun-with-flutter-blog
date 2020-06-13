+++
categories = []
date = 2020-06-12T08:24:00Z
description = "Using StateNotifier with Provider to easily manage state."
slug = ""
tags = ["State Managment"]
thumbnail = ""
title = "Provider with StateNotifier"
type = ""

+++
## Overview

In this tutorial we will take a look at the [State Notifier](https://pub.dev/packages/state_notifier) and [Flutter State Notifier](https://pub.dev/packages/flutter_state_notifier) packages, and how they can be used with [Provider](https://pub.dev/packages/provider).

As a demonstration we will create a counter application that also needs access to an external dependency to store the counter value to a "fake" local storage.

A simple example, but it serves to illustrate the simplicity of `StateNotifier` as well as its benefits.

{{< youtube 4J7rF3KirxM>}}

## Before we begin

**Note**: This tutorial assumes you are familiar with Provider and understand what a [ChangeNotifier](https://api.flutter.dev/flutter/foundation/ChangeNotifier-class.html "ChangeNotifier") and [ValueNotifier](https://api.flutter.dev/flutter/foundation/ValueNotifier-class.html "ValueNotifier") is.

If you are not familiar with `Provider` and `ChangeNotifier` then I recommend you first watch my video on `Provider` basics.

{{< youtube NeAMD0lQ5jw>}}

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
...

return ValueListenableBuilder(
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
> * It allows Dart packages with no dependency on Flutter to use these classes. This means that we can use them on AngularDart for example.
> * It allows solving some common problems with the original ChangeNotifier/ValueNotifier and/or their combination with provider.

This second bullet is what we are really interested in. The bit that says "their combination with provider".

By using `StateNotifier` instead of the original `ValueNotifier` we get a lot of benefits:

* A significant simplification of the integration with [Provider](https://pub.dev/packages/provider)
* Simplified testing/mocking
* Improved performances on `addListener` & `notifyListeners` equivalents
* Extra safety through small API changes

## Show me the code

First we need to add some dependencies to our app in `pubspec.yaml`.

_Take note of the versions used for this tutorial. Future versions may have different implementations than what is shown in this tutorial._

```dart
dependencies:
  flutter:
    sdk: flutter
  state_notifier: ^0.5.0
  flutter_state_notifier: ^0.4.2
  provider: ^4.1.3
```

We will begin by creating a `Counter` class that extends `StateNotifier` with a generic type `int`.

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

The code is pretty clear, we have an `increment` and `decrement` method, which directly change the `state`. This state object is retrieved from `StateNotifier` and is of type `int` (as we specified). Every time the state changes any subscribers to the `Counter` instance will be notified.

We can use this `Counter` like we would use a normal `ValueNotifier`, but for this tutorial we want to integrate our `Counter` with `Provider`. So let's do that.

### Using StateNotifier with Provider

We'll use the [Flutter State Notifier](https://pub.dev/packages/flutter_state_notifier) package which add extra Flutter bindings to `StateNotifier`. This package is what allows us to integrate `StateNotifier` with `Provider` and our other widgets.

***

#### StateNotifierProvider:

[StateNotifierProvider](https://pub.dev/documentation/flutter_state_notifier/latest/flutter_state_notifier/StateNotifierProvider-class.html) is the equivalent of [ChangeNotifierProvider](https://pub.dev/documentation/provider/latest/provider/ChangeNotifierProvider-class.html) but for [StateNotifier](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html).

Its job is to create a [StateNotifier](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html) and dispose of it when the provider is removed from the widget tree.

It is used like most providers, with a small difference. Instead of exposing one value, it exposes two values at the same time:

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

* obtain the [StateNotifier](https://pub.dev/documentation/state_notifier/latest/state_notifier/StateNotifier-class.html) in the widget tree, by writing `context.read<Counter>()`
* obtain and observe the current `int` state, through `context.watch<int>()`

Our final `MyApp` will look like this:

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

That's it, we have a working counter application. Our `_Body` is watching for changes on the `int` state and will rebuild whenever the state changes. And in our `FloatingActionButton`s we're accessing the `Counter` `StateNotifier` and calling `increment` and `decrement`.

## Service Locators

Now we're getting to the best part.

Say we want to use `Provider.of`/`context.read` to connect our `Counter` with external services. `StateNotifier` makes this easy, all we have to do is mix-in `LocatorMixin`.

As an example, let's pretend our counter application also needs to save the current counter value to local storage. For that we might need access to an external service within our `Counter` class. There are many ways we can access the service, for example using the [GetIt](https://pub.dev/packages/get_it) package or some other dependency injection framework. Or we could simply pass in the dependency through the constructor. However, `StateNotifier` makes this easy for us using locators.

Let's explore.

We'll create an `ILocalStorage` abstract class with one method called `saveInt`, and a `FakeLocalStorage` that implements it. We won't really be creating a local storage, instead we'll only print the value as this is only for demonstration purposes.

```dart
abstract class ILocalStorage {
  Future<void> saveInt(String key, int val);
}

class FakeLocalStorage implements ILocalStorage {
  @override
  Future<void> saveInt(String key, int val) async {
    print('saving $val to $key');
  }
}
```

Next we will modify our `runApp` function to also have a `Provider<ILocalStorage>` that provides a `FakeLocalStorage` implementation. We use a `MultiProvider` to create and provide everything.

```dart
void main() => runApp(
      MultiProvider(
        providers: [
          Provider<ILocalStorage>(
            create: (_) => FakeLocalStorage(),
          ),
          StateNotifierProvider<Counter, int>(
            create: (_) => Counter(),
          ),
        ],
        child: MyApp(),
      ),
    );
```

Then finally we need to be able to access the `ILocalStorage` in our `Counter` class:

```dart
class Counter extends StateNotifier<int> with LocatorMixin{
  Counter() : super(0);

  void increment() {
    state++;
  }
 
  void decrement() {
    state--;
  }

  @protected
  @override
  set state(int value) {
    read<ILocalStorage>().saveInt('count', value);
    super.state = value;
  }
}
```

A couple of things to take note of:

* We're adding the `LocatorMixin`
* We're overriding the `state` setter so that we can save the value to local storage every time the state change
* We're using the read method (from the `LocatorMixin`) to access the `ILocalStorage` dependency and calling `saveInt`.

That's that. As you can see it's easy to access other dependencies in our `StateNotifier` when using `Provider`.

## What's next?

Using `Provider` and `StateNotifier`  is a quick and easy way to create and expose state in our Flutter applications. Not only that, but seeing as `StateNotifier`is not dependent on Flutter we can use our state classes for other projects as well, for example CLI apps or AngularDart.

Instead of just passing around `int` values we can create more complicated state management using Sealed classes. Future tutorials will cover this in more detail.