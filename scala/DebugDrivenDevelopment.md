Debug Driven Development (DDD)
==============================

Debug driven development is a contested development  technique which I have seen multiple junior development teams use. It is the act of running the system on a
 developer's local machine and using breakpoints and code evaluators to build new functionality into legacy systems which they do not understand. I personally have in the past 
 found myself rather guilty and feeling ashamed of myself. 

Where did DDD come from?
------------------------

We have all seen and been that new developer that joins a well established team that has been working on a complex code base for months and years. 
The first Jira ticket they receive is always a small 1-3 point ticket (or so it seems). The lead developer on the team will walk through the code, maybe once, 
with an architecture diagram to show how it all vaguely hangs together. Then something remarkable happens, they discover the **metaphysical feature**.
 
##The metaphysical feature

You may be wondering what this metaphysical feature is. Let us first understand what metaphysics means.

>_metaphysics - abstract theory with no basis in reality_

Ok, enough with the philosophy!

And just like that, we know that there theoretically exists a feature in the code, as we can see it, with almost no calls to it in production. It is a low value 
low use feature, which has now manifested itself to complicate our code. At this point, our new developer is stuck, they cannot add their functionality 
without first understanding how they will deal with the dragon of technical debt in front of them. What is worse, is that the developer that wrote the code
has left the company is almost certainly lounging in a pool in the West Indies. 

## How have developers rationalised it?

I have seen developers time and time again attempt to tackle this issue. With languages such as Scala where monadic composition is possible, and you
can create a functional pipeline, contextual knowledge is the holy grail to fixing bugs and building features. Consider for example the following code snippet in Scala.

~~~~ 
  case class Coordinates(lat: Double, lon: Double)
   val dataCentreAndHostCoordinates : Map[Coordinates, Coordinates] = Map(
     Coordinates(30.0, 40.9) -> Coordinates(32.0, 46.7),
     Coordinates(32.0, 41.6) -> Coordinates(32.0, 46.7),
     Coordinates(33.7, 79.3) -> Coordinates(98.3, 114.7)
   )

   def compute(lat: Double) =
     dataCentreAndHostCoordinates.filter(_._2.lat % 2 == 0)
                                                  .filter(_._1 == lat)
                                                  .filterNot(_._2 == lat)
                                                  .withFilter(_._1 != 8)
                                                  .map((pair) => pair._1.lat + pair._2.lon)
                                                  .foreach { co =>
                                                    println(co)
                                                  }
   
~~~~

 Imagine we have some map of co-ordinates telling us the latitudes and longitudes of a client and the host which is servicing it's requests. The `computeTriangulation` function
 is (somewhat arbitrarily for the purpose of this example) applying filters onto this map to triangulate and print the optimal latitude to satisfy some condition. 
 For a new joiner to a development team, especially one who has little experience in Scala, the composition of the functional pipeline, the underscores and the filters is going to be 
 incredibly off-putting, while to an experienced developer, with knowledge of the problem, and solution, this makes perfect sense. 
  
 Unfortunately this knowledge is not fluid between people. The difficulty also arises when ```_._1``` and ```_._2``` does not allow the developer to infer information about types, and 
 contextually what this means. **Cue DDD***, it is at this point where documentation and test coverage is poor that I have seen developers tempted add additional functionality onto this
 by using the `evaluate expression` feature on their IDE's debugger while stepping through the code.
 
## Why is this bad practice?

Is there an evil behind this way of working? Almost definitely yes. 

1. More often than not, **this is not going to inspire the developer to invest time to understand why the code has been written** in such a way, but rather focus more on adding in their functionality.
2. The have used test driven development to go about completing their story, the only problem is that **they have not written any tests**, but rather executed them in their heads while playing with the 
 attached debugger. Once the code has been pushed, the coverage has forever been lost, and the next time a developer touches this code, we start the cycle again.
3. DDD **adds technical debt**. A better way of approaching this solution would be to refactor this code to use anonymous functions, and combine the filters as follows.
4. It is preventing junior developers to learn their craft. Much like an avid Vim user will tell you, **over-reliance on tooling and an IDE is not going to help you become a good developer**. Not at least when
 the majority of your development is actually taking place at runtime. Trial and error is a good way of learning, but brute forcing different functions into the expression evaluator is not!
5. DDD will usually only cater for the **happy path**.  
 
The following code makes the compute function more readable by using anonymous functions, and achieves the same functionality. There are no underscores or obscurely names variables.

~~~~~
  def compute(lat: Double) =
    dataCentreAndHostCoordinates
      .filter( {case (client, host) => host.lat %2 ==0 && host.lat!=lat})
      .filter( {case (client, host) => client.lat == lat  && client.lat !=8})
      .map( {case (client, host) => client.lat + host.lon})
      .foreach(println)
~~~~~
 
## What is a better approach than DDD?

1. Actually **understand and document the code**. Read the code, don't just turn the debugger on. Make the variables more readable, add a comment, add something.
2. If you can't tell what the code is doing, try to **verify your own assumptions about what the method does by writing some tests**. You will invariably be improving documentation, unit test coverage and establishing 
  truths about what the code does. It will also help identify if the existing code for metaphysical features has bugs in it.
3.**Leave the code in a better condition** that what you found it in. It you know that the underscores and complex data structures are daunting to new developers, then fix it. 
4. Make mistakes, but **be in charge of developing your story**. If you follow point 2. and actually increase code coverage, you are more likely to spot bugs earlier than in production when someone actually uses the metaphysical 
feature for the first time.
5. Accept that the code is complex and that it is likely to fail, being sceptical about your code is a good thing. Rather than trying to brute force your code into conforming to your happy path, **map out all of the 
failure modes and plan your recovery**.


 



