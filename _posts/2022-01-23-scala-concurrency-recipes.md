---
layout: post
author: Rizwan Minhas
title: Scala Concurrency Recipes
excerpt: Some examples of useful recipes when using concurrency in Scala. 
---

1. [Only wait for the first future response.](#one)
2. [Convert List of futures to future containing List of Success/Failure.](#two)

<a name="one"></a>
## 1. Only wait for the first future response.

```Scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration.DurationInt
import scala.concurrent.{Await, Future}
import scala.util.{Failure, Random, Success}

def futureCall(): Future[Int] = Future {
    val random = Random.nextInt(10) + 1
    val sleepFor = random * 300
    println(s"My answer will be $random, for now I am sleeping for $sleepFor milliseconds...")
    Thread.sleep(sleepFor)
    random
}

val result = Future.firstCompletedOf(List(futureCall(), futureCall()))

result.onComplete({
    case Failure(exp) => println(exp)
    case Success(value) => println(s"First call to complete was: $value")
})

Await.result(result, 3.seconds)
```

**Note:** `firstCompleteOf` only cares about the first successful future to complete, it doesn't care if the other futures fail or succeed.

<a name="two"></a>
## 2. Convert List of futures to future containing List of Success/Failure.

**Poblem**: If a function returns a `Future` of something and if that function can also throw an exception then calling `Future.sequence` on the response of such a function will stop the execution on the first exception. To avoid this problem you can use the following recipe:

```Scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration.DurationInt
import scala.concurrent.{Await, Future}
import scala.util.{Failure, Random, Success}

def mayBe(): Future[Int] = Future {
    val random = Random.nextInt(10)
    if (random < 5) {
        throw new Exception("oops...")
    } else {
        println(s"random: $random")
        random
    }
}

val listOfFutures = List(mayBe(), mayBe(), mayBe(), mayBe())

val listOfFuturesTrys = listOfFutures.map(_.map(Success(_)).recover { case e => Failure(e) })

val result = Future.sequence(listOfFuturesTrys)

result.onComplete(println)

Await.result(result, 2.seconds)
```

In the above example if the first call throws an exception and the rest of the calls return 6, 8 and 7 then it will it will print `Success(List(Failure(java.lang.Exception: oops...), Success(6), Success(8), Success(7)))`. Instead of `listOfFuturesTrys` if I would have used just `listOfFutures` then the sequence would have stopped on the exception.