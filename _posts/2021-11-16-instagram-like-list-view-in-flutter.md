---
layout: post
title:  "Instagram like Listview in Flutter"
date:   2021-11-15 09:21:06 -0500
categories: [Flutter, UX]
tags: [dart, flutter, UX]
---

In this article, we will simulate an Instagram search page. We will use a snap Listview for this task.

<video width="309" height="614" preload autoplay muted loop playsinline poster="">
  <source src="/assets/videos/2021/instagram-like-list-view-in-flutter-01.mp4">
</video>

```
Flutter (Channel stable, 2.5.3)
```

## Setup a new project

```shell
flutter create list_view_snap
```

After removing some code we don't use, we have a clear `main.dart` and an empty page. Let's start from here.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(debugShowCheckedModeBanner: false, home: MyHomePage());
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(body: Container());
  }
}
```

## Add a List View

Let's add a random color method for better UI:

```dart
import 'dart:math';

Color get randomColor => Color((Random().nextDouble() * 0xFFFFFF).toInt() << 0).withOpacity(1.0);
```

Build a vertical listview:

```dart
ListView.builder(
  scrollDirection: Axis.vertical,
  itemCount: 20,
  itemBuilder: (context, index) {
    return Container(
      width: double.infinity,
      height: 400,
      color: randomColor,
      margin: const EdgeInsets.all(20.0),
    );
  },
)
```

The full code of main.dart:

```dart
import 'dart:math';

import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(debugShowCheckedModeBanner: false, home: MyHomePage());
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  Color get randomColor => Color((Random().nextDouble() * 0xFFFFFF).toInt() << 0).withOpacity(1.0);

  Widget buildBody() {
    return ListView.builder(
      scrollDirection: Axis.vertical,
      itemCount: 20,
      itemBuilder: (context, index) {
        return Container(
          width: double.infinity,
          height: 400,
          color: randomColor,
          margin: const EdgeInsets.all(20.0),
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(body: buildBody());
  }
}
```

<video width="375" height="667"  preload autoplay muted loop playsinline poster="">
  <source src="/assets/videos/2021/instagram-like-list-view-in-flutter-02.mp4">
</video>

## Add Snap for Listview

We need a snap physics which is similar to [PageScrollPhysics](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/page_view.dart).

### SnapScrollPhysics

```dart
class SnapScrollPhysics extends ScrollPhysics {
  final double itemDimension;

  const SnapScrollPhysics({required this.itemDimension, ScrollPhysics? parent}) : super(parent: parent);

  @override
  SnapScrollPhysics applyTo(ScrollPhysics? ancestor) {
    return SnapScrollPhysics(itemDimension: itemDimension, parent: buildParent(ancestor));
  }

  double _getPage(ScrollMetrics position) {
    return position.pixels / itemDimension;
  }

  double _getPixels(double page) {
    return page * itemDimension;
  }

  double _getTargetPixels(ScrollMetrics position, Tolerance tolerance, double velocity) {
    double page = _getPage(position);
    if (velocity < -tolerance.velocity) {
      page -= 0.5;
    } else if (velocity > tolerance.velocity) {
      page += 0.5;
    }
    return _getPixels(page.roundToDouble());
  }

  @override
  Simulation? createBallisticSimulation(ScrollMetrics position, double velocity) {
    if ((velocity <= 0.0 && position.pixels <= position.minScrollExtent) ||
        (velocity >= 0.0 && position.pixels >= position.maxScrollExtent)) {
      return super.createBallisticSimulation(position, velocity);
    }
    final Tolerance tolerance = this.tolerance;
    final double target = _getTargetPixels(position, tolerance, velocity);
    if (target != position.pixels) {
      return ScrollSpringSimulation(spring, position.pixels, target, velocity, tolerance: tolerance);
    }
    return null;
  }

  @override
  bool get allowImplicitScrolling => false;
}
```

### Use the SnapScrollPhysics

Then in Listview, we use the SnapScrollPhysics. The `itemDimension` is the height of the container with the margin.

```dart
ListView.builder(
  scrollDirection: Axis.vertical,
  itemCount: 20,
  physics: const SnapScrollPhysics(itemDimension: 440),
  itemBuilder: (context, index) {
    return Container(
      width: double.infinity,
      height: 400,
      color: randomColor,
      margin: const EdgeInsets.all(20.0),
    );
  },
)
```

<video width="375" height="667"  preload autoplay muted loop playsinline poster="">
  <source src="/assets/videos/2021/instagram-like-list-view-in-flutter-03.mp4">
</video>

## Go further

### Add a special header

How about if we need a header on the first page? Let's do this.

Step 1: Add an extra element in Listview for the header.

```dart
const double headerHeight = 200.0;

itemBuilder: (context, index) {
  if (index == 0) {
    return Container(
      width: double.infinity,
      height: headerHeight,
      color: Colors.amberAccent,
      margin: const EdgeInsets.all(20.0),
      child: const Center(child: Text('The header', style: TextStyle(fontSize: 30))),
    );
  }

  // ...
}
        
```

Step 2: modify the SnapScrollPhysics to support an extra header element.

We add `headerDimension` for the ScrollPhysics to know the height of the header and modify `_getPage` and `_getPixels`.



```dart
final double headerDimension;

const SnapScrollPhysics({required this.itemDimension, this.headerDimension = 0, ScrollPhysics? parent})
    : super(parent: parent);

@override SnapScrollPhysics applyTo(ScrollPhysics? ancestor) {
  return SnapScrollPhysics(
    itemDimension: itemDimension, headerDimension: headerDimension, parent: buildParent(ancestor));
}
  
double _getPage(ScrollMetrics position) {
  if (position.pixels <= headerDimension) {
    return position.pixels / headerDimension;
  }

  return (position.pixels - headerDimension) / itemDimension + 1;
}

double _getPixels(double page) {
  if (page < 1) {
    return 0;
  }

  return (page - 1) * itemDimension + headerDimension;
}
```

<video width="375" height="667"  preload autoplay muted loop playsinline poster="">
  <source src="/assets/videos/2021/instagram-like-list-view-in-flutter-04.mp4">
</video>

### Add pull refresh

Step 1: Install [pull_to_refresh](https://pub.dev/packages/pull_to_refresh)

Step 2: Add pull refresh

```dart
final RefreshController refreshController = RefreshController(initialRefresh: false);

void onRefresh() async {
  await Future.delayed(const Duration(milliseconds: 1000));
  refreshController.refreshCompleted();
}

void onLoading() async {
  await Future.delayed(const Duration(milliseconds: 1000));
  refreshController.loadComplete();
}

@override
Widget build(BuildContext context) {
  return Scaffold(
    body: SmartRefresher(
      enablePullDown: true,
      enablePullUp: true,
      header: const WaterDropHeader(),
      controller: refreshController,
      onRefresh: onRefresh,
      onLoading: onLoading,
      child: buildBody(),
    ),
  );
}
```

<video width="375" height="667"  preload autoplay muted loop playsinline poster="">
  <source src="/assets/videos/2021/instagram-like-list-view-in-flutter-05.mp4">
</video>

The full code of [main.dart](https://gist.github.com/devon/1be70d54c86ef08f480dc34b88c6e19b).

