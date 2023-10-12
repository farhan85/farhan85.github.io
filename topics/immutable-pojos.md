# Immutable POJOs

One consequence of the Single Responsibility rule is that classes which store data must never have methods for
processing their data. This is two different responsibilities. We should always let data be data and define Function
classes to process data. Classes that only store data are called Plain Old Java Objects (POJOs).

If you're creating your own POJOs, there are some rules you should follow to keep the code safe and adhere to the
Single Responsibility principle:

- POJOs must be immutable. Once you create a POJO, you cannot change the data inside it.
- The constructor, or a builder class, should be the only way to set the attributes (i.e. specify attributes only when
  constructing the POJO object)
- If you need to change the values, create a new POJO object with the new values and leave the old object alone (use a
  Function object to create a new modified POJO).
- Never use setters. If you must use setters (e.g. an external dependency forces you to have setters in your classes),
  this is a situation that calls for comments. Explain why you are doing so in the class-level JavaDoc.
- POJOs must have `equals()` and `hashCode()` methods defined so that they are always comparable and hashable
- POJOs must have a `toString()` method defined so they can always be logged and debugging is easier

So what should you do if an object needs to be modified? Create a new one with the new values. However, you should also
be avoiding the `new` operator in your code (and using dependency injection to get the objects you need). So to avoid
using the `new` operator, create converter objects (given an object, and the values to change, produce a new object with
the new values). Yes, you'll be using the `new` operator in your converter classes, but at least these classes are
stateless and have one responsibility. By having a converter object, you will know exactly _when_ your objects can
change. This only happens when the converter is used. Now your unit tests will always know when updates are made.
Updates are made only when your converter objects are used. Thus, the unit tests can explicitly verify when the
converter was used and when it should not be used

In my [LibraryManagementSystem](https://github.com/farhan85/LibraryManagementSystem) demo project, I use the
[Immutables](https://immutables.github.io/) library for creating the POJOs. I prefer this library over Lombok mainly
because they create Optional getters for instance members that can be null. I also like how their generated POJO builders
will not let you build the object until all the required values are given, preventing you from creating incomplete POJOs.

## Why immutable objects and no setters?

The idea of only using immutable objects comes from Functional Programming. In that paradigm, functions are not allowed
to modify any object given to them. This is another way of saying they have no side effects. If their output produces an
object, it is either the one given to them, or an entirely new object. This allows the functions to be _predictable_.
If you call the function with the same input one hundred times, you'll get the same output one hundred times. This
cannot be guaranteed if you allow functions to have side effects, because you're allowing your object to be changed by
someone else without realising it, and the assumptions you make as you write your code could be wrong.

I have personally been burned by this, and learned the hard way how mutable objects makes debugging harder. I once had
to debug my code, wondering why a computation was producing the wrong results. When I was reading my top-level code,
there was no method call that modified the object, and yet somehow it was performing the final computation with wrong
values. I finally found the bug nested within a second-level method call which was resetting an object's attribute back
to zero.

My top-level code assumed it was operating on the same object the entire time, and it's a lot easier to code when you
make this assumption. If you know an object can never change, then it's easier to reason about your code. It's easier
to trust the algorithm you've implemented will work with the same input data every time.

If you allow an object’s values to change, then unit tests can never take care of every use case since an object can be
modified during any step within a function being tested. A function may pass the modified object to other functions, and
you might not have tested the use case for that particular modified object. This becomes much harder to test in a
multi-threaded application. If objects never change, and you only use Factory objects to create new objects, then unit
tests will become more reliable. Your unit tests will always know when a new object is created (by checking if the
Factory object was ever used), and they will always test with the same data (which cannot be modified mid-test). This is
why we can trust our unit tests. Our unit tests are saying "everytime we have these objects, and we call this method,
we _always_ get the same output, and our original objects are still the same".
