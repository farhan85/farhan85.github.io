# Dealing with Nulls

This is one of those contentious issues where people are highly opinionated and cannot agree with the opposing view.
After years of coding, I have finally chosen my side. _I. Hate. Nulls._ With a passion. I have spent way too much time
debugging NullPointerException (NPE) issues throughout my career. I now firmly believe if we spend the extra effort to
not allow nulls in the system, debugging issues will be much easier.

Of course, there will be many cases where a value is not known. All languages have different ways of handling this
situation. In Java, the fallback method is to pass null. From my experience, NPEs are the cause of many errors, and
littering the code base with if-not-null checks is harder to maintain. People will eventually start forgetting these
checks if they have to be put everywhere. But, you do need null-checks in some places. After all, we need to construct
our data objects at some point, and we may not have been given all the data during input. To handle this, there are few
rules I recommend following:

- Data can only be passed into constructors or builder classes. No setters are allowed.
- Data from the outside world is always passed into POJOs (via their constructor or Builder object). This is the only
  place where we have null-checks and other validation tests. Every other class/method works with data from a POJO
  which is guaranteed to contain valid data
- Do not pass raw data around, only pass POJOs around. POJOs are guaranteed to always have data (their constructors will
  throw exceptions if the POJOs were not constructed correctly)
- Every constructor should check for nulls. I like to use Guava'a `checkNotNull()` in my constructors.
- _Never pass nulls into any other method_. This is my second most important rule. Do not succumb to this rule. Be
  relentless in enforcing this. If a method says it wants something, you must give it a valid object. Constructors will
  check for nulls, no other method should. This is the cause of the majority of NPEs in every project. NPEs are thrown
  when someone tries to use an object they did not realise was a null the whole time.
- In Java, if a POJO contains a null value in an attribute, then it's getter method must return an `Optional` object.
  This informs the caller that they may or may not have a value, and forces them to deal with both use cases. This is
  the main benefit of using Optionals. You won't have surprise NPEs crashing your application because you have code that
  already handles the missing value use case.

## Constructors and Getters examples

There are occasions where a POJO will store nulls and is still a valid object. In this case the getters should return
an Optional object. Force the caller to think about what they should do if the value does not exist. Also, it's fine
for the POJO to have if-not-null checks. They are responsible for their own attributes, and they know that their
attributes can be nulls. But when people write code elsewhere that uses POJOs, they won't remember which attributes can
be null. So we need to remind them, by giving an Optional object. This will let them know they must be wary of the use
case where the value is missing.

```java
import java.util.Optional;

public class Person {
    public String getFirstName() {
        return firstName;
    }

    public Optional<String> getMiddleName() {
        return Optional.ofNullable(middleName);
    }

    public String getLastName() {
        return lastName;
    }
}
```

If you allowed the `getMiddleName` method to return a null, then someone calling this method might not realise there
are times where this value is missing:

```java
// Here, the method may return nulls
// public String getMiddleName() { return middleName; }
person.getMiddleName().toUpperCase();
```

The above will throw an NPE whenever the middle name is null. But if the method did return an optional, you're
telling the caller "hey this could be missing a value" and they will have to consider what to do if it was missing:

```java
// Use a default value
String value = person.getMiddleName().orElse("defaultValue");

// Or only run code if there is a value
person.getMiddleName().ifPresent(this::processMiddleName);
```

## Passing nulls into Constructors

There is another contentious issue that I have had many healthy discussions about. Some people argue that "if
a class is expected to not have values for every attribute, then what's wrong with giving nulls to a constructor?
You're still creating a valid object". My response to this is to ask "but are you creating the object you _expected_
to create?". Bugs are introduced when we write code making the wrong assumptions about the data were working with.
If someone gives you an object that has getter methods, it's natural to think the object must have a value for all
those attributes. Why else would it have those getter methods in the first place?

Let's go through an example of the pitfalls that arises from allowing someone to create an object that looks different
from what the top-level code is saying. Consider the following example of a class and its constructors:

```java
public class MyClass {
    public MyClass(SomeClass object1, AnotherClass object2) {
        this.object1 = checkNotNull(object1);
        this.object2 = object2;
    }

    public MyClass(SomeClass object1) {
        this(object1, null);
    }
}
```

This is not good coding practise. You are forced to remove one of the `checkNotNull` checks in the first
constructor. So now, someone can call that first constructor with the second parameter set to `null`. This is bad
practice and makes is easy for someone to write bad code. We should throw NPEs as early as possible so that an NPE
indicates when someone did not give a valid object.

> If a method or constructor says they want something, you have to give them something. Do not give a null.

Similarly:

> If a method says they are going to give you something, it should always give you something. Never return null.
> When someone calls a method with a return value, they are expecting to receive something.

The correct way to write those constructors is to have different constructors for the different valid use cases:

```java
public class MyClass {
    public MyClass(SomeClass object1, AnotherClass object2) {
        this.object1 = checkNotNull(object1);
        this.object2 = checkNotNull(object2);
    }

    public MyClass(SomeClass object1) {
        this.object1 = checkNotNull(object1);
        this.object2 = null;
    }
}
```

Yes, there is some repeated code, but I argue this makes the code safer to work with and it explicitly states the valid
use cases. It's extra work now, but the benefit is that it reduces all the time spent debugging in the future. I agree
we should aim to remove duplicate code as must as possible, but apply that in business logic code. Constructors serve
a different purpose. They are the entry point to your system. Their responsibility is to create valid objects and should
not allow bad data to enter your system. Put up a good shield around your system, then your inner code will be less
messy and can focus on the business logic.

Now when you see this in your code...

```java
MyClass myClassObj = new MyClass(someClassObj, anotherClassObj);
```

...you will know that when you read the rest of that code block, the MyClass object will always have both values set.
The second parameter will never be null. This is why it's easier to reason about your code. Your assumptions that you
make from what you see in the code will be correct, and also consistent because we don't allow for side effects (as the
object will never change since it is immutable).

If the above example still bothers you, a better alternative to using overloaded constructors is to use the Builder
pattern, which achieves the goal of being more explicit on what type of object you are creating:

```java
MyClass myClassObj = MyClass.builder()
    .withSomeClass(someClassObj)
    .withAnotherClass(anotherClassObj)
    .build();
```

The other problem with allowing nulls being passed into constructors is that the NPE may not get thrown until later on.
The code won't realise it's holding onto a ticking time bomb until it tries to do something with the null object.
Consider the following example:

```java
public class Processor {
    private final Metric metric;
    private final Function<Metric, String> jsonConverter;

    public Processor(Metric metric, Function<Metric, String> jsonConverter) {
        this.metric = metric;
        this.jsonConverter = jsonConverter;
    }

    public void addToQueue(Queue queue) {
        String json = jsonConverter.apply(metric.getValue());
        queue.add(json);
    }
}
```

The above class allows you to pass nulls into its constructor. So now imagine somewhere later in the code, you
accidentally passed in a null:

```java
public class Main {
    public class run() {
        Queue queue;
        Metric metric;
        //
        // do some work to get the queue and metric objects.
        //
        Worker worker = new Worker(queue);
        //
        // do some work
        //
        Processor processor = new Processor(metric, queue);
        //
        // do some work
        //
        worker.processMetrics(processor);
    }
}
```

Can you see where the null was given? Not really. It could be either the Queue or Metric object that are nulls. But we
won't know until the code is deployed and running. One problem here is that our code doesn't _show_ us where you can
potentially run into NullPointerExceptions. Another problem is what will the stack trace say when the NPE is finally
thrown? It'll contain the following section:

```text
java.lang.NullPointerException: null
    at com....MetricJsonConverter.apply(MetricJsonConverter.java:30)
    at com....Worker.processMetrics(Worker.java:20)
    at ...
```

So the application has crashed and we're now going through the logs. What is the stack trace telling us? It's saying
"The MetricJsonConverter could not process this object because it was a null". But what do we really want to know? We
want to know _who_ introduced the null in the first place. If we checked for nulls in every constructor, the NPE stack
trace will then be saying "hey, this other object just gave me a null" and when you begin debugging the issue, you're
already asking the right questions from the start, and you'll start your investigation from the constructor. You're
immediately trying to find out who gave the null to the Processor, instead of reading the code to find out why the
Processor object was holding onto a null for such a long time.

This example shows how we can write code that not just helps the initial developer with debugging, but also helps future
developers in the team when they need to debug similar issues later.
