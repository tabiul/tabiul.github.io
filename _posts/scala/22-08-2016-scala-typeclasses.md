## Introduction

When I started learning about TypeClasses in Scala I thought it was some feature of Scala, maybe a special keyword (which scala has lack of :) ). After some research I realized it is nothing but a pattern to solve a problem. As such let us talk about what sort of a problem that TypeClass helps us to solve

## Problem

Assuming you want to define a generic function across different types. The problem is that you have no control what sort of type your function will get. If you had then you could have gone through the trait route. So to reiterate you want to write a generic function that does a specific thing but you cannot enforce this new behaviour on other type. TypeClasses is an elegant solution to this problem.


In order to show some code, let me create a contrived problem. Let say I want to create a function that does concatenation. The problem with  concatenation is that different types does it differently. In **String** we use `+`, for **List** we use `++` etc. But now I want to have a generic function that encapsulate it

## Step 1

Define a trait for the logic that you want each type to have. In this case we want each type to have a Cons like method

    trait Cons[T] {
         def cons(t*: T): T
    }

## Step 2

    object Cons {

        implicit object StringCons extends Cons[String] {
            override def cons(t: String*): String = t.foldLeft("")(_ + _)
        }

        implicit object IntCons extends Cons[Int] {
            override def cons(t: Int*): String = Integer.parseInt(t.map(_.toString).foldLeft("")(_ + _))
        }

 
    }

Here we create two implementation of the method `cons` for two types `String` and `Int`. We could add other types if we want but for this example this would suffice.

## Step 3

    def Cons[T](t: T*)(implicit c: Cons[T]): T = {
       c.cons(t:_*)
    }

Here we can see the generic function `cons` that we wanted to create. The **magic** is in the `implicit`. So the Scala Compiler based upon the type of `T` replaces the implicit with the correct type.

## Step 4

Testing it out

    Cons("abc", "def", "ghi") # abcdefghi
    Cons(1, 2, 3, 4) # 1234

### Sugaring

Scala provides a way to make the syntax of the generic function better by using the concept [context bounds](http://stackoverflow.com/questions/4465948/what-are-scala-context-and-view-bounds) 

    def Cons[T : Cons](t: T*): T = {
       implicitly[Cons[T]].cons(t:_*)
    }


### Conclusion

I hope that this post sheds some light on what problem **TypeClasses** solve and when you might need to use them




