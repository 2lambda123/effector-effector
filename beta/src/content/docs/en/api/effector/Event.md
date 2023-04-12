---
title: Event
keywords:
  - event
  - unit
description: Event, its methods and properties
redirectFrom:
  - /api/effector/Event
  - /docs/api/effector/event
---

_Event_ is an intention to change the state. Let's imagine a life situation, you enter a shop and, according to etiquette, you have to say "hello" – **intention**, then you say "hello" – **event**.

Event calls always return its payload:

```js
import { createEvent } from "effector";

const event = createEvent();

console.log(event(1));
// => 1
console.log(event());
// => undefined
```

[Try it](https://share.effector.dev/iVBYDJjf)

<br/><br/>

# Methods {#methods}

## `watch(watcher)` {#watch-watcher}

It is a function which allows you to watch the event or to create side effects.

### Formulae {#watch-watcher-formulae}

```ts
const unwatch = event.watch(fn);
```

- Call `fn` on each `event` trigger, pass payload of `event` as argument to `fn`
- When `unwatch` is called, stop calling `fn` on each `event` trigger

<!--If you want to know, when watch is called, welcome to the advanced section-->

### Arguments {#watch-watcher-arguments}

1. `watcher` ([_Watcher_](/en/explanation/glossary#watcher)): A function that receives `payload`.

### Returns {#watch-watcher-returns}

[_Subscription_](/en/explanation/glossary#subscription): Unsubscribe function.

### Example {#watch-watcher-example}

```js
import { createEvent } from "effector";

const sayHi = createEvent();
const unwatch = sayHi.watch((name) => console.log(`${name}, hi there!`));

sayHi("Peter"); // => Peter, hi there!
unwatch();

sayHi("Drew"); // => nothing happened
```

[Try it](https://share.effector.dev/9YVgCl4C)

## `map(fn)` {#map-fn}

Creates a new event, which will be called after the original event is called, applying the result of a `fn` as a payload. It is a special function which allows you to decompose dataflow, extract or transform data.

### Formulae {#map-fn-formulae}

```ts
const second = first.map(fn);
```

- When `first` is triggered, pass payload from `first` to `fn`
- Trigger `second` with the result of the `fn()` call as payload

### Arguments {#map-fn-arguments}

1. `fn` (_Function_): A function that receives `payload`, [should be **pure**](/en/explanation/glossary#purity).

### Returns {#map-fn-returns}

[_Event_](/en/api/effector/Event): New event.

### Example {#map-fn-example}

```js
import { createEvent } from "effector";

const userUpdated = createEvent();
const userNameUpdated = userUpdated.map(({ name }) => name); // you may decompose dataflow with .map() method
const userRoleUpdated = userUpdated.map(({ role }) => role.toUpperCase()); // either way you can transform data

userNameUpdated.watch((name) => console.log(`User's name is [${name}] now`));
userRoleUpdated.watch((role) => console.log(`User's role is [${role}] now`));

userUpdated({ name: "john", role: "admin" });
// => User's name is [john] now
// => User's role is [ADMIN] now
```

[Try it](https://share.effector.dev/duDut6nR)

## `prepend(fn)` {#prepend-fn}

Creates an event, upon trigger it sends transformed data into the source event. Works kind of like reverse `.map`. In case of `.prepend` data transforms **before the original event occurs** and in the case of `.map`, data transforms **after original event occurred**.

If the original event belongs to some [domain](/en/api/effector/Domain), then a new event will belong to it as well

### Formulae {#prepend-fn-formulae}

```ts
const second = first.prepend(fn);
```

- When `second` event is triggered
- Call `fn` with payload from `second`
- Trigger `first` with the result of `fn()`

### Arguments {#prepend-fn-arguments}

1. `fn` (_Function_): A function that receives `payload`, [should be **pure**](/en/explanation/glossary#purity).

### Returns {#prepend-fn-returns}

[_Event_](/en/api/effector/Event): New event.

### Example {#prepend-fn-example}

```js
import { createEvent } from "effector";

const userPropertyChanged = createEvent();

userPropertyChanged.watch(({ field, value }) => {
  console.log(`User property "${field}" changed to ${value}`);
});

const changeName = userPropertyChanged.prepend((name) => ({
  field: "name",
  value: name,
}));
const changeRole = userPropertyChanged.prepend((role) => ({
  field: "role",
  value: role.toUpperCase(),
}));

changeName("john");
// => User property "name" changed to john

changeRole("admin");
// => User property "role" changed to ADMIN

changeName("alice");
// => User property "name" changed to alice
```

[Try it](https://share.effector.dev/XGxlG4LD)

## `filterMap(fn)` {#filterMap-fn}

:::info{title="since"}
[effector 20.0.0](https://changelog.effector.dev/#effector-20-0-0)
:::

Creates a new event, which will be called after the original event is called if `fn` returns a value other than **undefined**.
Consider a scenario where you walk into a grocery store with a specific task: you need to purchase 10 apples, but only if they're red. If they're not red, you're out of luck.
Let's consider by steps:

1. Take one apple;
2. Have a look, is it red(put in a pack) or not(take another).

And you repeat this until you complete the task. Now think about it in the effector terms, and we consider the positive case:

1. Take an apple – event;
2. Have a look, red or no – filter;
3. You keep it – map;
4. Put in pack – event.
5. Pack – store

You may see that we united `filter()` and `map()` methods, the reason for creating was an impossibility to event filtering.
The method is useful with JavaScript APIs those returns `undefined`.

### Formulae {#filterMa-formulae}

```ts
const second = first.filterMap(fn);
```

- When `first` is triggered, call `fn` with payload from `first`
  - If `fn()` returned `undefined` do not trigger `second`
  - If `fn()` returned some data, trigger `second` with data from `fn()`

### Arguments {#filterMap-fn-arguments}

1. `fn` (_Function_): A function that receives `payload`, [should be **pure**](/en/explanation/glossary#purity).

### Returns {#filterMap-fn-returns}

[_Event_](/en/api/effector/Event): New event.

### Example {#filterMap-fn-example}

```tsx
const listReceived = createEvent<string[]>();
const effectorFound = listReceived.filterMap((list) => list.find((name) => name === "effector"));

effectorFound.watch((name) => console.info("found", name));
listReceived(["redux", "effector", "mobx"]); // found effector
listReceived(["redux", "mobx"]);
```

[Try it](https://share.effector.dev/ARDanMAM)

## `filter({fn})` {#filter-fn}

Creates a new event, which will be called after the original event is called if `fn` returns `true`.

Let's assume a standard situation when you want to buy sneakers in the shop, but there is no size. You subscribe to the particular size of the sneakers' model, and in addition, you want to receive a notification if they have it, and ignore any other notification. Therefore, filtering can be helpful for that. Event filtering works in the same way. If `filter` returns `true`, the event will be called.

### Formulae {#filter-fn-formulae}

```ts
const second = first.filter({ fn });
```

- When `first` is triggered, pass payload from `first` to `fn`
- If `fn()` returns `true`, `second` will be triggered with payload from `first`

### Arguments {#filter-fn-arguments}

1. `fn` (_Function_): A function that receives `payload`, [should be **pure**](/en/explanation/glossary#purity).

### Returns {#filter-fn-returns}

[_Event_](/en/api/effector/Event): New event.

:::info
Object form is used because `event.filter(fn)` was an alias for [event.filterMap](/en/api/effector/Event#event-filterMap-fn)
:::

:::tip
[sample](/en/api/effector/sample) method with `filter` is the preferred filtering method
:::

### Example {#filter-fn-example}

```js
import { createEvent, createStore } from "effector";

const numbers = createEvent();
const positiveNumbers = numbers.filter({
  fn: ({ x }) => x > 0,
});

const $lastPositive = createStore(0).on(positiveNumbers, (n, { x }) => x);

$lastPositive.watch((x) => {
  console.log("last positive:", x);
});

// => last positive: 0

numbers({ x: 0 });
// no reaction

numbers({ x: -10 });
// no reaction

numbers({ x: 10 });
// => last positive: 10
```

[Try it](https://share.effector.dev/H2Iu4iJH)