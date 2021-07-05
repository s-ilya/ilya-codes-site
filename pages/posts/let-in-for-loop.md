---
layout: base.njk
title: Why let in for loop acts like it creates a new variable each time
date: 2021-06-05
tags: post
---
# {{ title }}

## Initializing variables in `for` statement

First, let's see this code example

```jsx
for (var i = 0; i < 3; i++) {
	setTimeout(() => console.log(i))
}
```

It will work like if `i` was a single memory chunk being changed in each loop and then read by scheduled functions

```jsx
3
3
3
```

One way to "fix" that is too create a copy of `i` each time. `i` is passed as an argument to immediately invoked function and a new variable is created each time (`copyOfi`) with the `i` value at a time:

```jsx
for (var i = 0; i < 3; i++) {
  ((copyOfi) => {
    setTimeout(() => console.log(copyOfi))  
  })(i)
}
```

```jsx
0
1
2
```

`for` loop with `let` initialization results in the same behavior. But why does it work like it creates a new variable each time? And if it creates a new variable, why can't we use `const`?

```jsx
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i))
}
```

```jsx
// Uncaught TypeError: Assignment to constant variable.
for (const i = 0; i < 3; i++) {
  setTimeout(() => console.log(i))
}
```

## How does JavaScript uses let variables in `for` statement?

Let's dig into the JavaScript specification to find out: [https://262.ecma-international.org/12.0/#sec-runtime-semantics-forloopevaluation](https://262.ecma-international.org/12.0/#sec-runtime-semantics-forloopevaluation)

What it says to us is: hey, I'm creating a new lexical scope for this loop and creating your variables here. So, at step 9 when we're ready to execute for loop for the first time and we have a freshly created scope with mutable `i` there with its initial value `0`

![Lexical scope with the initial variable in it](/img/let-in-for-loop/initial.png "For loop scope before any iteration")

Then comes ForBodyEvaluation: [https://262.ecma-international.org/12.0/#sec-forbodyevaluation](https://262.ecma-international.org/12.0/#sec-forbodyevaluation) What is interesting here is that it creates a new lexical scope for each loop iteration and copies previous initialization values there (but not those that were created inside the loop statement itself). Test and increment expressions, and `for` body itself run in this newly created scope.

![Two lexical scopes. The second one was created for the first loop of for statement. It contains its variable with the value copied from the "outer" scope. It shows how setTimeout function uses that particular variable from the second scope thus making it separate from the one in the first one](/img/let-in-for-loop/first-iteration.png "For loop scopes during the first iteration")

The same goes for the following loops.

![Sums up two previous images by adding the third scope for the second loop iteration. The third scope copies the value from the second one.](/img/let-in-for-loop/second-iteration.png "For loop scopes during the second iteration")

So, even though strictly speaking `i` is still the same single variable, it is effectively a new chunk of memory at each loop iteration, hence it fixes our `setTimeout` example. 

## But if it is a new piece of memory, why can't we use `const` in our example?

Again, let's check out the specification: [https://262.ecma-international.org/12.0/#sec-runtime-semantics-forloopevaluation](https://262.ecma-international.org/12.0/#sec-runtime-semantics-forloopevaluation) 

It's a bit easier: step 9 says, that only mutable variables go to each loop scope, while const ones stay in general `for` expression scope that was created once and being referenced by each loop scope as running execution context (step 6) and not re-created. So, trying to increment const will fail as expected