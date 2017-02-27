# Testing Scala in IntelliJ with scalatest

There are multiple libraries and testing methodologies for Scala,
but in this tutorial, we'll demonstrate one popular option from the scalatest framework
called [FunSuite](http://www.scalatest.org/getting_started_with_fun_suite).
We assume you know [how to build a project in IntelliJ](https://linktotutorial).

## Setup
1. Create an sbt project in IntelliJ.
* Add the scalatest dependency to your build.sbt file:
```
libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.1" % "test"
```
* this will cause sbt to pull down the scalatest library
* If you get a notification "build.sbt was changed", select **auto-import**.
* On the project pane on the left, expand `src` => `main`.
* Right-click on `scala` and select **New** => **Scala class**.
* Call it `CubeCalculator`, change the **Kind** to `object`, and click **OK**.
* Replace the code with the following:
```tut
object CubeCalculator extends App {
  def cube(x: Int) = {
    x * x * x
  }
}
```

## Creating a test
1. On the project pane on the left, expand `src` => `test`.
* Right-click on `scala` and select **New** => **Scala class**.
* Name the class `CubeCalculatorTest` and click **OK**.
* Replace the code with the following:
```
import org.scalatest.FunSuite

class CubeCalculatorTest extends FunSuite {
  test("CubeCalculator.cube") {
    assert(CubeCalculator.cube(3) === 27)
  }
}
```
* In the source code, right-click `CubeCalculatorTest` and select **Run
'CubeCalculatorTest'**.

## Understanding the code
Let's go over this line by line.

* `class CubeCalculatorTest` means we are testing the object `CubeCalculator`
* `extends FunSuite` lets us use functionality of scalatest's FunSuite class
such as the `test` function
* `test` is function that comes from the FunSuite library that collects
results from assertions within the function body.
* `"CubeCalculator.cube"` is a name for the test. You can call it anything but
one convention is "ClassName.methodName".
* `assert` takes a boolean condition and determines whether the test passes or fails.
* `CubeCalculator.cube(3) === 27` checks whether the output of the `cube` function is
indeed 27. The `===` is part of scalatest and provides clean error messages.

## Adding another test case
* Add another `assert` statement after the first one that checks for the cube
of `0`.
* Re-run the test again by right-clicking `CubeCalculatorTest` and selecting
'Run **CubeCalculatorTest**'.

## Conclusion
You've seen one way to test your Scala code. You can learn more about
scalatest's FunSuite on the [official website](http://www.scalatest.org/getting_started_with_fun_suite).
