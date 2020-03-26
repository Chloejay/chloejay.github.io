---
layout: post
title:  "Scala pattern - Monad is everywhere!"
date:   2020-03-26 16:40:00 +0800
categories: scala pattern
tags: scala
image: monad2.png
applause: true
short_description: pattern 
--- 

<div markdown="1" id="text">
Since I fall in love for Scala, by its pure functionality and the way it been expressed from both FP and OOP sides, so I can't stop but look this language further. The further leads me to tiny step into space of FP, not the way like lambda, closure, callback etc in Python, JavaScript, but the mathematically it shines with logic, imagination and universal language we are speaking.

#### Why Nomad
The monad is a common concept(cliches) in Scala for abstraction, it is talked so often in FP. Some(folks) elaborate this concept with Category Theory, some(others) called it as a container.Thinking this picture, a multiply gift boxes, look same outside, but varied gifts (surprises) inside.

Below is a funny expr Monad be described.
<blockquote>
The Monad is like a bellows:<br/>
it is empty yet infinitely capable.<br/>
The more you use it, <br/>
the more it produces;<br/>
the more you talk about it, <br/>
the less you understand.<br/>
– Monad Te Ching
</blockquote>

So zen! 😀

#### What resources and background linked 
For Monad is useful enough for the sake of essence of FP, so I decide to give it a shot. However, it comes so much unknown and unknown I don't know if is unknown when I study this concept. I read an existed <a href='https://medium.com/free-code-camp/demystifying-the-monad-in-scala-cc716bb6f534'>post</a> from experienced Scala programmer Sinisa Louc, also researched this in math world. Here I would like to recommend a book named **A Brief Course in Modern Math for Programmers**, which covers concept such as monoids, algebraic structures, categories, monomorphism, epimorphism, isomorphism, functors, monads. The book is very short, less than 200 pages, but comprehensive. 

As the writer said for this book, the purpose is to provide <blockquote>the reader with the food of thought, material for imagination, and ideas from modern mathematics that have been used in programming practice for a while now by those who know these things but which about 90% of practicing programmers would find totally alien.
</blockquote>

I enjoyed for partial reading by far and agreed for what been said for programming in this book, like "the math world is revoluting, which becomes more and more important in CS field". I agree based on known part at this moment, as I see such as new language born and old language retired or less used, or certain language produces much enterprise bugs for language itself setbacks, so that's sometimes we code in our code base to fill the enterprise waterfall.

#### Monad in Scala 
Bit of theory before code on, for if too much abstract but no real case practice, then at least will bury me in this wildland. 

Monad: a data structure, originated from `functor`(all monads are functors), it(endofunctor) has two operations(special families of morphisms), one is `unit`, one is `flatMap`. Abstarctly speaking, a morphism is a function that maps values of one type into values of another type; functors, which are defined by type constructors, usually map poorer types into richer types. Below the typical ***signature*** of Monad, which is a parametric type, take a type parameter T. 
```Scala
trait Monad[T]{
  //return flat lists 
  def flatMap[U](f: T=> Monad[U]): Monad[U]
}
//take value v of type T, return Monad[T] contains v 
def unit[T](v: T): Monad[T] 
```
Here use `unit` to map the value of type T to get an instance of Monad, Monad[T]; use `flatMap` to compose the operations in a pipeline. In programming, composition is common enough, like currying, which chains parameters to compose multiple parameters together in one running block or nested lines.  
- `unit` u[T] : T => F[T] (wrap T into Monad F) 
- `multiplication` m[T]: (F[T], T => F[B]) => F[B], be it flatMap.

For unit `u`, and method `m` are natural transformations, so it can be processed in any sequences as the image below, first unit, then flatMap, or other way. Another definition by Cats is <blockquote> A monad is a mechanism for sequencing computations.</blockquote> it has a good book <a href="https://books.underscore.io/scala-with-cats/scala-with-cats.html#what-is-a-monad">Scala with Cats</a>, worthy to have a read. So from the functions flow in Monad, here comes Monad law. 
<figure>
<img src='/assets/monad.png'>
<figcaption>the image credits for book *A brief course in modern math for programmer*
</figcaption>
</figure>

#### Monad Laws 
**Monad laws** in general for left/right unit identity and associativity, if any law fails, then it can't be monad:
```Scala
u(x).flatMap f == f(x)
m flatMap u == m
(m.flatMap g) flatMap(h) == m flatMap(b=>g(b) flatMap h)
```
#### Practice Monad in Scala 
Eough for theory, I think theory is used to validate and prove what we explore in practical way, therefore build the bridge for practice and theory is critical. So let's bear the basic surface and see how to use that unit to map and flatMap to decompose concept to build a monad. 

```Scala
class Monad[T](v: T) {
  def map[A](f: T => A) = new Monad(f(v))
  def flatMap[A](f: T => Monad[A]) = f(v)
  override def toString = v.toString
}

val sum= for {
  a <- new Monad(1)
  b <- new Monad(2) 
} yield a +b //3 
```
#### But, what the heck? - monad be used in practical ways, as a pattern
For example Scala's `List[T] or Option[T]` is monad from standard libarary, which can `map[T]` to an instance of all Type T, method flatten: `List[List[T]]=> List[T]; u(T)=> List[T]`, is unit transformation. Other popular library such as Cats is widely used for using monad concept. So read that documentations and understand how monad is used there is important along the way. However, I would like to practice the pure way, so that to understand this concept in backbone.  

One useful Monad is **Option Monad**, used often in pattern matching function, to avoid `Null` value in control flow by chaining multiply operations. Suppose we are fetching data via API, some data fields is null, so our function will return None. The code below use for comprehension, the syntatic sugar for Monad to improve code readibility, to be clean (aka. pure).
```Scala
val list1= List(1,2,3,4)
val list2= List(2,3,4,5)
val mapAdd= for {
  a<-list1
  b<-list2
} yield a+b 

//equal to
list1.flatMap{ a=>
  list2.map {b=> 
  a+b 
  }
}
```
Above shows a have a simple monad, flatMap it to add each elements in both Lists, same for data structure List and Dict in option `Null`condition. 

```Scala
//list monad 
val country = List("Germany", "China", "US", "France")
//map 
val city =
  Map("Germany" -> "Berlin", 
  "China" -> "Shanghai",
  "France"->"Nice")

val getCity= for {
  c <- country
  city <- capitals get(c) orElse(Some("None")) 
} yield(j)

//equal to 
country flatMap {
  c=> city.get(c).orElse(Some("None")) map {
    city=> city
  }
}
```

#### Take a break, then Next step 
As the title, Monad is everywhere and it's practice cases in real world are wildly varied, and rich mathematics is linked with it. so I will keep exploring this math term to expand the unknown, until which click in my head. But I need take a step, walk away from monad, for what I knew for Monad is just a tiny iceberge, aka. Ah I see monad now!&#128587;

Further explore required to understand monad in a deep way, such as Kleisli Category, IO monad, also without trying standard library like Cats mentioned above or Future from Scala standard library is hard to understand the how to make use of monad in practical way. 

