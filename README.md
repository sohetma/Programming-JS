# Programming in JS :computer:

## Synchronous and Asynchronous

By default, JavaScript tasks are functions that are executed sequentially in a single process. It’s single-threaded. Js code runs synchronously. This means that a line of code is executed, then the next one is executed, and so on. However, sometimes you can not wait and halt completely all the program. We don’t want a task to block other tasks. Almost all the I/O primitives in JavaScript are non-blocking. Network requests, Node.js filesystem operations, and so on. Being blocking is the exception, and this is why JavaScript is based so much on callbacks, and more recently on promises and async/await :punch:

Let's take a simple example with the function ```setTimeout(callback, milliseconds)```

```
  const hiAntoine = () => console.log('Hi Antoine');
  
  const waitForIt = () => {
     setTimeout(() => {
      console.log('You are patient enough to wait 2 seconds minumum');
    }, 2000);
  }
  
  const hiRosa = () => console.log('Hi Rosa'); 
  
  const hello = () => {
    hiAntoine();
    waitForIt();
    hiRosa();
  }
  
  hello();
``` 
The result in the console will be :
```
  Hi Antoine
  Hi Rosa
  You are patient enough to wait 2 seconds minumum
```

So what's happen? 
In this example, the code is not blocked thanks to the callback, and then 'Hi Rosa' is printed before (we don't wait for 2 seconds). The callback function is executed asynchronously.
To give a correct explanation, we need to introduce some concepts and go under the hood of a browser.

### Javascript Engine 
An engine is a program that translates JS into machine code and execute codes results on a CPU. The most popular JS engine is V8 and used by most popular browsers such as Chrome.

### Call Stack 
The call stack is a LIFO queue (last in, first out) of data storage that stores the current function execution context of a program. When we execute a function, JS runtime pushes frame(variables, parameters,..) on top of the stack and when we return from a function it pops off the frame.

### Event loop
The event loop continuously checks the call stack to see if there’s any function that needs to run.
In most browsers there is an event loop for every browser tab, to make every process isolated.

![JS Runtile - Web Apis - Event Loop](browser.png)

A step-by-step explanation of the previous example when the code starts to run.
1. ```hello()```is called and pushed onto the call stack.  Inside this function, three other functions are called.
2. ``` console.log('Hi Antoine') ``` is pushed onto the call stack and we see in the console the message and we pop it off the stack.
3. Then ``` waitForIt() ```is called and a that moment  ```setTimeout()``` is pushed.```setTimeout()``` provides from the browser and the callback function is pushed in the Web Apis and the browser starts the timer.
4. The third function is called, push to the stack and we see on the console ``` Hi Rosa  ```. Then it is popped.
5. Once the timer expires, the Web API pushes the callback function to the callback queue. If and only if the stack is empty (one execution at a time), the event loop will put the callback on the stack, and then the function is executed.
6. ```console.log('You are patient enough to wait 2 seconds minimum')``` is then pushed onto the call stack then popped.
7. ```hello()``` is popped.

In the previous example, there are no promises or other js events (like onClick event) but ECMAScript 2015 introduced the concept of the job queue and the event queue. The job queue is filled with Promise resolve and reject functions. The event queue contains all the callbacks event functions. It is important to note that callbacks in the job queue have a higher priority of execution than callbacks in the event queue. That means that the event loop will execute all of them one by one before any other callback in the event queue. 

Take an other exemple.

```
const start = new Date().getSeconds();

const blockThread = (ms) => {
  return new Promise(res => setTimeout(res,ms));
}

var myPromise = new Promise(function(resolve, reject) {
  if (true) {
    resolve("Stuff worked!");
  }
  else {
    reject(Error("It broke"));
  }
});

blockThread(3000).then(() => console.log("1) Ran after " + (new Date().getSeconds() - start) + " seconds"));

myPromise.then(() => console.log("2) Ran after " + (new Date().getSeconds() - start) + " seconds"));

console.log("3) Ran after " + (new Date().getSeconds() - start) + " seconds");

setTimeout(() => {
  console.log("4) Ran after " + (new Date().getSeconds() - start) + " seconds");
}, 0);

myPromise.then(() => console.log("5) Ran after " + (new Date().getSeconds() - start) + " seconds"));
```
Thanks to the previous explanation, we can easily understand the order of the ```console.log()```

```
3) Ran after 0 seconds
2) Ran after 0 seconds
5) Ran after 0 seconds
4) Ran after 0 seconds
1) Ran after 3 seconds
```

I invite all readers to check the online [tool](http://latentflip.com/loupe/?code=ZnVuY3Rpb24gcHJpbnRIZWxsbygpIHsNCiAgICBjb25zb2xlLmxvZygnSGVsbG8gZnJvbSBiYXonKTsNCn0NCg0KZnVuY3Rpb24gYmF6KCkgew0KICAgIHNldFRpbWVvdXQocHJpbnRIZWxsbywgMzAwMCk7DQp9DQoNCmZ1bmN0aW9uIGJhcigpIHsNCiAgICBiYXooKTsNCn0NCg0KZnVuY3Rpb24gZm9vKCkgew0KICAgIGJhcigpOw0KfQ0KDQpmb28oKTs%3D!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D) created by Philip Robers :heart_eyes:

## Closure in js
A closure is a feature in JavaScript where an inner function has access to the outer (enclosing) function’s variables — a scope chain.
The closure has three scope chains:
 - it has access to its own scope — variables defined between its curly brackets
 - it has access to the outer function’s variables
 - it has access to the global variables

Take an example

```
const greeting = (name) => {
  var message = 'Hello';
  const sayHello = () => {
    console.log(message+' '+name);
  }
  return sayHello;
}

var greetingFx = greeting('Fx');
var greetingNico = greeting('Nico');

greetingFx();
```

```greeting``` returns a closure that is assigned to ```greetingFx```. The function ```greetingFx``` has access to the parameter ```name```, the free variable ```message``` and ```sayHello```. Use ```console.dir(greetingFx)``` to see the scope and the closure.

Closures are useful because they let you associate data (the lexical environment) with a function that operates on that data. This has obvious parallels to object-oriented programming, where objects allow you to associate data (the object's properties) with one or more methods. Consequently, you can use a closure anywhere that you might normally use an object with only a single method.

Another small and fun example. Take a web page and go inside the main ```div``` with an id and copy-paste this code and replace the id in the code below. 

```
function changePageColor(color) {
  var size = 
  return function() {
    document.body.style.color = color;
  };
}

var textBlue = changePageColor('blue');
var textRed = changePageColor('red');

document.getElementById('ID').onclick = textBlue;
setTimeout(()=> {
 document.getElementById('ID').onclick = textRed;
},2000);
```

## Promises :rocket:



## The keyword ``` this ``` in js :hushed:

- By default, this refers to a global object, which is global in the case of NodeJS and a window object in the case of a browser
- When a method is called as a property of an object, then this refers to the parent object
- When a function is called with the new operator, then this refers to the newly created instance
- When a function is called using the call and apply methods, then this refers to the value passed as the first argument of the call or apply method


## Sources
 - [How JavaScript works in browser and node?](https://itnext.io/how-javascript-works-in-browser-and-node-ab7d0d09ac2f)
 - [What the heck is the event loop anyway? | Philip Roberts](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)
 - [JavaScript : Under The Hood of a Browser](https://medium.com/better-programming/javascript-internals-under-the-hood-of-a-browser-f357378cc922)
  - [Understanding Non-Blocking I/O in JavaScript](https://www.codementor.io/@theresamostert/understanding-non-blocking-i-o-in-javascript-cvmg1hp6l)
  - [Nodejs.org : blocking vs non-blocking](https://nodejs.org/en/docs/guides/blocking-vs-non-blocking/)
  - [Eploiringjs : JavaScript for impatient programmers](https://exploringjs.com/impatient-js/ch_variables-assignment.html#closures)
