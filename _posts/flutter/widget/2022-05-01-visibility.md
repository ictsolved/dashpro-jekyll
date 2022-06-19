---
layout: post
title: Flutter Visibility Widget Explained with Example
description: In this recipe, we are going to discuss about a handy widget of Flutter called Visibility. It helps to display a widget conditionally on the basis of it's boolean property.
category:
  - Flutter
  - Widget
dartpad: true
---

In this recipe, we are going to discuss about a handy widget of Flutter called Visibility. It helps to display a widget conditionally on the basis of it's boolean property `visible`. This property decides whether to include the child in the subtree or not. When it is set to false, the child is replaced with `SizedBox.shrink()` i.e. a zero-sized box by default. The best thing about the `visible` property is, it's state can be changed dynamically as per our need. This means, we can programatically show or hide a widget on the basis of a condition.

Let's take a look at it's constructor and fields:

```dart
class Visibility extends StatelessWidget {
const Visibility({
    Key? key,
    required this.child,
    this.replacement = const SizedBox.shrink(),
    this.visible = true,
    this.maintainState = false,
    this.maintainAnimation = false,
    this.maintainSize = false,
    this.maintainSemantics = false,
    this.maintainInteractivity = false,
  });

  final Widget child;
  final Widget replacement;
  final bool visible;
  final bool maintainState;
  final bool maintainAnimation;
  final bool maintainSize;
  final bool maintainSemantics;
  final bool maintainInteractivity;
}
```

`child`: Widget to display when `visible` property is true.

`replacement`: Widget to replace with when `visible` is false. Replaced with zero sized box if not specified.

`visible`: Boolean value that determines whether the `child` is visible or not.

`maintainState`: Defines whether to maintain the state objects of the `child` subtree when it is not `visible`. If false, the `child` subtree will be removed from the tree. When this property is set to true, `Offstage` widget replaces the child instead of `replacement`. Here is a piece of information on Offstage taken from Flutter documentation:

> A widget that lays the child out as if it was in the tree, but without painting anything, without making the child available for hit testing, and without taking any room in the parent.

> It is recommended to maintain the state only when it cannot be recreated on demand. It is potentially expensive to keep the state of subtree because the resources allocated by the objects are not released from the memory. 

`maintainAnimation`: Determines whether the animations within the `child` subtree should be maintained when `child` is not `visible`. It requires `maintainState` to be set too.

> It is even more expensive to keep the animations active when the widget is not visible.

`maintainSize`: It determines whether to maintain the space for `child` when it not visible. `maintinState` and `maintainAnimation` must also be set in order to set this.

`maintainSemantics`: Tells whether to maintain the semantics of the widget when not `visible`. When set to true, accessibility tools can find the `child` widget although it is hidden from the user. `maintainSize` must be set to use this.

`maintainInteractivity`: Allows the widget to be interactive, i.e. receive touch events when it is not `visible`. To set this property, `maintainSize` must also be set.

> The values of all maintain flags should be same in order to operate correctly. The state will be lost if the value of any of these flags get changed.

The following example demonstrates the use of the `Visibility` widget. Feel free to experiment with the values and code.

```run-dartpad:theme-dark:mode-flutter:run-true:null_safety-true:split-60:width-100%:height-500px:ga_id-visibility_widget_explained_with_example
import 'package:flutter/material.dart';

void main() {
  runApp(const MegaDash());
}

class MegaDash extends StatelessWidget {
  const MegaDash({Key? key}) : super(key: key);

  String get title => 'Flutter Visibility';

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: title,
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: FlutterVisibility(title: title),
      );
}

class FlutterVisibility extends StatefulWidget {
  const FlutterVisibility({required this.title, Key? key}) : super(key: key);

  final String title;

  @override
  State<FlutterVisibility> createState() => _FlutterVisibilityState();
}

class _FlutterVisibilityState extends State<FlutterVisibility> {
  bool visible = true;
  bool maintainState = false;
  bool maintainAnimation = false;
  bool maintainSize = false;
  bool maintainSemantics = false;
  bool maintainInteractivity = false;

  void toggleMaintainVisibility() {
    setState(() {
      visible = !visible;
    });
  }

  void toggleMaintainState() {
    setState(() {
      if (maintainState) {
        maintainAnimation = false;
        maintainSize = false;
        maintainSemantics = false;
        maintainInteractivity = false;
      }
      maintainState = !maintainState;
    });
  }

  void toggleMaintainAnimation() {
    setState(() {
      if (!maintainAnimation) {
        maintainState = true;
      } else {
        maintainSize = false;
        maintainSemantics = false;
        maintainInteractivity = false;
      }
      maintainAnimation = !maintainAnimation;
    });
  }

  void toggleMaintainSize() {
    setState(() {
      if (!maintainSize) {
        maintainState = true;
        maintainAnimation = true;
      } else {
        maintainSemantics = false;
        maintainInteractivity = false;
      }
      maintainSize = !maintainSize;
    });
  }

  void toggleMaintainSemantics() {
    setState(() {
      if (!maintainSemantics) {
        maintainState = true;
        maintainAnimation = true;
        maintainSize = true;
      }
      maintainSemantics = !maintainSemantics;
    });
  }

  void toggleMaintainInteractivity() {
    setState(() {
      if (!maintainInteractivity) {
        maintainState = true;
        maintainAnimation = true;
        maintainSize = true;
      }
      maintainInteractivity = !maintainInteractivity;
    });
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: Text(widget.title),
        ),
        body: Column(
          children: [
            Container(
              alignment: Alignment.center,
              color: Colors.lightBlue.shade100,
              padding: const EdgeInsets.all(8),
              height: 150,
              child: Visibility(
                visible: visible,
                maintainState: maintainState,
                maintainAnimation: maintainAnimation,
                maintainSize: maintainSize,
                maintainSemantics: maintainSemantics,
                maintainInteractivity: maintainInteractivity,
                replacement: SizedBox(
                  width: 200,
                  child: Card(
                      child: Container(
                          alignment: Alignment.center,
                          padding: const EdgeInsets.all(10),
                          child: const Text('Replaced Widget'))),
                ),
                child: Container(
                  width: 200,
                  height: 200,
                  decoration: const BoxDecoration(
                      shape: BoxShape.circle,
                      border: Border.fromBorderSide(
                          BorderSide(color: Colors.white))),
                  alignment: Alignment.center,
                  child: const Text('Visible Widget'),
                ),
              ),
            ),
            FittedBox(
              child: Row(
                children: [
                  const Text('Visibility'),
                  Switch(
                      value: visible,
                      onChanged: (_) {
                        toggleMaintainVisibility();
                      }),
                  const Text('Maintain state'),
                  Switch(
                      value: maintainState,
                      onChanged: (_) {
                        toggleMaintainState();
                      }),
                ],
              ),
            ),
            FittedBox(
              child: Row(
                children: [
                  const Text('Maintain animation'),
                  Switch(
                      value: maintainAnimation,
                      onChanged: (_) {
                        toggleMaintainAnimation();
                      }),
                  const Text('Maintain size'),
                  Switch(
                      value: maintainSize,
                      onChanged: (_) {
                        toggleMaintainSize();
                      }),
                ],
              ),
            ),
            FittedBox(
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  const Text('Maintain semantics'),
                  Switch(
                      value: maintainSemantics,
                      onChanged: (_) {
                        toggleMaintainSemantics();
                      }),
                  const Text('Maintain interactivity'),
                  Switch(
                      value: maintainInteractivity,
                      onChanged: (_) {
                        toggleMaintainInteractivity();
                      }),
                ],
              ),
            ),
          ],
        ),
      );
}
```

In the above example, removing `replacement` property will make the widget completely invisible when Visibility is toggoled off.
