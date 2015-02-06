---
title: Making My First IoC(ish) container part 1
layout: post
author: MegaWubs
categories: php
tags: php laravel ioc
---

This is the first post in a series of posts about the creation of my first IoC(ish) container. It'll explain the 
basics of the `ReflectionClass` that's heavily used throughout the rest of the series.

## Part 1, ReflectionClass


### Why I made it
For a project I did recently I was not allowed to use any kind of framework. This was a challenge, as I've been 
programming applications with a Laravel for a few years now. But, this did give me an amazing opportunity to 
flex my muscles a bit and not only implement a few concepts I had learned the past few years, but also program the 
logic of those concepts.

### My inspiration

The inspiration for my IoC container came from Laravel. From the moment I understood the working of an `App::make()` 
call I've been wanting to understand the logic behind it. As I dived into the source code I found that the real magic 
was preformed by the `ReflectionClass` php native class. Lets dig into this class!
   
### The ReflectionClass

"The `ReflectionClass` reports information about a class." That's what php.net has to say about it. But, how and what 
exactly? I'll show you with a bit of code.

Lets take the following class
{% highlight php %}

<?php
class Foo{
    
    private $bar;
    
    private $fooBar;
    
    public function __construct(Bar $bar, FooBar $fooBar){
        $this->bar = $bar;
        $this->fooBar = $fooBar;
     }
     
     public function getBar(){
        return $this->bar;
     }
}
{% endhighlight php %}

and examine it with `ReflectionClass`

{% highlight php %}
<?php

$reflectionFoo = new ReflectionClass('Foo');

{% endhighlight php %}

The code above makes the `ReflectionClass` instance with the name of the class `Foo` as a parameter. The 
ReflectionClass will now load in the `Foo` class and we are able to access information about, lets say, the 
constructor. This can be done like this:

{% highlight php %}
<?php
$constructor = $reflectionFoo->getConstructor();
{% endhighlight php %}

This method returns a `ReflectionMethod` object that is, just like the `ReflectionClass`, a way to access information
about a method from a class. With this object, we can get access to a number of things. For now, the only thing I'm 
interested in are the parameters of the constructor.

{% highlight php %}
<?php
$parameters = $constructor->getParameters();

foreach ($parameters as $parameter) {
    var_dump($parameter->getClass());
}
{% endhighlight php %}

We now get an array of `RreflectionParameters` back, and illiterate over them to get the `ReflectionClass` 
instance of the class that's type hinted before the parameter name. If you do not provide a type hint, the result of 
`$parameter->getClass()` will be `null`. The loop above will print out the following:

{% highlight javascript %}
object(ReflectionClass)[5]
  public 'name' => string 'Bar' (length=3)
object(ReflectionClass)[5]
  public 'name' => string 'FooBar' (length=6)

{% endhighlight javascript %}

The `ReflectionClass` has a number of other use full features. It can not only get methods from a class, it can also 
initiate the class with, or without, arguments. Like so.

{% highlight php %}
<?php

$reflectionFoo = new ReflectionClass('Foo');

$bar = new Bar();

$fooBar = new FooBar();

$foo = $reflectionFoo->newInstanceArgs([$bar, $fooBar]);

{% endhighlight php %}

Knowing this, creating a mechanism that can look at a class based on its full name and also look up it's dependencies
wouldn't be too hard. That's something I'm going to tackle in part 2.