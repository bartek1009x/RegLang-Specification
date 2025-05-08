# RegLang Specification
Specification and information about a non-existent (yet?) language that I designed.
I have very little knowledge on writing compilers for languages, so I lack the skills necessary to write the language myslef. However I decided that I might as well document the ideas I had for the language and share them online. Maybe someday someone will write a compiler for this language and make it a reality? We'll see!

# Genesis
I've been interested in programming language design for some time. After having many conversations with Grok 3 (the AI), gaining knowledge on various aspects of language design and getting inspiration, and also consulting stuff with a friend, I have put together all the things that I learned and things that seemed interesting to me, and decided to write a language specification document that, well, documents what I came up with.

# What is RegLang?
RegLang is a statically typed, AOT (ahead-of-time) compiled language with region based memory management (hence the name, **Reg**Lang). You've heared it right - no garbage collection, no borrow checker, no pointers. Something a bit different than what we're used to.

# Why?
Why not? - that would be the simplest answer.

But now a bit more seriously: region based memory management is something that's not seen in programming languages very often, especially when it's (almost - see `free(var)` later) the only way to manage memory. I think it's an underrated way to manage memory as it allows for manual managment, can be really fast, and is easier to use than pointers or a borrow checker (at least for me). It requires you to be disciplined about how you write code, as the lifespan of every variable allocated in a region is the same as the lifespan of the region itself, so you have to be careful with how you write regions. And of course, if you have a variable defined in the main region (which would basically mean it's kinda "global" since it can be accessed in all underlying regions) that you're only gonna use a few times, it will be allocated for the entire lifespan of a program, even when it's not needed anymore, which would basically be a memory leak. That's why there's a `free(var)` function that can deallocate a variable before the region that the variable has been defined in ends. More of that later.

Now let's get into explaining the aspects of the language more precisely.

# Data types
Let's start with data types.
```
bool var1 = true; -- 1 byte, true or false
byte var2 = 1; -- 8 bit int, 1 byte also but for numbers
short var3 = 1; -- 16 bit int
int var4 = 1; -- 32 bit int
long var5 = 1; -- 64 bit int
char var6 = "a"; -- a single unicode character
string var7 = "Hello, world!"; -- a wrapper for a char vector
```

I decided to go with data type names similiar to the ones in Java. If you want a 64 bit int you don't have to write `long long int` like in C++ which is too much boilerplate in my opinion. "Why not go with Rust's `i64`?" you might ask. To be honest, it's just a matter of preference for the most part, and I just prefer the Java data type names, but you could also make the argument that for begginer programmers or people who switch from higher level languages that just have a `number` type instead of `int`s (Lua, JavaScript), it might not be that clear what the `i` in `i64` stands for.

But in the case of `bool`, I think that it's more clear that it's a short for `boolean`, so the name of this data type doesn't match its Java counterpart.
