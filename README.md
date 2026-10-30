# Raga

Raga is a small, explicit language built from **one shape**. Almost everything you write, a variable, a function, a type,
a class, a loop, a call with a block, is the same sentence with different words in it:

```
[[attributes]]
[ const | let | Type ]  name  =  ( head [(args)] [|params|] [Type] [{ body }] )+
```

The part in `( )+` is written once, or again and again: each repeat is one more clause of the chain. Every repeat takes the same parts as the first
and starts on the line of the closing `}` before it. The `{ body }` after a call is its trailing block, which is the last
argument of the call written in place: a function body for a callback, the values of an `Array`, or the `name = value` lines of a
struct. A repeat after a block is a method of the value the call gave back. That is how `try`-like shapes
are built for your own code: with structs. If you can read that line, you can read Raga. This page explains it, and the ideas that follow from it.

The language itself is in [`raga/`](../raga), the editor support in [`vs-code-extension/`](../vs-code-extension).

The language is described by examples written in Raga itself, in the `templates/` folder: each one explains one area and can be
read top to bottom. The compiler, `ragac`, checks a file, and the editor extension checks code while you type.

---

## 1. The one shape

| Part | Meaning | Examples |
|---|---|---|
| `[attributes]` | **facts about the declaration**, on the line above, optional | `[page("/")]`, `[class(Inheritance.Final)]`, `[route("users/", "get")]` |
| `const` / `let` / `Type` | how the name is bound, exactly one of the three, never a type together with `const` or `let` | `const` cannot be rebound, `let` can, a type first picks the type: `u8 level = 5` |
| `name` | what you call it | `limit`, `area`, `Point`, `Student` |
| `=` | binds the name to what follows | every declaration has it, and a value |
| `head` | **what is being made or done** | a value, a type name, `function`, `method`, `struct`, `enum`, `union`, `class`, `interface`, `trait`, `attribute`, `get`, or a call |
| `(args)` | the **values** that go in, with optional defaults (`Type name = value`) | parameters of a function, arguments of a creation, the condition of an `if`. A call always has parentheses, `double(5)`, `answer()`: a name without them is the function itself. The one exception is a block as the only argument, `run { }` |
| `\|params\|` | the **names** that the head hands to its block | `for (items) \|item\| { }`, `catch \|e\| { }` |
| `Type` | the type of the result | `function(f64 r) f64 { }` |
| `{ body }` | the code or the members | the block |
| `( ... )+` | the **chain**: the same parts again, a head of its own, on the line of the closing `}` | `elif`, `else`, `catch`, `finally`, `extends(..)`, `implements(..)`, `use(..)`, a second `method(..)` (an overload), `set \|v\| { }`, `call \|..\| { }`, your own |

The same shape, in different clothes:

```raga
const limit = 100                                  // a value
u8 level = 255                                     // a value, with its type chosen first
const area = function(f64 radius) f64 {            // a function
    return(3.14159 * radius * radius)
}
const Point = struct { f64 x; f64 y }              // a type
const Color = enum { const Red = 0; const Green = 1; const Blue = 2 }   // a type
const route = attribute(String path)               // an attribute
[class(Inheritance.Final)]                         // attributes: facts about the declaration below
const Student = class {                            // a class, with a chain after the body
    String name = ""                               // a field is a variable
    const greet = method() String {                // a method is a binding whose head is method
        return(this.name)
    } method(String greeting) String {             // overloads are a chain on the closing brace
        return(String.join(greeting, this.name))
    }
    const marks = get { return(this._marks) } set |value| { this._marks = value }   // an accessor
} extends(People) implements(ISpeak)
```

Without a binding, the same shape is a **statement**. A condition, a loop, a call with a block are all
the same line without the name: `head (args) |params| { body }`, repeated as often as needed:

```raga
if (temperature > 30) {                            // head (args) { body } chain
    weather = "hot"
} elif (temperature > 20) {
    weather = "warm"
} else {
    weather = "cold"
}

for (items) |item, index| { print(item) }          // head (args) |params| { body }

try {
    check_age(200)
} catch |e| {                                      // chain: the next clause, on the line of the }
    print(e.message)
} finally {
    cleanup()
}

const doubled = apply(4) |n| { return(n * 2) }      // your own function takes a block the same way
```

Inside expressions Raga uses the usual operators (`+ - * / == && ...`), and reassigning a name, `count = count + 1`, is the
one statement that has no head.

A **chain** is the usual chaining of calls. A call that returns a value is continued with a dot, `user().select().limit(1)`. When the last
argument of a call is written as a block (a callback, an Array or a struct), a dot after the closing `}` would look strange, so the dot is
left out and the next name follows the brace: `user { } select { } limit (1)`. This is how Raga grows. Every repeat after the first is a head too, so it takes the same parts as the first:
`elif (b) { }`, `catch |e| { }`, `extends(People)`, `method(String lang) String { }`, `set |value| { }`. The repeats stay on the line of the
closing `}` before them, so the chain reads as one sentence. `if`, `try`, `catch` and `finally` are keywords of the language.
For your own code a function takes only one trailing block, for its last parameter, so a chain of blocks is built with structs:
each step returns a struct that has only the methods which may come next, so the return type of a step decides which repeat
can follow. The same struct again repeats a clause, another struct moves on, and a struct with no methods ends the chain.

```raga
when(a > 3) {
    ...
} or_when(a > 1) {
    ...
} otherwise {
    ...
}
```

---

## 2. What follows from the shape

**Everything is a binding.** A function, a type and an attribute are values bound to a name, declared like any variable:
`const area = function(...) { }`, `const Point = struct { }`, `const route = attribute(String path)`. There are no
special declaration forms to learn.

**Types in angle brackets, values in parentheses.** `List<i32>`, `Map<String, i32>`, `ptr<u8>` name a type.
`area(2.0)`, `Point(x = 1.0, y = 2.0)`, `u8(300)` take values. A cast is a type name called on a value.

**Brackets mean one thing each.** `( )` values, `< >` types, `{ }` a body or the members, `| |` names handed to a block,
`[ ]` attributes on the line above a declaration. Collections have literals that read like their constructors:
`"text"`, `{1, 2, 3}` (Array), `[1, 2, 3]` (List), `(1, 2.5)` (Tuple).

**Nothing is hidden.** There is no implicit conversion, no hidden return, no hidden `this`, no null, no overloading of
functions (methods can be overloaded). A function can be declared inside another function and then uses the names
around it (a closure): what it uses moves to the heap and lives as long as the function does. A function can return a function. `return` is a call, `return(value)` or `return()`, always written, and belongs to functions. A name must be declared
above its use. A struct is created by naming its fields, and a field with a default can be left out.

**Two kinds of values, and the name tells which.** A *primitive* (`i32`, `f64`, `bool`, `char`, `ptr<T>`, lowercase) is a
plain value that is copied. Everything else is *non-primitive* (`String`, `List`, `Function`, every struct and class, Capitalized):
the name holds a reference, an assignment shares it, `==` asks whether two names refer to the very same value, and `clone`
makes a copy. `const` protects the name, not the value.

**Failure is part of the type.** A call that can fail normally returns `Optional<T>` or `Result<T, E>` and is read with
`match`. Something that should never happen is thrown with `throw(Error(...))` and handled with `try` and `catch`.

**Memory is counted, not collected.** A non-primitive value is freed the moment nothing refers to it. `Weak<T>` breaks
cycles. Pointers and `alloc` exist for the rare case that needs manual memory.

**Facts about a declaration go above it.** An attribute is written in `[ ]` on the line above a declaration, and it is the
first line of the same shape. `[page("/")]` is data that your code can read, `[method(Ownership.Static, Visibility.Private)]`
changes what a method is. The modifiers are members of enums (`Visibility`, `Inheritance`, `Ownership`), not reserved words.
You declare your own attributes with the head `attribute`, in three patterns that build on each other:

```raga
const metadata = attribute(String name)                               // 1. metadata only: the arguments are stored

const routes = Map<String, BindingInfo>()
const route = attribute(String path) |target, path| {                  // 2. a build hook: runs once, while building
    routes.set(path, target)
} call |target, args, next| {                                          // 3. a call hook: runs on every call
    print(String.join("calling ", target.name))
    return(next(args))
}

[route("users/")]
const users = function(i32 first, i32 count) String { return("list") }
const path = attributes_of(users, "route").get(0).arguments.get(0).as<String>().unwrap()
```

The block names everything it uses between its `| |`, so nothing arrives under a hidden name.

**One file, one module.** `export { a, b }` ends a file, `const m = import("./path/to/file")` reads another. There is no
main function: the code of the file you run goes from the first line to the last.

---

## 3. Conventions

The conventions are not enforced, they are how Raga code is written so that it reads the same everywhere.

| | Style |
|---|---|
| types, enum members, union variants | `Capitalized` |
| variables, parameters, struct fields, functions, attributes | `lower_snake_case` |
| methods of a struct, enum or union | `lower_snake_case` |
| methods and fields of a class, interface or trait | `lowerCamelCase` |
| layout | a clause starts on the line of the `}` before it, `match` and `switch` hold one `case` per line |

---

## 4. What Raga does not have

No null, nothing to throw but an `Error`, no implicit casts, no operator overloading except with the `[operator]`
attribute, no inheritance for structs, no multiple inheritance and no overloading of plain functions.

---

## 5. Where to read next

Start with the quick start, then the templates in order. Each one explains one area in full, and none of them uses a
feature that a later one explains.

| | |
|---|---|
| [`hello.raga`](../raga/hello.raga) | the whole language in one page |
| [`01-basics`](../raga/templates/01-basics.raga) | variables, primitives, literals, operators, scope |
| [`02-data-structures`](../raga/templates/02-data-structures.raga) | Array, List, Tuple, Set, Map |
| [`03-control-flow`](../raga/templates/03-control-flow.raga) | `if`, `switch`, `match`, loops |
| [`04-functions`](../raga/templates/04-functions.raga) | parameters, callbacks, trailing blocks |
| [`05-structs-enums-unions`](../raga/templates/05-structs-enums-unions.raga) | your own types, `Optional` and `Result`, chaining |
| [`06-errors`](../raga/templates/06-errors.raga) | `Optional`, `Result`, `throw`, `try` |
| [`07-classes-and-traits`](../raga/templates/07-classes-and-traits.raga) | classes, inheritance, interfaces, traits |
| [`08-memory`](../raga/templates/08-memory.raga) | sharing, copying, reference counting, pointers |
| [`09-attributes-and-reflection`](../raga/templates/09-attributes-and-reflection.raga) | attributes (metadata, build hooks, call hooks), reflection, `compile` |
| [`10-modules`](../raga/templates/10-modules.raga) | `import` and `export` |
| [`11-standard-library`](../raga/templates/11-standard-library.raga) | the built-in functions and methods |
