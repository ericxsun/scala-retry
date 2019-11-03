# scala-retry [![Build Status](https://travis-ci.org/takezoe/scala-retry.svg?branch=master)](https://travis-ci.org/takezoe/scala-retry) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.takezoe/scala-retry_2.12/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.takezoe/scala-retry_2.12)

Offers simple retry functionality for Scala.

```scala
libraryDependencies += "com.github.takezoe" %% "scala-retry" % "0.0.3"
```

## Retry synchronously

`retryBlocking` runs and retries a given block on the current thread. If the block is successful, it returns a value. Otherwise, it throws an exception. Note that the current thread is blocked during retrying. If you don't want to block the current thread during retrying, use `retryAsync` instead.

```scala
import com.github.takezoe.retry._
import scala.concurrent.duration._

implicit val config = RetryConfig(
  maxAttempts = 3, 
  retryDuration = 1.second, 
  backOff = ExponentialBackOff, // default is FixedBackOff
  jitter = 1.second // default is no jitter
)

val result: String = retryBlocking {
  // something to retry
  "Hello World!"
}
```

## Retry Future

`retryFuture` takes `Future` (a block which generates `Future`, more precisely) instead of a block. It works as same as `retryAsync`, but it requires `ExecutionContext` additionally.

```scala
import com.github.takezoe.retry._
import scala.concurrent.duration._
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

implicit val rm = new RetryManager()
implicit val config = RetryConfig(
  maxAttempts = 3, 
  retryDuration = 1.second, 
  backOff = ExponentialBackOff
)

val future: Future[String] = retryFuture {
  Future {
    // something to retry
    "Hello World!"
  }
}

