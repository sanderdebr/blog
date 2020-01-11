---
layout: post
title:  "Functional vs Object Orientated Programming in Javascript"
date:   2020-1-2 15:12:58 +0100
categories: vanillajs
---

JavaScript is a multi-paradigm language, supporting imperative/procedural programming along with OOP (Object-Oriented Programming) and functional programming. JavaScript supports OOP with prototypal inheritance.

## What is functional programmming?
Functional programming is a paradigm that treats computation as the manipulation of value, and avoids changing state and mutable data.

In functional code, the output value of a function depends ONLY on the arguments that are passed to the function. In other words, calling a function f twice with the same value for an argument x produces the same result f(x) each time.

This is in contrast procedural and object-oriented programs, which can often depend on a local or global state.

React and Angular require state to be maintained through the use of pure functions and immutable state.

<strong>Benefits of functional programming</strong>

<ul>
    <li>Using the functional paradigm, programmers avoid any shared state or side-effects, which eliminates bugs caused by multiple functions competing for the same resources. With features such as the availability of point-free style (aka tacit programming), functions tend to be radically simplified and easily recomposed for more generally reusable code compared to OOP.</li>
</ul>

<strong>Cons of functional programming</strong>

<ul>
    <li>Over exploitation of FP features such as point-free style and large compositions can potentially reduce readability because the resulting code is often more abstractly specified, more terse, and less concrete.</li>
</ul>

## What is OOP?

An object is a thing that we interact with, it has properties and methods. The object is the heart of object-oriented programming, not only for JavaScript but also for Java, C, C++, and others.   

# Object literal

Create a new object in JavaScript by setting its properties inside curly braces.

{% highlight js %}
const book = {
   title: "Hippie",    
   author: "Paulo Coelho",  
   year: "2018"
}
{% endhighlight %}

# Object constructor

Object constructor is the same as a regular function. It will be called each time an object is created. We can use them with the new keyword. Object constructor is useful when we want to create multiple objects with the same properties and methods.

{% highlight js %}
function Book(title, author, year) { 
   this.title = title; 
   this.author = author; 
   this.year = year;
}
const book1 = new Book ('Hippie', 'Paulo Coelho', '2018');
{% endhighlight %}

# Class
Class is not an object — it is the blueprint of an object. Classes are special functions. You can define functions using function expressions and declarations and you can define classes that way as well. We can create the number of objects using the blueprint.

{% highlight js %}
class Book {
   constructor(name) {
      this.name = name
   }
}
class newBook extends Book { 
   constructor(name) {
      super(name);
   }
}
const book1 = new newBook("The Alchemist");
{% endhighlight %}

Class syntax is syntactical sugar — behind the scenes, it still uses a prototype-based model. Classes are functions, and functions are objects in JavaScript.

<strong>Benefits of OOP</strong>

<ul>
    <li>It’s easy to understand the basic concept of objects and easy to interpret the meaning of method calls. OOP tends to use an imperative style rather than a declarative style, which reads like a straight-forward set of instructions for the computer to follow.</li>
</ul>

<strong>Cons of OOP</strong>

<ul>
    <li>OOP Typically depends on shared state. Objects and behaviors are typically tacked together on the same entity, which may be accessed at random by any number of functions with non-deterministic order, which may lead to undesirable behavior such as race conditions.</li>
</ul>
