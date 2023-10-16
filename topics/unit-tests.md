# Unit tests

Unit (and component/integration) tests do not have to be so compartmentalized like production code. The should be quite
the opposite. Production code needs to be open for extension and are constantly being updated. Unit tests have a very
different behavior. They are almost never updated. Once you have a test method testing a particular use case, you should
aim to never change it, ensuring that logic is always consistent. Also in production code, if you change a shared code
block, it's usually because you want that logic to change for all use cases. This is not true in unit tests. If you
change the logic of a particular test, then no other test should be affected by this change.

> Once a test method is testing something, that logic should never change unless you are modifying that exact method.
> You should never be in a situation where you are suprised a unit test has changed what it is testing.

That means unit tests should be self contained. Minimize the amount of shared code between tests. Reading the entire
test method should be enough to know what are the Given/When/Then steps. You should not have to jump around all over
the place just to understand how a test is set up.

You read production code all the time. But the only time you come back to read a unit test is when something goes wrong.
When this happens, you'll want to go through the method, read how the test is setup, then quickly find the root cause.

One consequence of this rule is that there will be a lot of repeated code in unit tests. However this is actually
desirable. The number one priority of unit tests is that they prevent us making accidental backwards-incompatible
changes. So changing/breaking one unit test, should not cause you to break any other unit test. Changing how one test
behaves should not change how another test behaves. Every test describes their own contract for how the code and system
must behave.

Admittedly, this is another contentious issue that not everyone will agree with. I admit this style of coding does
result in a lot of duplicated code in your tests packages. And all programmers have been conditioned to hate duplicated
code. But in my experience, when I am debugging a failed test and reading the test code for the first time in over a
year, it helps immensely if the test is self-contained. After fixing a unit test, it would make me feel much more
comfortable knowing that modifying one test doesn't break multiple other tests. After all, we do not unit test our unit
tests to protect us from making unintended changes.

To achieve the requirements I mentioned above, I follow these rules:

- Unit tests do not call helper methods (that performs some computation or test) from any other class
- Unit tests do not even use helper methods within the same class. Reading a unit test from top-to-bottom should be
  enough to know what it is doing
  - When reading a unit test method, you should be able to know 1) what is the scenario/setup, 2) what is the method or
    or API being called, 3) what is the expected output and 4) how you are asserting the test is successful
- It's fine to have helper factories for creating POJOs for testing. This is not computation/logic. Make sure to use
  descriptive names explaining what the POJO looks like.
  - If you're finding yourself continuously updating/modifying these POJO generators, then you have not designed them
    correctly - changing code to fix one test should not affect the logic for any other test.
- Use static final (consts) variables as much as possible, and only use variables from within the same class

## Method naming conventions

Test methods should be _very_ descriptive. They should mention the setup, what is being tested, and what is the
expected outcome. Having descriptive methods like this gives these two benefits:
- When reading a test results report, it is easy to see what logic is failing
- When reading through the entire unit test class, it becomes easier to know if you have missed any test cases

And when I say "very descriptive". That's exactly what I mean

```java

// Horrible test name
public void testApplyDiscount() { ... }

// Slightly better, but not quite good enough
public void testApplyDiscountOnItemChangesPrice()

// Much bettter - Tests are basically contracts. They should detail what they do and never change
public void GIVEN_bookOrder_and_10percentDiscount_WHEN_calling_applyDiscount_THEN_return_bookPriceAt90percent() { ... }

// Similar rule applies for integration tests
public void GIVEN_multipleBooksWithSameAuthor_WHEN_calling_AuthorUpdateName_THEN_return_AuthorWithNewName_BooksWithNewAuthorName() { ... }
```

There are other naming conventions used for the setup/teardown methods of a unit test class. Here's an
example of a convention I used in a previous job:

```java
// This is an example of using the TestNG framework

public class ClassNameTest {
    @BeforeClass
    public void init() { ...  }

    @BeforeMethod
    public void setup() { ...  }

    @AfterMethod
    public void cleanup() { ...  }

    @AfterClass
    public void dispose() { ...  }

    @Test
    public void GIVEN_<description of input>_WHEN_calling_<method name>_THEN_<expected output>() { ...  }

    @Test(expected = <ExceptionType>.class)
    public void GIVEN_<description of input>_WHEN_calling_<method name>_THEN_throw_<ExceptionType>() { ...  }

    @Test
    public void GIVEN_<description of input>_WHEN_calling_<method name>_THEN_throw_<ExceptionType>_and_<other expected output>() {
        ...
        assertThrows(ExceptionType.class, testObj::methodName);
        // Other asserts
    }
}
```
