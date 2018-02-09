---
layout: post
title:  "TypeScript - An Overview"
author: Joshua Tanner
description: Using TypeScript for building more scalable JavaScript projects.
image: assets/images/tools.jpg
---

![test image]({{ site.url | absolute_path}}/assets/images/tools.jpg)

[ArcGIS-REST-JS](https://github.com/Esri/arcgis-rest-js/) is an open source library that Esri developers are actively working on that provdes an javascript abstraction interface to ArcGIS REST.  I'm interested in both using the library and contributing to it.  One initial hurdle... it's writtin in TypeScript, which I know very little about.  Time to roll up my sleeves and start learning.  Come join me along the way.

## What is TypeScript

[TypeScript](https://www.typescriptlang.org/) defines itself as being JavaScript that scales.  Unlike the regular interpreted language of JavaScript we are all used to, TypedScript is a compiled language.  Once compiled, the output is regular JavaScript.  TypeScript introduces concepts that aren't inherit to regular JavaScript programmers, like strict data types, interfaces, classes, and namespaces.  Although these can be implemented by design in JavaScript, they are not an inherit part of most traditional coding concepts for JavaScript developers.  However, modern JavaScript (ES2015, ES6) does have support for classes.

Next, let's go through the [tutorial here](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) for a quick introduction to TypeScript.  I'm also documenting the steps below.

## Installing TypeScript

Since TypeScript code will need to be compiled to JavaScript, we need to download the TypeScript compiler.

```
$ npm install -g typescript
```

The `-g` flag installs the nodejs package globally.

## Writing

Create a file called `greeter.ts` and write the following lines of code

```javascript
function greeter(person) {
  return "Hello, " + person;
}

let user = "Joshua Tanner";
```

So far, this looks like regular JavaScript.  We are however, using the `let` block level variable declaration.  This is available to modern JS engines, but not older browswers.  Let's see what happens if we compile this code.

```
$ tsc greeter.ts
$ cat greeter.js
```

TypeScript compiles greeter.ts into a new file, `greeter.js`.  Cat'ing it out to the console produces the following output:

```javascript
function greeter(person) {
    return "Hello, " + person;
}
var user = "Joshua Tanner";
```

Notice how the compiler converted the block level `let` variable to `var`.

## Typed Annotations

Using TypeScript we can start to have some better checking of our code.  Placing checks makes building software more robust, and the compiler can tell us ahead of time of there are issues with our code.  One thing we can check for is what kind of data type we expect go be passed as a parameter to a function.

In our previous example, we could pass any data type (string, boolean, array, object, etc) as a parameter to our `greeter` function.  Let's tighten this up by requiring that the parameter of of type `string`.

```javascript
function greeter(person: string) {
    return "Hello, " + person;
}

let user = [0, 1, 2];
```

The `person:string` defines that the person parameter must be of type `string`.  Let's see what happens if we try and compile this.

```
$ tsc greeter.ts
greeter.ts(7,35): error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.
```

Oops, tryint to pass the array `let user = [0, 1, 2];` threw an error.  The compiler still builds our greeter.js file, but warns us that there are problems.

## Interfaces

Interfaces are probably something you haven't worked with languages like Java or C#.  Essentially, an interface is a contract or definition about what something should look like that we can implement over and over again.

For example, a we might establish a person as an interface:

```typescript
interface Person {
  firstName: string,
  lastName: string
}
```

Let's build a simple program that has a function that implements this interface as a parameter:

```javascript
interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person: Person) {
  return "Hello, " + person.firstName + " " + person.lastName;
}

let user = { firstName: "Joshua", lastname: "Tanner"};
```

If we added `age` to our user object, the compiler would

## Classes

TypeScript supports class-based object-oriented programming with support for common patterns like inheritance.

For example:

```javascript
//classes.ts
class FourLeggedAnimal {
  legs: number;
  constructor() {
    this.legs = 4;
  }
}

class Cat extends FourLeggedAnimal {
  speak() {
    console.log('Meow! Meow!');
  }
}

class Dog extends FourLeggedAnimal {
  speak() {
    console.log('Ruff!  Ruff!');
  }
}

const cat = new Cat();
const dog = new Dog();

cat.speak();
dog.speak();

console.log(cat.legs == dog.legs);
```

You can run this directly in command line with nodejs:

```
$ tsc classes.ts
$ node classes.js
Meow! Meow!
Ruff!  Ruff!
true
```

## Modules

Another important feature of TypeScript is the use of modules.  If you have been working with modern JavaScript or nodejs, you should be familiar with the concept.  Modules allow us to export and import blocks of code between different javascript files.

You can read more about modules on [the TypeScript documentation guide](https://www.typescriptlang.org/docs/handbook/modules.html)

## Conclusion

When you are jumping into a project that uses TypeScript, don't be overwhelmed.  Most of your JavaScript knowldege will carry over just fine.  The essentials of TypeScript are easy to learn and their [documentation](https://www.typescriptlang.org/docs/handbook) is simple to follow for understanding more advanced features of the language.
