---
layout: post
title:  "Scala Pattern (aka. practice) - Part 1"
date:   2020-03-23 22:30:00 +0800
categories: scala pattern
tags: scala
image: Scala_logo.png
applause: true
short_description: pattern 
--- 

<div markdown="1" id="text">
For the learning, logic is very important, it needs thinking not memorization, so if something I can't understand it, then drop it, redo, but do not memorize it [ban!]. Therefore, to write this blog about Scala, based on so far what I understand, especially after watching <a href="https://www.youtube.com/watch?v=71yhnTGw0hY">12 steps to better Scala</a>. I plan to explain a few tips for better writing Scala by below toy code. My goal for Scala, I would like to experiment derivative learning, to practice something in Scala based on what similar I used in Python. Hope this can boostrap the learning and help structure my knowledge. (One reason for it, I strongly felt programming is the domain full of the rules (aka.pattern), only get enough or make peace with the strict rules, then there are some free space to be creative. *yeah, freedom comes with discipline, hard lesson learnt with pain*). <br/>

<hr>

1/4 tips out of 12 are: 
- Algebraic Data Type (ADT) to build data model.
- Use Type class more than interface (see Polymorphism in Scala post).
- How to avoid type erasure use `+ `

why need use ADT? For case class is good to construct data. Normally ADT has two groups, `Product(records)` and `Sum(enumerations)`. For data relations, Product is the group for logical "is-and", Sum for "is-or". Follow from abstract (generic) to concret (specific), Product is a container to hold all the common data types, abstract the Sum part deeper and fully and put that data section as one `field` in Product case class structure.

Abstraction is the key in programming, first came into my mind is when I was doing Spark workshop one more year ago, since now have been met this term often, is useful in programming which gives a way to let different data types to share the common structure. The term `algebra` should be the king on the top level, for it's the mathematical way to abstract all the reality ideas or domain objects we can think of, into the logic and computed by the algorithm we give it. In functional programming, ADT is used to construct domain objects of data, for data modeling. The base rule to construct the class is to use `sealed trait + case object, case class` pattern, traits, is used for type classes as used before in polymorphism subtype case and sum types; case classes, is used for product types. Be aware of this best practice **Mark case classes as final** from Scala Best Practice <a href='https://nrinaudo.github.io/scala-best-practices/tricky_behaviours/final_case_classes.html'>page</a>, final means you can't change during the run-time. 

*notes*: 
1. Companion object should be used together with trait, so should use paste mode, run `:paste` in the interactive shell 
2. Trait is the shared interface to be extended by case class/object
3. When class have no variables, either use empty () or use case object; case class must have a parameter vice versa

```Scala
//sum
sealed trait Price
object Price {
case object NoSalePrice extends Price 
final case class PreSalePrice (discount: Double) extends Price
final case class OnSalePrice (discount: Double, qty: Int) extends Price
}
//product 
final case class ProductSales(
  sku: Int,
  pktName: String,
  price: Price
)

val presale= new ProductSales(1, "coke",Price.PreSalePrice(0.9))
val onsale= new ProductSales(2, "sprite",Price.OnSalePrice(0.6, 2))

//defined trait Price
//defined object Price
//defined class ProductSales

println(presale.pktName) //coke 
println(onsale.discount) //0.6
```

It works just like the relational database, so we have the column and rows for data we just hard code. Next step we can do some operations on the toy data just generated. 

```Scala 
def calculatePrice(cost: Double, p: Price)={
   p match {
    case Price.NoSalePrice => 1
    case Price.PreSalePrice(discount) => cost *discount 
    case Price.OnSalePrice(discount,qty) => cost*discount*(qty*0.9) 
    case _=> None 
  }
}

//val presale_test= calculatePrice(1.2, Price.PreSalePrice(0.8))
//presale_test: Double = 0.96
```
Further more about class and complex operations in Scala, I will keep to explore on next article. 

How to avoid the run-time type error, which is compile validated, but for the unexpectation error on the run-time in JVM environment, it will remove the generic type for class, espeicially for the instance of subtype's can't be the subclass of the instance of supertype. The solution is to prefix `+`, called covariance annotation, it indicates that subtyping is covariant for that parameter, it's the declaration-site variance. Further for use `-` for contravariant and no sign means invariance. `+` used here is to provide two purposes, 1). subtype can extends supertype 2). the instance of subtype can extends instance of supertype. 
So think in a abstract way, if A is a subtype of B, in a sense that every instance of a: A is also the subclass of nstance of b:B.  

So the formula as above, construct the data and its subtypes use `sealed trait and object pattern`. 

```Scala
//supertype and subtype 
sealed trait Transportation[+T]
object Transportation {
  final case class Bikes[+T](param:T) extends Transportation[T]
  final case class Trains[+T](param:T) extends Transportation[T] 
  final case object Feet extends Transportation[Nothing] 
}

//superclass and subclass 
sealed trait MoveTool 
case object BlackBike extends MoveTool
case object Train extends MoveTool

def move(tool: Transportation[MoveTool]):Unit={
  println(s"move with $tool")
}

val bike= Transportation.Bikes(BlackBike)
println(move(bike))
```