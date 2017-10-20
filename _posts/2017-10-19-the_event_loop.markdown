---
layout: post
title:      "The Event Loop"
date:       2017-10-20 00:46:19 +0000
permalink:  the_event_loop
---


I have been preparing for common interview questions given to entry level web developers. One area of note is the Javascript event loop, where I have seen general questions about it as well as some code related questions. It makes sense that a web developer might get a question related to the event loop since it is a key underlying aspect of asynchronous behavior in Javascript.

Javacript is single threaded which means the Javascript engine runs through and processes your code line by line. It can only do one thing at a time. This means that if you have slow code that takes a long time to run, it will be holding up the rest of the code below it (called blocking). To avoid blocking, Javascript uses asynchronous callbacks and the event loop to work around this single threaded limitation.

When you make an asynch call such as a fetch request, the Javascript engine processes that initial action which sends the http request.

```
function fetchCurrent(){
  return fetch("someapi/current")
    .then(response => {
      console.log("current received");
      return response.json();
      })
}
```

Instead of waiting for a response and blocking the runtime, the Javascript engine will put the callback function on the task queue. In the above example we use some ES6 syntax for the callback function in the `.then` which logs a message to the console and returns the json response.

The callback will eventually get executed when it is pulled off the task queue in what is called the event loop.

The event loop is just the loop that runs whenever the Javascript runtime is free (the call stack is clear). So while the single threaded Javascript engine is going through code, the call stack is being used and the event loop does not run. But once the runtime has processed all its given code, the event loop can run. The event loop pulls the first item off the front of the task queue and the Javascript engine runs that code.

This pattern allows for asynchronous behavior in Javascript even though the runtime is single threaded. The engine can continue processing code without being blocked by slow actions such as network requests. The callbacks for those requests will be processed once everything else in the code has been processed.

## An Example Problem

```
function timing() {
   console.log(1); 
   setTimeout(function() { console.log(2); }, 1000); 
   setTimeout(function() { console.log(3); }, 0); 
   console.log(4);
}

timing();
```

The code above is similar to a question I had in an interview. I was asked to explain the order of the output and the reasons for that order.

The order of output for this function is 1,4,3,2. This question is basically testing the interviewee's understanding of the event loop and asynchronous callbacks in Javascript. Since `setTimeout` is an asynchronous function, it does not block the Javascript runtime. 1 will print, then 4 will print. 2 and 3 are in callbacks which will only execute after the the rest of the code has completed.

Once the last code has executed with `console.log(4)`, the event loop pulls the first task off the task queue. In this case it will be the function to `console.log(3)`. 3 preceeds 2 on the task queue since 2 has a timeout delay of 1000 milliseconds. 2 is only added to the task queue after that delay (this is handled by the browser), while the callback for 3 is added immediately.



