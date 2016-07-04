---
layout: post
title:  "Java 8 Map Streams - Part 1"
date:   2016-06-04 16:00:00
categories: java8, streams
---

Java 8 is a real relief for your Java code: it makes it less noisy with a good use of Lambdas and give you the opportunity to go functional style with the Stream API. I love it!

Lately, I was trying to make a piece of code more consise and readable by using Java 8 syntactic sugar. The code looked more or less like this one:

```java
  Map<String, Integer> months = new HashMap<>();
  months.put("January", 1);
  months.put("February", 2);
  // and so on...
  months.put("December", 12);

  for (Entry<String, Integer> entry : months.entrySet()) {
    String monthName = entry.getKey();
    int monthNum = entry.getValue();
    if (monthNum % 2 == 0) {
      System.out.printf("%02d -> %s%n", monthNum, monthName);
    }
  }
```

Don't bother trying to understand what this code is doing, this is not important.

Well, no doubt about it, the Stream API is a good candidate to replace this plain old foreach loop. Let's try it:

```java
  Map<String, Integer> months = new HashMap<>();
  months.put("January", 1);
  months.put("February", 2);
  // and so on...
  months.put("December", 12);

  months.entrySet().stream()
    .filter(entry -> entry.getValue() % 2 == 0)
    .forEach(entry -> System.out.printf("%02d -> %s%n", entry.getValue(), entry.getKey()));
```

This is a lot nicer, don't you think? this is much more readable than the old one...

Well, not that much! Let's face it: this code is way more cryptic than the first one. Obviously this entry stuff gets in the way. Once your data appears as Map entries, you're stuck with the obscur `getKey()` and `getValue()` methods. What is the key? what is the value? what does it mean?

We can do better! Let's rollback this change and take a minute to think about it.

If we were using Groovy syntax, this code would look like this:

```groovy
  def months = [
      'January': 1,
      'February': 2,
      // and so on...
      'December': 12
    ]

  months
    .findAll { monthName, monthNum -> monthNum % 2 == 0 }
    .each { monthName, monthNum -> println "${monthNum} -> ${monthName}" }
```

This looks nice. The key and the value are given an explicit name, which makes the code easy to understand. I like it!

Back to Java: let's do it!

```java
  Map<String, Integer> months = new HashMap<>();
  months.put("January", 1);
  months.put("February", 2);
  // and so on...
  months.put("December", 12);

  months.entrySet().stream()
    .filter((monthName, monthNum) -> monthNum % 2 == 0)
    .forEach((monthName, monthNum) -> System.out.printf("%02d -> %s%n", monthNum, monthName));
```

Done! Commit! Push in production! :P

Wait a minute. The syntax is correct. However, this code doesn't compile... :(

Let's take a closer look to the `java.util.stream.Stream` interface. And more precisely to the `filter()` and `forEach()` methods.

```java
  interface Stream<T> {
    Stream<T> filter(Predicate<? super T> predicate);
    void forEach(Consumer<? super T> action);
  }
```

The Stream API clearly doesn't fit our needs here. It only applies to single values, not key-value pairs.

As far as I know, Java 8 doesn't provide any kind of specific key-value Stream. What would be the interface of such Stream?

```java
  interface BiStream<T1, T2> {
    BiStream<T1, T2> filter(BiPredicate<T1, T2> predicate);
    void forEach(BiConsumer<T1, T2> action);
  }
```

That was easy! Take the Stream interface, prepend every functional interface name with `Bi`, and you're done! Almost all functional interfaces provided by the JDK have a `Bi` equivalent. By the way, why not naming this Stream a BiStream?

Obviously we're not done, but you see where I'm going with this.

Any thoughts about this? What do you think? Would it help you to have a BiStream in your toolbox?
