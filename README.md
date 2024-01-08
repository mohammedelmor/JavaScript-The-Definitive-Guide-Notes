# JavaScript: The Definitive Guide

My notes for JavaScript: The Definitive Guide

# Objects 

Any value in JavaScript is an `object` except:
- string
- number
- Symbol
- true or false 
- null
- undefined 


## Object are mutable
Objects are mutable and manipulated by reference rather than by value

```javascript
let x = {}
let y = x; // y holds a reference to the same object x references
```
Any modifications made to the object through the variable y are also visible through the variable x.

## Object property attributes

Each property has the following property attributes:
- name
- value
- The writable attribute specifies whether the value of the property can be set.
- The enumerable attribute specifies whether the property name is returned by a for/in loop
- The configurable attribute specifies whether the property can be deleted and whether its attributes can be altered.

Many of JavaScript’s built-in objects have properties that are read-only, non-enumerable, or non-configurable. By default, however, all properties of the objects you create are writable, enumerable, and configurable.

## Creating Objects

### Object Literals

```javascript
let empty = {};                          // An object with no properties
let point = { x: 0, y: 0 };              // Two numeric properties
let p2 = { x: point.x, y: point.y+1 };   // More complex values
let book = {
    "main title": "JavaScript",          // These property names include spaces,
    "sub-title": "The Definitive Guide", // and hyphens, so use string literals.
    for: "all audiences",                // for is reserved, but no quotes.
    author: {                            // The value of this property is
        firstname: "David",              // itself an object.
        surname: "Flanagan"
    }
};
```

An object literal is an expression that creates and initializes a new and distinct object each time it is evaluated. The value of each property is evaluated each time the literal is evaluated. This means that a single object literal can create many new objects if it appears within the body of a loop or in a function that is called repeatedly
```javascript
let o1 = {a: 1};
let o2 = {a: 1};
console.log(o1 === o2) // false
```

### Creating Objects with new

```javascript
let o = new Object();  // Create an empty object: same as {}.
let a = new Array();   // Create an empty array: same as [].
let d = new Date();    // Create a Date object representing the current time
let r = new Map();     // Create a Map object for key/value mapping
```
The `new` keyword must be followed by a function invocation. A function used in this way is called a constructor and serves to initialize a newly created object

### Prototypes

Almost every JavaScript object has a second JavaScript object associated with it. This second object is known as a prototype, and the first object inherits properties from the prototype

> All objects created by object literals have the same prototype object which is `Object.prototype`

> Objects created using the new keyword and a constructor invocation use the value of the prototype property of the constructor function as their prototype
```javascript
let a = new Object(); // Object.prototype
let o = {}; // // Object.prototype also
let b = new Array(); // Array.prototype
let c = new Date(); // Date.prototype
```
So the object created by new Object() inherits from `Object.prototype` and so on...

> Almost all objects have a prototype, but only a relatively small number of objects have a prototype property. It is these objects with prototype properties that define the prototypes for all the other objects.

#### Prototype chain
`Object.prototype` is one of the rare objects that has no prototype: it does not inherit any properties. Other prototype objects are normal objects that do have a prototype. Most built-in constructors (and most user-defined constructors) have a prototype that inherits from `Object.prototype`. For example, `Date.prototype` inherits properties from `Object.prototype`, so a Date object created by `new Date()` inherits properties from both `Date.prototype` and `Object.prototype`. This linked series of prototype objects is known as a **`prototype chain`**.

### Object.create()
`Object.create()` creates a new object, using its first argument as the prototype of that object:
```javascript
let o1 = Object.create({x: 1, y: 2});     // o1 inherits properties x and y.
o1.x + o1.y                               // => 3
```

You can pass null to create a new object that does not have a prototype, but if you do this, the newly created object will not inherit anything, not even basic methods like toString() (which means it won’t work with the + operator either):

```javascript
let o2 = Object.create(null);             // o2 inherits no props or methods.
```

so to create a normal object just like using Object Literals `{}` or using `new` as in `new Object()`, pass `Object.prototype` for `Object.create()`:
```javascript
let o3 = Object.create(Object.prototype); // o3 is like {} or new Object().
```

## Objects Inheritance

> Prototypes enables inheritance in JavaScript.

```javascript
let o = {};               // o inherits object methods from Object.prototype
o.x = 1;                  // and it now has an own property x.
let p = Object.create(o); // p inherits properties from o and Object.prototype
p.y = 2;                  // and has an own property y.
let q = Object.create(p); // q inherits properties from p, o, and...
q.z = 3;                  // ...Object.prototype and has an own property z.
let f = q.toString();     // toString is inherited from Object.prototype
q.x + q.y                 // => 3; x and y are inherited from o and p
```

Property assignment examines the prototype chain only to determine whether the assignment is allowed. If o inherits a read-only property named x, for example, then the assignment is not allowed. If the assignment is allowed, however, it always creates or sets a property in the original object and never modifies objects in the prototype chain. The fact that inheritance occurs when querying properties but not when setting them is a key feature of JavaScript because it allows us to selectively override inherited properties:
```javascript
let o = {x: 1}
let o1 = Object.create(o); // set o as the prototype of o1
console.log(o1.x) // => 1,  o1 ----inherits----> from o

o1.x = 100 // this will not modify o object it will create new property in o1
console.log(o1.x)
```

## Property Access Errors
It's not an error to query non-existing property, it will simply result in undefined
but you can't query `null` or `undefined` values as they don't have any properties and it is an error to query properties of these values
```javascript
let o = {
    person: null
}

console.log(o.data) // undefined
console.log(o.person.name) // error
```

Property access expressions will fail if the lefthand side of the . is `null` or `undefined`

## Deleting Properties

`delete` does not operate on the value of the property but on the property itself:
```javascript
let obj = {a: 1}
console.log(obj.a) // 1
console.log(delete obj.a) // A delete expression evaluates to true if the delete succeeded
console.log(obj.a) // undefined
```

The delete operator only deletes own properties, not inherited ones. (To delete an inherited property, you must delete it from the prototype object in which it is defined. Doing this affects every object that inherits from that prototype.)

```javascript
let o = {x: 1};    // o has own property x and inherits property toString
delete o.x         // => true: deletes property x
delete o.x         // => true: does nothing (x doesn't exist) but true anyway
delete o.toString  // => true: does nothing (toString isn't an own property)
delete 1           // => true: nonsense, but true anyway
```

delete does not remove properties that have a configurable attribute of false. Certain properties of built-in objects are non-configurable, as are properties of the global object created by variable declaration and function declaration. In strict mode, attempting to delete a non-configurable property causes a TypeError. In non-strict mode, delete simply evaluates to false in this case:
```javascript
// In strict mode, all these deletions throw TypeError instead of returning false
delete Object.prototype // => false: property is non-configurable
var x = 1;              // Declare a global variable
delete globalThis.x     // => false: can't delete this property
function f() {
}         // Declare a global function
delete globalThis.f     // => false: can't delete this property either

console.log(x) // 1
console.log(f) //  f() {} 
```

## Object Testing Properties Existence

To check whether an object has a property with a given name. You can do this with the:
- `in` operator
- `hasOwnProperty()` 
- `propertyIsEnumerable()` 
- querying the property

###  `in` operator
Returns 
> `true` if the object has an own property or an inherited property by that name:
> 
> `false` otherwise
```javascript
let o = {x: 1};
"x" in o         // => true: o has an own property "x"
"y" in o         // => false: o doesn't have a property "y"
"toString" in o  // => true: o inherits a toString property from Object.prototype
```


###  `hasOwnProperty()`
Returns:
> `true` for own properties
> 
> `false` for inherited properties
> 
> `false` otherwise
```javascript
let o = {x: 1};
o.hasOwnProperty("x")        // => true: o has an own property x
o.hasOwnProperty("y")        // => false: o doesn't have a property y
o.hasOwnProperty("toString") // => false: toString is an inherited property
```

###  `propertyIsEnumerable()`
Returns:
> `true` only if the named property is an own property and its enumerable attribute is true
>
> `false` for inherited properties
>
> `false` otherwise
```javascript
let o = {x: 1};
let o = { x: 1 };
o.propertyIsEnumerable("x")  // => true: o has an own enumerable property x
o.propertyIsEnumerable("toString")  // => false: not an own property
Object.prototype.propertyIsEnumerable("toString") // => false: not enumerable
Object.prototype.hasOwnProperty("toString") // true
```

###  Querying the property

```javascript
let o = {x: 1};
o.x !== undefined        // => true: o has a property x
o.y !== undefined        // => false: o doesn't have a property y
o.toString !== undefined // => true: o inherits a toString property
```

#### `in` operator vs Querying the property

```javascript
let o = {x: undefined};  // Property is explicitly set to undefined
o.x !== undefined          // => false: property exists but is undefined
o.y !== undefined          // => false: property doesn't even exist
"x" in o                   // => true: the property exists
"y" in o                   // => false: the property doesn't exist
delete o.x;                // Delete the property x
"x" in o                   // => false: it doesn't exist anymore
```


## Enumerating Properties

`for/in` loop
> loop once for each enumerable property (own or inherited) of the specified object,

```javascript
let o = {x: 1, y: 2, z: 3};          // Three enumerable own properties
o.propertyIsEnumerable("toString")   // => false: not enumerable
for (let p in o) {                    // Loop through the properties
    console.log(p);                  // Prints x, y, and z, but not toString
}
```

alternatives:
- `Object.keys()`
- `Object.getOwnPropertyNames()`
- `Object.getOwnPropertySymbols()`
- `Reflect.ownKeys()`

## Serializing Objects
Object serialization is the process of converting an object’s state to a string from which it can later be restored. The functions `JSON.stringify()` and `JSON.parse()` serialize and restore JavaScript objects.

```javascript
let o = {x: 1, y: {z: [false, null, ""]}}; // Define a test object
let s = JSON.stringify(o);   // s = '{"x":1,"y":{"z":[false,null,""]}}'
let p = JSON.parse(s);       // p = {x: 1, y: {z: [false, null, ""]}}
```