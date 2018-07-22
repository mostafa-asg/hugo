---
title: "Practical Cats: Functor and Applicative"
date: 2018-07-22T18:13:49+04:30
draft: false
tags: [Scala, Cats, Functional Programming, Functor, Applicative]
---
In this post, I 'm going to introduce you some useful aspects
of [Cats library](https://typelevel.org/cats/), mostly code snippets and I 'm
not going to delve into any theory or mathematics. Hopefully you will use them in
your daily codes, resulting in much more simpler and readable code.

## map using Functor
You are already familiar with the prominent **map** method in Scala. Informally,
a functor is any **context** with a **map** method. And by context I mean any type
that wraps other type, like List, Option, Future, etc. Here is a simple use of **Functor**,
 for mapping item's of a list:
```
import cats.instances.list._
import cats.Functor

val l = List(1, 2, 3, 4, 5)
Functor[List].map(l)(_ * 2)
// res0: List[Int] = List(2, 4, 6, 8, 10)
```
Not a very useful than built-in list's map. But what about mapping these
list items using built-in map:
```
val l = List(Some(1), None, Some(2), Some(3))
l.map(_.map(_ * 2))
// res1: List[Int] = List(Some(2), None, Some(4), Some(6))
```
Not bad, but there is room for improvements using Functor. You can achieve more readable code
by composing **Functors**:
```
import cats.instances.list._
import cats.instances.option._

Functor[List].compose[Option].map(l)(_ * 2)
// res2: List[Int] = List(Some(2), None, Some(4), Some(6))
```
### Generic Programming
You know that many of the types in Scala have *map* method, so you decided
to write a generic function, that accepts all types with *map* method, how do you
write such function? The first attempt may be is using type constraint, but each type
has its own definition of *map*. Here you can find the definition of map for List and Option:
```
// List's map
// At least until Scala version 2.12 :)
def map[B, That](f: A => B)(implicit bf: CanBuildFrom[List[A], B, That]): That
```
```
// Option's map
def map[B](f: A => B): Option[B]
```
Because method's parameter and return value type differs in each type, you can't
use type constraints, but with the existence of **Functor** it is very easy:
```
def map[F[_],A,B](fa: F[A])(f: A => B)(implicit F: Functor[F]): F[B] = {
    F.map(fa)(f)
}
```
Then as long as there exist an implicit Functor for your type in scope, your method is working as expected:
```
map(Option(1))(_ * 2)
// res3: Option[Int] = Some(2)
```
```
map(List(1, 2, 3))(_ * 3)
// res4: List[Int] = List(3, 6, 9)
```
```
map(Future(4)(_ * 4)
// res5: Future[Int] = Future(16)
```
## Lift a function using Functor
Let's investigate totally different example which **Functor** might be useful.
Imaging you have a simple function like:
```
def len(input: String): Int = input.length
```
and you have a value wrapped in an arbitrary context. Let's say Option:
```
val a = Some("Scala")
```
The question is *how do you call add method with such value?* The very naive solution might be:
```
if (a.isDefined) {
    val result = len(a.get)
}
```
Or better using for comprehensions:
```
val result = for {
    str <- a
} yield len(str)
```
Using Functor's **lift** method you can convert any **A => B** to **F[A] => F[B]**:
```
val newFunc = Functor[Option].lift(len)
// newFunc: Option[String] => Option[Int] = ...
```
now you can easily call your *new function* with your wrapped value:
```
newFunc(Some("Scala"))
// res3: Option[Int] = 5
```
But what about lifting a function that has more than one parameter? meet **Applicative**.

## Applicative
**Applicative** extends **Functor** with an **ap** and **pure** method. The above
sample using Applicative looks like this:
```
def len(input: String): Int = input.length
```
```
val a = Some("Mostafa")
Applicative[Option].ap(Some(len))(a)
// res4: Option[Int] = Some(7)
```
For functions more than one parameter, you can use **ap2**, **ap3**, ...:
```
val add: (Int,Int) => Int = _ + _
```
```
val a = Some(7)
val b = Some(9)
Applicative[Option].ap2(Some(add))(a,b)
// res5: Option[Int] = Some(16)
```
Notice that you should wrap your function in an appropriate context
like **Some(add)** or **Some(len)**. Fortunately you can achieve
the same goal without wrapping your function using equivalent
methods **map**, **map2**, **map3**, ...:
```
Applicative[Option].map2(a,b)(add)
```
Another reason that makes Applicative a real good candidate instead of
using for-comprehensions is **independent effects**. Take a look at this code:
```
for {
  user <- getUserFuture()
  data <- getDataFuture()
} yield Result(user, data)
```
As you can see *getDataFuture* is not depend on *getUserFuture*. We are not care
about **sequencing** data flow, but **Monads** is all about sequence flow. Why would we
need monadic flow, which forces us to view this code as a sequence of steps? I think the
better approach is using Applicative:
```
Applicative[Future].map2(getUserFuture(),getDataFuture())(Result.apply)
```
For deeper understanding please read **Krzysztof Ciesielski**'s
great article [The underrated applicative functor](https://softwaremill.com/applicative-functor/).

### Swapping context easily with traverse/sequence
Imagine you have list of futures (List[Future[Int]), with the power of **Applicative** you can
easily change this structure to Future[List[Int]]. Just call Cat's sequence extension method:
```
import cats.implicit._
List(Future(1), Future(2), Future(3)).sequence
// res6: Future[List[Int]] = ...
```
```
List(Option(1), Option(2)).sequence
// res7: Option[List[Int]] = Some(List(1,2))
```
If one of the items is *None*, the whole result is *None* also:
```
List(Option(1), None, Option(3)).sequence
// res8: Option[List[Int]] = None
```
**traverse** is very like *Future.traverse* with the difference that, not only works with Future,
but can work with every types that have Applicative instance:
```
List(1,2,3).traverse(x => Option(x))
// res9: Option[List[Int]] = Some(List(1,2,3))

List(1,2,3).traverse(x => if (x == 2) None else Option(x))
// res10: Option[List[Int]] = None
```

## Conclusion
[Cats library](https://typelevel.org/cats/) is awesome, and in my opinion
every Scala project can benefit from this library. There are a lot of
buzz words in functional programming paradigm, but thanks
to the [Cats](https://typelevel.org/cats/), there are not scary any more :).
