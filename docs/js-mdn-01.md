# JS-MDN-01

## Links
[Slide](https://goo.gl)

## Chapter #02 Grammar and types
This chapter discusses JavaScript's basic grammar, variable declarations, data types and literals.
___

### Basics
JavaScript borrows most of its syntax from Java, but is also influenced by Awk, Perl and Python.

JavaScript is case-sensitive and uses the Unicode character set. For example, the word Früh (which means "early" in German) could be used as a variable name.

```javascript
var Früh = "foobar";
```
But, the variable früh is not the same as Früh because JavaScript is case sensitive.

In JavaScript, instructions are called statements and are separated by semicolons (;).

A semicolon is not necessary after a statement if it is written on its own line. But if more than one statement on a line is desired, then they must be separated by semicolons. ECMAScript also has rules for automatic insertion of semicolons (ASI) to end statements. It is considered best practice, however, to always write a semicolon after a statement, even when it is not strictly needed. This practice reduces the chances of bugs getting into the code.

___

### Comments

The syntax of comments is the same as in C++ and in many other languages:

```javascript
// a one line comment
 
/* this is a longer, 
 * multi-line comment
 */

/* You can't, however, /* nest comments */ SyntaxError */
```
Comments behave like whitespace and are discarded during script execution.
___

###Declarations
There are three kinds of declarations in JavaScript.

* var
* let
* const

####var
The var statement declares a variable, optionally initializing it to a value. The scope of a variable declared with var is its current execution context.

```javascript
function varTest() {
 if (true) {
   var x = 2;  // scoped whole function!
   console.log(x);  // 2
 }
 console.log(x);  // 2
}
```

####let
let allows you to declare variables that are limited in scope to the block, statement, or expression on which it is used. 

```javascript
function letTest() {
 let x = 1;
 if (true) {
   let x = 2;  // different variable scoped within if block
   console.log(x);  // 2
 }
 console.log(x);  // 1
}

```

####const
Constants are block-scoped, much like variables defined using the let statement. The value of a constant cannot change through reassignment, and it can't be redeclared. 
The const declaration creates a read-only reference to a value.

```javascript
const obj = { 'key': 'value' };
obj = { 'anotherKey': 'value' };  //ERROR
obj.key = 'otherValue';  //Executes Without Problems

```

___

###Variables
You use variables as symbolic names for values in your application. The names of variables, called identifiers, conform to certain rules.

A JavaScript identifier must start with a letter, underscore (_), or dollar sign ($); subsequent characters can also be digits (0-9). Because JavaScript is case sensitive, letters include the characters "A" through "Z" (uppercase) and the characters "a" through "z" (lowercase).

You can use most of ISO 8859-1 or Unicode letters such as å and ü in identifiers. You can also use the Unicode escape sequences as characters in identifiers.

Some examples of legal names are Number_hits, temp99, $credit, and _name.

A variable declared using the var or let statement with no assigned value specified has the value of undefined.


```javascript
var a;
console.log('The value of a is ' + a); // The value of a is undefined

console.log('The value of b is ' + b); // The value of b is undefined - see 'Variable hoisting' below to know why
var b;

console.log('The value of c is ' + c); // Uncaught ReferenceError: c is not defined

let x;
console.log('The value of x is ' + x); // The value of x is undefined

console.log('The value of y is ' + y); // Uncaught ReferenceError: y is not defined
let y;
```

####Variable Scopes

When you declare a variable outside of any function, it is called a global variable, because it is available to any other code in the current document. When you declare a variable within a function, it is called a local variable, because it is available only within that function.

JavaScript before ECMAScript 2015 does not have block statement scope; rather, a variable declared within a block is local to the function (or global scope) that the block resides within. For example the following code will log 5, because the scope of x is the function (or global context) within which x is declared, not the block, which in this case is an if statement.

```javascript
if (true) {
  var x = 5;
}
console.log(x);  // x is 5
```

This behavior changes, when using the let declaration introduced in ECMAScript 2015.

```javascript
if (true) {
  let y = 5;
}
console.log(y);  // ReferenceError: y is not defined
```
___

###Undefined
A variable declared using the var or let statement with no assigned value specified has the value of undefined. The undefined value behaves as false when used in a boolean context.

```javascript
var input;
if (input === undefined) {
 doThis();  //This line will execute
} else {
 doThat();
}

```

___


###null
The value null represents the intentional absence of any object value. It is one of JavaScript primitive values. null is an assignment value. It can be assigned to a variable as a representation of no value.

The null value behaves as ‘0’ in numeric contexts and as false in boolean contexts.

```javascript
var n = null;
console.log(n * 32);   // Will log 0 to the console

```
___

###Variable Hoisting

The JavaScript engine treats all variable declarations using “var” as if they are declared at the top of a functional scope or global scope regardless of where the actual declaration occurs. This essentially is “hoisting”.
However, variables that are hoisted return a value of undefined. So even if you declare and initialize after you use or refer to this variable, it still returns undefined.

So variables might actually be available before their declaration.

```javascript
console.log(x); // Undefined
var x = 3;	// Using var
console.log(x); // 3


console.log(y);  // Error
let y = 5;	// Using let
console.log(y);  // 5
```
___

###Function Hoisting

For functions, only the function declaration gets hoisted to the top and not the function expression.

```javascript
/* Function declaration */

foo(); // "bar"

function foo() {
  console.log('bar');
}


/* Function expression */

baz(); // TypeError: baz is not a function

var baz = function() {
  console.log('bar2');
};
```
___

###JavaScript Data Types

The latest ECMAScript standard defines seven data types:

Primitive:

* __Boolean__		true and false

* __null__			A special keyword denoting a null value.

* __undefined__		A primitive value automatically assigned to variables that have just been declared.

* __Number__		integer or floating point number. 42 or 4.326

* __String__ 		A sequence of characters that represent a text value.

* __Symbol__		(new in ES6) A data type whose instances are unique and immutable.

and Object.

Although these data types are relatively few, they enable you to perform useful functions with your applications. Objects and functions are the other fundamental elements in the language. You can think of objects as named containers for values, and functions as procedures that your application can perform.

####Data type conversion

JavaScript is a dynamically typed language. That means you don't have to specify the data type of a variable when you declare it, and data types are converted automatically as needed during script execution.

```javascript
var answer = 42;		//Number
answer = 'Thanks for all the fish...'; //Same Variable String
```
In expressions involving numeric and string values with the + operator, JavaScript converts numeric values to strings.

```javascript
x = 'The answer is ' + 42 // "The answer is 42"
```

####Converting strings to numbers

In the case that a value representing a number is in memory as a string, there are methods for conversion.

* parseInt ( ) : 		Only returns whole numbers 
* parseFloat ( ) : 	Returns a floating point number.

An alternative method of retrieving a number from a string is with the + (unary plus) operator:

```javascript
x = '2.1' + '4.6' // '2.14.6'
x = (+'1.1') + (+'1.1') // 2.2
```
___

###Literals
You use literals to represent values in JavaScript. These are fixed values, not variables, that you literally provide in your script.

* Array Literals
* Boolean Literals ( true and false )
* Floating-points Literals
* Integers	
* Object Literals
* RegEx Literals (Pattern enclosed between slashes:  /ab+c/ )
* String Literals

####Array Literals
An array literal is a list of zero or more expressions, each of which represents an array element, enclosed in square brackets [ ] .

```javascript
var coffees = ['French Roast', 'Colombian', 'Kona'];
```

If you put two commas in a row, the array is created with undefined for the unspecified elements.

```javascript
var fish = ['Lion', , 'Angel'];       //length = 3
var myList = [, 'home', , 'school'];  //length = 4
```


####Numeric Literals
Integers can be expressed in decimal (base 10), hexadecimal (base 16), octal (base 8) and binary (base 2).

* A leading 0 (zero) on an integer literal, or a leading 0o (or 0O) indicates it is in octal.
* A leading 0x (or 0X) indicates a hexadecimal integer literal.
* A leading 0b (or 0B) indicates a binary integer literal.

```javascript
0, 117 and - 345  (decimal, base 10)
015, 0001 and - 0o77  (octal, base 8)
0x1123, 0x00111 and - 0xF1A7  (hexadecimal, "hex" or base 16)
0b11, 0b0011 and - 0b11 (binary, base 2)

```

####Floating
A floating-point literal can have the following parts:

* A decimal integer which can be signed (preceded by "+" or "-")
* A decimal point ( ‘.‘ )
* A fraction (another decimal number)
* An exponent. ( "e" or "E" followed by an integer )

```javascript
3.1415926     //Decimal
-.123456789   //Signed
-3.1E+12      //Exponent
.1e-23
```

####Object Literals
An object literal is a list of zero or more pairs of property names and associated values of an object, enclosed in curly braces { } . 

```javascript
var car = { myCar: 'Saturn', getCar: carTypes('Honda'), special: sales };

console.log(car.myCar);   // Saturn
console.log(car.getCar);  // Honda
console.log(car.special); // Toyota
```

Property names that are not valid identifiers can be accessed and set with the array-like notation [ ] .

```javascript
var unusualPropertyNames = {
 '': 'An empty string',
 '!': 'Bang!'
}
console.log(unusualPropertyNames.'');   // SyntaxError: Unexpected string
console.log(unusualPropertyNames['']);  // An empty string
console.log(unusualPropertyNames.!);    // SyntaxError: Unexpected token !
console.log(unusualPropertyNames['!']); // Bang!
```

####String Literals
A string literal is zero or more characters enclosed in double (") or single (') quotation marks.

```javascript
'foo' , "bar" , '1234', "John's cat"
```

In addition to ordinary characters, you can also include special characters in strings preceding it with a backslash. They are known as escape characters.

```javascript
'one line \n another line'
'C:\\temp' //including double slash to put literal slash in string
"He read \"The Cremation of Sam McGee\" by R.W. Service.";
```

___

> THANK YOU!