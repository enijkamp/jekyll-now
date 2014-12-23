---
layout: post
title: "How to avoid try/catch blocks in Scala"
resume: "As I’m running low on time these days, but want to keep the steady flow of Scala buzz alive, here’s another simple trick. Dealing with Java I/O you probably know the unavoidable burden of exception handling ..."
---

As I'm running low on time these days, but want to keep the steady flow of Scala buzz alive, here's another simple trick. Dealing with Java I/O you probably know the unavoidable burden of exception handling:

{% highlight java %}
import java.io.*;

public final class CatchMeIfYouCan {
  public static void main(String ... args) {
     BufferedReader reader = null;
     try {
        reader = new BufferedReader(new FileReader(...));
        String line = null;
        while((line = reader.readLine()) != null) {
             ...
        }
     } catch(IOException e) {
             ...
     } finally {
        try {
            if(reader != null) reader.close();
        } catch(IOException e) {
             ...
        }
     }
  }
}
{% endhighlight %}

Ouch! It takes 16 lines of code to read some lines from a file. In an ideal world we would be able to factor out this reoccuring pattern of nested try/catch blocks.
What we could do in Java is to simply put the logic which operates on the "line" string into another class and pass it to a factorized version of the code above:

{% highlight java %}
public final class StillNotThere {
 interface Read { void line(String str); }

 private static void read(Read read) {
    BufferedReader reader = null;
    try {
       reader = new BufferedReader(new FileReader(...));
       String line = null;
       while((line = reader.readLine()) != null) {
            read.line(line);
       }
    } catch(IOException e) {
            ...
    } finally {
       try {
           if(reader != null) reader.close();
       } catch(IOException e) {
            ...
       }
    }
 } 

 public static void main() {
   read(new Read() { public void line(String str) { ... } });
 }
}
{% endhighlight %}

Looks better! But still feels clumsy to define an anonymous class and ... nah. In Java 7 the try / catch syntax was enhanced to support the java.lang.AutoClosable interface. This effectively removes most of the boiler plate code, but at the same time reflects how limited the expressive power of Java is. We don't want to touch the syntax of a language just to get some minor things right.

So, let's see what we can do about it in Scala! We introduce a function named "using" which takes an I/O handle (using duck-typing to ensure the type defines a close() method) and in turn passes it to a given function f.

{% highlight scala %}
object IO {
  def using[R, S <: R): R = {
    try {
      f(s)
    } finally {
      if (null != s) {
        try {
          s.close()
        } catch {
          case _ => {}
        }
      }
    }
  }
}
{% endhighlight %}

Equivalent to the Java code above:

{% highlight scala %}
object Better {
  def main(args:Array[String]) = using(new Reader(...)) {
    reader => reader.getLines.foreach(line => ...)
  }
}
{% endhighlight %}

I hope your brain screams "DAMN!". Doesn't "using" almost look like a first-class citizen?
