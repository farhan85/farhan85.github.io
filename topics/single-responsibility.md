# Single responsibility

If I had to pick a design principle to be ranked the number one most important, it would be _Single Responsibility_.
Officially it means a module should be responsible for one, and only one, action. This applies to classes and methods.
Especially methods. If a method has only one responsibility, then it's name will accurately describe what it is used for
and you now have a self-commenting method.

From my personal experience, the two biggest benefits of Single Responsibility are 1) It's easier to reason about your
class/method, and 2) It's easier to write unit tests. If your class/method truly does only one thing, then it should be
easy to read it and know all possible code paths it can take. If the class is small, then its unit test class should
have only 2 or 3 test methods. The more methods you have to put in your unit test class, the harder it will be to notice
if there are any missing test cases.

Suggestions on adhering to the Single Responsibility principle:

- Each Class should be responsible for only one piece of work. To help enforce this, each class should have only one
  public method.
- If you need a class to perform multiple logical steps, use composition to inject other objects (that each have their
  own single responsibility) and delegate work to them.
- Classes should be small. Try to make classes small enough to fit on the screen so that scrolling is not required.
- If you're writing nested if statements, consider moving the inner logic to another helper method or class. Evey time
  you have a nested if statement or loop, you are switching contexts. It's easier to understand code when reading a
  method (months after you've written it) if it only has to deal with one context.

One last suggestion, which does not technically fall under Single Responsibility, is to make the top-level public
method not contain any loops. Make it a "list of instructions" (do this, then do that...) and move any loops to a
private helper method. When you need to read this class later in the future, the top level method should be easier to
understand what the class is doing (without needing to read the exact/low-level details yet).
