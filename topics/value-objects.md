# Value objects

Consider the following two method calls:

```java
createConfig(configName, featureName, upperLimit, lowerLimit)

createConfig(featureName, configName, lowerLimit, upperLimit)
```

Which one is the correct invocation? If you wrote both method calls in your code, when will you know which one is wrong?
Maybe you have a good unit or integration test that will tell you one of them is wrong? Or maybe you won't know until
the application is running and either crashes, or even worse, produces wrong results. That is worse because nothing is
telling you something is wrong until someone actually looks at the output themselves.

There is a better way to catch this type of error quickly. The quickest way to catch errors, is to have the compiler
tell you that something is wrong. That is why spelling errors are never deployed to prod in a strongly typed program.
To have the compiler help you in this situation, create _value objects_. Value objects are wrappers around primitive
types, which allows us to write some basic validation in the constructor and also prevents us from giving the wrong
values:

```java
public class ConfigName {
    public class ConfigName(String name) {...}
}

public class FeatureName {
    public class FeatureName(String name) {...}
}

public class LowerLimit {
    public class LowerLimit(int limit) {...}
}

public class UpperLimit {
    public class UpperLimit(int limit) {...}
}
```

Now, when we define the method that needs all four variables, we can guarantee the ordering:

```java
public Config createConfig(ConfigName configName, FeatureName featureName, LowerLimit lowerLimit, UpperLimit upperLimit) {...}
```

And now the compiler will tell us whenever we've passed in the variables in the wrong order. This will also help
guarantee that if any method needs a `UpperLimit` object, it won't be given an incorrect integer by mistake.
