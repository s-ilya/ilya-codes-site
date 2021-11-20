---
layout: base.njk
title: Jest mocking practices
date: 2021-11-17
tags: post
---
# {{title}}

Mocking dependecies is a necessary tool for any unit testing environment. Let's not dwell too much on basic usage of `jest.fn` which is described brillaintly in the [official documentation](https://jestjs.io/docs/mock-functions) and hop on to the problems one can encounter in daily work

In order to isolate function under test from surroundings we usually choose to mock its dependencies. The choise to provide dependencies via imports as opposed to injection as function parameter is arguable, but it seems to be widely adopted in JS world, so we have to be prepared. Compare these examples:
```ts
import {fortyTwo} from 'fortyTwo'

// implicitly uses dependecy, needs import mocking as a result
function fortyThree() {
  return 1 + fortyTwo()
}

// explicitly states its dependencies, so allows to have jest.fn passed as an argument
function addOne(fortyTwo: () => number) {
  return 1 + fortyTwo()
}
```
We will be focusing more on `fortyThree` and alike because it helps us to explore mocking possibilities of Jest further

## Default and named exports
Jest does good job at providing us with an api to mock packages or local dependecies. All we need is just to call `jest.mock` with the path to the dependency as an argument (should look pretty much like the one in import expression)
```ts
export function fortyTwo(): number {
  throw new Error("42")
}
```
```ts
import { fortyTwo } from "./fortyTwo";

export function fortyThree(): number {
  return 1 + fortyTwo()
}
```
Then mocking `fortyTwo` is as trivial as
```ts
 // mind the placement. Either with help of babel plugin or manually jest.mock call should be placed before actual import
jest.mock("../src/fortyTwo");

import { fortyThree } from "../src/fortyThree";
import { fortyTwo } from "../src/fortyTwo";

describe("fortyThree", () => {
  it("should return 43", () => {
    // sadly ts doesn't really know fortyTwo now of "jest.Mock & () => number" type
    (fortyTwo as jest.Mock).mockReturnValueOnce(42)

    expect(fortyThree()).toBe(43);
  });
});
```
Things get a bit more hairy with default export
```ts
export default function fortyTwo(): number {
  throw new Error("42")
}
```
```ts
import fortyTwo from "./fortyTwo";

export function fortyThree(): number {
  return 1 + fortyTwo()
}
```
In this case we should mimick the way js expects module to look like
```ts
jest.mock("../src/fortyTwo", () => {
  return {
    __esModule: true,
    default: jest.fn(), // by convention this is how default export looks like
  }
});

import { fortyThree } from "../src/fortyThree";
import fortyTwo from "../src/fortyTwo";

describe("fortyThree", () => {
  it("should return 43", () => {
    (fortyTwo as jest.Mock).mockReturnValueOnce(42)

    expect(fortyThree()).toBe(43);
  });
});
```

Still, Jest team does a great job explaining that in [their docs](https://jestjs.io/docs/jest-object#jestmockmodulename-factory-options)

## Globals
Imagine using some library not imported via js modules, but added to global namespace
```ts
declare function fortyTwo(): number

export function fortyThree(): number {
  return 1 + fortyTwo()
}
```

In order to mock global propery we will create it on [node's `global` object](https://developer.mozilla.org/en-US/docs/Glossary/Global_object)
```ts
import { fortyThree } from "../src/fortyThree";

describe("fortyThree", () => {
  it("should return 43", () => {
    Object.defineProperty(global, "fortyTwo", {
      value: jest.fn().mockReturnValueOnce(42)
    })

    expect(fortyThree()).toBe(43);
  });
});

```
## Testing async code
Rarely it's the case that we just use synchronously executed code. Testing async code is a crusial ability of unit testing frameworks. Thankfully it's a breeze with Jest
```ts
export async function fortyTwo(): Promise<number> {
  throw new Error("42")
}
```
```ts
import { fortyTwo } from "./fortyTwo"

export async function fortyThree(): Promise<number> {
  return 1 + await fortyTwo()
}
```
Testing is pretty much similar to sync function, mind adding `async`, `await` and changed mocking method call:
```ts
jest.mock("../src/fortyTwo")

import { fortyThree } from "../src/fortyThree";
import { fortyTwo } from "../src/fortyTwo"

describe("fortyThree", () => {
  it("should return 43", async () => {
    (fortyTwo as jest.Mock).mockResolvedValueOnce(42)

    expect(await fortyThree()).toBe(43);
  });
});
```
## Working with time 
Let's cover another example: using timers in your code
```ts
export function fortyTwo(): number {
  return 42
}
```
```ts
import { fortyTwo } from "./fortyTwo"

export async function fortyThree(): Promise<number> {
  return new Promise((resolve) => {
    setTimeout(() => resolve(1 + fortyTwo()), 5000)
  })
}
```
It's possible to test async function `fortyThree` as we discussed earlier, but it will take 5 real seconds to do so. In order to get around such inconvenience we will use Jest's fake timers api
```ts
jest.mock("../src/fortyTwo")

import { fortyThree } from "../src/fortyThree";
import { fortyTwo } from "../src/fortyTwo"

jest.useFakeTimers()

describe("fortyThree", () => {
  it("should return 43", async () => {
    const fortyTwoMock = (fortyTwo as jest.Mock & typeof fortyTwo)
    fortyTwoMock.mockReturnValueOnce(42)
    
    const answer = fortyThree()
    // for illustration purpose check that we haven't called fortyTwo dependency yet
    expect(fortyTwoMock).not.toBeCalled()

    // execute all scheduled actions
    jest.runAllTimers()

    expect(fortyTwoMock).toHaveBeenCalledTimes(1)
    expect(await answer).toBe(43);
  });
});
```
Please consider reading through [Jest timers docs](https://jestjs.io/docs/timer-mocks) which provides a broad toolset to cover pretty much any case of testing timers in your code