---
title:  "Useful Java libs: AssertJ"
published: true
tags: java lib
---

This article will be a short one. Today I was trying to write a test case which I had to compare 2 maps, one expected map against an output map.

I could use **[Hamcrest][1]** to accomplish the task but that would involve importing another lib into the project. I noticed **[Spring Boot][2]** imports an lib called **AssertJ** by default. Following the lib documentation, I had a happy surprise seeing how easy and natural is to write test cases with **AssertJ** syntax.

The example below contains how to test the map and some additional examples about how to use **AssertJ**. The best thing is that different from **Hamcrest** you don't need to know the matchers beforehand. The autocomplete feature of your favorite IDE will do that job for you.

{% gist hugomarques/a6d05f7154f464bb047c4803678a7781 AssertJMapTest.java %}

For more info see the [official documentation][3]. If you are interested how to accomplish the same with **Hamcrest** I found a good example at [Stack Overflow][4].

Have fun and keep coding!

[1]: http://hamcrest.org/JavaHamcrest/
[2]: https://projects.spring.io/spring-boot/
[3]: http://joel-costigliola.github.io/assertj/assertj-core-quick-start.html
[4]: http://stackoverflow.com/questions/2509293/map-equality-using-hamcrest

