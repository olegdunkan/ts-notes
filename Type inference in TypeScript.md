# Some notes on type argument inference in Typescript

Let say we have this snippet:

```ts
function f<T>(x:T) {
  return x;
}
```
`f` is a **generic function** with the **type parameter** `T`. [Spec](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#6.5) <br>
`T` is a type. [Spec](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#3)<br>
`T` can be used in a parameter list, in return type annotation and in a function body.<br>
In our example we use `T` in parameter list as **parameter type**.<br>

When we write this:

```ts
f<number>(1);
```

We substitute `T` with `number`. `number` is a **type argument** in our call expression.
We use the type `number` as a type argument explicitly. <br>
But we can do that implicitly:
```ts
f(1);
```
Typescript inferes that `T` is a `number`.<br>

How can we use functions with type parameters, what for?
Example above allows us to use function like identity function keeping type safety.
```ts
class A {
  x:number;
}
function identity<T>(x:T) {
  return x;
}
identity(new A).x = 0; 
```
Typescript sees that the return type of the `identity` function is `A`.<br>
If we wouldn't have generic functions in typescript then in order to keep identity functionality we would write the identity function 
for each type to keep type checking or would use overload resolution to make it shorter.   
```ts 
function identityA(x:A) {
  return x;
}
function identityB(x:B) {
  return x;
}
//or 
function identity(x:A);
function identity(x:B);
function identity(x:A|B) {
  ...
}
```
Identity function is one of usage of generic functions without contrains on type parameter.
Additionally we can write generic functions without constrains on type parameters to produce some values.
For example:
```ts
function produceTuple<T>(x:T) {
  return [x, x]; 
}
produceTuple(new A);
//or
function produceArray<T>(x:T, size:number) {
  const a:T[];
  
  for (let i = 0; i < size; i++) {
    a.push(x);
  }
  return a; 
}
produceArray(new A, 3);

```
We can expand our field of usage of generic function by introducing constrains on type parameters.
```ts
interface IArea {
  toArea():number;
}

class Rectangle implements IArea {
    constructor(public x: number, public y: number) {}
  toArea() {
    return this.x * this.y;
  }
}
class Circle implements IArea {
  constructor(public r:number) {}
  toArea() {
    return Math.PI * this.r * this.r;
  }
}

function sum<T extends IArea>(args:T[]) {
    return args.reduce((x, y) => x + y.toArea(), 0);
}

sum([new Rectangle(2, 2), new Circle(2)]);

//but if we would use the rest parameter
function sum<T extends IArea>(...args:T[]) {
    return args.reduce((x, y) => x + y.toArea(), 0);
}
//we would get error
sum(new Rectangle(2, 2), new Circle(2)); //error
//we will know why it was an error later 
```
We have changed **type parameter list** adding constraint `IArea` on `T`.<br>
**Constraint** is an **object type** in most broad sence. [Spec](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#361-type-parameter-lists)<br>
**Object type** is a main building block of whole type system in typescript. [Spec](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#1.3)<br>
Object type has members (properties, index (number and string) signatures, call signatures and construct signatures)<br>
`IArea` is the name that has **type meaning** and reference to this name is an object type that has one `toArea` member (property) described as a **method signature**. <br>
(Technically word `IArea` is a **type reference** in `var i:IArea` declaration.)   
That means we can expect that `T` type will have all members declared in `IArea`. 
And all **named values** (parameters and variables declared inside function body) associated with type 'T', can access public members of `IArea`.
```ts
class A {
  private x:number;
  y:number; //by default public
}

class B {
  y:number; 
}

function f<T extends A>(x:T) {
  x.x; //error x is private
  x.y; //ok
}

f(new A); //ok
f(new B); //error, property 'x' is missing in type B
```
Now we know what is the **object type, generic function, type parameter list, function parameter list, type parameter, parameter type, type argument and constraint.**  
We can go further to explore the type argument inference more closely.




