---
layout: post
title:  "Const Var Let"
date:   2017-08-06 19:25:46 +0000
---


When I first learned about `const` and `let` I tried to look into how they were different from `var`. I was met by lengthy posts discussing block scope and new ES6 implementations. Since `const` and `let` were introduced with ES6 many of the write-ups about it came from an experienced developer standpoint, with a similar intended audience. This post is intended to be a short and simple explanation of the differences between `const` `var` and `let` in Javascript.

## var

The key difference between `var` and `const` and `let` is scope. When declaring a variable with `var`, the scope of that variable extends to the function it is in (global if it is not in a function).

```
function counter(num) {
	for (var i = 1; i <= num; i++) {
		console.log(i);
	}
	console.log(`In function scope ${i}`);
}

counter(4);
console.log(`Global scope ${i}`);
```

Here we have a simple function that prints to console from 1 to the num argument. The for loop uses the variable `i` (declared with `var`) to iterate from 1 through num (4 in our example). Immediately after the for loop the variable `i` is used again to print to the console. Using template literals, "In function scope 5" is printed to the console. Since variables declared with `var` have a **function** scope, we are still able to access `i` at this point. Variable `i` is equal to 5 here since it was increased from 4 to 5 prior to ending the for loop.

Our attempt to print the variable `i` on the last line fails because `i`'s scope is only within the counter function.

## let and const

Unlike what we saw above, `let` and `const`  are **block** scope. That means their scope only extends to their current block; their block being the curly braces that encompass their declaration. Using the same example as before we can see this difference.

```
function counter(num) {
	for (let i = 1; i <= num; i++) {
		console.log(i);
	}
	console.log(`In function scope ${i}`);
}

counter(4);
console.log(`Global scope ${i}`);
```

We changed the example to declare `i` using `let` rather than `var`. The counter function still prints out 1 through 4 during the for loop. However now we get a reference error when we try to access `i` at line 5 after the for loop. Now that `i` has a block scope it cannot be accessed outside of the for loop curly brace block. Also, the attempt to access `i` at the global level continues to fail.

Declaring variables with `const` results in block scope just like `let`. The difference with `const` is that, as the name implies, the variable is constant. So a variable declared with `const` cannot be reassigned or redeclared (within the same scope).

Hopefully this post gave you a better understanding of `var`, `let`, and `const`. If you are looking for further detail on this subject I suggest Wes Bos' post at http://wesbos.com/javascript-scoping/.
