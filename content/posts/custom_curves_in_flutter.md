---
title: "Creating Animation Curves in Flutter"
slug: custom-curves-in-flutter
description: null
date: 2020-02-24T14:25:21+07:00
type: posts
draft: false
categories:
- General
tags:
- Animation
series:
-
thumbnail: "flutter_custom_curves.png"
description: "How to create custom animation curves in Flutter"
---
In this blog post we will take a look at how we can create our own custom [Curve](https://api.flutter.dev/flutter/animation/Curve-class.html) in Flutter. And as an example we will create two curves: a Sine and Spring curve.

A curve in Flutter can be any mapping of a function over a time period (*t*) from **0.0** to **1.0**.

> Easing curves are used to adjust the rate of change of an animation over time, allowing them to speed up and slow down, rather than moving at a constant rate.

> A curve must map *t=0.0* to **0.0** and *t=1.0* to **1.0**.

So to illustrate this we need to define some function that takes a time *t* and outputs some value, however, we have to respect the conditions that when *t=0.0* then the function should output **0.0**, and when *t=1.0* the function should output **1.0**.

As an example, let's say we have a normal linear function:

```
f(t) = mt+c
```

This is a straight line, with constants *m* and *c*, and the function is defined by input *t*.

Let's make this function even more boring, and define a value of **0** for *c* and **1** for *m*. Now we have:

```
f(t) = 1t +0
     = t
```

![Straight Line](/pictures/custom_curves_in_flutter/straight_line.jpg)

We just defined the default animation curve in Flutter: [Curves.linear](https://api.flutter.dev/flutter/animation/Curves/linear-constant.html).

Borrrinnngg.

## Sine Curve

Let's create something more interesting and create a looping animation using a [Sine](https://en.wikipedia.org/wiki/Sine) curve. We'll define the following function:

```
f(t) = sin(t)
```

![Plain Sine Curve](/pictures/custom_curves_in_flutter/plain_sine_curve.jpg)

But now we have a problem, remember that our Flutter curve must map *t=0.0* to **0.0** and *t=1.0* to **1.0**.

As you can see in the image above, or if you take a look at the graph [here](https://www.desmos.com/calculator/p4vpbfrtny), you will note that this is not the case.

You will also note that the curve does not do any oscillations (or loops) within the time frame from 0 to 1.

So we want to modify the sine curve slightly:

```
f(t) = sine(3*2pi*t)*0.5 +0.5
```

We multiple *t* by 3*2pi to get 3 oscillations, we decrease the height of the curve by multiplying everything by 0.5, and then offset the curve to make sure it returns a positive value by increasing the output of the function by 0.5.

Now we have this:

![Custom Sine Curve](/pictures/custom_curves_in_flutter/custom_sine_curve.jpg)

Take a look at the graph here: https://www.desmos.com/calculator/omh7xyuzxk

This is much better, however, keen observers will note that it still does not map a value of **0.0** at *t=0.0* and **1.0** at *t=1.0*.

Technically this would be a problem if we do a once of animation, but we want to create an animation that loops. For example, by calling the [repeat](https://api.flutter.dev/flutter/animation/AnimationController/repeat.html) method on our [AnimationController](https://api.flutter.dev/flutter/animation/AnimationController-class.html).

Take a look at this demo application that I made, to see an example of the animation in action:
https://dartpad.dev/a3c7d5b24443efdf28ecd3ba19222555

The application is a [PageView](https://api.flutter.dev/flutter/widgets/PageView-class.html) with multiple examples, keep scrolling until you hit the *Sine demo* page. You will note that the animation does not satisfy the 0 and 1 rule, however, it works because it is looping. It restarts the animation at the same point where it ended.

So let's take this function and create a curve in Flutter.

We need to create a new class which extends the [Curve](https://api.flutter.dev/flutter/animation/Curve-class.html) class, and then override the [transformInternal](https://api.flutter.dev/flutter/animation/Cubic/transformInternal.html) method, which receives a value *t* of type double, that we can use to pass in to our own function and return that value.

See below:

{{< highlight dart >}}
class SineCurve extends Curve {
  final double count;

  SineCurve({this.count = 3});

  // t = x
  @override
  double transformInternal(double t) {
    var val = sin(count * 2 * pi * t) * 0.5 + 0.5;
    return val; //f(x)
  }
}
{{< /highlight >}}

There we go. Now you can use this as you normally would when using a curve.

## Spring Curve

In a similar way we can create a spring curve, such as the one below:

![Spring Curve](/pictures/custom_curves_in_flutter/spring_curve.jpg)

And the code for this:

{{< highlight dart >}}
class SpringCurve extends Curve {
  const SpringCurve({
    this.a = 0.15,
    this.w = 19.4,
  });
  final double a;
  final double w;

  @override
  double transformInternal(double t) {
    return -(pow(e, -t / a) * cos(t * w)) + 1;
  }
}
{{< /highlight >}}

Note that this curves satisfy the rule of returning **0.0** at *t=0.0* and **1.0** at *t=1.0*. At least, it almost satisfies that rule, at *t=1.0* it's not exactly **1.0**, but close enough :)

If you need unique animations in your application, or animations that require specific behavior, then knowing how to create your own curve will be of great value. There is no limit to what curve you can create.

Here is the link to the spring curve, play around with it and see what you can make:

https://www.desmos.com/calculator/6gbvrm5i0s

![Learning More](/memes/learning.gif)

**Want to learn more?** Check out my *Mastering Animation in Flutter* course.
The course goes into great detail discussing everything you need to be a pro at animation in Flutter. It's the handbook to animate everything in Flutter, and it will teach you what you need to know in a structured and fun way. You might also be lucky and get it at a discount, use the promo code **FUN** for a **75% discount**.

https://fun-with-flutter.teachable.com/

Each lecture has a dedicated video, code examples, and an interactive DartPad to get you interacting and working with the code examples.