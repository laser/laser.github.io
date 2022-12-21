# Hanging up on Callbacks: Generators in ECMAScript 6

> December 1, 2013

I hear people whine about asynchronous callbacks in JavaScript constantly. I
admit that wrapping your head around control flow in the World of JavaScript
(also known as "Callback Hell" or the "The Pyramid of Doom" by aforementioned
whiners) can be a bit of a mind-explosion if you're used to a top-down,
synchronous programming style. "Just deal with it" has been my go-to response;
after all, do we expect programming in all languages to look and feel the same?
Of course not.

This all changed after a recent review of the the ECMAScript 6 Draft, which
describes generators - a language feature that will greatly change the way we
write both server and client-side JavaScript. With generators, we can transform
nested callbacks into easy-to-read top down-style code without blocking our
single event loop thread. An example (adapted from a blog post by Toby Ho), to
illustrate my point:

```javascript
setTimeout(function(){
    _get("/something.ajax?greeting", function(err, greeting) {
        if (err) { console.log(err); throw err; }
        _get("/else.ajax?who&greeting="+greeting, function(err, who) {
            if (err) { console.log(err); throw err; }
            console.log(greeting+" "+who);
        });
    });
}, 1000);
```

...could be written as:

```javascript
sync(function* (resume) {
    try (e) {
        yield setTimeout(resume, 1000);
        var greeting = yield _get('/something.ajax?greeting', resume)
        var who = yield _get('/else.ajax?who&greeting=' + greeting, resume)
        console.log(greeting + ' ' + who)
    }
    catch (e) {
        console.log(e);
        throw e;
    }
});
```

Interesting stuff, right? Centralized exception handling and a 
easy-to-understand flow control. Note: If you just have to know how "sync" is
implemented, scroll to the "Blocking Ajax" example below.

## Uhhh, ECMAScript 6?

The examples in this document work in Chrome Canary version 33.0.1716.0. With
the exception of the XHR examples, they should all work in Node.js with the
"-harmony" flag. The generator implementation provided by JavaScript 1.7+ does
not adhere to the ECMAScript 6 draft - so you'll have to make some changes in
order to get my examples to work in Firefox. If you want to see these examples
running in a browser (Canary), you can do so here.

## ES6 Generators: Quick n' Drrty

In order to understand what's going on in the example above, we need to talk
about what an ES6 Generator is and what it provides you.

According to the ECMAScript 6 Draft, generators are "First-class coroutines,
represented as objects encapsulating suspended execution contexts." For those
of you who prefer a little less specificity in their tea: Generators are 
functions that can suspend themselves (using the yield keyword) and be resumed 
(from the outside world) by calling their "next" method. From your perspective, 
the JavaScript engine is still doing only one thing at a time - but it's now 
able to suspend execution in the middle of a (generator) function body and 
context-switch to do something else. Generators aren't enabling parallelism and
they don't have anything to do with threads.

## A Modest Iterator

Whew. Now that we've gotten that out of the way, let's see some code. We'll
build a simple iterator to demonstrate the suspend / resume semantics:

```javascript
function* fibonacci() {
   var a = 0, b = 1, c = 0;

   while (true) {
      yield a;
      c = a;
      a = b;
      b = c + b;
   }
}

function run() {
   var seq = fibonacci();
   console.log(seq.next().value); // 0
   console.log(seq.next().value); // 1
   console.log(seq.next().value); // 1
   console.log(seq.next().value); // 2
   console.log(seq.next().value); // 3
   console.log(seq.next().value); // 5
}

run();
```

Here's what's happening:

1. The caller, function "run," first initializes the fibonacci generator
   (denoted by the "function*" syntax). Unlike a normal function, this does
   not cause the code in its body to be run - it simply returns a new
   generator object.
2. When "run" calls the generator's "next" method (a synchronous operation),
   the code is the generator's body run… up until the "yield" keyword.
3. Evaluating the "yield" operator suspends the generator and yields the
   generator's result back to the caller. Operations following the yield
   have not yet been evaluated. The value (the operand, "a" of "yield")
   will be accessible to the caller through the "value" property of the
   generator result.
4. When the caller is ready to resume the generator, the "next" method is
   called and processing the code in the generator's body continues  
   immediately after where the prior "yield" left-off. You may be wondering
   if the generator function will ever return. The answer is "no," it will
   loop as many times as someone calls the "next" method.

## Following the Flow: A Digression

As mentioned in the prior example, code in the generator function's body
encountered after the yield operation won't be run until the generator is
resumed. The generator can also be passed an argument, which will be
substituted into the generator's function body where yield left off:

```javascript
function* powGenerator() {
  var result = Math.pow(yield "a", yield "b");
  return result;
}

var g = powGenerator();
console.log(g.next().value);   // "a", from the first yield
console.log(g.next(10).value); // "b", from the second
console.log(g.next(2).value);  // 100, the result
```

The first time the generator's body is run, "a" is yielded back to the caller
(and made available through the "value" property of the returned object). The
caller then resumes the generator, passing 10. Using substitution to visualize
what's happening:

```javascript
function* powGenerator() {
  var result = Math.pow(----10----, yield "b");
  return result;
}
````

The generator then hits the second "yield" statement and is suspended. The
value "b" is available on the returned object. Finally, the caller resumes the
generator, passing 2. With substitution:

```javascript
function* powGenerator() {
  var result = Math.pow(----10----, ----2----);
  return result;
}
```

The "pow" method is then called and the return value stored in the "result"
variable which is then returned to the caller.

## Fake Synchronicity: Blocking Ajax

Fibonacci sequence-emitting iterators and math functions with multiple entry
points are interesting, sure - but I promised to show you a way to eliminate
callback functions from your otherwise-callback-heavy JavaScript code. As it
turns out, we can pick from what I've already showed you some patterns that
will get us most of the way there.

Before we jump into the next example, note the "sync" function. This function
calls the generator function with a resume function, and then calls "next" on it 
to get things started. Whenever the generator function needs an async call, it 
supplies resume as the callback and yields. When the async call executes resume, 
it calls "next" (with a value) on the generator, allowing it to continue 
execution with the result of the async call.

Okay, back to the codez:

```javascript
// **************
// framework code

function sync(gen) {
  var iterable, resume;

  resume = function(err, retVal) {
    if (err) iterable.raise(err);
    iterable.next(retVal); // resume!  
  };

  iterable = gen(resume);
  iterable.next();
}

function _get(url, callback) {
  var x = new XMLHttpRequest();

  x.onreadystatechange = function() {
    if (x.readyState == 4) {
      callback(null, x.responseText);
    }
  };

  x.open("GET", url);
  x.send();
}

// ****************
// application code

sync(function* (resume) {
  log('foo');
  var resp = yield _get("blix.txt", resume); // suspend!
  log(resp);
});

log('bar'); // not part of our generator function's body
```

Can you guess what you're going to see in the console? If you said "foo,"
"bar", and "whatever was in blix.txt," felicidades, compa. You're right. By 
putting the code that we want to run in series inside a suspendable generator 
function, we can make it behave in a synchronous, top-to-bottom manner. We 
aren't blocking the event loop thread; we suspend the generator and resume the 
non-generator code at the point at which we called "next." The generator has 
been suspended but has not been garbage collected. The callback, called at some 
point in the future on a different tick of the event loop, resumes our 
generator, passing a value.

## Centralized Exception Handling

Centralizing exception handling across various asynchronous callback functions
is a pain. Take, for example, the following:

```javascript
try {
  firstAsync(function(err, a) {
    if (err) { console.log(err); throw err; }
    secondAsync(function(err, b) {
      if (err) { console.log(err); throw err; }
      thirdAsync(function(err, c) {
        if (err) { console.log(err); throw err; }
        callback(a, b, c);
      });    
    });
  }); 
}
catch (e) {
  console.log(e);
}
```

The catch block will never be hit (unless for some reason the synchronous calls
to "firstAsync" or "secondAsync" or "thirdAsync" cause an error to be thrown)
due to the execution of the callback being a part of a completely different
call stack, on a separate tick of the event loop. Exception handling must be 
done in the callback body itself. One could write higher-order functions to 
eliminate some of the error-throwing duplication and remove some of the nesting 
with a library like async, but if we follow the Node.js error as "first 
argument" convention, we can write a generalized handler that will propagate 
all errors back to the generator:

```javascript
function sync(gen) {
   var iterable, resume;

   resume = function(err, retVal) {
      if (err) iterable.raise(err); // raise!
      iterable.next(retVal);
   };

   iterable = gen(resume);
   iterable.next();
}

sync(function* (resume) {
   try {
      var x = firstAsync(resume);
      var y = secondAsync(resume);
      var z = thirdAsync(resume);
      // … do something with your data
   }
   catch (e) {
      console.log(e); // will catch errors from any of the three calls
   }
});
```

Now an error thrown inside any of these three calls will be caught by the
single catch block. And - just like in the vanilla JavaScript example - an
exception thrown from inside of any of the three calls will prevent the
subsequent functions from being called. Very nice.

## Concurrent Operations

Just because your generator code runs from top-to-bottom doesn't mean you can't
handle multiple asynchronous operations concurrently. Libraries like genny and
gen-run and co provide APIs do this, and basically reduce to yielding some
enumeration of asynchronous operations to be completed before the generator is
to be resumed. We can add basic support for concurrent operations to our sync
method like so:

```javascript
function sync(gen) {
  var iterable, resume, check, vals, ops;

  vals = [];
  ops  = 0;

  check = function() {
    if (vals.length == ops) {
      if (ops == 1) { 
        iterable.next(vals[0]);
      }
      else {
        iterable.next(vals); // resume with an array
      }
    }
  }

  resume = function(err, retVal) {
    var slot = ops;
    ops++;

    return function(err, retVal) {
      if (err) { 
        iterable.raise(err);
      }
      else {
        vals[slot] = retVal;
        check();
      }
    };
  };

  iterable = gen(resume);
  iterable.next();
}
```

…which then requires us to invoke the resume function, passing its result as
the callback to our asynchronous operation:

```javascript
sync(function* (resume) {
  var oneAndTwo = yield [_get("test1.txt", resume()), _get("test2.txt", resume())]
  var three     = yield _get("test3.txt", resume())
  
  log(oneAndTwo[0] + " - " + oneAndTwo[1] + " " + three)
});
```

## Conclusion

Asynchronous callbacks as a programming style has been the de-facto JavaScript
pattern for a long while - but the with the introduction of generators in the
browser (Firefox since JavaScript 1.7 and Chrome Canary as of a few months
ago), it doesn't have to stay that way. Leveraging the new control flow
constructs provided by generators can enable a very different coding style -
one which I think will evolve to contend with the nested-callback style - as
the ECMAScript 6 standard is implemented by the JavaScript engines of tomorrow.
