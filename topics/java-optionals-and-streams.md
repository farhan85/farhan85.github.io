# Using Java Optionals and Streams

Never write if-statements like the following:

```java
if (person.getMiddleName().isPresent()) {
    String value = persion.getMiddleName().get();
}
```

If you're writing if statements, you're using Optionals wrong. Let the Optional class deal with that logic. I like to
think of the code block inside Optionals as "the section of code where you don't have to worry about nulls"

```java
// Out here we need to worry about nulls
...

myOptional.ifPresent(value -> {
    // In here, we never have to worry about value being null
    ...
});
```

So when writing code with optionals, you should say to your self "run this code if we have a value" and let the
Optional class deal with the "if-not-null" logic. Always favour `ifPresent/orElse` over `isPresent/get`.

```java
myOptional.ifPresent(this::doSomethingWithValue);

myValue = myOptional.orElse("defaultValue");

myValue = myOptional.orElseGet(this::getDefaultValue);
```

If you need to use an `if-then-else` construct with an Optional, then use the `.map().orElseGet()` pattern:

```java
myValue = myOptional
    .map(this::processValue)           // If there is a value
    .orElseGet(this::getDefaultValue)  // If there is no value
```

As you become more proficient with Optionals, you will reduce the number of if-statements in your code, and your code
will be more concerned with what you want to do with your variables, rather then "when to run your logic". There will
be less indented code, and it will more resemble a "list of instructions".

Similar logic applies to Java Streams. When you're "inside the stream" you write code for a single element. Let the
Stream class deal with the "for-each-element" logic.

```java
myCollection.stream()
    .map(singleElement -> {
        // In here, we only have to think about what to do with a single element
    })
    .map(transformedSingleElement -> {
        // We're still only thinking about what to do with a single element
    })
    .toList(); // Let the Stream class deal with this logic for you
```

I also prefer making method calls inside each `.map()` method, because then reading the Stream chain will tell me at
a high level what I'm doing. So I can focus on "why" instead of low-level details of "how" until I need to.

```java
myCollection.stream()
    .map(this::extractName)
    .map(this::getIdForName)
    .map(this::getLatestBook)
    .forEach(System.out::println)
```

## Don't overuse them

Optionals are great, but not perfect. Don't go overboard with them. We all have a natural tendency to overuse many new
tools before we learn from our mistakes and readjust. There are times where it's okay not to use Optionals:
- It's okay to use null checks inside the class where you define the nullable variable. That class _knows_ the value can
  be null. This knowledge/logic/responsibility can stay inside this class. The point of Optionals is to let _external_
  callers know they have to deal with the null case to avoid suprise NPEs
- Optionals should never be stored in a variable. It is not part of the definition of your POJO. The Java developers
  decided not to make Optionals serializable because their purpose is not to hold data, but to inform the caller there
  may be missing data. You should only be using Optionals when returning values from a method, to let the caller know
  they may not be getting any data.
- Java does not support an `.ifPresent().orElse()` pattern. In these cases, I think it's fine to use an if-else
  statement. Don't force the Optional to do something it cannot do.
