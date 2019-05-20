# Scala style guide

Supplements the [official scala guide](https://docs.scala-lang.org/style/).

## Import ordering

Group imports according to whether they're stdlib, third party, or the local project.
Sort alphabetically within the three blocks:

```scala
<stdlib>
__blank line__
<library imports>
__blank line__
<imports from local project>
```

📎**Example:**

```scala
import java.io.File
import scala.concurrent.ExecutionContext

import akka.actor.{Actor, ActorSystem}
import akka.stream._
import com.amazonaws.services.s3.AmazonS3

import com.example.myproject.Config
```

🔍**Explanation:**

This method of sorting imports provides very easy reference of what you're importing and where it
comes from, and makes it very easy to see whether whether it's a third party dependency, standard
library, or your own type that you're importing. Alphabetising makes it easy to find imports in
the blocks and see duplicates etc. This pattern is well-recognised not only in scala, but also in
many other languages.

The reason I've included this section is that it's somewhat different to what Intellij provides
out of the box, the default being quite a strange set of groupings. You can override this in
intellij with XML config a bit like the following (or you can configure it using the GUI config
in the IDE).

```xml
<option name="importLayout">
  <array>
    <option value="java"/>
    <option value="scala"/>
    <option value="_______ blank line _______"/>
    <option value="all other imports"/>
    <option value="_______ blank line _______"/>
    <option value="com.example"/>
  </array>
</option>
```

## Indentation and bracket matching

The official scala style guide is very inconsistent in its examples about how to do this and
doesn't seem to give it much thought. As a general rule:

* **Always** put opening brackets / braces at the end of the line they're opening for
* **Always** put closing brackets / braces of multi-line statements on a new line,
  do not leave them dangling at the end
* **Always** ensure closing brackets / braces line up with the line containing the opening bracket
  / brace
* **Always** put arguments on the next line if you're going to be wrapping the argument list across
  multiple lines
* **Always** indent argument lists 2 spaces in from the opening bracket line

These points also apply to other types of wrapping, but are most often done wrong in argument lists
for classes and defs.

🚫**Don't:**

```scala
// Lots of wasted space in the left gutter, brackets don't visibly match
def longArgumentList(argument1: Boolean = true,
                     argument2: String = "long arguments",
                     argument3: Int = 1337) = ???

someMethodCall(argument1,
               argument2,
               argument3,
               argument4)
```

✅**Do:**

```scala
// No wasted space, arguments all line up, spacing is consistent, brackets match
def longArgumentList(
  argument1: Boolean = true,
  argument2: String = "long arguments",
  argument3: Int = 1337
) = ???

someMethodCall(
  argument1,
  argument2,
  argument3,
  argument4
)
```

🚫**Don't:**

```scala
// Some guides tell you over-indent the class argument list so that arguments don't
// seem to line up with methods, but this results in inconsistent indentation
class Foo(
    arg1: String,
    arg2: String,
    arg3: String,
    arg4: String
) extends Bar {
  def method(): Unit = ???
}
```

✅**Do:**

```scala
// Instead, visually separate the blocks with a blank line at the start of the body to avoid
// confusion and ensure everything is consistently 2-space indents
class Foo(
  arg1: String,
  arg2: String,
  arg3: String,
  arg4: String
) extends Bar {

  def method(): Unit = ???
}
```

## Scaladoc style

Use normal javadoc style.

🚫**Don't:**

```scala
/**
  * Asterisks lined up on the right
  */
def foo(): Unit = ???
```

✅**Do:**

```scala
/**
 * Asterisks lined up on the left
 */
def foo(): Unit = ???
```

🔍**Explanation:**

The explanation for this one is simply that the official styleguide suggesting the different lining
up has never really been accepted by the community; you'll likely find a lot more scala using
javadoc-style asterisk alignment. The documentation renders correctly no matter which you choose.

If you use IntelliJ, you can tell it to do this with:

```xml
<option name="USE_SCALADOC2_FORMATTING" value="false"/>
```

## Resources
### IntellJ
I don't personally use IntelliJ, I write all my scala in vim. IntelliJ is by far the most used
editor for scala, though, so I have acquired an IntelliJ config which applies a lot of these
rules: [IntelliJ Config](../resources/intellij-style.xml). Note that the import ordering will need
to be edited to set the third block to your own project rather than to `com.example`.
