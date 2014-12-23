---
layout: post
title: "The build pattern in Scala"
---

So, you are a Senior Developer with solid experience in java, c#, or c++? You know the famous <a href="http://en.wikipedia.org/wiki/Builder_pattern" target="_blank">GoF Builder pattern</a> by heart, applying it is a matter of seconds and there's nothing more to learn?

In this case Rafael's article <a href="http://blog.rafaelferreira.net/2008/07/type-safe-builder-pattern-in-scala.html" target="_blank">"Type-safe Builder Pattern in Scala"</a> will probably shake your world.

To put it in simple terms, let's say we have a constructor signature for type Foo:

{% highlight scala %}
case class Foo(val a:A, val b:Option[B])
{% endhighlight %}

And the corresponding builder which emulates a Java-ish implementation:

{% highlight scala %}
class FooBuilder  {
  var a:Option[A] = None
  var b:Option[B] = None

  def withA(a:A) = {this.a = Some(a); this}
  def withB(b:B) = {this.b = Some(b); this}

  def build() = new Foo(a.get, b)
}
{% endhighlight %}

This is self-explanatory. A careful reader might wonder what happens when the mandatory variable 'a' remains unbound?

{% highlight bash %}
scala> new FooBuilder() withB(B) build
java.util.NoSuchElementException: None.get
{% endhighlight %}

Simple answer, for non-optional parameters (here: a:Option[A]) the Option[A].get method throws an exception at run-time if a is unassigned (or more precisely: is assigned to an instance of None).
Okay, but where's the magic, you say? Good, imagine we could statically ensure that non-optional parameters are bound at compile-time. Got it? Then let's take a shortcut - we transform the builder code into a functional-inspired form and use phantom types as a constraint for instantiation:

{% highlight scala %}
object BuilderPattern {
  case class A
  case class B
  case class Foo(val a:A, val b:Option[B])

  abstract class TRUE
  abstract class FALSE

  class FooBuilder[HasA, HasB](val a:Option[A], val b:Option[B]) {
     def withA(a:A) = new FooBuilder[TRUE, HasB](Some(a),b)
     def withB(b:B) = new FooBuilder[HasA, TRUE](a,Some(b))
  }

  implicit def enableBuild(builder:FooBuilder[TRUE,FALSE]) = new { 
     def build() = new Foo(builder.a.get, builder.b) }

  def builder = new FooBuilder[FALSE,FALSE](None, None)
}
{% endhighlight %}

Let's give it a try:

{% highlight bash %}
scala> import BuilderPattern._
scala> builder withA(A()) build
scala> builder withB(B()) build
:12: error: value build is not a member of BuilderPattern.FooBuilder[BuilderPattern.FALSE,BuilderPattern.TRUE]
{% endhighlight %}

I hope your brain screams "SWEET!". Didn't get it? Then enjoy Rafael's lengthy version <a href="http://blog.rafaelferreira.net/2008/07/type-safe-builder-pattern-in-scala.html" target="_blank">"Type-safe Builder Pattern in Scala"</a>.
