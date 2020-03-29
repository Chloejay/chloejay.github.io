---
layout: post
title:  "Polymorphism in Scala"
date:   2020-03-22 16:20:00 +0800
categories: scala pattern
tags: scala
image: Scala_logo.png
applause: true
short_description: polymorphism pattern  
--- 

<div markdown="1" id="text">
Polymorphism is one key part in OOP when we use class, as we all known class servers two purposes in software programming. 1).construct values(instance) 2).define a data type. Here is one good explanation from <a href="https://stackoverflow.com/questions/1031273/what-is-polymorphism-what-is-it-for-and-how-is-it-used">SO</a>, which summarizes that polymorphism is the ability (in programming) to present the same interface for differing underlying forms (data types). Therefore, what is to say in OOP, is transform the "verb", the behavior of the group of subjects, be it class A, B,C,D as in one abstract object (interface), then implement concrete in its own building block, so the variable or function applied are able to take many different forms. It's one of the ways to follow `DRY` principle, reuse code, which is the high level abstraction. <br/>

In Scala, based on class **Functional Programming Principles** in Scala I'm attending from <a href="https://www.coursera.org/learn/progfun1/home/welcome">Coursera</a>, polymorphism means that a function type comes "in many forms". 1). the function can be applied to arguments of many types; 2). the type can have instances of many types. <br>
Common types of polymorphism:
- subtyping, which is typical in OOP usage, also be used often in Python class hierarchy for superclass and subclass, called inheritance. It defines types that permits "subset" relations, which one type is defined to be a subeset or superset of another type. 
- parametric, instances of a function or class are created by type parameterization, called parametric polymorphism. type parameters, to have more than one kind of type. 
- ad-hoc type classes, define a common interface and overriding the methods in its implementation, one good example is from <a href="https://typelevel.org/cats/typeclasses.html">Cats</a>.

I would like to implement two cases here, one is subtyping, one is ad-hoc type classes. Before starting, the concept that involves here should be `algebraic data type`(ADT), a type defined by providing several alternatives, each of which comes with its own constructor, such as case class, ADT is useful for data modeling. `trait`: which is defined with the trait keyword, is like an abstract class that cannot take any value parameters and can be “mixed into” classes or other traits via the process known as mixin composition, compare to trait, class, case class, object. The reason to use trait, for Scala don't allow class to extend multiply classes use with, but can extends multiply traits. `superclass` and `subclass`, `abstract class`. <br/>

Notes: my experience from the programming language I learnt, learning the concept(aka. culture) is very important, but also vocabulary, so here is the <a href="https://docs.scala-lang.org/glossary/">full glossary list</a> for Scala. Always read its source code. 

Now coming to Stage 1: the most easy case when we start Scala, that we don't care about type! Like we can do this in the universal type free language Python. 

{% highlight Scala %}
val name: List[String]=List("Chloe", "Emma")
val age: List[Int]= List(28,24)
val isEmpty= (xs: List[_])=>xs == Nil 
val checkAge= isEmpty(age)//res0: Boolean = false
{% endhighlight %}

But wait, this is not the real case when coming to type safe language, such as `Any` is very useful for type, which is the supertype(the top level in Scala type structure, for normal types are subtype of `Any`, such as `List`, `Int`, etc.). However, we should not use `Any` if not really needed. So then if we put the same behavior but different data types in Scala-Compiler, it will have overloading error messages during compile time. Ah, so it won't work when running above, so here comes the pattern. 

Stage 2: use subtyping 
<blockquote>Liskov Substitution Principle. The Liskov substitution principle says that if a function accepts an input of type A, then it should be valid to pass the function any subtype of A, without altering the correctness of the function. Similarly, for a function returning an output of type B, it should be valid for the function to return a subtype of B. The Liskov substitution principle enables principled reasoning about subtyping.</blockquote>

Subtyping, which I first knew was doing **inheritance**, so in my mental model is all linked with subclass to extend superclass. Officially, Wikipedia defines "when a name denotes instances of many different classes related by some common superclass." So what is to say is polymorphism allows us to combine common functionality in a single class and implement the details from that class in subclasses. A good example is from <a href="https://howtoscala.wordpress.com/2016/10/25/polymorphism-in-scala/">Scala official website</a>. 

So logic is: first abstract then concrete, factory the supertype as abstract type, then implement the concrete method(override) on its subtype, can relate and think about ABC package in Python. For below supertype Language, Deutsch and Chinese is the subset of Language. 

{% highlight Scala %}
abstract class Language(name: String){
  def greetings
}
class Deutsch(name: String) extends Language(name){
  override def greetings:Unit = println(s"Halo!! $name")
}
class Chinese(name: String) extends Language(name){
  override def greetings:Unit = println(s"Ni Hao! $name")
}
def say(lang: Language):Unit={
  println(lang.greetings)
}
val test= say(new Chinese("chloe"))
{% endhighlight %}

So we don't need to write the same operation multiply times, keep it DRY, at the same time, the data type can be multiply forms. Note: trait can be used here if we don't use any parameters, otherwise it will give error message, `error: traits or objects may not have parameters`

{% highlight Scala %}
trait Language{
  def greetings: String
}
class Deutsch extends Language{
  val greetings= "Halo!!"
}
class Chinese extends Language{
  val greetings = "Ni Hao!"
}
def say(lang: Language):Unit={
  println(lang.greetings)
}
val test= say(new Chinese)
{% endhighlight %}

Stage 3: use ad-hoc polymorphism 
To use type classes, a good understanding of ‘implicit’ is required. The keyword "implicit" indicates to Scala compiler to look for ways to convert from one data type to another, such as from Int to Double.

Goal: I will use typical data types Int, Double, String to combine all this in the function and not break it during the run time. First abstract the behavior output type as `T`(or any name that preferred).

{% highlight Scala %}
trait DataSource[T]{
    def add(x:T, y:T):T   
}

object DataSource[T]{
  implicit object IntData extends DataSource[Int]{
    override def add(x: Int, y: Int)= x + y 
}
  implicit object DoubleData extends DataSource[Double] {
    override def add(x: Double, y: Double)= x + y 
 }
  implicit object StringData extends DataSource[String]{
    override def add(x: String, y: String)= x + y 
}
}
def concat[T](x: T, y: T)(implicit data: DataSource[T]):T=
data.add(x,y) 

val dataInt= concat(1,2)
val dataDbl= concat(1.0, 2.0)
val dataStr= concat("kulture","menchen")
{% endhighlight %}

Viola, it's cool type classes pattern, the structure is clear from how the function is designed. Personally I prefer type classes, for it's more easy for me to understand and reasoning. 
