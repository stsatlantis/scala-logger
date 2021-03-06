# Scala Logger [![Build Status](https://travis-ci.org/emartech/scala-logger.svg?branch=master)](https://travis-ci.org/emartech/scala-logger) [![Maven Central](https://img.shields.io/maven-central/v/com.emarsys/scala-logger_2.12.svg?label=Maven%20Central)](https://search.maven.org/search?q=g:%22com.emarsys%22%20AND%20a:%22scala-logger_2.12%22)

## Usage

Add to sbt:

```sbtshell
libraryDependencies += "com.emarsys" %% "scala-logger" % "0.3.1"
```

### Examples

Basic effectful logging:

```scala
import com.emarsys.logger.syntax._

val context = LoggingContext("job1")

log[IO].info("My first log!")(context).unsafeRunSync()
```

Passing context implicitly:

```scala
import com.emarsys.logger.syntax._

implicit val context: LoggingContext = LoggingContext("job1")

log[IO].info("Implicit context!").unsafeRunSync()
```

Adding contextual information:

```scala
import com.emarsys.logger.syntax._

implicit val context: LoggingContext =
  LoggingContext("job1") <>
    "id" -> 1 <>
    "customer" -> "Joe"

log[IO].info("Contextual information!").unsafeRunSync()
```

Passing context as typeclass

```scala
import com.emarsys.logger.{Logging, Context}
import com.emarsys.logger.syntax._

def work[F[_]: Logging: Context](): F[Unit] = {
  log[F].info("Typeclasses!")
}
```

Adding contextual information when using typeclasses

```scala
import com.emarsys.logger.{Logging, Context}
import com.emarsys.logger.syntax._

def work[F[_]: Logging: Context](): F[Unit] = {
  extendContext("id" -> 1, "job" -> "job01") {
    log[F].info("Typeclasses!")
  }
}
```

Providing implementation for a tagless final algebra with logging

```scala
import com.emarsys.logger.Logged
import com.emarsys.logger.syntax._
import cats.syntax.applicative._

trait Clock[F[_]] {
  def now(): F[Long]
}

class SystemClock extends Clock[Logged[IO, ?]] {
  override def now(): Logged[IO, Long] = withContext { implicit ctx =>
      for {
        time <- IO { System.currentTimeMillis() }
        _ <- log[IO].info(s"Current time: $currentTime")
      } yield time
  }
}

```
