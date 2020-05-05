---
layout: post
title:  "Monads, the typeclass"
date:   2020-05-05 18:56:00 +0800
categories: scala 
tags: scala
image: monad_post2.png
applause: true
short_description: Monad for typeclass 
--- 

<div markdown="1" id="text">
Monad attracts me, for its abstraction and logical reasoning, so I started try to dig into by studying Type and Category Theory, read one book <cite>Category Theory for Programmer</cite>, not finished yet. Now listening to <a href="https://www.youtube.com/watch?v=BoJGIqyriCc">Monads song</a> from Dylan Beattie, I think maybe it's a high time to review Monads, what the new mixed in my brain from that rabbit hole. 

From last time when I knew Monads, which I learnt superficially that monad holds a signature, a `functor` plus `flatMap` method, and sugared syntax `for comprehension` for sequence computation (monadic operation).

Simply abstracting over above from bottom up angle, 
1. Just a `functor` with a `flatMap` method, to transform data from type A to B in `context` F, based on `trait Monad[F[_]]` constructor. Method flatMap works like sequence computation, that's why use for comprehension to return immediate results and chain them with next return. 
```Scala
// below is the abstract level flatMap 
def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B] 
```
2. what is functor, functor is `semigroup` with empty method, so map over something with no side effect, functor must follow the identity law and associativity law. Identity provides flexibility to `fold` data structure, associativity offers composability. Composition is the hardcore of FP. 
```Scala
trait Functor[F[_]] {
  def pure[A](a: A): F[A] //F can be List, Option, Either etc 
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```
3. what is semigroup. similar to functor, but just combine data structure, without pure method. Associativity law is the only rule to follow. 
```Scala
trait Semigroup[A] {
  def combine(x: A, y: A): A
}
```
<i>My feeling is, after understanding above concept, make use of the library such as Cats or Scalaz will make code more efficient. So the cost is learning and practicing framework.</i>

Back to Monad, in metaphor way to infer is a container or data constructor to build relational data structure, by creating which if we need functions to have map and flatten methods(`flatMap`). The most popular Monads are List, Option, Future, IO. 

In Scala related library or some practice, two main terms get involved, 
- `implicit`, to make the instance (by using factory method ) accessible in scope.
- `Higher Kinded Type(HKT)`, to lift types to a new abstract level, also be called `type constructor`, which provides a `context`, a type wraps another type. It's very simple to understand is HKT provides a type constructor which can be List, Option, then a placeholder to hold type, can be Int, Double, String etc. 

Let's have one toyed code for this typical pattern. 
```Scala
sealed trait Monad[F[_]]{

  def pure[A](a: A): F[A]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
}

object Monad {

def apply[F[_]](implicit v: Monad[F]) = v 

implicit val listInstance: Monad[List] = new Monad[List] {
  def pure[A](a: A): List[A] = List(a) 
  def flatMap[A, B](fa: List[A])(f: A => List[B]): List[B] = 
      fa.foldLeft(List[B]()){(acc, x)=> acc:::f(x)}
  }

implicit val optionInstance: Monad[Option] = new Monad[Option{
  def pure[A](a: A): Option[A] = Some(a) 
  def flatMap[A, B](fa: Option[A])(f: A => Option[B]): Option[B] = 
  fa.flatMap(f)
  }
}
```
Above `sealed trait and companion object pattern` is the convention, can be useful in most of Scala typeclass implementation. 

Monad is core of category theory, also itself is a typeclass, so as I wrote on the previous article, Monad is everywhere. My understanding is Monad can be used to construct ADT or polymorphic class. Let me elaborate one practice for typeclass. 

Before start with typeclass implementation, concept used here are:
1. subclass use extends such as from B extends A, so B is-an A, to make sure code sharing in both scopes
2. type bounds(type parameter `A` bound to type, such as supertype) and variance (covariant`+A` and contravariant`-A`), the relations between groups of types. 
3. (abstract class or sealed trait)`xor` and class`has` for ADT, use trait is a good starting, personally I like trait in Scala, to extend functionality with flexibility when using `mixin`

```Scala
abstract class Beverage {
  def price: Double 
  def `package`: Boolean //for package is reserved words, use ``
}
case class Coke (override val price: Double, 
                 override val `package`: Boolean, 
                 favor: String,
                 score: Double,  
                 region: String) extends Beverage

case class Sprite (override val price: Double, 
                   override val `package`: Boolean,
                   score: Double,  
                   region: String) extends Beverage

object Beverage{
  def compare[A <: Beverage](coke: A, sprite: A): A = {
    //add upper type bound, the constraint to type A 
    if (coke.price > sprite.price) coke 
    sprite
  }
}

//then another group of type comes, and relation is positive, the same direction, so annotate as +A 
abstract class Classifier [+A <: Beverage]{ //covariant 
  def cls(v: Coke): Unit 
}

case class CokeClassification() extends Classifier[Coke] {
def cls(v: Coke): Unit = v match {
    case Coke(_, _, _, v, _) if v > 0.8 => println("is coke") 
    case _ => println("go back to retrain")
 }
}

object Comp{
  import Beverage._ 
  def main(args: Array[String]): Unit = {
  val coke = Coke(2.5, true, "light", 0.8, "cn")
  val sprite = Sprite(2.8, false, 0.6, "cn")
  val res = Beverage.compare(coke, sprite) 
  println(res)  
  CokeClassification().cls(Coke(2.9, true, "light", 0.9, "cn"))
  }
}
```
//TODO, implement a new user story for Monad usage and @typeclass.

Next, another corner of Monad, is `Monad transformer`. The implementation will be covered on next article. 
</div>