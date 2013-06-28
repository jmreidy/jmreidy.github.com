---
title: What Does Coffeescript's "do" Do? 
layout: post
comments: true
---
When reading over some Node.js source code written in Coffeescript, I came across
a couple uses of the Coffeescript `do` operator. While `do` is a very familiar tool
for Ruby developers, I wasn't sure how Coffeescript used it, so I took at look at
its documentation, which states: 

>When using a JavaScript loop to generate functions, it's common to insert a closure
>wrapper in order to ensure that loop variables are closed over, and all the 
>generated functions don't just share the final values. CoffeeScript provides the 
>do keyword, which immediately invokes a passed function, forwarding any arguments.

"All the generated functions don't just share the final values." Huh. This statement
didn't mean a lot to me. A little searching on Google led to a fairly helpful post
on [Stack Overflow](http://stackoverflow.com/questions/643542/doesnt-javascript-support-closures-with-local-variables/643664#643664). The question there presented the following Javascript:


``` javascript
var closures = [];
function create() {
  for (var i = 0; i < 5; i++) {
    closures[i] = function() {
      alert("i = " + i);
    };
  }
}

function run() {
  for (var i = 0; i < 5; i++) {
    closures[i]();
  }
}

create();
run();
```

The question stated that the result of running this code was `5,5,5,5,5` instead
of the expect `0,1,2,3,4`. The answer which created the "correct" result
provided the code below:

``` javascript

function create() {
  for (var i = 0; i < 5; i++) {
    closures[i] = (function(tmp) {
      return function() {
        alert("i = " + tmp);
      };
    })(i);
  }
}
```

and also explained:

> JavaScript's scopes are function-level, not block-level, and creating a closure just means that the enclosing scope gets added to the lexical environment of the enclosed function.

OK, this was even less helpful. But after going back and reading the Coffeescript
docs again, the concept made sense. As always with Javascript, the *context* in 
which a function is called demands as much attention as the function itself. Reading
the code in the Stack Overflow post, a developer's brain automatically thinks of the
returned function at the time of evaluation. But in reality, each of those functions
created in the `create` function aren't called until *after* the for loop has completed.
Thus, by the time each function is called, it evaluates `i` when `i` has already
finished the loop with a value of `5`. The closure-wrapping provided in the answer
is exactly what the `do` keyword in Coffeescript provides!

Here's an example of the solution's function, in Coffeescript: 

``` coffeescript
closures = []
create ->
  for i in [0..5]
    closures[i] = do (i) ->
      -> alert i

```

Even with this contrived example, I find the Coffeescript easier to read, although
of course it's personal preference. If you wanted to go really crazy with Coffeescript
syntax, you could write:

``` coffeescript
  closures = ((do (i) -> -> alert i) for i in [0..5])
```

Two points for cool implementation, no points for readability!
