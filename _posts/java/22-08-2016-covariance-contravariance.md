---
---

The difference between covariance and contravariance is something that has always bugged me. So,  I have finally decided to research more on this and clarify my understading.

Covariance and Contravariance is related to subtyping. What this means, is that the compiler will check is it possible for me to replace this function with another function. Can this function be considered a subtype of the other function.

In Java terms, the question here is that is this method overriding the super class. Covariance is related to return type of a function whereas contravariance is related to to arguments of a function. Let me quote from wikipedia [wikipedia](http://en.wikipedia.org/wiki/Covariance_and_contravariance_%28computer_science%29)

>> contravariant in the input type and covariant in the output type

Let us start with some example. 

Assume that we have a abstract class `Animal` and a subclass `Cat`. We also have an abstract class `AnimalShelter` and subclass `CatShelter`.

	  public abstract class Animal {
	    public abstract void displayAnimal();
	  }

	  public class Cat extends Animal {
	    public void displayAnimal() {
		System.out.println("Cat");
	    }
	  }

	  public abstract class AnimalShelter {
	    public abstract Animal getAnimalForAdoption();
	  }

	  public class CatShelter extends AnimalShelter
	  {
	    public Cat getAnimalForAdoption() {
		return new Cat();
	    }
	  }

If you notice in the class `CatShelter` for the method  `getAnimalForAdoption`, I am returning the class `Cat` rather then what was defined in the base class `Animal`. Even though the signature of the method is changed, java still consider this an override as such there is no error. This is **covariance** in action. So if a language supports covariance like java, basically what it means that the subtype of a type can have a function that return a more specialized type compared to that of the base type (In this case, we returned `Cat` instead of `Animal`).

Let us now add a new type called `AnimalLover`.  We will also create a subtype called `CatLover`. We will then modify the signature of the method `getAnimalForAdoption` to accept the type `CatLover`. We then modify the method `getAnimalForAdoption` in the class `CatShelter` and that takes the parameter of type `AnimalLover`
  
	  public abstract class AnimalLover {
	     public void animalLoveType() {
		System.out.println("Animal Lover");
	     }
	   }

	  public class CatLover extends AnimalLover {
	    public void animalLoveType() {
		System.out.println("Cat Lover");
	    }
	  }

	  public abstract class AnimalShelter {
	    public abstract Animal getAnimalForAdoption(CatLover lover); // add a new parameter CatLover
	  }

	  public class CatShelter extends AnimalShelter
	  {
	    public Cat getAnimalForAdoption(AnimalLover lover) {   // modifying to refer to the base class
		return new Cat();
	    }
	  }

This will not run as java does not support **contravariance**. In this case contravariance means we are replacing an argument  from ***specialized*** type to a more ***generic type***. If you do this, java will consider this as an overload not as an override. There are languages that support contravariance like [sather](http://en.wikipedia.org/wiki/Sather)

 Let us do one last experiment. What if we make the method `getAnimalForAdoption` take `AnimalLover` in the base class `AnimalShelter` and then in the subclass `CatShelter`, we replace it with `CatLover`. Will this work?
 
	  public abstract class AnimalShelter {
	    public abstract Animal getAnimalForAdoption(AnimalLover lover); // modify to base class now
	  }

	  public class CatShelter extends AnimalShelter
	  {
	    public Cat getAnimalForAdoption(CatLover lover) {   // refer to the sub class
		return new Cat();
	    }
	  }

This will also not work. As for java, the argument type is no variance i.e. it must match exactly the same type. There are language that allow covariance for arguments like [Eiffel](http://en.wikipedia.org/wiki/Eiffel_%28programming_language%29).
 
We can use generic in order to allow this to happen. We can update our class `AnimalShelter` and `CatShelter` class accordingly as shown below

	   public abstract class AnimalShelter<T extends AnimalLover> {
	      public abstract Animal getAnimalForAdoption(T lover);
	   }
	   public class CatShelter extends AnimalShelter<CatLover>{
	    public Cat getAnimalForAdoption(CatLover lover) {
		return new Cat();
	    }
	   }

There is a good [stackoverflow](http://stackoverflow.com/questions/2501023/demonstrate-covariance-and-contravariance-in-java) link that provide more examples 