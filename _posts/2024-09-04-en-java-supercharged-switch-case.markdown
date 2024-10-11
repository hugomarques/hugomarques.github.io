---
title: "🇺🇸 Java: Supercharged Java 17 `Switch Case`! 🚀"
published: true
description: Small post about Java 17/21 Switch Case new features
tags: #java #programming #beginners 
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-09-04 23:37 +0000
---

> Disclaimer: The original post is available in PT-BR https://dev.to/hugaomarques/o-novo-switch-case-no-java-17-e-21-3ffg

The `switch case` in Java 17 and 21 brought several improvements that make Java programming easier. From compile-time errors to `yield`, there's a lot of cool stuff to explore! Let's check out the main updates! 🎉

### 1. Forget the `default`!

Now the compiler will notify you if you forget a case, making your code safer. If all possible values are covered, you can skip the `default`. The best part? Compile-time errors ensure no scenario is left out! 🎯

Let's look at an example showing a compile-time error when a value is added to an enum but not covered in the `switch case`.

```java
enum Shape { CIRCLE, SQUARE }

// Later we add TRIANGLE to the enum
enum Shape { CIRCLE, SQUARE, TRIANGLE }

Shape shape = ...;
switch (shape) {
    case CIRCLE -> System.out.println("It's a circle!");
    case SQUARE -> System.out.println("It's a square!");
}
```

Here, the compiler will throw an error saying that the value `TRIANGLE` has not been handled in the `switch`. This happens because all enum values need to be covered in the `switch case`—ensuring no scenario is left out and keeping the code more robust and secure! Without the `default`, you can be sure all cases are handled. ✨

Compile-time error:
```
Error: The switch statement does not cover all possible values of the enum: TRIANGLE
```

### 2. Yield!

Now the `switch` can return values with `yield`, making the code cleaner and more expressive. Forget about confusing returns and enjoy the simplicity. The result? Cleaner and more direct code! 🔄

### Example with `yield`

```java
String message = switch (day) {
    case MONDAY -> {
        System.out.println("Starting the week!");
        yield "Back to work!";
    }
    case FRIDAY -> {
        System.out.println("Weekend is coming!");
        yield "Almost there!";
    }
    case SATURDAY, SUNDAY -> "Enjoy the weekend!";
};
```

Here, `yield` allows us to return values directly within the switch without complications. 🚀

### 3. Support for Enums and Sealed Types

Support for `enums` and `sealed types` in `switch case` has been improved! Now, `enums` are handled more fluidly, and with `sealed types`, we can ensure all subtypes are covered in the switch, making the code more predictable and secure. 🚀

### Example with Sealed Types

```java
sealed interface Shape permits Circle, Square {
    // Can add common methods or properties
}

final class Circle implements Shape {
    // Implementation of Circle class
}

final class Square implements Shape {
    // Implementation of Square class
}

Shape shape = ...;
switch (shape) {
    case Circle c -> System.out.println("It's a circle with radius: " + c.getRadius());
    case Square s -> System.out.println("It's a square with side: " + s.getSide());
}
```

In this example, `Shape` is a `sealed interface`, and we ensure that all possible subclasses of `Shape` (in this case, `Circle` and `Square`) are covered in the switch case. The compiler helps ensure that no case is missed, increasing the security of the code. 🔒

### 4. Type Deconstruction with Pattern Matching

And it doesn't stop there! We also have **type deconstruction** in the `switch case`, which allows us to perform `pattern matching` directly on objects. This makes the code more powerful and flexible.

### Example with Pattern Matching

```java
switch (obj) {
    case Point(int x, int y) -> System.out.println("Point with coordinates: (" + x + ", " + y + ")");
    case Circle c -> System.out.println("Circle with radius: " + c.getRadius());
}
```

The power of pattern matching gives us greater control over types! 🔥

---

### Stay Tuned! 🚨

Coming soon, we'll discuss **Sealed Types** and how, along with the new `switch`, they open a new paradigm in Java programming 👀. So stay tuned for next posts!
