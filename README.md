# RegLang Specification
Specification and information about a non-existent (yet?) language that I designed.
I have very little knowledge on writing compilers for languages, so I lack the skills necessary to write the language myslef. However I decided that I might as well document the ideas I had for the language and share them online. Maybe someday someone will write a compiler for this language and make it a reality? We'll see!

# Genesis
I've been interested in programming language design for some time. After having many conversations with Grok 3 and ChatGPT, gaining knowledge on various aspects of language design and getting inspiration, and also consulting stuff with [@herb-ert](https://github.com/herb-ert), I have put together all the things that I learned and things that seemed interesting to me, and decided to write a language specification document that, well, documents what I came up with.

In terms of inspiration from other languages, it draws inspiration mainly from Java, C, C++, Rust, Lua, Luau and maybe even Kotlin or Go? (I'm not sure what exactly inspired me to use the `func` keyword for functions, but I know these languages use it so maybe they did?)

# What is RegLang?
RegLang is a statically typed, AOT (ahead-of-time) compiled language with region based memory management (hence the name, **Reg**Lang). You've heared it right - no garbage collection, no borrow checker, no pointers. Something a bit different than what we're used to.

It uses `.regl` as its file extension (`.reg` would be cooler, but it's already used by the Windows Registry).

# Why?
Why not? - that would be the simplest answer.

But now a bit more seriously: region based memory management is something that's not seen in programming languages very often, especially when it's (almost - see `free()` later) the only way to manage memory. I think it's an underrated way to manage memory as it allows for manual managment, can be really fast, and is easier to use than pointers or a borrow checker (at least for me). It's also probably safer than using pointers, allocating with `malloc()` or stuff like that. It requires you to be disciplined about how you write code, as the lifespan of every variable allocated in a region is the same as the lifespan of the region itself, so you have to be careful with the way you structure your code. And of course, if you have a variable defined in the main region (which would basically mean it's kinda "global" since it can be accessed in all underlying regions) that you're only gonna use a few times, it will be allocated for the entire lifespan of a program, even when it's not needed anymore, which would basically be a memory leak. That's why there's a `free()` function that can deallocate a variable before the region that the variable has been defined in ends. More on that later.

Now let's get into explaining the aspects of the language more precisely.

# Contents
1. [Primitive data types](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#primitive-data-types)
2. [Nulls](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#nulls)
3. [Comments](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#comments)
4. [Operators](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#operators)
5. [Variables](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#variables)
6. [Constants](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#constants)
7. [If statements](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#if-statements)
8. [Arrays](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#arrays)
9. [Functions](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#functions)
10. [Classes](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#classes)
11. [Generics](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#generics)
12. [Standard library](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#standard-library)
13. [Vectors](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#vectors)
14. [String](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#string)
15. [Regions](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#regions)
16. [Anonymous regions](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#anonymous-regions)

# Primitive data types
Let's start with data types.
```
bool var1 = true; -- 1 byte, true or false
byte var2 = 1; -- 8 bit int, 1 byte too, but for numbers
short var3 = 1; -- 16 bit int
int var4 = 1; -- 32 bit int
long var5 = 1; -- 64 bit int
float var6 = 1.5; -- 32 bit floating-point
double var7 = 1.5; -- 64 bit floating-point
char var6 = 'a'; -- a single unicode character, uses the single quote symbol, unlike strings which use the double quote symbol
```

I decided to go with data type names similiar to the ones in Java. If you want a 64 bit int you don't have to write `long long int` like in C++ which is too much boilerplate in my opinion. "Why not go with Rust's `i64`?" you might ask. To be honest, it's just a matter of preference for the most part, and I just prefer the Java data type names, but you could also make the argument that for begginer programmers or people who switch from higher level languages that just have a `number` type instead of `int`s (Lua, JavaScript), it might not be that clear what the `i` in `i64` stands for.

But in the case of `bool`, I think that it's more clear that it's a short for `boolean`, so the name of this data type doesn't match its Java counterpart.

What should also be mentioned is that in RegLang, every data type is its own thing. What does that mean? A boolean is a boolean, and an int is an int. Boolean values (`true`, `false`) are **not** the same as `0` and `1` values of an int. Those are two completely different types.

```C
int main() {
  int x = 0;

  if (x) {
    printf("x is true");
  }

  return 0;
}
```

This C code would not print anything out, as the if statement's condition would evaluate to `false`, because any non-zero value will be treated as `true` in C, while zeros are treated as `false`.

In RegLang, if you tried to write similiar code, the if statement's condition would evaluate to `true`. Why? Because booleans are their own type and ints (or any other types for numbers) are their own types, not related to each other. `0` is a number, it shouldn't be equal to something that's not a number.

# Nulls
Because every data type is its own thing, you also can't assing `null` to a variable (unless it's explicitly marked as nullable) simply because `null` won't correspond to any type other than itself. If a variable is explicitly marked as nullable, then it means it can be either of the specified type or null, making it possible to use nulls for that variable.

Nullable variables work similarly to the ones in Kotlin.

```
mut String? str = null;

if (str == null) {
  str = "Not null anymore!";
}
```

As you can see, in order to make a variable nullable, we need to add `?` to the data type.

Of course, if we tried to assign null to a variable that's not nullable, or try to access a non-nullable variable that has never been initialized, the compiler won't compile the program.

```
String s;
println(s); -- that would be impossible, because s has never been initialized and it hasn't been marked as nullable

String? s;
println(s); -- prints out null
```

This also means that all non-nullable variables have to be initialized before usage. When dynamic input or something else prevents the compiler from being sure that a non-nullable variable isn't being accessed before initialization, it has to be marked as nullable.

# Comments
Not much to say about them. Single line comments are done with `--` like in Lua, while multi-line comments are done with `/*` and `*/` like in Java and other similiar languages.
The language doesn't have increment/decrement operators like C or Java, so the usage of `--` isn't a problem.

```
-- this is a single line comment

/*
this
is
a
multi-line
comment
*/
```

# Operators
For logical operators, there are:
- `&&` - and,
- `||` - or,
- `!` - not (the opposite of the condition).

For relational operators, there are:
- `==` - equal to,
- `!=` - not equal to,
- `>` - greater than,
- `<` - less than,
- `>=` - greater than or equal to,
- `<=` - less than or equal to.

For arithmetic operators, there are:
- `+` - addition,
- `-` - subtraction,
- `*` - multiplication,
- `/` - division,
- `//` - floor division,
- `**` - exponentiation,
- `%` - modulus,
- `-` - unary negation.

You might notice there are no increment (`++`) or decrement (`--`) operators. I decided not to include them in the language, because they can unnecessarily complicate the code. You can just use the `+=` and `-=` compound operators to add or subtract 1 and assign the new value. It's clearer than `x++` or `x--`, which may get confused with `++x` and `--x`.

For compound assignment:
- `+=` addition (`x = x + y`),
- `-=` - subtraction (`x = x - y`),
- `*=` - multiplication (`x = x * y`),
- `/=` - division (`x = x / y`),
- `//=` - floor division (`x = x // y`),
- `**=` - exponentiation (`x = x ** y`),
- `%=` - modulus (`x = x % y`).

For bitwise operators:
- `&` - bitwise and,
- `|` - bitwise or,
- `^` - bitwise xor,
- `<<` - left shift,
- `>>` - right shift,
- `~` - bitwise not.

# Variables
You could have already seen how to create variables in the [Primitive data types](https://github.com/bartek1009x/RegLang-Specification?tab=readme-ov-file#primitive-data-types) example code. However what hasn't been said yet is that **all varaibles are immutable by default**. In order to make them mutable, you have to use the `mut` variable modifier.

```
int x = 10; -- this variable is immutable
mut int y = 10; -- this variable is mutable

x += 1; -- this would be impossible to do as the value of an immutable variable can't change
y += 1; -- this would work as you'd expect since y has been defined as a mutable variable
```

Of course if a variable is immutable its value can't be changed, but it can be reassigned.

# Constants

Immutable variables should not be confused with constants. RegLang features both immutable variables and constants. The value of an immutable variable can't change, but you can still reassign it, as it's just a variable. However when it comes to constants, they aren't variables at all, they're constants. Defining a constant is very similiar to defining a variable, but a constant is inlined at compile-time. It also means that it can't be reassign later on.

```
int x = 10; -- this is an immutable variable
const int y = 10; -- this is a constant

int x = 5; -- I can reassign the x variable
const int y = 5; -- but reassigning a constant like this wouldn't be possible
```

Constants must be known at compile-time, so they should be used for something like e.g. the value of PI.

```
const double PI = 3.141592653589793;
```

# If statements

```
int number = 5;

if (number > 5) {
  println("The number is higher than 5");
} else if (number < 5) {
  println("The number is lower than 5");
} else {
  println("The number is equal to 5");
}
```

As you can see, the if statements are basically the same as in every other language.
What's worth noting is that it requires parenthasis for the condition like C or Java, and it uses `else if` instead of a dedicated keyword like `elseif` in Lua or `elif` in Python. Also uses brackents instead of `then` and `end` like Lua.

# Arrays
Arrays in RegLang are very similiar to Java arrays, though in Java it's possible to write `[]` both after the data type and after the variable name. Writing C style arrays (`[]` after the variable name) won't work here though, only `[]` after the data type is accepted. I decided that only one of the ways to write arrays should be possible RegLang to unify syntax and reduce confusion.

```
int[] arr = new int[] {1, 2, 3, 4, 5}; -- this array will have max 5 elements
println(arr[0]); -- prints out 1
println(arr.length); -- prints out 5
println(arr.bytes); -- prints out the amount of bytes that the array takes

mut int[] arr = new int[4]; -- this array will have max 4 elements, but currently it's empty
arr[3] = 100; -- works as you'd expect
arr[4] = 100; -- index out of bounds, wouldn't let you compile your code
```

When you're trying to access index 4 when the max length is 4 (so max index is 3), the compiler *might* not let you compile your program at all. Of course in cases where dynamic input comes into play, or anything else that could make it impossible to evaluate whether an index out of bounds is being used anywhere in your code, the compiler would let you compile your program and you'd get an error during runtime. However, for cases like the one in the example above, it's easy to tell that there's an index out of bounds, so the program won't compile. Checking whether there are known out of bounds array index usages in your program makes compilation slower, but it's safer. 

# Functions
The function syntax looks like this:
```
func nothing() : void {} -- an empty function that does nothing
func something() : void {
  println("Something");
}

nothing(); -- calling the nothing function - obviously, nothing happens
something(); -- calling the something function - it prints Something
```

As you can see, the `func` keyword is used to define a function. I think it's a sweet spot between the long `function` (in languages like Lua or JavaScript) and short `fn` (in Rust). Followed by the `func` keyword, we have the name of the function and the parenthasis, in which function arguments are specified. After that we specify the return type of the function, `void` if nothing is returned. Then the last thing is the function body.

```
func add(int firstNumber, int secondNumber) : int {
  return firstNumber + secondNumber;
}

int x = add(2, 5);
println(x); -- prints 7
```

Function arguments are specified similiarly to Java - type first, then the argument's name. The `return` statement obviously does what it says it does - it returns something that matches the type of the specified function return type.

However, it's actually not necessary to write the function return type. The compiler should be able to figure out what the function returns automatically.

```
func add(int firstNumber, int secondNumber) {
  return firstNumber + secondNumber; -- the function return type has not been specified, but the compiler can figure out that what gets returned is an int, since what the function returns is assigned to an int variable later
}

int x = add(2, 5); -- since we're assinging the result of add() to int x, the compiler will know that the return type of the add function is int
println(x); -- prints 7
```

However, if we had multiple variables of multiple types assigning the result of a function without a return type specified, the compiler wouldn't be able to figure out what to return.

```
func add(int firstNumber, int secondNumber) {
  return firstNumber + secondNumber;
}

int x = add(2, 5);
short y = add(1, 4);
```

In this example, the compiler doesn't know if `add` should return an `int` or a `short`. In a scenario like this you'd have to explicitly cast what gets returned from the `add` function to a `short` to let the compiler know it can return an `int`.

```
func add(int firstNumber, int secondNumber) {
  return firstNumber + secondNumber;
}

int x = add(2, 5);
short y = (short) add(1, 4);
```

`x` requires an int and `y` casts whatever gets returned from `add` to a `short`, so `add` can safely return an `int` as it will satisfy the needs of all variables that assign the result of `add`.

And of course, if you tried to return multiple types in one function, the compiler wouldn't let you compile your code.

```
func someFunction(int x) {
  if (x >= 10) {
    return true;
  } else {
    return 0;
  }
}
```

This function tries to return either a `bool` or some number data type, which is invalid, because functions can only have one return type. The compiler won't compile the code until this issue is resolved.

```
func add(int firstNumber, int secondNumber) {
  return firstNumber + secondNumber;
}

int x = add(2, 5);
add(1, 4);
```

In a scenario like this where the function's result is once assigned to a variable and once called without assigning it to a variable, its return type should be the type of the variable that it gets assigned to, in this cas `int`.
If the function's result wasn't ever assigned to any variable, the return type would be `void`.

Overall this is a feature that's made primarly to reduce boilerplate, so you don't have to write `: void` in functions that don't return anything and you don't need to specify the return type for functions that are returning an obvious type (e.g. if you have a function called getUsername(), it's pretty obvious it's going to return a String, because what else could the username be?). **Generally it's advised to specify the return type for clarity**, unless in your case the function doesn't return anything or has a name clearly suggesting what it's going to return.

When it comes to argument passing in the functions, it works the same way as in Java. Primitive types get copied and the copies get passed to the function, while for everything else, so classes, arrays, etc. it's a copy of the reference to the object that gets passed to the function.

```
func example(int number) {
  number = 5; -- doesn't modify the original x variable
}

int x = 10; -- whether x is mutable or not doesn't matter in this case
example(x);
println(x); -- prints out 10
```
```
func example(int[] numbers) {
  numbers[0] = 5; -- modifies the first element in the array, even in the original x variable
  numbers = new int[] {1, 2}; -- doesn't modify the original array from the x variable
}

mut int[] x = {1, 2, 3, 4, 5, 6, 7};
println(x[0]); -- prints out 1
example(x);
println(x[0]); -- prints out 5
```

If you want a function that doesn't have a specified amount of arguments and the amount of them can vary, you can use `...` before the argument type to tell the compiler that there will be an unknown amount of arguments of this type passed in. You will then be able to use the passed arguments as an array inside the function.

```
func add(...int numbers) {
  mut int endResult = 0;

  for (int num : numbers) {
    endResult += num;
  }

  return endResult;
}

println(add(1, 2, 3, 4, 5)); -- prints out 15
println(add(10, 50, 25)); -- prints out 85
```

Even though we passed in multiple `int`s to the function, not an `int` array, it will be packed into an `int` array for usage inside the function.

Additional arguments besides the ones that will be packed into an array can only be passed before the `...` ones, not after.

```
func addOrSubtract(boolean add, ...int numbers) {
  mut int endResult = 0;

  if (add) {
    for (int num : numbers) {
      endResult += num;
    }
  } else {
    for (int num : numbers) {
      endResult -= num;
    }
  }

  return endResult;
}

println(add(true, 1, 2, 3, 4, 5)); -- prints out 15
println(add(false, 50, 20)); -- prints out 30
```

This works, but if you wrote the arguments the other way around, like: `addOrSubtract(...int numbers, boolean add)`, this wouldn't work. This is because if there was e.g. `...int numbers, int amount, int somethingElse` it could get confusing, when passing the arguments, where the `numbers` argument ends and the other arguments start.

# Classes
C has structs, which can't have functions inside them. But then there's Rust, which also has structs, but its structs **can** have functions inside them (they can be added through a `impl structName { functions here }` block). Structs with functions are already kind of used like classes, but they are a bit lower level and less flexible as a result. That's why I think that instead of adding structs with functions, I might as well just add classes. I think it's a more flexible approach that basically lets you do stuff more easily. And also I just like object oriented programming.

All classes extend a built-in `Object` class, like in Java. It has some basic functions like e.g. className() which returns the name of the class.

Let me demonstrate how classes should be made, with a generic Animal class example:

```
abstract class Animal {
  public:
    String name;
    int age = 0; -- default age

    func sound() {}
  private:
    func age() {
      age += 1;
    }
}

class Dog extends Animal {
  public:
    String name = "Dog";
    // age is not defined here, but it's inherited from Animal

    -- overrides the empty sound function from Animal
    func sound() {
      println("Bark");
    }

    func printAge() {
      println(age);
    }
  // there's no private section in the Dog class, but the private age() method is inherited from Animal
}

-- an instance of Animal can't be made since it's an abstract class

mut Dog dogInstance = new Dog(); -- similarly to Java, you need the new keyword when creating a new instance of a class
dogInstance.sound(); -- prints Bark
dogInstance.age = 10;
dogInstance.printAge(); -- prints 10
```

As you can see, I took an approach to classes that's a mix between C++ and Java. It's mostly the same as in Java, except for the `public` and `private` sections that are like the ones in C++. It removes the unnecessary boilerplate of writing `public` or `private` in the definition of every variable or function that you want to be public or private.

When it comes to keywords, it uses similiar keywords to Java's class definition keywords, like `extends` or `abstract`. Why not `:` instead of `extends` like in C++? Honestly I just prefer `extends`, though an argument could also be made that it's more readable.

Similarly to Java, besides being abstract, classes can also be final. That of course means that a class can't be extended.

```
final class Animal {}

class Dog extends Animal {} -- this would be impossible to do, because Animal has been defined as a final class in this example
```

Classes themselves are always "public", which means they can be used anywhere after they have been defined. Additionally, you can't nest class definitions, so you can't have a class definition inside another class definition.

Classes also have **constructors**. Constructors are basically functions that are called when you create a new instance of a class.

```
class Person {
  public:
    String name;
    
    Person(String name) {
      this.name = name;  
    }
}

Person newPerson = new Person("Bart");
println(newPerson.name); -- prints out Bart
```

Similiarly to Java, there's a `this` keyword. When in the constructor you have an argument that's called the same as one of the class's variables, it makes it possible to distinguish between the two.

There's also some syntax sugar for creating instances of classes that reduces boilerplate.

```
class Person {
  public:
    String name;
    
    Person(String name) {
      this.name = name;  
    }
}

Person newPerson = ("Bart"); -- you can skip writing new Person, and just write the parenthasis with the constructor's arguments. However, in this case, it's probably better to include new Person for clarity
println(newPerson.name); -- prints out Bart
```

# Generics
RegLang supports generic types for functions and classes. Those should be used when you want to create a function or class that should work with any kind of data type, like e.g. a `Vector`.

```
func makeString(<T> anything) : String {
  return "Hello, " + anything;
}

println(makeString(1)); -- prints out Hello, 1
println(makeString("World")); -- prints out Hello, World
```

As you can see in the example above, when a function has a `<T>` argument, it means that the argument can be of any type. The function returns a String that concatenates "Hello, " with the argument. It doesn't matter if the argument is a String, an int, or anything else. As long as it's possible to concatenate a string with the argument type, this code will work properly.

```
class Vector<T> {
  public:

    Vector<T>(...<T> elements) {
      for (<T> element : elements) {
        println(element);
      }
    }
}

Vector<String> = new Vector<>("Hello", ", ", "World!");
Vector<int> = (1, 2, 3, 4, 5); -- of course you can skip writing new Vector<> and instead just pass the arguments to the constructor in the parenthasis, like you can with other classes
```

The above example isn't an actual Vector as it only prints out the arguments, for an actual Vector implementation you can check out the [Vector.regl example in this repository](https://github.com/bartek1009x/RegLang-Specification/blob/main/examples/Vector.regl). However what you can see is how generics are used in classes. There's `<T>` after the class name in the class declaration (first line) and in the constructor. The constructor's arguments are packed into an array with the `...` (information about this can be found in the section about functions) and their type is specified as <T>, which means that it will be the generic type of the vector.

Writing just `<T>` allows any type, whether it's a primitive type or a class. In case you'd want only specific classes to be allowed, you could write `<T extends CLASS_NAME>`, where the CLASS_NAME would obviously be the name of your class.

```
func getClassName(<T extends Object> someClass) : String { -- every class extends the Object class, which is the initial class for all classes in the language
  return someClass.className();
}

println(getClassName(new String())); -- prints out String
println(getClassName('a')); -- a is a primitive type (char), not a class, so this won't work (it won't let you compile)
```

# Standard Library
One of the things that I immediately liked when I started coding in Java was its rich standard library and all the things that it had built-in (well, most of them were in the JDK, not in the language itself, but you know what I mean). While RegLang is meant to be in the lower level category of langauges (C, C++, etc.), so lower level than Java, I think that it should still have a standard library covering all the functionality used daily, unlike the C std which is not too extensive.

The language should have built-in functions and classes covering:
- OS (for platform specific needs),
- Processes,
- Filesystem,
- I/O,
- Math,
- Random,
- Strings,
- Vectors,
- Maps,
- Coroutines,
- Threads.

Those are **essential** for a modern language in my opinion. Of course, coverage of other functionality could also be considered for the std, but what I have listed are the essentials, the bare minimum.

# Vectors
Basically dynamic arrays that resize themselves when needed, like the ones in C++, Rust or the Java ArrayList. Implemented as classes.

```
mut Vector<int> vec = new Vector<>(); -- initializing an empty vector
mut Vector<int> vec = new Vector<>(1, 2, 3, 4, 5); -- initializes a vector with values
mut Vector<int> vec = (1, 2, 3, 4, 5); -- some syntax sugar - you can just skip writing new Vector<> entirely and just pass in the arguments (values)

vec.push(7); -- the vector now has values: 1, 2, 3, 4, 5, 7
println(vec.get(5)); -- prints out 7
```

Though accessing the internal array could be made possible by making it public, it should be kept private. Accessing the internal array would be faster, but would compromise safety, because the `get()` function of a Vector always checks bounds. Modifying the internal array could also lead to problems. If a programmer knows what they're doing, they can just make their own implementation of a vector class and make the internal array public. But it shouldn't be this way in the standard library's vector.

# String
A String would basically be a built-in wrapper class that has a `char` vector. It should have all the basic string properties and also some utility methods too. Internal variables like the length or size of the string should be kept private, and getter functions should be used. Exposing the variables by making them public could lead to users modifying the variables and that could lead to issues. Let's keep things safer.

```
mut String str = "Hello, world!"; -- as with Vectors, there's syntax sugar for Strings. it makes it possible to just write the text in "" without the need to write mut String str = new String("Hello, world!"). while vectors and other classes use parenthasis for the arguments, Strings are a special exception and only use the double quote symbols. you could write String str = ("Hello, world!"); as well, but it's not necessary in this case
println(str); -- prints out Hello, world! as you'd expect
println(str.length()); -- prints out 13
println(str.bytes()); -- prints out the number of bytes that the string takes
str = str.replace("Hello", "Hi"); -- an example of a string manipulation method - it returns a new string, so you have to assign the returned value to the str variable
println(str); -- prints out Hi, world!

-- string concatenation
mut String str2 = "Hello!";
str2 += " How are you?"; -- using the compoound operator is of course the same as writing str2 = str2 + " How are you?";
println(str2); -- prints Hello! How are you?
```

There's also an alternative string formatting option if you want to e.g. easily insert variable values into a string, inspired by Luau's string formatting.

```
int x = 1;
String s = `The value of x is equal to {x}` -- the {varName} wouldn't do anything if the string was made with "" instead of ``
```

# Regions
Now the main unique feature of the language - regions.

```
region r1 {}
```

As you can see, a region is created using the `region` keyword followed by the region's name and the region body.

```
region r1 {
  int x = 1;

  region r2 {
    int x = 5; -- a new variable called x in this region. the original x from r1 will no longer be accessible in r2, as in r2 a variable with the same name has been defined
    int y = 10;

    println(x); -- prints out 5
    println(y); -- prints out 10
  } -- here, when the region ends, both x and y get deallocated from memory. only r2's x though, not the x from r1, as those are two completely different variables, despite having the same name

  println(x); -- prints out 1
}
```

Regions can be nested. When a region ends, everything that was allocated inside it gets deallocated.
What's also worth noting is that **functions can't be defined inside regions**, as they are always either global or tied to classes.

```
region r1 {
  int x = 1;

  region r2 {
    region r3 {
      println(x);

      region r4 {
        region r5 {
          region r6 {
            region r7 {
              -- and so on
            }
          }
        }
      }
    }
  }
}
```

In this scenario we have many regions. Imagine that each one has its own code and variables. The `x` variable defined in region `r1` is not used past `r3`, so keeping it allocated for the entire duration of the program's execution would be a memory leak. We don't need `x` in `r4`, `r5` and so on, so we can deallocate it after using it for the last time using the built-in `free()` function, like this:

```
region r1 {
  int x = 1;

  region r2 {
    int x = 10;

    region r3 {
      println(x);
      free(x @ r1);
    }
  }
}
```

Normally, the `x` variable would get deallocated when the region it was defined in (`r1`) ends, so in this case it would be at the end of the program's execution. However, we have used the `free()` function after it was used for the last time, so it was deallocated earlier. The `@` (at) symbol tells the compiler which x to deallocate. We have variables called `x` both in the `r1` region and the `r2` region, so it would be unclear for the compiler which x variable to free - the one in `r1` or the one in `r2`. Because of this, the `@` operator has to be used to specify in which region the compiler should search for the variable to deallocate.

You can also use the `@` operator to access variables from outer regions if there are multiple with the same name.

```
region r1 {
  int x = 1;

  region r2 {
    int x = 5;

    region r3 {
      println(x); -- prints out 5
      println(x @ r2); -- also prints out 5
      println(x @ r1); -- prints out 1
    }
  }
}
```

Accessing variables from other regions isn't the only use case for the `@` operator though - when you have nested regions and you're in one of the nested ones, but you want to allocate a variable in one of the outer region, you can use the `@` operator to specify in which region you want the variable to be allocated, like this:

```
region r1 {
  region r2 {
    -- there are no variables defined in r2 initially

    region r3 {

      int x @ r2 = 10; -- x will be allocated in r2, not in r3 where this code is written

      println(x); -- of course now it will also be accessible in all the inner nested regions from r3 to the last one
      region r4 {
        region r5 {
          region r6 {
            region r7 {
              -- and so on
            }
          }
        }
      }
    }

    println(x); -- the x variable can be accessed in r2 now, even though it wasn't originally allocated in r2
  } -- the x variable will of course get deallocated when the r2 region ends
}

```

You'll probably be nesting a lot of regions when writing more complex programs. Since you manage memory with the way you write regions, you need to be careful with the structure of your code. To keep the code readable, remember to wrap code in functions where it makes sense.

# Anonymous regions
In the previous section about regions, we learned how to create regions with names that identify them, which allows for the usage of the `@` operator. However, it's possible to create a region without giving it a name. That would be an anonymous region.

```
{
  println("Anonymous region 1");
  {
    println("Anonymous region 2");
  }
}
```

As you can see, anonymous regions are created with just curly brackets to define where they start and where they end. Those regions are pretty much the same as blocks made with curly brackets in languages like C, Java or Rust (with the exception that anonymous regions can't return anything, unlike the Rust blocks), or `do end` blocks in Lua. RegLang just uses different terminology that fits the language better, so that's why these aren't called blocks.

Since anonymous regions don't have names, you can't use the `@` operator to access or allocate variables in outer regions, when you're inside a nested region.

And of course in if statements, functions, loops, etc. when you write `{` and `}` to define where the scope starts and ends, that is an anonymous region too.
However, nothing stops you from writing non-anonymous regions for those if you'd want to.

```
func example() : void r1 { -- the region name goes after the return type for functions
    println("Example");
}

func exampleWithNoReturnType() r1 {
    println("Example");
}

int number = 10;

if (number > 5) r1 {
  println("The number is higher than 5");
  region r2 {
    int x @ r1 = 10;
  }
} else if (number < 5) r1 {
  println("The number is lower than 5");
} else r1 {
  println("The number is equal to 5");
}
```

As you can see in the example, you can write the name of the region after the condition, thus making it a normal, non-anonymous region and allowing you to use the `@` operator in regions that will be nested in the main one.

When creating a normal, non-anonymous region for an if statements or loop condition, you don't write `region` like you usually do when creating a new region, because in this case it would be unnecessary.
