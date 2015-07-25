
---
title: JavaScript Instantiation Patterns
tags: Javascript, Angular, Backbone, Bookshelf.js
---

JavaScript lacks an explicit class constructor and, as a result, developers must rely upon other techniques for building classes. There are four commonly found instantiation patterns that can be used for constructing class objects. In this post I illustrate each instantiation pattern and discuss the pros and cons.

### Functional Instantiation
```
var Dog = function(name, age) {
  var dog = {};
  var name = name;
  var age = age;
 
  dog.bark = function() {
    console.log("woof");
  };
 
  dog.hasBirthday = function() {
    age++;
  };
 
  dog.age = function() {
    return age;
  };
 
  return dog;
};
```

In the functional instantiation pattern, class methods are defined in the constructor function. Each instance of our dog class has its own version of the class methods. This makes functional instantiation a memory-inefficient pattern - if you have many instances of a class declared at once, a large amount of memory might be squandered on remembering the same methods over and over again. Since all class methods have access to variables declared within the class object (complements of closure scope), it is not necessary to use the keyword this to acces class variables. For example, we can access the age variable directly in the hasBirthday function. In practice, this pattern is rarely used.

### Functional Shared Instantiation
```
var Cat = function(name, age) {
  var cat = {};
  cat.name = name;
  cat.age = age;
  // extend(cat, catMethods);
  return extend(cat, catMethods);
};
 
var catMethods = {};
 
catMethods.meow = function() {
  console.log("Meow!");
};
 
catMethods.hasBirthday = function() {
  this.age++;
}
 
var extend = function(baseObj) {
  for (var i = 1; i < arguments.length; i++) {
    for (var prop in arguments[i]) {
      baseObj[prop] = arguments[i][prop];
    }
  }
  return baseObj
};
```

The functional shared instantiation pattern is very similar to the functional instantiation pattern, but has one key difference: class methods are defined outside of the constructor. In the example above, I invoke an extend method which transfers the key value pairs from a method-containing object into the cat object that is returned when the Cat constructor is instantiated. Because class methods are defined outside of the class constructor, they must use the this keyword to access class variables.

### Prototypal Instantiation
```
var Cow = function(name, age) {
  var newCow = Object.create(cowMethods);
 
  newCow.name = name;
  newCow.age = age;
 
  return newCow;
}
 
var cowMethods = {};
 
cowMethods.moo = function() {
  console.log("Mooo!");
}
 
cowMethods.hasBirthday = function() {
  this.age++;
}
```

In the prototypal instantiation pattern, an object is created and returned within the class constructor (var newCow = Object.create(cowMethods);). This line of code establishes a delegation relationship between the Cow class and a method-containing object called cowMethods. The methods declared in the cowMethods object are not actually copied to Cow class objects that are instiantiated. Rather, cowMethods becomes the Cow class prototype. This means that if a new method is added to cowMethods once a Cow object has been instantiated, the Cow object is able to access the new method. If a conflicting method is defined directly within the class constructor, it will override the inherited method it interferes with.

### Pseudoclassical Instantiation

```
var Duck = function(name, age) {
  this.name = name;
  this.age = age;
}
 
Duck.prototype.quack = function() {
  console.log("Quack!");
}
 
Duck.prototype.hasBirthday = function() {
  this.age++;
}
```

The pseudoclassical instantiation pattern looks similar to the prototypal instantiation pattern, but methods are added directly into the class prototype, outside of the class constructor. This pattern is the most commonly used because it closely resembles class instantiation patterns found in other languages. It is also the simplest to write - you don't need to delegate the prototype to another object and the encapsulation is intuitive. When an object from a class using the pseudoclassical pattern is instantiated, make sure to use the new keyword (e.g. var daffey = new Duck("daffey", 25)).






