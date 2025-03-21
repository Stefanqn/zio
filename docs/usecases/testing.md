---
id: usecases_testing
title:  "Testing"
---

**ZIO Test** is a zero dependency testing library that makes it easy to test effectual programs. Begin by adding the required configuration in your SBT settings:

```scala
libraryDependencies ++= Seq(
  "dev.zio" %% "zio-test"     % zioVersion % "test",
  "dev.zio" %% "zio-test-sbt" % zioVersion % "test"
),
testFrameworks := Seq(new TestFramework("zio.test.sbt.ZTestFramework"))
```

From there the fastest way to start writing tests is to extend `DefaultRunnableSpec`, which creates a Spec that is also an executable program you can run from within SBT using `test:run` or by using `test` with the SBT test runner.

```scala
import zio._
import zio.console._
import zio.test._
import zio.test.Assertion._
import zio.test.environment._

import HelloWorld._

object HelloWorld {
  def sayHello: ZIO[Console, Nothing, Unit] =
    console.putStrLn("Hello, World!")
}

object HelloWorldSpec
    extends DefaultRunnableSpec(
      suite("HelloWorldSpec")(
        testM("sayHello correctly displays output") {
          for {
            _      <- sayHello
            output <- TestConsole.output
          } yield assert(output, equalTo(Vector("Hello, World!\n")))
        }
      )
    )
```

In **ZIO Test**, all tests are immutable values and tests are tightly integrated with ZIO, so testing effectual programs is as natural as testing pure ones. In the example above, our test involved the effect of printing to the console but we didn't have to do anything differently in our test because of this other than use `testM` instead of `test`.

**Test Environment**

The library also includes built-in test versions of all the standard ZIO environmental effects (`Clock`, `Console`, `System`, and `Random`), so when we tested our program above the `helloWorld` method didn't actually print a String to the console but instead wrote the String to a buffer that could access for testing purposes. If you ever do need to access the "live" environment just use the `live` method in the `mock` package or specify the live environment in your type signature like `Live[Console]`.

**Property Based Testing**

Support for property based testing is included out of the box through the `check` method and its variants and the `Gen` and `Sample` classes. For example, here is how we could write a property to test that integer addition is associative.

```scala
val associativity =
  check(Gen.anyInt, Gen.anyInt, Gen.anyInt) { (x, y, z) =>
    assert((x + y) + z, equalTo(x + (y + z)))
  }
```

If a property fails, the failure will be automatically shrunk to the smallest failing cases to make it easier for you to diagnose the problem. And shrinking is integrated with the generation of random variables, so you are guaranteed that any shrunk counterexample will meet the conditions of your original generator.

**Results Reporting**

When tests do fail, it is easy to see what went wrong because the test reporter will show you the entire assertion that failed and the specific part of the assertion that failed. To facilitate this, a variety of assertion combinators are included in the `Assertion` class.

**Test Aspects**

Test aspects are powerful tools for modifying behavior of individual tests or even entire suites that you have already written. Convenient syntax `@@` is provided for applying test aspects. So for example you could apply a timeout to a test by using `test @@ timeout(60.seconds)` or only run a test on javascript by using `test @@ jsOnly`. Test aspects are highly composable so you can combine multiple test aspects together or apply them only to certain tests that match a predicate you specify.

**Zero Dependencies**

As a library with zero third party dependencies, this project is available on the JVM, ScalaJS, Dotty, and will be available on Scala Native in the near future. So you can write your tests once and make sure that your code works correctly across all platforms that you support.
