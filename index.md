# My philosophy on coding

There are different ways to approach coding, and the situations determine which approach makes more sense. In some
cases you need to optimise for speed and memory usage, in other cases, shaving off one extra second doesn't give you
much because the entire system takes a few minutes to complete its computation, or network calls are the actual
bottlenecks. As more data gets processed, more memory and storage is used, and we must choose between scaling up,
reducing memory usage, or scaling out horizontally. In a professional setting, developer time is probably the most
precious commodity. If the developers are too busy debugging code and re-reading super convoluted, tightly coupled,
somewhat cryptic and unreadable code, then time is being lost for the more appealing feature work.

During my 10+ years as a software engineer I have increasingly found myself putting more value in making code easy to
debug, and harder to introduce bugs. When I first learned programming, I did what everyone does the first time they
learn something new: practice all the time. I wrote lots of code, created parent/child classes and overloaded methods
everywhere. It wasn't long before I came crashing back down and learned the hard way how reading old code is not easy,
that comments always fall behind, missing logs makes debugging much harder, why composition is better than inheritance
and why it's easy to miss unit test cases when your classes being tested are way too big. I have also seen myself over
the years write smaller and smaller methods, reaching to the point where I now have an aversion to nested indentations
in a method. The more my method looks like a flat list of instructions, the easier it is for me to know, at a high level,
what my method is doing and why.

That is the context behind my philosophy of coding. My aim is not to write the most optimal piece of assembly code
ever, but to make coding, and even debugging, more fun and easy for me and my future team members. If you see yourself
in the same boat, then read on. Hopefully you'll find my approaches to coding can also be applied to yours.

# Defensive coding

What is our number one priority when writing code? Maybe sometimes it's to solve the problem at hand using the best
algorithm possible. But if there is one thing we can be certain about in every project, it is that there will always
be errors we need to debug. Debugging takes _a lot_ of time and slows down every project. The longer I spent working
on a project or maintaining a service, the more I wished the time spent debugging did not take so long. Band-aid fixes
were not enough. True, it would fix the particular problem I was debugging, but it did not guarantee I would not run into
a similar problem again. Writing clean code can reduce bugs, but having the mindset that you want to solve the class of
problems behind the root cause of the issue, will also move you faster towards a stable system.

I have had many "healthy" discussions about writing optimal code. Some people were horrified that I chose to add more
method calls, or inject more classes. But for the situations that I was usually writing code for, choosing safety over
speed simply made more sense to me. By taking this approach to coding, I try to write code that makes it difficult to
introduce bugs, and easy to discover root causes of errors. There are many ways to achieve this

- Keeping class sizes small (single responsibility) then it is easier to reason it is doing the right thing, and it is
  easier to know if you have missed a test case in your unit test
- Using descriptive names to reduce a reliance on comments and making it easier to understand the logic behind your code
- Using interfaces and encapsulation to write logic at a higher level, and leave the low level details in their own
  (single responsible) classes and methods
- Writing unit tests that are self-contained and have no effect on other tests. Once they are testing a particular use
  case, they must always test that same use case
- Write lots of logs
- And many more...

Anyone can write code that completes a task. But it's the best of us that can write code that makes maintenance and
extension easy and fun for our future colleagues that will also work on the same code base.

# Topics

1. [Single Responsibility](single-responsibility.md)
