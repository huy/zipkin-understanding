# Programming model

Zipkin is based on twitter finagle runtime library. Finagle's foundation is non blocking asynchronous programming model with 3 classe

* `Future`
* `Service`
* `Filter`

**Future**

Future is abstract class that represent result of an asynchronous operation. There are range of Future's combinators defined to 
express different sematic of computation's workflow. 

* `flatmap` : combine 2 Futures to form sequential computation
* `collect` : combine list Futures to form  scratter-garther (i.e. fan-out or fork-join) parallel computation.
* `select` : combine list Futures to form parallel computation until one succeeds.

The Future is implemented using Promise see detail in https://github.com/twitter/util

* `util-core/src/main/scala/com/twitter/util/Future.scala`
* `util-core/src/main/scala/com/twitter/util/Promise.scala`
