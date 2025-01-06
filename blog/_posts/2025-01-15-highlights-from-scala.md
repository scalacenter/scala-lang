---
category: blog-detail
post-type: blog
by: Scala Core Team
title: Highlights From Scala - January 2025
---

This year, we’re excited to share key updates across the Scala ecosystem, showcasing advancements in performance, usability, and developer experience.

Here’s what’s new:

- [Scala 3.5](#scala-35) introduces binary integer literals, named tuples, and an experimental syntax for context bounds.
- [Scala.js](#scalajs) unveils an experimental WebAssembly backend.
- [Scala Native](#scala-native) gains multithreading and preliminary 32-bit architecture support.
- [Scala CLI](#scala-cli) now ships with the default Scala distribution, streamlining prototyping, testing, and publishing.
- [sbt](#sbt) introduces pipelined compilation and early support for Scala 3.x in build definitions
- [Metals](#metals) enhances IDE functionality with a new "best effort" compilation mode for smoother workflows.

These updates mark significant progress in making Scala more powerful and accessible. Let’s explore the details!

## Scala 3.5

### Binary integer literals

Integer literals can now start with `0b` or `0B`. They will be interpreted as numbers in base 2.

```scala
assert(0B1 == 1)
assert(0B10 == 2)
assert(0B_1000_0010 == 130)
assert(0B_1000_0010 == 0x82)
```

### Experimental: Named tuples

Scala 3.5 introduces experimental support for named tuples. The feature, hidden behind the `language.experimental.namedTuples` import, allows you to give meaningful names to tuple elements and use those names during constructing, destructuring, and pattern matching.

```scala
import scala.language.experimental.namedTuples

type Point = (x: Int, y: Int)
val point: Point = (x = 1, y = 2)
val is_it_point = (x = 5, y = 3)
val it_is: Point = is_it_point

println(point.x) // prints 1

point match
  case (x = real, y = 0) => println(s"Real number: $real")
  case _ => println("Point doesn't represent a real number")
```

This is an implementation of [SIP-58](https://docs.scala-lang.org/sips/named-tuples.html).

### Experimental: New syntax for givens and context bounds

Another experimental feature introduced in Scala 3.5 is the new syntax for type classes. Some of these improvements are: `Self` type member instead of the type parameter, auxiliary type alias `is` or named context bounds, just to name a few.
The full list of proposed improvements and their examples can be found under [Modularity Improvements](https://scala-lang.org/api/3.5.0/docs/docs/reference/experimental/modularity.html) and [Better Support for Type Classes](https://scala-lang.org/api/3.5.0/docs/docs/reference/experimental/typeclasses.html).
To test the new syntax you would be required to use both the source version `future` and the additional language import `experimental.modularity`. 

```scala
//> using options -source:future -language:experimental.modularity

trait Ord:
  type Self
  extension (x: Self)
    def compareTo(y: Self): Int
    def < (y: Self): Boolean = compareTo(y) < 0
    def > (y: Self): Boolean = compareTo(y) > 0

given intOrd: (Int is Ord) with
  extension (x: Int) def compareTo(y: Int) = if x < y then -1 else if x > y then +1 else 0

def descending[T: Ord as asc]: T is Ord = new:
  extension (x: T) def compareTo(y: T) = asc.compareTo(y)(x)

def maximum[T](xs: List[T])(using T is Ord): T =
  xs.reduceLeft((x, y) => if (x < y) y else x)

val xs = List(1, 2, 3)
val _ = maximum(xs)
val _ = maximum(xs)(using descending)
val _ = maximum(xs)(using descending(using intOrd))
```

This is an implementation of [SIP-64](https://docs.scala-lang.org/sips/sips/typeclasses-syntax.html).

### Other new features

- `var` in refinements ([3.5.0](https://scala-lang.org/blog/2024/08/22/scala-3.5.0-released.html#var-in-refinements))
- Add sources of synthetic classes to source jar ([3.5.1](https://github.com/scala/scala3/releases/tag/3.5.1))
- Add the -Wall option that enables all warnings ([3.5.2](https://github.com/scala/scala3/releases/tag/3.5.2))
- Add origin filter to WConf, DeprecationWarning ([3.5.2](https://github.com/scala/scala3/releases/tag/3.5.2))

## Scala.js

### Experimental: WebAssembly Backend

Starting with this release, Scala.js 1.17.0 ships with an experimental WebAssembly backend.
Under some conditions, you may use it as a drop-in replacement for the usual JavaScript backend.

You can set it up as follows:

```scala
// Emit ES modules with the Wasm backend
scalaJSLinkerConfig := {
  scalaJSLinkerConfig.value
    .withExperimentalUseWebAssembly(true) // use the Wasm backend
    .withModuleKind(ModuleKind.ESModule)  // required by the Wasm backend
},

// Configure Node.js (at least v22) to support the required Wasm features
jsEnv := {
  val config = NodeJSEnv.Config()
    .withArgs(List(
      "--experimental-wasm-exnref", // required
      "--experimental-wasm-imported-strings", // optional (good for performance)
      "--turboshaft-wasm", // optional, but significantly increases stability
    ))
  new NodeJSEnv(config)
},
```

Make sure node -v reports at least v22.0.0. If not, install a newer version.

You are then set up to run and test your codebase with the WebAssembly backend from sbt.

Read more detailed information about the [WebAssembly backend in the docs](https://www.scala-js.org/doc/project/webassembly.html).

Check out the other enhancements in [Scala.js 1.17.0](https://www.scala-js.org/news/2024/09/28/announcing-scalajs-1.17.0/).

## Scala Native

### Multithreading support

Scala Native now allows to use system threads based on `java.lang.Thread` implementation, along with all the necessary primitives to work with concurrency:

- `synchronized` blocks are now using object monitors following partially JVM semantics - these might be less performant when compared to JVM, but are going to achieve improvements in the followup patch releases.
- JVM compliant support for `@volatile` annotations
- Configurable mostly JVM-compliant support for final fields semantics, see [multithreading guide](https://scala-native.org/en/stable/user/lang.html#multithreading)
- Thread safe implementation of most [java.util.concurrent] types and methods, including atomics, thread pools, etc. See list of currently implemented primitives in our [meta-issue on GitHub](https://github.com/scala-native/scala-native/issues/3165). Be aware that our Java Standard Library implementation might still contain thread-unsafe implementations of types not listed in the tracker.
- Support for most of `scala.concurrent` and `scala.collection.concurrent` types.
- Multithreading-aware implementation of all garbage collections.

### Support for 32-bit architectures

We’ve introduced preliminary, experimental support for targeting 32bit architectures, like ARMv7. To allow for representing variable-length integers in C bindings, we’ve introduced 2 new types `scala.scalanative.unsafe.{Size, USize}` which correspond to `ssize_t`, `size_t` in C. During linking these would be transformed into 32 or 64 bits longs integer. Multiple C standard library and POSIX bindings were adapted to use these new types.

### Other new features

- Initial support for source level debugging ([0.5.0](https://scala-native.org/en/stable/changelog/0.5.x/0.5.0.html))
- SIP-51 support: Drop forward binary compatibility of the Scala 2.13 standard library ([0.5.0](https://scala-native.org/en/stable/changelog/0.5.x/0.5.0.html))
- Improved missing symbols reports ([0.5.0](https://scala-native.org/en/stable/changelog/0.5.x/0.5.0.html))
- JVM service providers supports ([0.5.0](https://scala-native.org/en/stable/changelog/0.5.x/0.5.0.html))
- User build settings ([0.5.0](https://scala-native.org/en/stable/changelog/0.5.x/0.5.0.html))

## Scala CLI

### The new Default Runner

[Scala CLI](https://scala-cli.virtuslab.org/) is a popular tool among Scala devs. It allows lightning-fast running, testing, and prototyping of Scala scripts and single-module projects. In 3.5.0, it becomes part of the default Scala distribution. If you install Scala through popular package managers, such as Brew or SDKMAN!, the installed `scala` command allows you to compile, run, test, and even publish your code on Maven Central.  It has out-of-the-box support for `using` directives, toolkits, compilation to JavaScript and native binaries, and other goodies formerly available only if you installed a separate tool.

Given the following file named `biggerThan.scala`

```scala
//> using dep com.lihaoyi::os-lib:0.10.3

@main def run(path: String, size: Int) =
  os.list(os.Path(path, os.pwd))
    .filter: p =>
      os.size(p) > size * 1024
    .foreach(println)
```

We can run `scala biggerThan.scala -- ../my-directory 10`. This will download os-lib and its transitive dependencies, compile the source code, and finally run the compiled program, printing all files in `../my-directory` bigger than 10 kB. Subsequent invocations of the command will use cached bytecode, or if the source code changed, trigger incremental compilation.

To learn the full scope of the new capabilities, read about Scala CLI [commands](https://scala-cli.virtuslab.org/docs/commands/basics/) and [using-directives](https://scala-cli.virtuslab.org/docs/reference/directives) or glance at [the cookbook](https://scala-cli.virtuslab.org/docs/cookbooks/intro).

Merging Scala CLI into the distribution doesn't change the behavior of popular build tools that work with Scala, such as sbt, Mill, Maven, and Gradle. It does, however, allow for maintaining and publishing single-module multi-platform libraries without any build tool.

_**Note**: Installation of a new Scala runner using Coursier requires it to be updated or reinstalled to Coursier 2.1.10 or above. Earlier versions did not support installing native packages, defaulted to the old runner JAR._

### Experimental: JMH options and directives

It is now also possible to control JMH with using directives:

- `//> using jmh` allows to enable JMH for the project, being the equivalent of the `--jmh` option.
- `//> using jmhVersion <version>` allows to set the JMH version to use, being the equivalent of the `--jmh-version` option.

```scala
//> using jmh
//> using jmhVersion 1.37
package bench

import org.openjdk.jmh.annotations.*
import java.util.concurrent.TimeUnit

@BenchmarkMode(Array(Mode.AverageTime))
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 1, time = 100, timeUnit = TimeUnit.MILLISECONDS)
@Measurement(iterations = 10, time = 100, timeUnit = TimeUnit.MILLISECONDS)
@Fork(0)
class Benchmarks {
  @Benchmark
  def foo(): Unit = {
    (1L to 1000L).sum
  }
}
```

Expect more improvements in this area in the future.
Also, do play with it and give us feedback about the current implementation!

### Other new features

- Environment variable help with `--env-help` ([v1.4.0](https://github.com/VirtusLab/scala-cli/releases/tag/v1.4.0))
- Running the REPL with the test scope included ([v1.4.0](https://github.com/VirtusLab/scala-cli/releases/tag/v1.4.0))
- Pass compiler args as an @argument file ([v1.4.1](https://github.com/VirtusLab/scala-cli/releases/tag/v1.4.1))
- Explicitly enable or disable multithreading in Scala Native ([v1.4.1](https://github.com/VirtusLab/scala-cli/releases/tag/v1.4.1))
- Support for auto-completions in fish ([v1.5.0](https://github.com/VirtusLab/scala-cli/releases/tag/v1.5.0))
- Support for launching apps from dependencies without other inputs ([v1.5.0](https://github.com/VirtusLab/scala-cli/releases/tag/v1.5.0))
- Experimental: Support for exporting to a Maven project ([v1.5.0](https://github.com/VirtusLab/scala-cli/releases/tag/v1.5.0))
- Initial support for emitting Wasm with a command line option and a directive ([v1.5.2](https://github.com/VirtusLab/scala-cli/releases/tag/v1.5.2))
- `--source` is now deprecated and scheduled for removal in Scala CLI v1.6.x ([v1.5.2](https://github.com/VirtusLab/scala-cli/releases/tag/v1.5.2))

## sbt

### Pipelined build in Scala 3.5

Scala 3.5.0 supports pipelined compilation. It can be enabled by setting `ThisBuild/usePipelining := true` in sbt build definition. This can result in significant speedups in compilation time for multi-module projects.

You can learn more about how the pipelined compilation works and what benefits you can expect from [the talk by Jamie Thompson](https://www.youtube.com/watch?v=1uuFxEAiPuQ&t=1473s).

### Beta: Scala 3 support for build definitions

In the recent 2.0.0-M2 beta version, you can use Scala 3 for your build definition.

You can try it out by upgrading the sbt version in `project/build.properties`:
```
sbt.version=2.0.0-M2
```

Note that many sbt plugins are not yet available for sbt 2.0.0-M2.
You can help us cross build them to both sbt 1.x and 2.x:
```scala
crossScalaVersions := Seq("2.12.20", "3.3.4")
pluginCrossBuild / sbtVersion := {
  scalaBinaryVersion.value match {
    case "2.12" => "1.5.8"
    case _      => "2.0.0-M2"
  }
},
```

For more details see [Migrating from sbt 1.x](https://www.scala-sbt.org/2.x/docs/en/changes/migrating-from-sbt-1.x.html).

### Other beta features in sbt 2.x

- Bare settings are added to all sub-projects ([2.0.0-M2](https://eed3si9n.com/sbt-2.0.0-beta))
- Local/remote cache system that is Bazel-compatible ([2.0.0-M2](https://eed3si9n.com/sbt-2.0.0-beta))
- `compile` and `test` are both rewritten to be cacheable tasks ([2.0.0-M2](https://eed3si9n.com/sbt-2.0.0-beta))
- `test` changed to incremental test ([2.0.0-M2](https://eed3si9n.com/sbt-2.0.0-beta))
- Project matrix is in-sourced ([2.0.0-M2](https://eed3si9n.com/sbt-2.0.0-beta))
- Extension of the slash syntax to support query of sub-projects ([2.0.0-M2](https://eed3si9n.com/sbt-2.0.0-beta))
- Build Server Protocol improvements: `run` task is non-blocking ([2.0.0-M2](https://eed3si9n.com/sbt-2.0.0-beta))

## Metals

### Best effort compilation

When using Metals, the IDE functions work great on code that compiles. However, once the user starts changing the code, the support drops in quality. This gets worse with a larger number of compilation errors and subsequent modifications. In 3.5.0, we have fixed this problem. We introduced a new mode of compilation called Best Effort Compilation. When enabled, the compiler will output BETASTy files for not-currently-compiling code. It can be used by tooling to provide autocompletion and other IDE features.

To enable the use of BETASTy files in Metals, start the language server with `-Dmetals.enable-best-effort=true` or put that into `metals.serverProperties` setting in VS Code. We plan to enable this feature by default soon after gathering feedback from users.

### Inlay hints

Metals will now use the standard LSP way of showing decorations via Inlay Hints instead of a custom solution. Thanks to this, the decorations behave much more naturally like usual code, you can navigate to definition and get proper hovers. It will also allow all editors to support decorations without implementing the additional extension to the protocol.

![alt text](/resources/img/blog/2025-01-15/hinlay-hint.gif)

### Other new features in Metals

- Java Home improvements ([v1.3.0](https://scalameta.org/metals/blog/2024/04/15/thalium))
- Automatic imports setting ([v1.3.0](https://scalameta.org/metals/blog/2024/04/15/thalium))
- Preferred build server setting ([v1.3.0](https://scalameta.org/metals/blog/2024/04/15/thalium))
- Exhaustive matches for union types ([v1.3.0](https://scalameta.org/metals/blog/2024/04/15/thalium))
- Go to implementation for dependency sources ([v1.3.0](https://scalameta.org/metals/blog/2024/04/15/thalium))
- Completions for implicit classes ([v1.3.0](https://scalameta.org/metals/blog/2024/04/15/thalium))
- Bloop 2 (([v1.4.0](https://scalameta.org/metals/blog/2024/10/24/palladium)))
- Detect custom mains (([v1.4.0](https://scalameta.org/metals/blog/2024/10/24/palladium)))

## How to support Scala?

The Scala Center is the Scala language foundation coordinating Scala governance, community, education, and OSS library/tool development.

It contributes to the core language, open source Scala tooling and libraries, and delivers high-quality education materials. It fosters conversations in the community and coordinates with various parties to unblock and improve the Scala ecosystem.

Joining the Center’s Advisory Board is an effective way to participate in Scala governance, have your voice heard, as well as supporting the Center to achieve its goals.

For more information:

- [Home page](https://scala.epfl.ch/)
- [Joining the Advisory Board](https://scala.epfl.ch/corporate-membership.html)
- [5 Year Impact Report](https://www.scala-lang.org/blog/2024/02/06/scala-center-2024-roadmap.html)
