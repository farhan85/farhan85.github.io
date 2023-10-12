# Writing logs

This one is not quite a design standard, but something I had a long time to think throughout my years of debugging.
Some people will write logs using the smallest characters possible, like `x=123`. Others will write logs that use more
natural language. Which is understandable. People read logs when debugging. So it should be easy to read.

Firstly, logs have to be very descriptive. We read logs before diving into our code. Our logs must tell us what happened
and it should be easy to know this just from reading the log entry. Don't use shortened words or abbreviations. It's no
significant extra work for a computer to write it out.

I have had to read many log files with well over tens of thousands of lines. They were so big we had to use `grep` to
search for entries (or use other solutions like AWS CloudWatch Logs). We would write queries to search for log entries,
and to also extract data to gather information about the impact. To handle both cases, my preference is now to write
logs in the following format:

```text
<Description of issue> - <context (key/value pairs)>

// Example
Could not add book to list because of invalid subject. BookId=... BookSubject=... ExpectedSubject=...
```

When logs are written in this format, you simply search on the description, which is a fixed string, and then extract
from the key/value pairs.

Alternatively, log in JSON format, and use an application that parses the log for you, which will make querying easier.

## Logging for easy reading vs logging for easy parsing

Most people write logs that are easy for humans to read. For example:

```java
// Java code
log.error("Could not add book {} to list because its subject {} was not Mystery", book.getId(), book.getSubject());

// Log entry
Could not add book 123 because its subject Fantasy was not Mystery
```

Usually we look at logs are when things go wrong. For the example above, you may find yourself writing the following
grep command:

```shell
> grep 'Could not add book' application.log
```

But what if there were other similar log entries:

```text
Could not add book 123 because its subject Fantasy was not Mystery
Could not add book 456 because its title did not match filter
Could not add book 789 because its subject Fantasy is not allowed in this order
Could not add book 321 because its subject Thriller is not allowed in this order
```

Then the query would need to be updated with an additional filters

```shell
> grep 'Could not add book' application.log | grep 'because its subject' | grep 'was not'
```

And how would we now find all the subjects whose lists were not being populated correctly?

```shell
> grep 'Could not add book' application.log \
| grep 'because its subject' \
| grep 'was not' \
| awk '{print $NF}' \
| sort \
| uniq
```

Notice how the grep queries have to account for the subtle differences in each log entry. If we instead used the
description/context format from above:

```text
Book not added because of invalid subject. BookId=123 BookSubject=Fantasy ExpectedSubject=Mystery
Book not added because title does not match filter. BookId=456 Title=<some book title>
Book not added because subject is not allowed in order. BookId=789 Subject=Fantasy OrderNo=1234
Book not added because subject is not allowed in order. BookId=321 Subject=Thriller OrderNo=5678
```

Then the query would simply be 1) Search for the fixed string, 2) Extract the key/val:

```shell
> grep 'Book not added because of invalid subject' application.log \
| grep -o 'BookSubject=[^ ]\+' \
| cut -d= -f2
```

If you know that your systems will be generating a large volume of logs, and that you have to write search queries to
find the log entry you need, then I recommend you look into storing logs in JSON format. Every log entry now becomes an
event entry, and you can use a much richer log search/analysis toolkit provided by the log aggregation solution of your
choice and you won't need to remember any complex regex expressions to find your log entries.
