---
layout: post
title: "Factory JS : Dependency Injection Containers + AOP"
author: webspinner
tags:
- JavaScript
- Dependency Injection
- Factory
- AOP
---

Over the course of building many front end projects, we've learned a few lessons. One big finding is that that reusing code between these projects can be difficult, tedious and error prone. Interfaces for a given piece of reusable code are often intertwined with the current stack. Extracting that behavior involves using much larger code segments than the reusing code requires. Testing this code becomes a chore since the testing must occur in each new context.

TODO: I'd add a brief note here explaining why alternatives don't solve the problem well, to more fully set context.

AMD Dependency Soup
-------------------

To solve these issues we introduce Factory JS (TODO href to github). A dependency injection container maker so that you can organize components (types) and behaviors (mixins). Built on underscore, backbone and jquery it can encapsulate and reuse any type of library without too much effort. One of the first benefits of this utility is that it can reduce the argument soup required for an AMD module definition. Consider the following:
```
    define(['a','b','c'], function(A, B, C){
      // we now have connascence of name and order
    });
```
we can leverage a factory to clean this up:
```
    define(['factory', 'a', 'b', 'c'], function(factory){
      // now it doesn't matter which order the dependencies are in
      // except for the factory, which must be first.
    });
```
To achieve this the modules 'a', 'b' and 'c' might have code looking something like the following:
```
    // a.js

    define(['factory'], function(factory){
      function A(){
        ...
      }

      factory.define('A', A);
    });
```
As long as this code executes the factory will return an instance of A when invoked using the get method:
```
    var a = factory.get('A');
```
And if the A class takes arguments, no big deal, just pass those in as well:
```
    var a = factory.get('A', arg1, arg2, arg3);
```
You can define a constructed method on any class. Constructed methods execute after object creation and after applying all mixins to the instance. You can think of them as post-composition initializers. They will receive the arguments used in the constructor.

Mixins
------

So we have a constructor container, whoopadidoo. The real value is that this allows you to abstract behavior code into reusable chunks that can be applied in other applications. We do that here by using mixins and the factory's defineMixin method.
```
    defineMixin(name, mixinDefinition, mixinSettings)
```
Name is the string you will use to add the behavior to a definition. A mixinDefinition is an object. It contains methods and properties to mix into instances using this behavior. A mixinitialize method can be define on the mixinDefinition. Mixinitialize is invoked during the composition of an instance.

Mixins defined this way can be used in any object in the factory by adding the mixins: ['MixinName'] option in the define options:
```
    // Lion.js
    factory.defineMixin('Lion', {
      roar: function(){
        console.log("ROOARR!!!");
      },
      mixinitialize: function(){
        console.log('It behaves like a lion', this);
      }
    });

    // Tiger.js
    factory.defineMixin('Tiger', {
      pounce: function(){
        console.log("POUNCE!!!");
      },
      mixinitialize: function(){
        console.log('It behaves like a tiger', this);
      }
    });

    // Liger.js
    factory.defineMixin('Liger', {
      magic: function(){
        console.log("PHWOOOOSH!!!");
      },
      mixinitialize: function(){
        console.log('It is known for it\'s skills in magick!', this);
      }
    }, {
      mixins: ['Lion', 'Tiger']
    });

    // FavoriteAnimal.js
    factory.define('FavoriteAnimal', Animal, {
      mixins: ['Liger']
    });
```
Mixins can also define a mixins array in their mixinSettings to depend on other mixins. You can compose behaviors together and apply them to objects at will.

Singletons
----------

Factory JS supports singletons as a native concept. Singletons are used in javascript because the application loads as the result of a single request. It is destroyed when the page changes location or reloads. This alleviates much of the concern of singletons in other languages. With longer running execution paths singletons can present problems with maintaining long term state.

To make a class a singleton just pass the singleton: true flag to the definition option. Once defined the first call to get will create the object and following calls will return the same instance. In general, singleton constructors do not take arguments. This prevents you from initializing the object with the wrong arguments.
```
    factory.define('TheOne', A, { singleton: true });

    var theone = factory.get('TheOne');
    factory.get('TheOne') === theone // => true
```
Evented Factory
---------------

One of the more interesting features of the factory is that it supports an evented interface. Let's take a look at the events you can bind on:

    - define: Factory emits this anytime you define a type.
    - defineMixin: Factory emits this when you define a mixin.
    - create: Factory emits this when you create an instance. Used for tag support.
    - dispose: Factory emits this when you dispose an instance.

These events support the AOP and mirroring features we will talk about later. You can also use them to depend on functionality or track object creation and disposal.

Tags
----

Tags mark definitions as being part of a group. Tags can reach across type and mixin boundaries to apply behaviors to objects. This allows you to do aspect oriented things like logging, error handling and security in a consistent way across object types in your application. This is the effective equivalent of executing a callback against any instance in memory that has a tag, and binding that same callback to be applied to any instance that comes into memory with the same tag.

Let's imagine that we have a large group of models and non model objects that rely on persistence strategies where the default is ajax. Let us also imagine that we have established an alternate strategy for when we are in maintenance mode where things will be routed to a local storage container for later upload. We have tagged all these model and non model objects with 'Persists'.
```JavaScript
// TODO: Also show how you set the tag in the factory

    // let's say we get a socket call informing us of the maintenance mode
    socket.on('change:mode', function(data){
      if (data.mode === 'maintenance') {
        // we need to switch to the offline strategy
        factory.offTag('Persists');
        factory.onTag('Persists', function(instance){
          instance.strategy('offline');
        });
      } else {
        // we need to switch back to the online strategy
        factory.offTag('Persists');
        factory.onTag('Persists', function(instance){
          instance.strategy('online');
        });
      }
    });
```
Now we don't need a global singleton to maintain this state, the objects can be tested in isolation as can the persistence behavior. Because onTag will execute against all the instances in memory and all instances created in the future we don't have to worry about the objects having the wrong state if they are created after maintenance mode is started or stopped.

Mirror
------

As you can see factory js is designed to take core object domains and wrap them together into a single access container. This is great unless you want to use multiple domains of objects in a single project. Let's say that you want to use a BackboneFactory (included in factory js) as well as defining your own factory of object definitions. You can easily do this by mirroring the BackboneFactory in your own factory, then any definitions that are added to the BackboneFactory will automatically be added to your factory.
```
    define(['Factory', 'BackboneFactory'], function(Factory, BackboneFactory){
      var MyFactory = new Factory(function(){...});

      MyFactory.mirror(BackboneFactory);
      MyFactory.hasDefinition('View'); // => true
      return MyFactory;
    });
```
This will allow you to compose factories from all over the place and utilize their mixins and definitions in your own factory without having to carry around multiple factories.

Summary
-------

TODO: Add a very brief summary here including another link to the github repo.