

class: middle, center

# Crafting an Interpreter

---

class: middle, center
## Motivation

---

## Why?

--

* *Understanding!*

--

  Gaining insights into how
  * programming languages
  * compilers
  * VM's
  * computers

  work and why they're made the way they are.

--

* A test of programming skill.

--

  Become a better programmer.

--

* A frequent need of DSL's.

--

  * Scripting languages
  * Template engines
  * Markup formats
  * Configuration files
  * Etc..

---

class: quiz
## Quiz!

--

* Know what a compiler does?

--

* What does it do?

--

* What doesn't it do?

---

## Compiler vs. Interpreter

Subtle differences.

Definitions:

--

 * Compiler

--

  Translating from one source to another, e.g. bytecode, machine code.

--

 * Interpreter
 
--

  Executing source code directly.

---

class: middle, center
![](http://www.craftinginterpreters.com/image/a-map-of-the-territory/venn.png)

---


class: middle, center

We'll be discussing...

--

# Lox

--

... actually jlox

---

## Lox

We are making an interpreter.

--

A programming language being executed in a host language (Java).

--

Specifically a "tree-walk interpreter" (executing code as the AST is traversed).

--

<br/>
_Disclaimers!_

--

* This talk is _not_ a technical guide to making an interpreter. It's a look at _some_ of the interesting aspects of the process.

--

* I'm not an expert in compilers/interpreters.

--

* Please interrupt with questions.

--

* There's a test at the end!

--

* _We have to be quick!_ (⊙_☉)

---

class: middle, center

```lox
print "hello world";
```

---

## What is Lox?

* A high-level language with a familiar syntax.

* Dynamically typed. Types: Boolean, Number, String, Nil

* Functions as first class. Lambda functions/closures. Anonymous functions.

* Object oriented. Classes. Inheritance.

![](http://www.craftinginterpreters.com/image/the-lox-language/class-lookup.png)

* Standard "library".

---

class: gray
# Demo Time!

--

### Hangman demo & source code

---

class: middle, center
Enough introduction!

Let's get to...

---

class: middle, center
# The Journey

---

## The Journey

![](http://www.craftinginterpreters.com/image/a-map-of-the-territory/mountain.png)

---

## The Journey

Phases of an interpreter:

--

1. **Scanning** (lexing): <tt>Source code ➠ Tokens</tt>

--

1. **Parsing**: <tt>Tokens ➠ AST</tt>

--

1. **Static analysis**: <tt>AST ➠ AST</tt>

--

1. **Type checking**: <tt>AST ➠ Typed AST</tt>

--

1. **Optimizing**: <tt>Typed AST ➠ Typed AST</tt>

--

1. _Compiler_: **Code generation**: <tt>Typed AST ➠ Target code</tt>

--

1. _Interpreter_: **Interpreting**: <tt>Typed AST ➠ Execution</tt>

<!-- TODO: Add more phases here? -->

---

## Scanning

Converting raw source code into _tokens_.

--

![](http://www.craftinginterpreters.com/image/scanning/lexigator.png)

---

## Scanning

From ...

![](http://www.craftinginterpreters.com/image/a-map-of-the-territory/string.png)

to ...

![](http://www.craftinginterpreters.com/image/a-map-of-the-territory/tokens.png)

--

Not that complex to implement.

Can be done with a single-pass scanning with only a single character lookahead (i.e. fast).


---

## Parsing

For parsing, we need to define two things:

--

* An evaluation order of expressions (language grammar)

--

* A representation of code/expressions (structure)

---

## Parsing: Language Grammar

Defining the language grammar. It is a context free grammar consisting of nonterminals and terminals.

--

```log
expression → literal | unary | binary | grouping ;
literal    → NUMBER | STRING | "true" | "false" | "nil" ;
grouping   → "(" expression ")" ;
unary      → ( "-" | "!" ) expression ;
binary     → expression operator expression ;
operator   → "==" | "!=" | "<" | "<=" | ">" | ">=" | "+"  | "-"  | "*" | "/" ;
```

--

Recursive grammar. E.g. `expression` → `grouping` → `"(" expression ")"`.

--

Defines a recursive structure.

---

## Parsing: Language Grammar

The recursive data structure is a natural fit for the recursive nature of the grammar.

It's called an _Abstract Syntax Tree_ (AST).

--

For instance:

```lox
1 + 2 * 3 - 4
```

![](http://www.craftinginterpreters.com/image/representing-code/tree-evaluate.png)

<!-- _Post-order_ traversal. -->

---

## Parsing: Language Grammar

We want to avoid ambiguity when parsing.

--

For instance:

![](http://www.craftinginterpreters.com/image/parsing-expressions/tokens.png)

... can be parsed as either ...

![](http://www.craftinginterpreters.com/image/parsing-expressions/syntax-trees.png)

---

## Parsing: Language Grammar

We can avoid ambiguity by having operator precedence.

A common solution is to use _Recursive Decent Parsing_ (top-down parser).

![](http://www.craftinginterpreters.com/image/parsing-expressions/direction.png)

---

## Parsing: Syntax Trees

<!-- The AST is our representation of code for the parser to produce and the interpreter to consume. -->

The nodes in the AST is typed and can have meta-data.

![](http://www.craftinginterpreters.com/image/a-map-of-the-territory/ast.png)

---

## Parsing: Handling Syntax Errors

What we want to happen when encountering syntax errors:

--

 * Detect and report errors.

--

 * Don't crash.

--

 * Report many distinct errors.

--

 * Minimize cascading errors.

---

background-image: url(http://www.craftinginterpreters.com/image/parsing-expressions/panic.png)

---

## Parsing: Panic Mode

_Panic mode_ is an error reporting technique.

Panic mode is entered when the parser detects a syntax error.

--

Panic mode error recovery is a process called _synchronization_. It synchronize the next tokens to match rules.

--

<br/>
Synchronization process:

 * Mark a synchronization point

  * Jump out of nested productions & discard tokens.

  * Usually between statements.

  * Implemented as an exception that unwinds the parser.

<!-- Some errors don't need panic mode (e.g. exceeding max number of function parameters). -->

---

## Evaluating Expressions

--

![](http://www.craftinginterpreters.com/image/evaluating-expressions/lightning.png)

---

## Evaluating Expressions

![](http://www.craftinginterpreters.com/image/evaluating-expressions/skeleton.png)

---

## Evaluating Expressions

We now need to evaluate expressions to start interpreting.

--

![](http://www.craftinginterpreters.com/image/a-map-of-the-territory/ast.png)

---

## Evaluating Expressions

To evaluate the AST, do a recursive _post-order_ traveral using the _Visitor_ pattern.

![](http://www.craftinginterpreters.com/image/representing-code/tree-evaluate.png)

--

A missing aspect: The _environment_.

---

## Evaluating Expressions: Environment

Keeping track of scoped variables.

![](http://www.craftinginterpreters.com/image/statements-and-state/environment.png)

---

## Evaluating Expressions: Environment

Scoping:

```lox
{
  var a = "first";
  print a; // "first".
}

{
  var a = "second";
  print a; // "second".
}
```

![](http://www.craftinginterpreters.com/image/statements-and-state/blocks.png)

---

## Evaluating Expressions: Environment

Nested scoping:

```lox
var global = "outside";
{
  var local = "inside";
  print global + local;
}
```

![](http://www.craftinginterpreters.com/image/statements-and-state/chaining.png)

--

Defines a recursive structure for our environment.

---

class: quiz
## Quiz!

What does this code print?
--

```lox
var a = 1;
{
  var a = a + 2;
  print a;
}
```

--

<br/>

Answer: `3`. Should it be?

--

No, because `a` in `a + 2` referers to the inner `a`, which -- when referenced in the variable initializer -- will always be an uninitialized variable.

We'll fix it in the static analysis phase!

---

## Control Flow

To make our language Turing-complete we need to add conditionals.

Loops would also be nice.

We'll skip the details of implementing this.

---

class: quiz
## Quiz!

Does the `else` belong to the `first` or `second` `if`-statement?

--

```c++
if (first) if (second) whenTrue(); else whenFalse();
```

--

<br/>

Answer: Ambiguious!

![](http://www.craftinginterpreters.com/image/control-flow/dangling-else.png)

--

Most languages simply choose the nearest `if`.

---

## Control Flow: Desugaring

Desugaring for _fun_.

<center><img src="http://www.craftinginterpreters.com/image/control-flow/sugar.png" style="width:60%"/></center>

---

class: quiz
## Quiz!

* What is syntactic sugar?

--

**Answer:** Code syntax that is not _required_ but makes the code more pleasant to write.

--

* What is desugaring?

--

**Answer:** The process of converting pleasant code to "unpleasant" code.

---

## Control Flow: Desugaring

Suppose we have implemented `while`-loops.

We can then implement `for`-loops by converting them to `while`-loops.

--

For instance, we can rewrite

```lox
for (var i = 0; i < 10; i = i + 1) {
  print i;
}
```

as 

```lox
{
  var i = 0;
  while (i < 10) {
    print i;
    i = i + 1;
  }
}
```

---

## Functions

<center><img src="http://www.craftinginterpreters.com/image/functions/lambda.png" style="width:70%"/></center>

---

## Functions

Function name must be bound in the environment (same as variables).

--

When calling a function we need to validate _arity_.

```lox
fun a(b, c) {}
a(1,2);
```

--

Handle `return` statements that affect the flow of evaluation.

```lox
fun a() {
  while (true) {
    if (x > 2) return;
    //...
  }
}
```

Returning from inside loops can be implemented using exceptions.

---

## Functions: Binding Environment

```lox
fun add(a, b, c) {
  print a + b + c;
}

add(1, 2, 3);
```

![](http://www.craftinginterpreters.com/image/functions/binding.png)

---

## Functions: Binding Environment

The beautiful code that implements this (from `lox/LoxFunction.java`):

```Java
@Override
public Object call(Interpreter interpreter, List<Object> arguments) {
  Environment environment = new Environment(this.closure);
  for (int i = 0; i < this.declaration.params.size(); i++) {
    environment.define(this.declaration.params.get(i).lexeme, arguments.get(i));
  }

  interpreter.executeBlock(this.declaration.body, environment);
  return null;           
}
```

--
<br/>
Pseudo-code:
```java
function execute_function_call(interpreter, arguments[]) {
  var environment = new Environment()
  for param in parameters {
    environment.define_variable(param.lexeme, arguments[i])
  }

  interpreter.execute_block(function_body, environment)
}
```

---

## Functions: Local Functions and Closures

.pull-left[
```lox
fun makeCounter() {
  var i = 0;
  fun count() {
    i = i + 1;
    print i;
  }

  return count;
}

var counter = makeCounter();
counter(); // "1".
counter(); // "2".
```
]

--

.pull-right[
![](http://www.craftinginterpreters.com/image/functions/closure.png)
]

---

## Scope Resolving and Binding

Semantic analysis to ensure semantic correctness.

--

Traverse the AST with a Visitor, ensuring that all references are accessible from the different scopes.

---

## Static Analysis

Static analysis works almost like an interpreter.

--

... exceptions:

* No side effects.

* No control flow.

* Visiting _all_ branches exactly once, ignores short-curcuiting.

--

Examples of static checks:

* Disallow using local variables in its own initializer. I.e. `var a = a + 1;`

* Disallow dublicate variable names in the same scope. I.e. `var a; var a;`

* Disallow invalid return statements (i.e. outside functions).

* Disallow unused variables.

* Disallow `this` outside method.

* (...)

---

## Classes

A container of fields for variables and functions.

<center><img src="http://www.craftinginterpreters.com/image/classes/circle.png" style="width:70%"/></center>

---

## Classes

We need to implement:

* _Get_ expressions, e.g. `print person.name;`

* _Set_ expressions, e.g. `person.name = "John";`

* Methods, e.g. `person.sayHello();`

  * `this`

---

## Classes: Method calls

![](http://www.craftinginterpreters.com/image/classes/method.png)

---

## Classes: _Get_ expression

![](http://www.craftinginterpreters.com/image/classes/zip.png)

---

## Classes: _Set_ expression

![](http://www.craftinginterpreters.com/image/classes/setter.png)

---

class: quiz
## Quiz!

Should it be possible to "detach" methods from objects?

```lox
var m = object.method;
m(argument);
```

--

<br/>

In Lox: Yes.

---

class: quiz
## Quiz!

What about this?

```lox
class Box {}

fun notMethod(argument) {
  print "called function with " + argument;
}

var box = Box();
box.method = notMethod;
box.method("argument");
```

--

<br/>

In Lox: Yes.

---

class: quiz
## Quiz!

What is printed here?

```lox
class Person {
  sayName() {
    print this.name;
  }
}

var jane = Person();
jane.name = "Jane";

var bill = Person();
bill.name = "Bill";

bill.sayName = jane.sayName;
bill.sayName(); // ?
```

--

<br/>

In Lua and JavaScript: `"Bill"`. ("Methods" are really functions fields)

--

In Python, C#, Lox: `"Jane"`. (Methods are bound to objects)

---

## Classes: _this_

```lox
class Egotist {
  speak() {
    print this;
  }
}

var method = Egotist().speak;
method();
```
Should print `"Egotist instance"`.

--

We need `this` as a function variable.

--

We need functions to be _closures_.

---

## Classes: Constructors

Constructors ensure that objects start in a valid state.

* Allocate memory for a fresh instance.

* User-provided code to initialize the new object.

* Special method `init` is automatically invoked, e.g. `var p = Person();`

---

## Inheritance

```js
class Doughnut {
  // General doughnut stuff...
}

class BostonCream < Doughnut {
  // Boston Cream-specific stuff...
}
```

![](http://www.craftinginterpreters.com/image/inheritance/doughnuts.png)

---

## Inheritance: Methods

```js
class Doughnut {
  cook() {
    print "Fry until golden brown.";
  }
}

class BostonCream < Doughnut {}

BostonCream().cook();
```

---

## Inheritance: Calling Superclass Methods

```js
class Doughnut {
  cook() {
    print "Fry until golden brown.";
  }
}

class BostonCream < Doughnut {
  cook() {
    super.cook();
    print "Pipe full of custard and coat with chocolate.";
  }
}

BostonCream().cook();
```

--
<br/>
Prints:

<pre>
Fry until golden brown.
Pipe full of custard and coat with chocolate.
</pre>

---

class: quiz
## Quiz!

What is printed?

```lox
class A {
  method() {
    print "A method";
  }
}

class B < A {
  method() {
    print "B method";
  }

  test() {
    super.method();
  }
}

class C < B {}

C().test();
```

--

<br/>
Answer: `"A method"`

---

## Inheritance: Calling Superclass Methods

.pull-left[
```lox
class A {
  method() {
    print "A method";
  }
}

class B < A {
  method() {
    print "B method";
  }

  test() {
    super.method();
  }
}

class C < B {}

C().test();
```
]

--

.pull-right[
Inside `test()`, `this` is an instance of `C`.

Lookup should start on _the superclass of the class containing the super expression_.

![](http://www.craftinginterpreters.com/image/inheritance/classes.png)
]

---

## Inheritance: Some pitfalls

```lox
class Oops < Oops {}
```

```lox
var NotAClass = "I am totally not a class";

class Subclass < NotAClass {} // ?!
```

```lox
class NoSuperclass {
  method() {
    super.method();
  }
}
```

```lox
super.notEvenInAClass();
```

```lox
class A < B {}
class B < A {}
```

---

class: middle, center
# Bonus section!

--
<br/>
_Disclaimer:_ <br/><span style="font-size:5em;">🤷🏻‍♂️</span>

---

# Bonus section

* Type Checking
* Bytecode Generation
* Virtual Machine
* Front-end and back-end optimization

---

class: quiz
# Bonus: Type Checking

--

What is the return _type_ of `square`?
```js
fun square(x) {
    return x * x;
}
```

--
<br/>
_Answer_: In Lox: Number.

When could it be something else?

---

class: quiz
# Bonus: Type Checking

What is the return _type_ of `f`?
```js
fun f(arg) {
    return arg;
}
```

--
<br/>
_Answer_: The type of `arg`.


---

class: quiz
# Bonus: Type Checking

```js
fun add(x, y) {
  return x + y;
}
```

Is this valid?
```js
add(2, 2) * 3;
```

--
<br/>
_Answer_: Yes.

What about this?
```js
add("hello", " world") * 3;
```

--
<br/>
_Answer_: In Lox: No.

---

# Bonus: Type Checking

Let's try to walk through these examples...

---

# Bonus: Type Checking

```js
fun add(x, y) {
  var z = x + y;
  return z;
}

add(2, 2) * 3;
```

--

<hr/>

```js
fun add(x, y) {
  var z = x + y; // 2: number (x) + number (y) => number (z)
  return z;      // 3: return type: number (z)
}

add(2, 2) * 3;   // 1. x: number, y: number
                 // 4. number (add(2, 2)) * number (3)
                 // 5. [type match]
```

---

# Bonus: Type Checking

```js
fun add(x, y) {
  var z = x + y;
  return z;
}

add("hello", " world") * 3;
```

--

<hr/>

```js
fun add(x, y) {
  var z = x + y;            // 2: string (x) + string (y) => string (z)
  return z;                 // 3: return type: string (z)
}

add("hello", " world") * 3; // 1. x: string, y: string
                            // 4. string (add("hello", " world")) * number (3)
                            // 5. [type mismatch]
```

---

# Bonus: Bytecode Generation

Generating lower-level instructions. Can improve performance and are platform agnostic.

One typical approach is to generate _bytecode_ where each instruction identifier fits in a byte.

--
### Example

Source code:
```js
print 2
```

Byte code:
```js
push_num 2
print
```

---

# Bonus: Bytecode Generation

Example:
```js
for i in 2..5 {
    print i
}
```
to
```bash
push_num 2
set_local 0
get_local 0
push_num 5
less
jump_if_false 22
pop  1
get_local 0
print
get_local 0
push_num 1
add
set_local 0
pop 1
jump -35
pop 1
```

---

# Bonus: Bytecode Generation

We need to patch jump instructions.

![](https://craftinginterpreters.com/image/jumping-back-and-forth/patch.png)

---

# Bonus: Bytecode Generation

Example: 

```js
...
10 jump_if_false XXX // jump to where?
```
then...
```js
...
10 jump_if_false XXX // jump to where?
...
30 ... // jump to here
...
```
then...
```js
...
10 jump_if_false 20
...
30 ...
...
```

---

# Bonus: Virtual Machine

A stack based virtual machine (alternatively a register based virtual machine).

<center><img src="https://craftinginterpreters.com/image/a-virtual-machine/pancakes.png" style="width:60%"/></center>

---

# Bonus: Virtual Machine

```js
Code: 3 - 1
```

![](https://craftinginterpreters.com/image/a-virtual-machine/reverse.png)

---

# Bonus: Virtual Machine

Algorithm (pseudo code):

```haxe
var program = ...; // bytecode array
var pos = 0;
var stack = new Array();

while (pos < program.length) {
    var code = program[pos++];

    switch code {
        case PushNumber:
            var value = program[pos];
            pos += 4; // size of float
            stack.push(value);
        case Print:
            print(stack.pop() + '\n');
        ...
    }
}
```

---

# Bonus: Front-end optimization

Simplifying code using static analysis. Post-traversal of the AST.

E.g. from

```js
print 4 + 3 + 2 + 1
var j = 'hello' + ' ' + 'world'
print j
```

to

```js
print 10
print 'hello world'
```

---
# Bonus: Back-end optimization

Improving runtime performance by optimizing the generated code.

Example pseudo code:

```js
ifcmp (e0, l0)
ldc_int (i0)
goto (l1)
label (l2)
ldc_int (i1)
label (l3)
```

to

```js
ifcmp (negate e0, l4)
ldc_int (i1)
label (l3)
```

---

class: middle, center

That's it...

---

class: quiz
# Pop quiz!

--

* What is a statement and an expression?

--

* What is a function (free function)?

--

* What is a method?

--

* What is a first class function?

--

* What is an anonymous function?

--

* What is a closure?

--

* What is a field?

--

* What is function arity?

--

* What is a parameter?

--

* What is an argument?

--

* What does the scanner/lexer do?

--

* What is a token?

---

class: quiz
# Pop quiz!

* What does the parser do?

--

* What does the panic mode do?

--

* What is syntatic sugar?

--

* What is an environment?

--

* What is the scope of an environment?

--

* What does the static analyser do?

--

* What does the type checker do?

--

* When is the type checker unable to infer a type?

--

* What does it mean for a language to by dynmically typed?

--

* What are the advantages of bytecode evaluation over AST evaluation?

---

![](http://www.craftinginterpreters.com/image/inheritance/superhero.png)

---

## Recap

* Tokens and lexing.
* Abstract syntax trees.
* Recursive descent parsing.
* Prefix and infix expressions.
* Runtime representation of objects.
* Interpreting code using the Visitor pattern.
* Lexical scope.
* Environment chains for storing variables.
* Control flow.
* Functions with parameters.
* Closures.
* Static variable resolution and error detection.
* Classes.
* Constructors.
* Fields.
* Methods.
* Inheritance.
* Type Checking.
* Bytecode Generation.
* Virtual Machine.
* Front-end and back-end optimization.

---

class: middle, center, gray

![](https://craftinginterpreters.com/image/header.png)

```C
// TODO: Go read the book at craftinginterpreters.com
```

---

class: middle, center

# EOF
