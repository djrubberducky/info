# S.O.L.I.D The first 5 principles of Object Oriented Design with JavaScript

![logo](https://cdn-images-1.medium.com/max/800/1*qfk6AFv4OF1GRd1WZJc25A.jpeg)

# S.O.L.I.D. STANDS FOR:
  - [**S**](#single-responsibility-principle) - Single responsibility principle
  - [**O**](#open-closed-principle) - Open closed principle
  - [**L**](#liskov-substitution-principle) - Liskov substitution principle
  - [**I**](#interface-segregation-principle) - Interface segregation principle
  - [**D**](#dependency-inversion-principle) - Dependency Inversion principle

## Single responsibility principle
    A class should have one and only one reason to change,
    meaning that a class should only have one job.

For example, say we have some shapes and we wanted to sum all the areas of the shapes. Well this is pretty simple right ?
```javascript
const circle = (radius) => {
  const proto = { 
    type: 'Circle',
    //code 
  }
  return Object.assign(Object.create(proto), {radius})
}
const square = (length) => {
  const proto = { 
    type: 'Square',
    //code 
  }
  return Object.assign(Object.create(proto), {length})
}
```
First, we create our **shapes** factory functions and setup the required parameters.

## What is a factory function ?
In JavaScript, any function can return a new object. When it’s not a constructor function or class, it’s called a factory function. [Why to use factory functions, this article provides a good explanation](https://medium.com/javascript-scene/javascript-factory-functions-vs-constructor-functions-vs-classes-2f22ceddf33e#.m5h2jj8a7) and [this video also explain it very clear](https://www.youtube.com/watch?v=ImwrezYhw4w)

Next, we move on by creating the **areaCalculator** factory function and then write up our logic to sum up the area of all provided shapes.
```javascript
const areaCalculator = (s) => {
  const proto = {
    sum() {
      // logic to sum
    },
    output () {
     return `
        <h1>
          Sum of the areas of provided shapes:
          ${this.sum()} 
        </h1>
      `;
    }
  }
  return Object.assign(Object.create(proto), {shapes: s})
}
```

To use the **areaCalculator** factory function, we simply call the function and pass in an array of shapes, and display the output at the bottom of the page.

```javascript
const shapes = [
  circle(2),
  square(5),
  square(6)
];

const areas = areaCalculator(shapes);
console.log(areas.output());
```

The problem with the output method is that the **areaCalculator** handles the logic to output the data. Therefore, what if the user wanted to output the data as json or something else?

All of the logic would be handled by the **areaCalculator** factory function, this is what ***‘Single Responsibility principle’*** frowns against; the **areaCalculator** factory function should only sum the areas of provided shapes, it should not care whether the user wants ***JSON*** or ***HTML***.

So, to fix this you can create an **SumCalculatorOutputter** factory function and use this to handle whatever logic you need on how the sum areas of all provided shapes are displayed.

The **sumCalculatorOutputter** factory function would work liked this:
```javascript
const shapes = [
  circle(2),
  square(5),
  square(6)
];

const areas  = areaCalculator(shapes);
const output = sumCalculatorOputter(areas);
console.log(output.JSON());
console.log(output.HAML());
console.log(output.HTML());
console.log(output.JADE())
```

Now, whatever logic you need to output the data to the users is now handled by the **sumCalculatorOutputter** factory function.

## Open-closed Principle
    Objects or entities should be open for extension,
    but closed for modification.
    
*Open for extension means that we should be able to add new features or components to the application **without breaking** existing code.*

*Closed for modification means that **we should not introduce breaking changes to existing functionality**, because that would force you to refactor a lot of existing code — [https://medium.com/@_ericelliott](Eric Elliott)*

In simpler words, means that a class or factory function in our case, should be easily extendable without modifying the class or function itself. Let’s look at the **areaCalculator** factory function, especially it’s **sum** method.
```javascript
sum () {
  const area = []
  
  for (shape of this.shapes) {
    if (shape.type === 'Square') {
      area.push(Math.pow(shape.length, 2)
    } else if (shape.type === 'Circle') {
      area.push(Math.PI * Math.pow(shape.length, 2)
    }
  }
  
  return area.reduce((v, c) => c += v, 0)
}
```

If we wanted the sum method to be able to **sum** the areas of more shapes, we would have to add more **if/else** blocks and that goes against the **open-closed principle**.

A way we can make this **sum** method better is to remove the logic to calculate the area of each shape out of the sum method and attach it to the shape’s factory functions.

```javascript
const square = (length) => {

  const proto = {
    type: 'Square',
    area () {
      return Math.pow(this.length, 2)
    }
  }
  
  return Object.assign(Object.create(proto), {length})
}
```

The same thing should be done for the **circle** factory function, an **area** method should be added. Now, to calculate the sum of any shape provided should be as simple as:
```javascript
sum() {
  const area = []
  
  for (shape of this.shapes) {
    area.push(shape.area());
  }
  
  return area.reduce((v, c) => c += v, 0);
}
```

Now we can create another shape class and pass it in when calculating the sum without breaking our code. However, now another problem arises, how do we know that the object passed into the **areaCalculator** is actually a shape or if the shape has a method named **area**?

Coding to an interface is an integral part of **S.O.L.I.D.**, a quick example is we create an interface, that every shape implements.

Since JavaScript doesn’t have interfaces, I’m going to show you, how this will be achieved in TypeScript, since TypeScript models the classic OOP for JavaScript, and the difference with pure JavaScript Prototypal OO.

```typescript
interface ShapeInterface { 
  area(): number 
}

class Circle implements ShapeInterface {     
  let radius: number = 0
 
  constructor (r: number) {
    this.radius = r
  }
 
  public area(): number {
    return MATH.PI * MATH.pow(this.radius, 2);
  }
}
```

In the example above demonstrates how this will be achieved in TypeScript, but under the hood TypeScript compiles the code to pure JavaScript and in the compiled code it lacks of interfaces since JavaScript doesn’t have it.

So how can we achieved this, in the lack of interfaces?

**Function Composition** to the rescue!

First we create **shapeInterface** factory function, as we are talking about interfaces, our shapeInterface will be as abstracted as an interface, using function composition, [for deep explanation of composition see this great video](https://www.youtube.com/watch?v=wfMtDGfHWpA).
```typescript
const shapeInterface = (state) => ({
  type: 'shapeInterface',
  area: () => state.area(state)
})
```

Then we implement it to our **square** factory function.
```typescript
const square = (length) => {

  const proto = {
    length,
    type: 'Square',
    area: (args) => Math.pow(args.length, 2),
  };
  
  const basics = shapeInterface(proto);
  const composite = Object.assign({}, basics);
  
  return Object.assign(Object.create(composite), {length});
}
```

And the result of calling the **square** factory function will be the next one:
```typescript
const s = square(5)
console.log('OBJ\n', s)
console.log('PROTO\n', Object.getPrototypeOf(s))
s.area()

/* output
  OBJ
   { length: 5 }
  PROTO
   { type: 'shapeInterface', area: [Function: area] }
  25
*/
```

In our **areaCalculator** sum method we can check if the shapes provieded are actually types of **shapeInterface**, otherwise we throw an exception:
```typescript
sum() {
  const area = []
  
  for (shape of this.shapes) {
    if (Object.getPrototypeOf(shape).type === 'shapeInterface') {
      area.push(shape.area())
    } else {
      throw new Error('this is not a shapeInterface object')
    }
  }
  
  return area.reduce((v, c) => c += v, 0)
}
```

and again, since JavaScript doesn’t have support for interfaces like typed languages the example above demonstrates how we can simulate it, but more than simulating interfaces, what we are doing is using closures and function composition if you don’t know [what are a closure is this article explains it very well](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36) and for [complementation see this video](https://www.youtube.com/watch?v=CQqwU2Ixu-U).

## Liskov substitution principle
    Let q(x) be a property provable about objects of x of type T.
    Then q(y) should be provable for objects y of type S
    where S is a subtype of T.
    
All this is stating is that every subclass/derived class should be substitutable for their base/parent class.

In other words, as simple as that, a subclass should override the parent class methods in a way that does **not break functionality from a client’s point of view**.

Still making use of our **areaCalculator** factory function, say we have a **volumeCalculator** factory function that extends the **areaCalculator** factory function, and in our case for extending an object without breaking changes in ***ES6*** we do it by using `Object.assign()` and the `Object.getPrototypeOf()`:
```javascript
const volumeCalculator = (s) => {
  const proto = {
    type: 'volumeCalculator'
  }
  const areaCalProto = Object.getPrototypeOf(areaCalculator())
  const inherit = Object.assign({}, areaCalProto, proto)
  return Object.assign(Object.create(inherit), {shapes: s})
}
```

## Interface segregation principle
    A client should never be forced to implement
    an interface that it doesn’t use or clients
    shouldn’t be forced to depend on
    methods they do not use.
    
Continuing with our shapes example, we know that we also have solid shapes, so since we would also want to calculate the volume of the shape, we can add another contract to the **shapeInterface**:
```javascript
const shapeInterface = (state) => ({
  type: 'shapeInterface',
  area: () => state.area(state),
  volume: () => state.volume(state)
})
```

Any shape we create must implemet the **volume** method, but we know that squares are flat shapes and that they do not have volumes, so this interface would force the **square** factory function to implement a method that it has no use of.

**Interface segregation principle** says no to this, instead you could create another interface called **solidShapeInterface** that has the **volume** contract and solid shapes like cubes etc. can implement this interface.
```javascript
const shapeInterface = (state) => ({
  type: 'shapeInterface',
  area: () => state.area(state)
})
const solidShapeInterface = (state) => ({
  type: 'solidShapeInterface',
  volume: () => state.volume(state)
})
const cubo = (length) => {
  const proto = {
    length,
    type   : 'Cubo',
    area   : (args) => Math.pow(args.length, 2),
    volume : (args) => Math.pow(args.length, 3)
  }
  const basics  = shapeInterface(proto)
  const complex = solidShapeInterface(proto)
  const composite = Object.assign({}, basics, complex)
  return Object.assign(Object.create(composite), {length})
}
```

This is a much better approach, but a pitfall to watch out for is when to calculate the sum for the shape, instead of using the **shapeInterface** or a **solidShapeInterface**.

You can create another interface, maybe **manageShapeInterface**, and implement it on both the flat and solid shapes, this is way you can easily see that it has a single API for managing the shapes, for example:
```javascript
const manageShapeInterface = (fn) => ({
  type: 'manageShapeInterface',
  calculate: () => fn()
})
const circle = (radius) => {
  const proto = {
    radius,
    type: 'Circle',
    area: (args) => Math.PI * Math.pow(args.radius, 2)
  }
  const basics = shapeInterface(proto)
  const abstraccion = manageShapeInterface(() => basics.area())
  const composite = Object.assign({}, basics, abstraccion)
  return Object.assign(Object.create(composite), {radius})
}
const cubo = (length) => {
  const proto = {
    length,
    type   : 'Cubo',
    area   : (args) => Math.pow(args.length, 2),
    volume : (args) => Math.pow(args.length, 3)
  }
  const basics  = shapeInterface(proto)
  const complex = solidShapeInterface(proto)
  const abstraccion = manageShapeInterface(
    () => basics.area() + complex.volume()
  )
  const composite = Object.assign({}, basics, abstraccion)
  return Object.assign(Object.create(composite), {length})
}
```

As you can see until now, what we have been doing for *interfaces in JavaScript* are factory functions for function composition.

And here, with **manageShapeInterface** what we are doing is abstracting again the **calculate** function, what we doing here and in the other interfaces (if we can call it interfaces), we are using *“high order functions”* to achieve the abstractions.

If you don’t know what a higher order function is you can go and [https://www.youtube.com/watch?v=BMUiFMZr7vk](check this video).

## Dependency inversion principle
    Entities must depend on abstractions not on concretions.
    It states that the high level module must
    not depend on the low level module,
    but they should depend on abstractions.
    
As a dynamic language, JavaScript doesn’t require the use of abstractions to facilitate decoupling. Therefore, the stipulation that *abstractions shouldn’t depend upon details* isn’t particularly relevant to JavaScript applications. The stipulation that *high level modules shouldn’t depend upon low level modules* is, however, relevant.

    From a functional point of view,
    these containers and injection concepts
    can be solved with a simple higher order
    function, or hole-in-the-middle type pattern
    which are built right into the language.

[http://softwareengineering.stackexchange.com/questions/103508/how-is-dependency-inversion-related-to-higher-order-functions](How is dependency inversion related to higher-order functions?) is a question asked in stackExchange if you want a deep explanation.

This might sound bloated, but it is really easy to understand. This principle allows for decoupling.

And we have made it before, lets review our code with the **manageShapeInterface** and how we accomplish the **calculate** method.
```javascript
const manageShapeInterface = (fn) => ({
  type: 'manageShapeInterface',
  calculate: () => fn()
});
```

What the **manageShapeInterface** factory function receives as the argument is a higher order function, that decouples for every shape the functionality to accomplish the needed logic to get to final calculation, let see how this is done in the shapes objects.
```javascript
const square = (radius) => {
  // code
 
  const abstraccion = manageShapeInterface(() => basics.area())
 
 // more code ...
}
const cubo = (length) => {
  // code 
  const abstraccion = manageShapeInterface(
    () => basics.area() + complex.volume()
  )
  // more code ...
}
```

For the square what we need to calculate is just getting the area of the shape, and for a cubo, what we need is summing the area with the volume and that is everything need to avoid the coupling and get the abstraction.

## Complete code examples

 - You can get it here: [solid.js](https://gist.github.com/Crizstian/ba39c6fdbb30a40c738e3c07ea83b5c1)
 
## Further reading and references
 - [SOLID the first 5 principles of OOD](https://scotch.io/bar-talk/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)
 - [5 Principles that will make you a SOLID JavaScript Developer](http://thefullstack.xyz/solid-javascript)
 - [SOLID JavaScript Series](http://aspiringcraftsman.com/2011/12/08/solid-javascript-single-responsibility-principle)
 - [SOLID principles using Typescript](https://samueleresca.net/2016/08/solid-principles-using-typescript)

## Conclusion
    “If you take the SOLID principles
    to their extremes, you arrive at
    something that makes Functional
    Programming look quite attractive”
                        - Mark Seemann

JavaScript is a multi-paradigm programming language, and we can apply the solid principles to it, and the great of it is that, we can combine it with the functional programing paradigm and get the best of both worlds.

Javascript is also a dynamic programming language, and very versatile
what i have presented is just a way of achieving this principles with JavaScript, they may be more better options in achieving this principles.

I hope you enjoyed this post, i’m currently still exploring the JavaScript world, so i am open to accept feedback or contributions, and if you liked it, recommend it to a friend, share it or read it again.

You can follow me #twitter [@cramirez_92](https://twitter.com/cramirez_92)

Until next time 😁👨🏼‍🎨👨🏻‍💻
