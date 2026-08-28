# 1. Import

import 'package:flutter/material.dart';

### `import`

`import` means:

> Bring code from another library/package into my current file so I can use it.

### `'package:flutter/material.dart'`

This is Flutter's **Material Design library**.

It gives you widgets such as:

- `MaterialApp`
- `Scaffold`
- `AppBar`
- `Text`
- `Center`
- `ElevatedButton`

### `;`

The semicolon marks the end of this Dart statement.

So this entire line means:

> Import Flutter's Material Design widgets.

---

# 2. The `main()` function

void main() {

  runApp(const MyApp());

}

## `void`

`void` means this function **does not return a value**.

For example:

main() does something

       ↓

doesn't send a value back

So its return type is `void`.

---

## `main`

`main` is the **starting point of a Dart application**.

When you start your Flutter app, Dart begins here:

Flutter App Starts

       ↓

    main()

       ↓

   runApp()

       ↓

   MyApp

---

## `()`

The parentheses contain the function's parameters.

Your `main` function has no parameters:

main()

---

## `{ }`

Curly brackets contain the **body of the function**—the code that runs when `main()` is called.

Inside:

{

  runApp(const MyApp());

}

---

# 3. `runApp`

runApp(const MyApp());

`runApp()` tells Flutter:

> Start the application and display this widget.

The widget you're giving it is:

MyApp

So:

runApp

   ↓

MyApp()

   ↓

Flutter starts building your UI

---

## `const`

const MyApp()

`const` tells Dart:

> This object can be created as a constant because its value will not change.

Flutter can use this as an optimization.

You will learn more about `const` later, but for now remember:

> Use `const` when the widget and its values are known and don't need to change.

---

## `MyApp()`

This creates an **object/widget** from the `MyApp` class.

You define that class below:

class MyApp extends StatelessWidget

---

# 4. Creating a class

class MyApp extends StatelessWidget {

Let's break this into pieces.

## `class`

A `class` is like a **blueprint**.

You define what something should contain and how it should behave.

Here:

class MyApp

means:

> I am creating a blueprint called `MyApp`.

---

## `MyApp`

This is the name of your class.

Dart convention says class names use **PascalCase**:

- `MyApp` ✅
- `myApp` ❌ convention-wise
- `my_app` ❌ convention-wise

---

## `extends`

`extends` means:

> MyApp inherits from another class.

Here:

MyApp extends StatelessWidget

means:

> MyApp is a type of StatelessWidget.

---

## `StatelessWidget`

A `StatelessWidget` is a widget whose UI doesn't manage changing state.

For example, initially:

Text → Hello Flutter!

It stays the same.

So conceptually:

MyApp

  IS A

StatelessWidget

---

# 5. Constructor

const MyApp({super.key});

This is one of the more difficult lines for beginners.

## `MyApp`

This is the constructor for the `MyApp` class.

When you write:

MyApp()

Dart calls this constructor.

---

## `const`

This allows objects of `MyApp` to be created with `const`.

That's why this works:

const MyApp()

---

## `{ }`

Inside function or constructor parameters, curly brackets mean the parameter is **named**.

Here:

{super.key}

means you can provide the parameter by name.

For example conceptually:

MyApp(key: something)

---

## `super`

`super` refers to the **parent class**.

Your parent is:

StatelessWidget

So:

super.key

passes the `key` to the parent `StatelessWidget`.

You don't need to deeply understand keys yet. For now:

> Flutter uses keys to help identify and manage widgets.

---

# 6. `@override`

@override

This means:

> I am replacing or implementing a method that already exists in my parent class.

`StatelessWidget` expects you to provide a `build()` method.

So you override it.

---

# 7. The `build()` method

Widget build(BuildContext context) {

This line is extremely important.

## `Widget`

This is the **return type**.

It means:

> This function must return a Widget.

Your `build()` method eventually returns:

MaterialApp

And `MaterialApp` is a widget.

---

## `build`

`build` is the method Flutter calls to create your UI.

Conceptually:

Flutter:

"Tell me what should appear on the screen."

  

        ↓

  

build()

        ↓

  

returns Widgets

---

## `BuildContext`

`BuildContext` contains information about **where this widget exists in the widget tree**.

Don't worry about mastering it yet.

For now:

> `context` helps Flutter widgets understand their position and surrounding environment.

---

## `context`

This is simply the variable name for the `BuildContext`.

You could technically name the variable differently, but `context` is the standard convention.

---

# 8. `return`

return MaterialApp(

`return` sends a value back from a function.

Earlier, we saw:

Widget build(...)

That means `build()` must return a `Widget`.

So:

return MaterialApp(...)

means:

> My build function is returning a MaterialApp widget.

---

# 9. `MaterialApp`

MaterialApp(

`MaterialApp` is usually the **main wrapper** around a Material Design Flutter application.

Conceptually:

MaterialApp

     ↓

Your Flutter application

It can manage things such as:

- Themes
- Navigation
- Routes
- Application configuration

In your code, you're using its:

home:

property.

---

# 10. `home`

home: Scaffold(

`home` means:

> This is the first/main screen of the application.

The value of `home` is:

Scaffold

So:

MaterialApp

    ↓

 home

    ↓

Scaffold

---

# 11. `Scaffold`

Scaffold(

A `Scaffold` provides the basic structure of a Material Design screen.

Think of it as a screen framework:

Scaffold

 ├── AppBar

 ├── Body

 ├── Drawer

 ├── FloatingActionButton

 └── BottomNavigationBar

Your code uses:

- `appBar`
- `body`

---

# 12. `appBar`

appBar: AppBar(

`appBar` is a property of `Scaffold`.

You are telling the `Scaffold`:

> Put an AppBar at the top of this screen.

The value is:

AppBar(...)

---

# 13. `AppBar`

AppBar(

`AppBar` creates the top application bar.

For example:

┌─────────────────────────┐

│       Nivida App        │

├─────────────────────────┤

│                         │

│                         │

│      Hello Flutter!     │

│                         │

└─────────────────────────┘

---

# 14. `title`

title: const Text('Nivida App'),

`title` is a property of `AppBar`.

You're telling the `AppBar`:

> My title should be this Text widget.

---

# 15. `Text`

Text('Nivida App')

`Text` displays text on the screen.

The text is:

Nivida App

So:

AppBar

   ↓

title

   ↓

Text

   ↓

"Nivida App"

---

# 16. `body`

body: const Center(

`body` is the main content area of the `Scaffold`.

Everything below the `AppBar` can go inside the body.

Here you're saying:

> Put a `Center` widget in the body.

---

# 17. `Center`

Center(

`Center` places its child in the center of the available space.

Conceptually:

┌─────────────────────┐

│                     │

│                     │

│    Hello Flutter!   │

│                     │

│                     │

└─────────────────────┘

---

# 18. `child`

child: Text('Hello Flutter!'),

`Center` accepts **one child**.

So:

Center

   ↓

child

   ↓

Text

Remember this difference:

child     → one widget

children  → multiple widgets

---

# 19. The final widget tree

Your complete application structure is:
```

main()

 │

 └── runApp()

      │

      └── MyApp

           │

           └── MaterialApp

                │

                └── home

                     │

                     └── Scaffold

                          │

                          ├── appBar

                          │    │

                          │    └── AppBar

                          │         │

                          │         └── title

                          │              │

                          │              └── Text("Nivida App")

                          │

                          └── body

                               │

                               └── Center

                                    │

                                    └── child

                                         │

                                         └── Text("Hello Flutter!")

```
## The most important thing to understand

Flutter UI is basically a **tree of widgets inside other widgets**.

You can read your code as a sentence:

> Start the app with `MyApp`. `MyApp` builds a `MaterialApp`. The `MaterialApp` has a home screen called `Scaffold`. The `Scaffold` has an `AppBar` with the title "Nivida App" and a body containing a `Center`, which contains the text "Hello Flutter!"

Once you understand this **parent → property → child widget** structure, `Row`, `Column`, forms, buttons, and most basic Flutter UI become much easier to understand.