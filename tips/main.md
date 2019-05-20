# Tips & common gotchas

## Logging

Firstly, I'd strongly recommend using [scala-logging](https://github.com/lightbend/scala-logging),
a very fast, lazy, and convenient slf4j wrapper, and [logback](https://logback.qos.ch/), which
provides very easy log configuration as an XML file. Have your class extend `StrictLogging`, add a
little logback config, and you're good to go. These snippets assume slf4j, at least, but generally
apply to other logging frameworks too.

### Logging exceptions
This one is a simple failure to make full use of logging features.

🚫**Don't:**

```scala
val e = new RuntimeException("foo")
logger.error(s"Oh no, $e")
logger.error(s"Oh no, ${e.getClass.getSimpleName): ${e.getMessage}")
logger.error(s"Oh no, ${e.getClass.getSimpleName}: ${e.getMessage}: ${e.getStackTrace)")
```

✅**Do:**

```scala
logger.error("Oh no!", e)
```

🔍**Explanation:**

These are all examples of reinventing the wheel by trying to extract information out of an
exception at the point of writing the log line, instead of making use of the logging features and
simply passing the exception into the log statement along with a message, allowing it to be logged
usefully and consistently (and configurably).

## Futures

When using Futures, it's important to design it from the ground up so that the futures permeate
your stack, allowing eveything to be non-blocking, and futures are only awaited at the very top.
If you make a call which returns a Future, you should also return the Future, having transformed
it as necessary.

There is a lot to know about Futures, so you should seek tutorials in how to use Futures if in
doubt, but here are some common gotchas:

### Don't forget the future can fail

```scala
def makeToast(): Future[Toast] = Future {
  if (math.random() < 0.5) throw IOException()
  new Toast()
}

val toast = makeToast()
toast foreach serveToast  // But where's my toast half the time?
```
You should always be aware that your futures may contain failures, and at some point you'll want to
deal with them, even if it's just to log them. You should actively decide whether you want to leave
failed futures as they are and propagate the failure up further or recover them with some other
success value when you're dealing with them, and the right decision depends on context.

### Accidentally discarding nested futures AKA "oh no, I should have flatMapped!"

Make sure you're careful about knowing what types you're dealing with inside your futures,
especially when you have nested futures, as you can cause confusing issues by accidentally
discarding nested futures. Consider the following:

```scala
def writeToDatabase(s: String): Future[Unit] = Future.successful(())

val twoWrites: Future[Unit] = writeToDatabase("foo") map { _ => writeToDatabase("bar") }
```

So, in the above, twoWrites is a Future[Unit] and will return when we're done writing our two
records, right?

Wrong: it's a Future[Future[Unit]] and it'll return as soon as the outer future is finished; the
inner future has likely been forgotten, along with any errors it had, and any callbacks on the
Future[Future[Unit]] will proceed sooner than intended! This can be obscured by the type, because
`Future[Any] <: Future[Unit]`.

In the above scenario, the intent was to flatMap:

```scala
def writeToDatabase(s: String): Future[Unit] = Future.successful(())

val twoWrites: Future[Unit] = writeToDatabase("foo") flatMap { _ => writeToDatabase("bar") }
```
Now the future will complete only when the inner future has also completed, and will contain any
failures from that inner future as well (assuming the first completed ok).
