# An Introduction to ADTs and Structural Pattern Matching in TypeScript

> January 1, 2018

## Preface

To quote Rúnar Bjarnason:

```text
One of the great features of modern programming languages is structural pattern 
matching on algebraic data types. Once you've used this feature, you don't ever 
want to program without it. You will find this in languages like Haskell and 
Scala.
```

I couldn't agree more myself. That said, I spend most of my time writing 
programs with languages that don't have first-class support for algebraic data 
types (ADTs). So what's a programmer to do? This blog post provides examples of 
two ways to approximate structural pattern matching in TypeScript. The 
class-based example borrows heavily from Bjarnason's excellent blog post 
[Structural Pattern Matching in Java][1], and the discriminated union example 
was inspired by the Advanced Types section of the TypeScript documentation and 
countless conversations with Michael Avila, one of my coworkers.

## What's an ADT Look Like? (Obligatory Haskell Example)

In Haskell, we define an algebraic data type Failable - a type which represents 
the disjoint union of success and failure-values - with the following syntax:

What we're saying here is that a value of type `Failable t e` is either a 
`Success t` ("we succeed with a value of type t") or a `Failure e` ("we failed" 
with a value of type `e`). Functions that return a `Failable` can indicate one 
or the other but not both. When interacting with a value of type `Failable`, we 
need to pattern match on the type's constructor if we want to perform operations 
on their underlying values.

```haskell
failable :: (t -> c) -> (e -> c) -> Failable t e -> c
failable f g r = case r of
(Success x) -> f x
(Failure y) -> g x
```

## Discriminated Unions

According to the docs, the programmer needs three things in order to create an 
algebraic data type in TypeScript:

1. Types that have a common, singleton type property - the _discriminant_.
2. A type alias that takes the union of those types - the _union_.
3. Type guards on the common property.

Following these guidelines, let's approximate our Haskell type Failable:

```typescript
interface Failure<E> {
    tag: "failure";
    reason: E;
}

interface Success<T> {
    tag: "success";
    value: T;
}

type Failable<T, E> = Failure<E> | Success<T>;
```

In the above example, the `Failure` and `Success` interfaces both have a common, 
singleton type property - tag (item #1). These interfaces are unioned together 
to form our ADT, `Failable` (item #2). For any function accepting a Failable to 
be well typed it must first examine the value of the `Failable`‘s tag before 
making use of `Failure` or `Success`-specific properties (item #3). We can use 
the tag type guard to build a type safe failable function, too:

```typescript
function failable<T, U, E>(
    r: Failable<T, E>,
    f: (_: Success<T>) => U,
    g: (_: Failure<E>) => U
): U {
    switch (r.tag) {
        case "success": return f(r);
        case "failure": return g(r);
    }
}
```

### Interesting Property A: Exhaustiveness Checking

With the compiler option `strictNullChecks` enabled, TypeScript will fail to 
compile our failable function if it omits one or more of the type's singleton 
properties (tag, in this example), because the function would implicitly return 
`undefined` for the unhandled type and `undefined` is not an inhabitant of our 
return type, `U`.

### Interesting Property B: Inextensibility

An interesting property of a TypeScript disjoint union type is that it cannot be 
extended with additional types after its initial declaration. This is a good 
thing, as allowing library consumers to extend a disjoint union with their own 
types would cause compilation errors in existing code (due to failed 
exhaustiveness checks).

### Interesting Property C: Naming the Switching Function

A thing that I find irritating about this approach is the fact that I will need 
to come up with an original name for the matching function associated with each 
disjoint union type. While `failable` seems sensible, I've run into things like 
`templateElementLabelResolutionResult`.

## The Classical Approach

If discriminated unions aren't your thing, ADTs can be approximated using an 
abstract class:

```typescript
type Function1 <T, U> = (x: T) => U;

abstract class Failable <T, E> {
    abstract match <U> (
        f: Function1<Success<T, E>, U>,
        g: Function1<Failure<T, E>, U>
    ): U;
}

class Success <T, E> extends Failable <T, E> {

    public value: T;

    constructor(value: T) {
        super()
        this.value = value;
    }

    match <U> (
        f: Function1<Success<T, E>, U>, 
        g: Function1<Failure<T, E>, U>
    ): U { return f(this); }
}

class Failure <T, E> extends Failable <T, E> {

    public reason: E;

    constructor(reason: E) {
        super()
        this.reason = reason;
    }

    match <U> (
        f: Function1<Success<T, E>, U>, 
        g: Function1<Failure<T, E>, U>
    ): U { return g(this); }
}
```

### Interesting Property A: Familiarity

This is the pattern that I use in Java. If you do too, perhaps it makes sense to 
stick with it in your TypeScript instead of introducing a new pattern.

### Interesting Property B: Extensibility

Unlike the disjoint union type, our abstract class can be extended by any 
programmer who imports it (to my knowledge, as TypeScript contains no mechanism 
by which a programmer can "seal" a class). That said, such a thing would have 
little effect, as the Failable#match method would not contain a parameter for 
this new type.

## A Real-World Example

ADTs are useful for providing meaning to the input and/or output-types of a 
function beyond what's possible with JavaScript primitives, and without 
introducing a hierarchy of classes. Consumers of these types get compile time 
exhaustiveness guarantees (no runtime instanceof stuff required) provided that 
they guard on the common property. 

In the following example, we define a function parseLogLine which accepts a line 
of text from a log file as a String and returns a `ErrorMessage`, 
`WarningMessage`, `InfoMessage` (if the string was parseable as a log-line) or 
`Unknown` (if it was not).

```typescript
interface ErrorMessage {
    tag: "error",
    timestamp: number,
    message: string,
    code: number
}

interface WarningMessage {
    tag: "warning",
    timestamp: number,
    message: string
}

interface InfoMessage {    
    tag: "info",
    message: string
}

interface Unknown {
    tag: "unknown";
    fullLogLine: string;
}

type LogLineParseResult = ErrorMessage | WarningMessage | InfoMessage | Unknown

function logLineParseResult<T>(
    r: LogLineParseResult, 
    f: (_: ErrorMessage) => T, 
    g: (_: WarningMessage) => T,
    h: (_: InfoMessage) => T,
    i: (_: Unknown) => T
): T {
    switch (r.tag) {
        case "error": return f(r);
        case "warning": return g(r);
        case "info": return h(r);
        case "unknown": return i(r);
    }
}

// usage

const parseLogLine = function(logLine: string): LogLineParseResult {
    const words = logLine.split(" ");
    const level = words[0];
    const timestamp = parseInt(words[1], 10);
    const errorCode = parseInt(words[2], 10);

    if (level === "E" && timestamp && errorCode) {
        return {
            "tag": "error",
            "timestamp": timestamp,
            "message": words.slice(2).join(""),
            "code": errorCode
        };
    }
    else if (level === "W" && timestamp) {
        return {
            "tag": "warning",
            "timestamp": timestamp,
            "message": words.slice(2).join("")
        };
    }
    else if (level === "I") {
        return {
            "tag": "info",
            "message": words.slice(1).join("")
        };
    }
    else {
        return {
            "tag": "unknown",
            "message": logLine
        };
    }
};

const errorLogLine1  = "E 1513877434 503 Service Unavailable";
const errorLogLine2  = "E 1513878191 502 Bad Gateway";
const warningLogLine = "W 1513878016 Running low on RAM";
const garbageLogLine = "It's like love in an elevator";

const errorCodes = [garbageLogLine, warningLogLine, errorLogLine1, errorLogLine2]
    .map(parseLogLine)
    .map(r => logLineParseResult(r, (e) => e.code, (w) => 0, (i) => 0, (u) => 0))
    .filter(r => r);

console.log(errorCodes); // [503, 502]
```

## Takeaways

I find algebraic data types to be a fun and useful tool in my tool belt, even if 
the language of my day job doesn't provide first-class support for them.

[1]: https://web.archive.org/web/20200930153519/http://blog.higher-order.com/blog/2009/08/21/structural-pattern-matching-in-java/
