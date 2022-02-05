---
layout: post
author: Rizwan Minhas
title: Scala Concurrency - async, sync, blocking and non-blocking
excerpt: Scala concurrency async, sync, blocking and non-blocking. 
---

Today I am going to `async`, `sync`, `blocking` and `non-blocking`.

Just because something is `async` doesn't necessarily mean it is also `non-blocking`

Let's take a code example:

```scala
    import scala.concurrent.ExecutionContext.Implicits.global
    import scala.concurrent.duration.DurationInt
    import scala.concurrent.{Await, Future}

    val start = System.currentTimeMillis()

    // An blocking async function that returns a future but puts the thread to sleep for 3 seconds
    def asyncFunction(value: Int): Future[Int] = Future {
        Thread.sleep(3000)
        value
    }

    val futureOfSeqOfInt: Future[Seq[Unit]] = Future.sequence(
        (1 to 100)
        .map(id =>
            asyncFunction(id).map(_ => println(s"$id took ${System.currentTimeMillis() - start} milliseconds."))
        )
    )

    Await.result(futureOfSeqOfInt, 25.seconds) // wait for the result for at most 25 seconds 
    println("done")
```

If you run this code you will notice that it will wait for 3 seconds and then print the statement `{id} took n milliseconds.` in batches where each batch size will be equal to the number of threads on your machine. In my case, on a 8 core and 16 thread machine it waits for 3 seconds then it prints 16 statesments and then it repeats the process until all 100 statements are printed. The entire loop takes about `21100 milliseconds`. 

The reason of this behavior is the threads are explicity put to sleep for 3 seconds therefore they can't process other things during that time.

Now let's see how `async` and `non-blocking` code can improve the performance of this same problem:

```scala
    import java.util.concurrent.{Executors, TimeUnit}

    val start = System.currentTimeMillis()
    val oneThreadScheduleExecutor = Executors.newScheduledThreadPool(1)

    def nonBlockingTask(id: Int): Runnable = () => {
        println(s"${Thread.currentThread().getName()} start-$id")
        val endTask: Runnable = () => {
        println(s"${Thread.currentThread().getName()} end-$id | took = ${System.currentTimeMillis() - start}")
        }
        oneThreadScheduleExecutor.schedule(endTask, 3, TimeUnit.SECONDS)
    }

    val result = (1 to 100).map(n => nonBlockingTask(n).run())

    oneThreadScheduleExecutor.shutdown()
```

In the above code instead of sleeping for 3 seconds I am scheduling the task after 3 seconds, this code will complete all the 100 tasks in about 3 seconds. 
