---
layout: post
title: "Intuition about Functors, Applicatives and Monads"
excerpt: "An article explaining my comprehension in a simple way."
categories: articles
tags: [functor, monad, applicative]
author: ybr
comments: true
share: true
image:
  feature: #so-simple-sample-image-7.jpg
  credit: #WeGraphics
  creditlink: #http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---

I went to [scala.io](http://scala.io/) 2014 and had the chance to sit in the talk [OptimusPrimeT](https://speakerdeck.com/larsrh/optimusprimet) by [Lars Hupel](https://twitter.com/larsr_h). I really like his intuitive way to describe functional buildings.

The Functor and Monad buildings are very easy to come by since any Scala programmer faces them immediatly. As soon as one tries to understand behaviours of Option and List, it will read these words Functor and Monad.
While Applicative Functors are less highlighted and I never met one in the standard library.

We will try to get an intuition about these functional buildings setting a particular focus on Applicative.

# Functor

> "Functors are about transforming data" - Lars Hupel

Functors do not allow creation nor deletion of data but just transformation. Also they are intended to keep their behaviour.

Here is a functor type class definition

{% highlight scala %}
trait Functor[F[_]] {
  def map[A, B](fa: F[A], f: A => B): F[B]
}
{% endhighlight %}

We can infer from types that we need an A to compute a B, the reamining is functor's responsibility. The function f does not take a F[A] nor it returns a F[B], meaning you have no way to influence the functor's logic.

Considering scala List as our Functor and applying f on data of the List will result in another List of the exact same length in the exact same positions.

{% highlight scala %}
List(1, 2, 3).map(_ - 2)
: List[Int] = List(-1, 0, 1)
{% endhighlight %}

Use `Simple transformation`

# Monad

> "Monads are about transforming dependent pieces of data" - Lars Hupel

On the other hand Monads allow both creation and deletion of data as well as transformation. Furthermore the behaviour of the Monad can be switched to some less happy path.

Here is the Monad type class definition

{% highlight scala %}
trait Monad[F[_]] {
  def flatMap(fa: F[A], f: A => F[B]): F[B]
}
{% endhighlight %}

Types tell that an A is required to compute a F[B], and the rest is left apart as for the Monad's logic. The function f allows to switch between the many shapes of a Monad. We deduce here that the effect expressed by F[B] depends on A so Monads are really about chaining dependent pieces of data.

For instance a List will apply f to each of its elements, when f returns a List containing more than 1 element it results in adding elements to the List.

{% highlight scala %}
List(1, 2).flatMap(x => List(x * 10, x * 10 + 1))
: List[Int] = List(10, 11, 20, 21)
{% endhighlight %}

Conversely when f returns an empty List then this element is removed from the List.

{% highlight scala %}
List(1, 2).flatMap(x => List[Int]())
: List[Int] = List()
{% endhighlight %}

Use `Short circuit error handling`, `Parser combinators`, `Value-oriented paralleslism`, ...

# Applicative

> "Applicatives are about transforming independent pieces of data" - Lars Hupel

Applicatives are more confidential, less explicitly exposed but you certainly use them quite often under the cover.

Applicatives are like Monads in that they can create, modify and delete data but in a slightly different way. An Applicative Functor does not rely on a specific function of yours to create or delete data but only on the supplied F[_] arguments and its very own logic.

Here is the Applicative type class definition

{% highlight scala %}
trait Applicative[F[_]] {
  def <*>[A, B, C](fa: F[A], ff: F[A => B]): F[B]
}
{% endhighlight %}

Applicatives are more confusing when compared to Monads, why would we ever need to have a F[A => B] ? As its name suggests Applicatives try to apply a function A => B with a label F (F[A => B]) to an A with the same kind of label F (F[A]).

Focusing on types, from an A we know how to compute a B. We don't know how to compute F[B], once more it is Applicatives responsibility. Finally we can't bring out any relationship between F[A] and F[B] but our A => B, this means Applicatives know how to generically manage things nicely. Chaining application through an Applicative makes very clear it deals with independent pieces of data.

Say we have some untrusted generic data structure and want some other trusted data structure, respectively JSON and case class. Surely we know how to extract data from a json tree for the first field of the class as well as we know for the second field and so on. Then we need some way to express these extracted pieces of data should overall fail when combined together if at least one of them can not be extracted.

For example we have a Person

{% highlight scala %}
case class Person(name: String, age: Int)
{% endhighlight %}

Of course, we know how to create a Person from the data needed for its fields.

{% highlight scala %}
Person.curried
: String => (Int => Person) = <function1>
{% endhighlight %}

Here for ff, A is String and B is Int => Person.

It is Applicatives responsiblity to decide what to do with zero, one or more failing pieces of data, should it be to accumulate failures, keep the first or the last one.

Going back to the untrusted data structure we can create a way of extracting each piece of data from the JSON tree and make these pieces available as Options. We can extract both the name to get an Option[String] and extract the age to get an Option[Int]

Now we mix everything together to see what can happen:

{% highlight scala %}
Option(Person.curried) <*> extract("name")[String] <*> extract("age")[Int]
: Option[Person]
{% endhighlight %}

This expression has the following type:

{% highlight scala %}
Option[String => Int => Person] <*> Option[String] <*> Option[Int]
: Option[Person]
{% endhighlight %}

In case data is missing somewhere in the application chain then it is logical to result in a None.

We understand the pieces of data are independent one from another and Applicative Functors allow to combine them in order to compute meaningful results.

Use `Accumulative error handling`
