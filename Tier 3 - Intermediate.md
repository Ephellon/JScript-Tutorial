# JScript at Work — Tier 3: Intermediate

Welcome to Tier 3. By the end you'll write complete utility scripts solo: scripts that process files, run commands, parse arguments, work with dates, and use the full power of modern JavaScript via baseline.

What we'll cover:

- **Template literals** — proper coverage now that we're committed to modern syntax
- **Objects** — key/value structures, the foundation of larger code
- **Classes** — bundled state and behavior
- **Date and time** — JScript's `Date` and the manual formatting you'll need
- **FileSystemObject** — reading, writing, listing files
- **WScript.Shell** — running commands, environment variables, capturing output
- **Regular expressions** — pattern matching for text
- **Modules** — splitting code across files via `import`/`export`
- **Argument parsing** — named vs positional, building parsed-args objects
- **The rest of modern syntax** — generators, `async`/`await`, exponentiation, polyfilled globals

The capstone for Tier 3 is a real maintenance script — file processing, error handling, structured output. The kind of thing that earns its place in the toolkit folder.

From here on, examples use modern syntax. If you see something unfamiliar, check Lesson 2.8.

---

## Lesson 3.1: Template literals — a closer look

> **New words in this lesson**
>
> - **template literal** — a string written between backticks (`` ` ``) instead of regular quotes
> - **interpolation** — inserting a value into a string with `${...}`
> - **backtick** — the `` ` `` character (not the same as a single quote)
> - **expression** — anything that produces a value (a number, a calculation, a function call, etc.)
> - **ternary** — the `condition ? a : b` shortcut for if/else
> - **escape** — using `\` to include a special character literally

You met template literals in Lesson 2.8 — backtick strings with `${}` interpolation. We're covering them properly now, since the rest of Tier 3 leans on them heavily.

### Basic syntax

A template literal is a string written between backticks instead of single or double quotes:

```js
const greeting = `Hello, world!`;
WScript.Echo(greeting); // Hello, world!
```

By itself that's no different from `"Hello, world!"`. What backticks unlock is two related features: **interpolation** and **multi-line strings**.

### Interpolation with `${}`

Inside a template literal, `${expression}` evaluates the expression and inserts its string form:

```js
const name = "Sam";
const age = 34;

WScript.Echo(`${name} is ${age} years old.`);
// Sam is 34 years old.
```

The expression can be anything that produces a value:

```js
const a = 5;
const b = 7;

WScript.Echo(`The sum is ${a + b}.`);                   // The sum is 12.
WScript.Echo(`The square is ${a * a}.`);                // The square is 25.
WScript.Echo(`Status: ${a > b ? "first" : "second"}.`); // Status: second.
WScript.Echo(`Random: ${Math.random()}`);               // Random: 0.483...
```

Function calls work too:

```js
function upper(s) {
    return s.toUpperCase();
}

const name = "sam";
WScript.Echo(`Hello, ${upper(name)}!`); // Hello, SAM!
```

You can nest template literals inside `${}`, though it gets ugly quickly:

```js
const items = ["apple", "banana"];
WScript.Echo(`First: ${`item is ${items[0]}`}`);
// First: item is apple
```

If a `${}` expression evaluates to a non-string, it gets converted to a string the same way `+` would — `${42}` becomes `"42"`, `${null}` becomes `"null"`, `${[1,2,3]}` becomes `"1,2,3"`.

### Multi-line strings

Backtick strings preserve line breaks literally:

```js
const block = `line one
line two
line three`;

WScript.Echo(block);
// line one
// line two
// line three
```

No `\n` escapes needed.

Indentation inside the string is preserved exactly as you write it — which is occasionally a footgun:

```js
function makeBlock() {
    return `line one
        line two`;
}

WScript.Echo(makeBlock());
// line one
//         line two   ← indented because the source was indented
```

For multi-line content where indentation would corrupt the output, either start the string on its own line and don't indent, or build it from an array and `.join`:

```js
const block = [
    "line one",
    "line two",
    "line three"
].join("\n");
```

That's the same multi-line error pattern from Lesson 2.6 — still preferred for error messages and other content where mixing source indentation with output would be confusing.

### Escaping backticks and `${`

If you need a literal backtick or a literal `${` inside a template, escape with `\`:

```js
WScript.Echo(`This is a backtick: \``);     // This is a backtick: `
WScript.Echo(`Not interpolated: \${name}`); // Not interpolated: ${name}
```

In practice you rarely need either.

### When to use templates — and when not

Reach for a template literal when:

- You're inserting one or more values into a fixed string format (`Hello, ${name}!`).
- You're building a multi-line string clearer as a literal than as `\n`-laden concatenation.
- You're constructing a log line, a path, or a generated piece of code.

Stick with a regular string when:

- It's a fixed string with no interpolation. (`"Hello, world!"` is fine — no need to switch.)
- You're building a multi-line message where indentation would corrupt the output — use the `Array.join` pattern from 2.6.
- The content is user-facing and the dual-quote convention applies. Doubles for user-facing, singles for symbols. Backticks live in their own category, for *generated* or *interpolated* output.

In the codebase, you'll find templates everywhere code is being constructed: error messages, generated source, formatted reports, log lines, and CLI banners.

### Exercises

1. **Greeting upgrade.** Take any `Echo` from Tier 2 that used `+` concatenation to insert a value, and rewrite it as a template literal. Run both versions to confirm identical output.
2. **Status report.** Create three variables: `taskName` (string), `taskCount` (number), `isUrgent` (boolean). Write a single template literal that produces a sentence like *"Task 'cleanup': 5 items, urgent."* — using interpolation for all three values, and a ternary to make the `, urgent` part appear only when `isUrgent` is true.
3. **Multi-line tribute.** Build a multi-line string (using a backtick literal) that prints a 4-line poem of your choosing, with `Echo` outputting it in one call.
4. **Multi-line via join.** Build the same 4-line output, but using `Array.join("\n")` instead of a backtick literal. Note which one you'd reach for in production code where source indentation could change.
5. **Path builder.** Write a function `buildPath(folder, filename)` (with header comment) that returns a Windows-style path using a template literal: `"C:\\Reports\\<folder>\\<filename>"`. Test with `buildPath("2025", "summary.txt")`.

---

## Lesson 3.2: Objects

> **New words in this lesson**
>
> - **object** — a collection of named values (key/value pairs)
> - **property** — one named value inside an object
> - **key** — the name half of a property
> - **value** — the data half of a property
> - **method** — a function attached to an object
> - **`this`** — a special variable that refers to the current object inside a method
> - **reference** — a "pointer" to an object; copying a reference doesn't copy the object itself
> - **lookup table** — an object used to look up values by name
> - **prototype chain** — JScript's internal mechanism for inheritance (you can mostly ignore it)
> - **destructuring** — pulling values out of an object/array into separate variables in one statement
> - **spread** — unpacking an array or object inline using `...`
> - **rest** — collecting remaining items into an array using `...`
> - **shallow merge** — copying only top-level properties; nested objects stay shared
> - **optional chaining** — safely reading nested properties using `?.`
> - **nullish coalescing** — providing a default with `??` only when something is `null`/`undefined`

An **object** is a collection of key-value pairs, also called **properties**. Where an array is an ordered list, an object is a labeled bag. Use objects when each piece of data has a distinct name (a person's `name`, `age`, `email`); use arrays when each piece is just one of many of the same kind.

### Object literals

The literal syntax:

```js
const person = { name: "Sam", age: 34, isOnTheTeam: true };

const empty = {};
```

**Style note:** unlike arrays, object literals **are** padded inside the braces. `{ name: "Sam" }`, never `{name:"Sam"}`. Empty objects are `{}` (no padding for empty).

Multi-property objects can span multiple lines:

```js
const person = {
    name: "Sam",
    age: 34,
    isOnTheTeam: true
};
```

### Keys

Property keys are strings. You can write them with or without quotes:

```js
const obj1 = { name: "Sam" };          // unquoted — common
const obj2 = { "name": "Sam" };        // quoted — equivalent
const obj3 = { "first name": "Sam" };  // quoted — required (contains a space)
```

Unquoted keys must be valid identifiers. When in doubt, quoting is safe.

### Property access

Two ways to read or write a property.

**Dot notation** — when the key is a known identifier:

```js
const person = { name: "Sam", age: 34 };

WScript.Echo(person.name); // "Sam"
person.age = 35;
WScript.Echo(person.age);  // 35
```

**Bracket notation** — when the key is dynamic (in a variable) or contains special characters:

```js
const person = { name: "Sam", "first name": "Samuel" };

WScript.Echo(person["name"]);       // "Sam"
WScript.Echo(person["first name"]); // "Samuel"

const key = "name";
WScript.Echo(person[key]); // "Sam" — uses the variable
```

Use dot notation by default; switch to brackets when you have a dynamic key or a key with special characters.

### Adding and removing properties

Assigning to a new key adds it:

```js
const person = { name: "Sam" };
person.age = 34;
WScript.Echo(person.age); // 34
```

Removing requires the `delete` operator:

```js
delete person.age;
WScript.Echo(person.age); // undefined
```

Reading a missing key returns `undefined` — no error.

### Methods — functions as properties

A property whose value is a function is a **method**:

```js
const dog = {
    name: "Rex",
    bark: function() {
        WScript.Echo("Woof!");
    }
};

dog.bark(); // Woof!
```

There's a shorthand for methods in object literals:

```js
const dog = {
    name: "Rex",
    bark() {
        WScript.Echo("Woof!");
    }
};

dog.bark(); // Woof!
```

Both produce the same object. Use the shorthand when it works.

### `this`

Inside a method, `this` refers to the object the method was called on:

```js
const dog = {
    name: "Rex",
    introduce() {
        WScript.Echo(`Hi, I'm ${this.name}.`);
    }
};

dog.introduce(); // Hi, I'm Rex.
```

`this` works as expected in:

- Object methods called via `obj.method()`
- Class methods (Lesson 3.3)
- Regular `function` declarations called appropriately

`this` does **not** work the same way in arrow functions — an arrow function inherits `this` from the surrounding scope, not the object it's attached to. Sometimes useful, sometimes a footgun. Rule of thumb: use regular `function` (or method shorthand) for object methods, arrow functions for everything else.

### Reference semantics — important

Objects are stored and passed **by reference**. This is different from primitives.

```js
const a = 5;
let b = a;
b = 10;
WScript.Echo(a); // 5 — unchanged

const obj1 = { count: 5 };
const obj2 = obj1;
obj2.count = 10;
WScript.Echo(obj1.count); // 10 — both names point to the same object
```

`obj2 = obj1` doesn't copy the object; it copies the reference. Both names now point to the same underlying object.

This matters when passing objects to functions:

```js
function increment(o) {
    ++o.count;
}

const obj = { count: 5 };
increment(obj);
WScript.Echo(obj.count); // 6 — modified inside the function, change visible here
```

For primitives, the function gets a copy. For objects, it gets a reference to the same object. **Functions can mutate objects passed to them.** Sometimes that's what you want; sometimes it's a bug.

The other consequence: `===` on objects compares **references**, not contents.

```js
const a = { x: 1 };
const b = { x: 1 };
const c = a;

WScript.Echo(a === b); // false — different objects, same contents
WScript.Echo(a === c); // true  — same object
```

Comparing object contents requires checking properties one by one, or using `JSON.stringify` for a structural compare.

### Iterating with `for...in`

To loop over all keys in an object, use `for...in` — and remember the labeled-loop rule:

```js
const counts = { apple: 3, banana: 5, cherry: 2 };

iter_counts: for(const key in counts) {
    WScript.Echo(`${key}: ${counts[key]}`);
}
```

### `for...in` gotcha — the prototype chain

`for...in` iterates over **all enumerable properties**, including inherited ones from the prototype chain. If anyone modifies `Object.prototype` (a known anti-pattern that legacy code occasionally does), those properties leak into every `for...in` loop on every object.

Defensive pattern using `hasOwnProperty`:

```js
iter_counts: for(const key in counts) {
    if(!counts.hasOwnProperty(key))
        continue iter_counts;

    WScript.Echo(`${key}: ${counts[key]}`);
}
```

That's a lot of ceremony for every loop. The codebase prefers a different solution.

### `Object.create(null)` — the lookup-table pattern

For objects used as **lookup tables** — where keys are dynamic and values are looked up by them — the style guide mandates creating them with `Object.create(null)`:

```js
const counts = Object.create(null);
counts.apple = 3;
counts.banana = 5;
counts.cherry = 2;
```

The result is an object with **no prototype chain**. No inheritance, no `.hasOwnProperty`, no `.toString`, nothing. Just the keys you've put in.

Why? Two reasons:

1. **Safety** — `for...in` always iterates only your keys.
2. **Correctness** — `key in obj` accurately tells you whether `key` is one of yours, without false positives like `"constructor" in {}` returning `true`.

The cost: you can't call methods like `.hasOwnProperty` on it directly (no prototype). Almost never an issue in practice.

Use the regular `{}` literal for objects representing **records** with a known shape (`{ name, age, email }`). Use `Object.create(null)` for objects acting as **maps** or **lookup tables**.

### Listing keys with `Object.keys`

`Object.keys(obj)` returns an array of an object's own enumerable property names:

```js
const counts = { apple: 3, banana: 5, cherry: 2 };
const keys = Object.keys(counts);

iter_keys: for(let i = 0; i < keys.length; ++i)
    WScript.Echo(`${keys[i]}: ${counts[keys[i]]}`);
```

Often cleaner than `for...in` because:

- It only returns the object's own keys (no prototype-chain surprises).
- You get a regular array — sortable, filterable, etc.
- It plays well with the standard array iteration pattern.

`Object.values(obj)` and `Object.entries(obj)` exist and behave as you'd expect, but `Object.keys` plus indexing is the most universally available pattern.

### Destructuring

Destructuring lets you pull values out of objects (or arrays) into separate variables, in one statement.

**Object destructuring:**

```js
const person = { name: "Sam", age: 34, email: "sam@example.com" };

const { name, age } = person;

WScript.Echo(name); // "Sam"
WScript.Echo(age);  // 34
```

The variable names must match the property names. To rename:

```js
const { name: fullName, age: yearsOld } = person;

WScript.Echo(fullName); // "Sam"
WScript.Echo(yearsOld); // 34
```

Default values for missing properties:

```js
const { email = "unknown" } = { name: "Sam" };
WScript.Echo(email); // "unknown" — default kicked in
```

**Array destructuring:**

```js
const [first, second, third] = [10, 20, 30];

WScript.Echo(first);  // 10
WScript.Echo(second); // 20
WScript.Echo(third);  // 30
```

Skip elements with empty slots:

```js
const [, , third] = [10, 20, 30];
WScript.Echo(third); // 30
```

Destructuring works in function parameters too:

```js
function describe({ name, age }) {
    WScript.Echo(`${name} is ${age}`);
}

describe({ name: "Sam", age: 34 });
```

Heavily used in the codebase — most functions that take "options" objects destructure them right in the parameter list.

### Spread (`...`)

The spread operator unpacks the contents of an array or object inline.

**Spread an array into a function call:**

```js
const nums = [1, 2, 3];
WScript.Echo(Math.max(...nums)); // 3
```

That's the answer to "how do I pass an array to a variadic function?" — spread it.

**Spread an array into another array:**

```js
const a = [1, 2, 3];
const b = [4, 5, 6];
const combined = [...a, ...b];

WScript.Echo(combined); // 1,2,3,4,5,6
```

**Spread an object into another object:**

```js
const base = { theme: "dark", fontSize: 14 };
const override = { fontSize: 16 };
const merged = { ...base, ...override };

WScript.Echo(merged.fontSize); // 16 — later overrides earlier
WScript.Echo(merged.theme);    // "dark"
```

The merge is **shallow** — only top-level keys are merged. Nested objects are still shared by reference.

### Rest (`...`)

The rest operator collects remaining elements into an array (or object).

**Rest in function parameters:**

```js
function sum(...numbers) {
    let total = 0;

    sum_loop: for(let i = 0; i < numbers.length; ++i)
        total = total + numbers[i];

    return total;
}

WScript.Echo(sum(1, 2, 3));    // 6
WScript.Echo(sum(1, 2, 3, 4)); // 10
```

This replaces the older `arguments` object — `numbers` is a real array, not array-like.

**Rest in destructuring:**

```js
const [first, ...others] = [10, 20, 30, 40];

WScript.Echo(first);  // 10
WScript.Echo(others); // 20,30,40 (a real array)

const { name, ...rest } = { name: "Sam", age: 34, email: "sam@example.com" };

WScript.Echo(name);       // "Sam"
WScript.Echo(rest.age);   // 34
WScript.Echo(rest.email); // "sam@example.com"
```

### Optional chaining (`?.`)

When accessing nested properties, any missing intermediate value normally throws:

```js
const data = { user: null };
WScript.Echo(data.user.name); // TypeError — can't read .name of null
```

The optional chaining operator short-circuits the access if anything in the chain is `null` or `undefined`:

```js
const data = { user: null };
WScript.Echo(data.user?.name); // undefined — no error
```

Works for method calls (`obj.method?.()`) and bracket access (`arr?.[i]`):

```js
const obj = { greet: void null };
obj.greet?.(); // no error, just nothing happens

const arr = null;
WScript.Echo(arr?.[0]); // undefined
```

This replaces a lot of defensive `if(data && data.user && data.user.profile)` chains.

### Nullish coalescing (`??` and `??=`)

The nullish coalescing operator returns its right side **only if** its left side is `null` or `undefined`:

```js
const a = null;
const b = a ?? "default";
WScript.Echo(b); // "default"

const c = 0;
const d = c ?? "default";
WScript.Echo(d); // 0 — zero is not null/undefined
```

Compare to `||`, which uses falsy:

```js
const e = 0 || "default";
WScript.Echo(e); // "default" — zero is falsy

const f = "" || "default";
WScript.Echo(f); // "default" — empty string is falsy
```

Use `??` when you want defaults *only* for "not set" — keeping `0` and `""` as valid values. Use `||` when you genuinely want any falsy value to fall through.

The assignment form `??=` assigns only if the left side is null/undefined:

```js
const config = { theme: "dark" };

config.theme ??= "light"; // already "dark", no change
config.fontSize ??= 14;   // was undefined, now 14

WScript.Echo(config.theme);    // "dark"
WScript.Echo(config.fontSize); // 14
```

### Exercises

1. **Build and inspect.** Create an object representing a book with `title`, `author`, and `year`. Use both dot and bracket notation to read each property. Add a `genre` property after creation.
2. **Iterate cleanly.** Create a `counts` object as a lookup table using `Object.create(null)`. Add three keys with numeric values. Iterate with a labeled `for...in` loop and print each key/value pair using a template literal.
3. **Reference vs copy.** Write a function `addOne(obj)` (with header comment) that adds `1` to a property called `count`. Pass it an object and `Echo` the object's `count` before and after the call. Note the mutation. Now write `addOneCopy(obj)` that creates a *new* object using spread and returns it, leaving the original untouched.
4. **Destructure a config.** Given `const config = { width: 80, height: 24, theme: "dark" };`, write a function `describe({ width, height, theme = "light" })` that prints all three values using a template literal. Test it with the full config and with a partial one (`{ width: 100, height: 30 }`).
5. **Merge objects.** Given `defaults = { tabSize: 4, theme: "light" }` and `userPrefs = { theme: "dark" }`, use spread to produce a `merged` object where user prefs override defaults. Print the result.
6. **Safe access.** Given `const data = { user: { profile: null } };`, use optional chaining to safely read `data.user.profile.name` and `data.account?.balance`. Use `??` to provide a default of `"anonymous"` when the name is missing.
7. **Variadic average.** Write a function `average(...numbers)` (with header comment) that returns the average of any number of arguments, or `0` if no arguments are passed. Test with `average(10, 20, 30)` and `average()`.
8. **Word frequency map.** Given an array of strings, write a function `wordFreq(words)` that returns a lookup-table object (use `Object.create(null)`) mapping each unique word to its count. Test with `["a", "b", "a", "c", "a", "b"]` (expected: `{ a: 3, b: 2, c: 1 }`).

---

## Lesson 3.3: Classes

> **New words in this lesson**
>
> - **class** — a template for creating objects with shared structure and behavior
> - **constructor** — the function that runs when you create a new instance with `new`
> - **instance** — one specific object created from a class
> - **inheritance** — when one class extends another, gaining its properties and methods
> - **`super`** — refers to the parent class
> - **static method** — a method belonging to the class itself, not to instances

A **class** is a template for objects with shared structure and behavior. Where an object literal is one specific thing, a class is a recipe for making things of that kind.

### Class declaration

```js
class Counter {
    constructor() {
        this.count = 0;
    }

    increment() {
        ++this.count;
    }

    get() {
        return this.count;
    }
}

const c = new Counter();
c.increment();
c.increment();
c.increment();
WScript.Echo(c.get()); // 3
```

The pieces:

- **`class Counter { ... }`** declares a class named `Counter`.
- **`constructor()`** runs once when you create a new instance via `new Counter()`. Use it to set up initial state.
- **`increment()`** and **`get()`** are methods, attached to every instance.
- **`new Counter()`** creates a fresh instance with its own `count`.

Each `new Counter()` produces a separate object. They don't share state:

```js
const a = new Counter();
const b = new Counter();
a.increment();
WScript.Echo(a.get()); // 1
WScript.Echo(b.get()); // 0 — separate instance
```

### Constructor parameters

The constructor takes arguments like any other function:

```js
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    introduce() {
        WScript.Echo(`Hi, I'm ${this.name} and I'm ${this.age}.`);
    }
}

const sam = new Person("Sam", 34);
sam.introduce(); // Hi, I'm Sam and I'm 34.
```

### Static methods

Methods marked `static` belong to the class itself, not to instances. Useful for utility functions related to the class:

```js
class Logger {
    static info(message) {
        WScript.Echo(`[INFO] ${message}`);
    }

    static error(message) {
        WScript.Echo(`[ERROR] ${message}`);
    }
}

Logger.info("Script started");  // [INFO] Script started
Logger.error("File not found"); // [ERROR] File not found
```

You can't call static methods on an instance — only on the class itself.

### Inheritance

A class can extend another, inheriting its properties and methods:

```js
class Animal {
    constructor(name) {
        this.name = name;
    }

    speak() {
        WScript.Echo(`${this.name} makes a sound.`);
    }
}

class Dog extends Animal {
    speak() {
        WScript.Echo(`${this.name} barks.`);
    }
}

const rex = new Dog("Rex");
rex.speak(); // Rex barks.
```

`Dog` inherits `Animal`'s constructor and methods, then overrides `speak`. To call the parent's version, use `super`:

```js
class Dog extends Animal {
    speak() {
        super.speak(); // Rex makes a sound.
        WScript.Echo(`${this.name} also barks.`);
    }
}
```

When a child class needs its own constructor, it must call `super(...)` first to run the parent's:

```js
class Cat extends Animal {
    constructor(name, indoor) {
        super(name);
        this.indoor = indoor;
    }
}
```

### Classes are transformed under the hood

In our codebase, the class syntax above gets transpiled to ES3-compatible patterns by baseline. The output uses constructor functions, prototype assignments, and explicit chain construction — verbose, `var`-based, and unrecognizable as a "class" at first glance. You don't normally need to read or write that output, but you should know:

- A `class` you write becomes a constructor function plus prototype assignments after transformation.
- `extends` becomes manual prototype chaining and a `super` shim.
- The transformation isn't trivial. If you ever encounter a bug in a class hierarchy that doesn't make sense in the source, the transformed output is the actual code running.

Write classes in modern syntax and trust the tool. If something behaves weirdly, the journal probably has notes on known transform issues.

### When to use classes

Classes are powerful but not always the right tool. Use them when:

- You have **state** tightly coupled to **behavior** (a counter, a parser, a buffer).
- You'll create **multiple instances** with their own state.
- There's a clear **inheritance** relationship.

For one-off groupings of data with no behavior, prefer plain object literals. For utility functions with no state, prefer plain functions or static methods.

### Exercises

1. **Counter with floor.** Write a class `BoundedCounter` whose constructor takes a `floor` value. It has methods `increment()`, `decrement()`, and `get()`. The counter starts at `floor`, can be incremented freely, but `decrement` never takes it below `floor`. Test with `floor = 0`.
2. **Greet variations.** Write a class `Greeter` whose constructor takes a name. Add methods `formal()`, `casual()`, and `enthusiastic()`, each printing a different style of greeting using `this.name` and template literals.
3. **Static helpers.** Write a class `MathExtra` with three static methods: `clamp(n, min, max)` (returns `n` clamped to the range), `lerp(a, b, t)` (returns `a + (b - a) * t`, linear interpolation), and `randInt(min, max)` (uses your `randomInt` formula from Tier 2). Test each.
4. **Inherit and override.** Write a base class `Shape` with a constructor taking `name` and an `area()` method that throws a "not implemented" error. Write `Rectangle extends Shape` and `Circle extends Shape`, each implementing `area()` correctly. Test with a rectangle (4 × 3) and a circle (radius 5).

---

## Lesson 3.4: Date and time

> **New words in this lesson**
>
> - **`Date` object** — JScript's built-in type for representing a moment in time
> - **timestamp** — a date represented as a single number (milliseconds since 1970)
> - **epoch** — the fixed reference point dates are measured from (Jan 1, 1970, UTC)
> - **UTC** — Coordinated Universal Time, a global timezone-independent reference
> - **locale** — region-specific formatting conventions (don't trust them)
> - **ISO 8601** — an international date-format standard (`YYYY-MM-DD`)
> - **overflow** — when a date component goes past its normal range and wraps (e.g., month `13` becomes January of the next year)

JScript has a `Date` object for working with timestamps. It's the same `Date` you'd see in any JavaScript environment — but with one big caveat: WSH JScript's date *formatting* methods are locale-dependent and unreliable, so we usually format manually.

### Creating a Date

```js
const now      = new Date();                    // current date and time
const epoch    = new Date(0);                   // Unix epoch (Jan 1, 1970, UTC)
const specific = new Date(2024, 5, 15, 14, 30); // Year, month (0-based!), day, hour, minute
```

**Critical gotcha:** the `month` argument is **zero-indexed**. January is `0`, December is `11`. The `day` argument is one-indexed. So `new Date(2024, 5, 15)` is **June 15**, not May.

You can also parse from a string, but parsing is wildly inconsistent across JavaScript engines. Avoid it; construct dates explicitly when you can.

### Reading components

```js
const d = new Date(2024, 5, 15, 14, 30, 45);

WScript.Echo(d.getFullYear()); // 2024
WScript.Echo(d.getMonth());    // 5 (June, zero-indexed)
WScript.Echo(d.getDate());     // 15 (day of month)
WScript.Echo(d.getDay());      // 6 (day of week — Saturday, where 0 = Sunday)
WScript.Echo(d.getHours());    // 14
WScript.Echo(d.getMinutes());  // 30
WScript.Echo(d.getSeconds());  // 45
```

`getMonth()` returns `0`–`11`. `getDay()` returns `0`–`6` (with `0` = Sunday). Both are common sources of confusion.

There are UTC variants — `getUTCFullYear`, `getUTCMonth`, etc. — that return components in UTC instead of local time.

### Timestamps

`d.getTime()` returns the date as a number — milliseconds since the Unix epoch:

```js
const d = new Date();
WScript.Echo(d.getTime()); // e.g., 1734345600000
```

Useful for:

- **Comparing dates.** `dateA.getTime() < dateB.getTime()` reliably tells you which came first.
- **Date math.** Subtract two timestamps to get the difference in milliseconds.
- **Storage.** A timestamp is a single number; easy to log or persist.

```js
const a = new Date(2024, 0, 1);
const b = new Date(2024, 0, 15);

const diffMs = b.getTime() - a.getTime();
const diffDays = diffMs / (1000 * 60 * 60 * 24);

WScript.Echo(`${diffDays} days apart.`); // 14 days apart.
```

### Don't trust `.toString()`, `.toLocaleString()`, etc.

The Date object has built-in formatting methods:

```js
const d = new Date();
WScript.Echo(d.toString());       // varies wildly by environment
WScript.Echo(d.toLocaleString()); // depends on locale settings
```

In WSH JScript these are unreliable. The output format depends on regional settings, the Windows version, and sometimes seemingly the phase of the moon. **Don't use them** for any output that needs to be parseable or consistent.

### Manual formatting — the standard pattern

Build the format string yourself from components:

```js
/*
 * formatDate
 * Formats a Date as YYYY-MM-DD HH:MM:SS in local time.
 *
 * Input:  d (Date)
 * Output: string in YYYY-MM-DD HH:MM:SS format
 */
function formatDate(d) {
    const pad = (n) => (n < 10 ? `0${n}` : `${n}`);

    const yyyy = d.getFullYear();
    const mm   = pad(d.getMonth() + 1); // +1 to undo zero-indexing
    const dd   = pad(d.getDate());
    const hh   = pad(d.getHours());
    const mi   = pad(d.getMinutes());
    const ss   = pad(d.getSeconds());

    return `${yyyy}-${mm}-${dd} ${hh}:${mi}:${ss}`;
}

WScript.Echo(formatDate(new Date())); // 2025-04-12 14:30:45
```

The `pad` helper turns `9` into `"09"`. The `+1` on the month corrects for zero-indexing.

This — or some variant — is the reliable way to format a date in our codebase. ISO 8601 (`YYYY-MM-DD`) is the safe default since it sorts lexically as well as chronologically and parses correctly in any environment.

### Modifying dates

Set components individually:

```js
const d = new Date();
d.setFullYear(2030);
d.setMonth(0); // January
d.setDate(1);
```

Date components also overflow — setting month to `13` advances the year:

```js
const d = new Date(2024, 11, 1); // Dec 1, 2024
d.setMonth(d.getMonth() + 1);    // overflows to Jan 1, 2025
WScript.Echo(formatDate(d));
```

This is occasionally what you want. More often it's a bug. Be deliberate.

### Common date patterns

Yesterday:

```js
const yesterday = new Date();
yesterday.setDate(yesterday.getDate() - 1);
```

N days ago:

```js
function nDaysAgo(n) {
    const d = new Date();
    d.setDate(d.getDate() - n);

    return d;
}
```

Difference in days:

```js
function daysBetween(a, b) {
    const diffMs = Math.abs(b.getTime() - a.getTime());

    return Math.floor(diffMs / (1000 * 60 * 60 * 24));
}
```

### Exercises

1. **Today's stamp.** Write a script that prints today's date as `YYYY-MM-DD` using the manual formatting pattern.
2. **Format helper.** Implement `formatDate(d)` exactly as shown above (with header comment), then use it to print the current time three times in a row separated by `WScript.Sleep(1000)` calls. (`WScript.Sleep` pauses for the given milliseconds — handy for testing date logic.)
3. **Days until.** Write a function `daysUntil(year, month, day)` (with header comment, and remember the month is zero-indexed in `Date`) that returns the number of days from today until that date. Test with a date in the future and a date in the past (negative result expected for past dates).
4. **Yesterday's log.** Write a script that opens (or creates) `C:\\daily.log` and appends a single line: `[<timestamp>] running on <username>`, using `formatDate` and `WScript.Shell` for the username.

---

## Lesson 3.5: Working with files — FileSystemObject

> **New words in this lesson**
>
> - **COM** — Component Object Model; Windows' way of letting programs talk to each other
> - **`ActiveXObject`** — JScript's way of creating COM components
> - **progID** — the program identifier string for a COM component (like `"Scripting.FileSystemObject"`)
> - **`FileSystemObject` (FSO)** — the COM component for working with files and folders
> - **stream** — a flowing sequence of data; used here for reading or writing a file
> - **iomode** — a number telling the file system whether to read, write, or append (1 / 2 / 8)
> - **`Enumerator`** — JScript's adapter for iterating over a COM collection (which isn't a regular array)
> - **path operations** — utilities for splitting and joining file paths

A lot of our utility scripts read or write files: log filters, report generators, file renamers, data ingesters. JScript exposes file system access through a COM object called the **FileSystemObject** (FSO).

### The ActiveXObject pattern

JScript doesn't have built-in file APIs. Instead, it talks to COM components installed on the machine. The pattern:

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");
```

`new ActiveXObject(progID)` creates an instance of a registered COM component. `"Scripting.FileSystemObject"` is the program ID for the file system object — that string is fixed and case-insensitive.

Once you have the instance, you call methods on it like any other object:

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");

if(fso.FileExists("C:\\Reports\\summary.txt"))
    WScript.Echo("Found it.");
```

You'll meet several COM components by name throughout this tier — `Scripting.FileSystemObject`, `WScript.Shell`, `WScript.Network`, etc. The `ActiveXObject` pattern is the same for all of them.

### Reading a text file

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");
const stream = fso.OpenTextFile("C:\\notes.txt", 1); // 1 = ForReading

const text = stream.ReadAll();
stream.Close();

WScript.Echo(text);
```

`OpenTextFile(path, iomode)` returns a stream object. The `iomode` constants:

- `1` — ForReading
- `2` — ForWriting (creates a new file or overwrites existing)
- `8` — ForAppending

The stream methods you'll use most:

- `ReadAll()` — reads the entire file as a single string.
- `ReadLine()` — reads one line.
- `AtEndOfStream` — boolean property, `true` when there's no more to read.
- `Close()` — closes the stream. **Always close streams** when you're done.

For line-by-line reading:

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");
const stream = fso.OpenTextFile("C:\\big.log", 1);

read_lines: while(!stream.AtEndOfStream) {
    const line = stream.ReadLine();
    WScript.Echo(line);
}

stream.Close();
```

This is the right pattern for large files where loading the whole thing into memory is wasteful.

### Writing a text file

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");
const stream = fso.OpenTextFile("C:\\out.txt", 2, true); // 2 = ForWriting, true = create if missing

stream.WriteLine("Hello,");
stream.WriteLine("world!");
stream.Close();
```

The third argument is a `create` flag. Passing `true` means "create the file if it doesn't exist." Passing `false` (the default) makes the call fail if the file is missing.

`Write(text)` writes without a line break. `WriteLine(text)` writes a line followed by `\r\n` (Windows-standard line ending).

There's also `CreateTextFile(path, overwrite)` as a shortcut for "create new file for writing":

```js
const stream = fso.CreateTextFile("C:\\out.txt", true); // true = overwrite if exists
stream.WriteLine("...");
stream.Close();
```

### Appending

Append by opening with iomode `8`:

```js
const stream = fso.OpenTextFile("C:\\app.log", 8, true);
stream.WriteLine(`[${new Date()}] Script ran`);
stream.Close();
```

The `true` create-if-missing flag ensures the first run doesn't fail.

### Always close streams (and prefer try/finally)

If your script throws an error before closing a stream, the file may be left in an unflushed or locked state. The defensive pattern uses `try`/`finally`:

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");
const stream = fso.OpenTextFile("C:\\important.txt", 2, true);

try {
    stream.WriteLine("critical data");
    doSomethingThatMightThrow();
    stream.WriteLine("more data");
} finally {
    stream.Close();
}
```

`finally` runs whether or not the `try` block threw, ensuring the stream is closed either way. Use this pattern any time you have a stream open across non-trivial work.

### Existence checks

```js
fso.FileExists("C:\\notes.txt") // boolean
fso.FolderExists("C:\\Reports") // boolean
```

Both check existence on disk and return `true`/`false` without throwing.

### Listing directory contents

A folder's files and subfolders are accessed through its `Folder` object, but the iteration is unusual:

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");
const folder = fso.GetFolder("C:\\Reports");

const files = new Enumerator(folder.Files);

list_files: for(; !files.atEnd(); files.moveNext())
    WScript.Echo(files.item().Name);
```

That's a different iteration pattern — `Enumerator` is JScript's adapter for COM collections (which aren't regular arrays). The pattern:

- `new Enumerator(collection)` wraps it.
- `.atEnd()` returns `true` when iteration is complete.
- `.moveNext()` advances to the next item.
- `.item()` returns the current item.

The same pattern works for `folder.SubFolders`:

```js
const subs = new Enumerator(folder.SubFolders);

list_subs: for(; !subs.atEnd(); subs.moveNext())
    WScript.Echo(subs.item().Path);
```

Each `Files` item is a `File` object with properties like `.Name`, `.Path`, `.Size`, `.DateLastModified`. Each `SubFolders` item is a `Folder` object.

### `Array.enumerate` — the baseline shortcut

Our `baseline` package provides a helper that wraps the `Enumerator` pattern and behaves like the modern `Array.from`:

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");
const folder = fso.GetFolder("C:\\Reports");

const fileNames = Array.enumerate(folder.Files, (file) => file.Name);

list_files: for(let i = 0; i < fileNames.length; ++i)
    WScript.Echo(fileNames[i]);
```

`Array.enumerate(enumerable, mapper, self)`:

- `enumerable` — a COM collection (or anything with the iteration interface).
- `mapper` *(optional)* — a function applied to each item before it's added to the array.
- `self` *(optional)* — value to use as `this` in the mapper.

Mirrors the standard `Array.from(iterable, mapper, self)` API but works on COM collections. **Reach for this first** in our codebase — the raw `Enumerator` pattern above is mostly for understanding what's happening underneath, and for situations where you want to stream rather than materialize the whole collection.

Without a mapper, you get the items themselves:

```js
const allFiles = Array.enumerate(folder.Files); // array of File objects

for_each_file: for(let i = 0; i < allFiles.length; ++i)
    WScript.Echo(allFiles[i].Name);
```

### Path operations

FSO provides path helpers — useful because Windows paths get awkward with manual string manipulation:

```js
const fso = new ActiveXObject("Scripting.FileSystemObject");

WScript.Echo(fso.BuildPath("C:\\Reports", "summary.txt"));      // C:\Reports\summary.txt
WScript.Echo(fso.GetParentFolderName("C:\\Reports\\sum.txt"));  // C:\Reports
WScript.Echo(fso.GetFileName("C:\\Reports\\sum.txt"));          // sum.txt
WScript.Echo(fso.GetBaseName("C:\\Reports\\sum.txt"));          // sum
WScript.Echo(fso.GetExtensionName("C:\\Reports\\sum.txt"));     // txt
```

Use these instead of hand-rolled `.indexOf("\\")` math wherever you can — they handle edge cases (trailing slashes, missing extensions, etc.) for free.

### Other useful operations

Brief tour of operations you'll occasionally need:

- `fso.CopyFile(source, dest)` — copies a file.
- `fso.MoveFile(source, dest)` — moves a file (also used to rename).
- `fso.DeleteFile(path)` — **destructive**, removes the file. Verify first.
- `fso.CreateFolder(path)` — creates a directory.
- `fso.CopyFolder` / `fso.MoveFolder` / `fso.DeleteFolder` — folder versions.

For destructive operations (`DeleteFile`, `DeleteFolder`), get into the habit of checking existence first and printing what you're about to do before doing it. Easy mistake to delete the wrong path; hard to undo.

### Exercises

1. **Read and echo.** Write `cat.js` that takes one filename as a CLI argument and prints its contents. Use `try`/`catch` to print a friendly error if the file doesn't exist or can't be opened.
2. **Line counter.** Write `wc.js` that takes one filename and prints the number of lines in the file. Read line-by-line — don't use `ReadAll` (so it works for large files).
3. **Append a note.** Write `note.js` that takes any number of CLI arguments, joins them with spaces, and appends them as a single line to `C:\\notes.log` along with a timestamp (use `new Date()`). Create the file if it doesn't exist.
4. **List directory.** Write `ls.js` that takes a folder path and prints each file in the folder (just the name, not the full path), one per line. Use `Array.enumerate` with a mapper for cleanliness, then optionally rewrite using the raw `Enumerator` pattern to feel the difference.
5. **Safe rename.** Write a function `safeMove(fso, source, dest)` (with header comment) that moves `source` to `dest`, but only if `dest` does **not** already exist. Throw an `Error` if `dest` exists. Test it interactively.

---

## Lesson 3.6: Running commands and reading the environment

> **New words in this lesson**
>
> - **Shell** — `WScript.Shell`, the COM object for running commands and reading the environment
> - **environment variable** — a named value the OS makes available (like `USERNAME` or `PATH`)
> - **`Run`** — fire off a command, optionally waiting for it to finish
> - **`Exec`** — run a command and capture its output
> - **stdout** — standard output, where a program writes its normal output
> - **stderr** — standard error, where a program writes error messages
> - **exit code** — a number a program returns when it ends; `0` means success
> - **redirection** — sending output into a file with `>` or piping it with `|`
> - **special folder** — a well-known system folder (Desktop, Documents, etc.)

For invoking other programs, reading environment variables, and accessing system folders, JScript uses the **Shell** COM object — `WScript.Shell`.

### Creating a Shell object

```js
const shell = new ActiveXObject("WScript.Shell");
```

Same `ActiveXObject` pattern as FSO. The `progID` is `"WScript.Shell"`.

### `Run` — fire and (optionally) wait

`Run(command, windowStyle, waitOnReturn)` executes a command:

```js
const shell = new ActiveXObject("WScript.Shell");

shell.Run("notepad.exe", WINDOW.ACTIVATE, false);
```

The arguments:

- **`command`** — the command line to run, including any arguments. Quote paths with spaces.
- **`windowStyle`** — controls how the launched window appears. Use the `WINDOW` enum from our shared globals (table below). Most scripts pass `WINDOW.ACTIVATE`; for headless work, `WINDOW.HIDE`.
- **`waitOnReturn`** — `true` blocks until the command finishes; `false` returns immediately.

#### The `WINDOW` enum

Window style values are held in a global `WINDOW` enum:

| Name | Value | Meaning |
|------|------:|---------|
| `WINDOW.HIDE` | 0 | Hide the window |
| `WINDOW.ACTIVATE` | 1 | Activate and display *(most common)* |
| `WINDOW.MINIMIZE` | 2 | Show minimized |
| `WINDOW.MAXIMIZE` | 3 | Show maximized |
| `WINDOW.RESTORE_AND_SKIP` | 4 | Show without changing focus |
| `WINDOW.RESTORE` | 5 | Show in current size and position |
| `WINDOW.MINIMIZE_AND_SKIP` | 6 | Minimize and don't activate |
| `WINDOW.MINIMIZE_AND_KEEP` | 7 | Show minimized; keep focus elsewhere |
| `WINDOW.DISPLAY` | 8 | Show in current state |
| `WINDOW.ACTIVATE_FROM_MINIMIZED` | 9 | Restore and activate |
| `WINDOW.INHERIT` | 10 | Use parent's setting |

You'll see legacy code using bare numbers (`shell.Run("...", 1, false)`); prefer the named constants in new code.

When `waitOnReturn` is `true`, `Run` returns the command's exit code:

```js
const code = shell.Run(`cmd /c "dir C:\\Reports > C:\\report.txt"`, WINDOW.HIDE, true);
WScript.Echo(`Exit code: ${code}`);
```

Use the leading `cmd /c` when you need shell features like redirection (`>`, `|`) — `Run` itself doesn't interpret those.

### `Exec` — capture output

`Run` doesn't give you the command's output. For that, use `Exec`:

```js
const shell = new ActiveXObject("WScript.Shell");
const process = shell.Exec("ipconfig");

while(!process.StdOut.AtEndOfStream)
    WScript.Echo(process.StdOut.ReadLine());
```

`Exec(command)` returns a process object. The relevant properties:

- **`process.StdOut`** — text stream of standard output.
- **`process.StdErr`** — text stream of standard error.
- **`process.ExitCode`** — the exit code (only valid after the process completes).
- **`process.Status`** — `0` while running, `1` when complete.

For a simple capture-everything pattern:

```js
const process = shell.Exec("git status");
const output = process.StdOut.ReadAll();

WScript.Echo(output);
```

`ReadAll()` blocks until the process is done. For long-running processes whose output you want to stream, use `ReadLine` in a loop as in the first example.

### Environment variables

Read environment variables via `Environment`:

```js
const shell = new ActiveXObject("WScript.Shell");
const env = shell.Environment("Process");

WScript.Echo(env("USERNAME"));
WScript.Echo(env("COMPUTERNAME"));
WScript.Echo(env("PATH"));
```

The `"Process"` argument selects the current process's environment (the one your script inherited). Other valid values: `"User"`, `"System"`, `"Volatile"`. For most scripts, `"Process"` is what you want.

There's also `ExpandEnvironmentStrings` for the common case of expanding `%VARS%` inside a path:

```js
const path = shell.ExpandEnvironmentStrings("%USERPROFILE%\\Documents");
WScript.Echo(path); // C:\Users\Sam\Documents
```

Easier than reading `USERPROFILE` and concatenating.

### Special folders

```js
const shell = new ActiveXObject("WScript.Shell");

WScript.Echo(shell.SpecialFolders("Desktop"));
WScript.Echo(shell.SpecialFolders("MyDocuments"));
WScript.Echo(shell.SpecialFolders("Programs"));
WScript.Echo(shell.SpecialFolders("StartMenu"));
```

Returns the path. Useful when your script needs to drop a file in a well-known location regardless of which user is running it.

### Current directory

```js
const shell = new ActiveXObject("WScript.Shell");

WScript.Echo(shell.CurrentDirectory);

shell.CurrentDirectory = "C:\\Reports";
```

Read or write. Note: this sets the *script's* current directory, not the parent shell's. If you change directory and your script ends, the parent `cmd` is unaffected.

### Practical pattern: running a command with error handling

```js
/*
 * runCmd
 * Runs a command, captures stdout, and throws on non-zero exit.
 *
 * Input:  cmdline (string)
 * Output: stdout content (string), or throws Error on failure
 */
function runCmd(cmdline) {
    const shell = new ActiveXObject("WScript.Shell");
    const proc = shell.Exec(cmdline);

    const out = proc.StdOut.ReadAll();
    const err = proc.StdErr.ReadAll();

    if(proc.ExitCode !== 0)
        throw new Error(`Command failed (exit ${proc.ExitCode}): ${err.trim()}`);

    return out;
}

try {
    const result = runCmd("ipconfig /all");
    WScript.Echo(result);
} catch(e) {
    WScript.Echo(`Error: ${e.message}`);
}
```

A good template for shelling out from a script — gives you the output on success, or a meaningful error on failure.

### Exercises

1. **Echo username.** Write a script that prints `Hello, <USERNAME>!` using the `Environment` collection.
2. **Run and report.** Write a script that runs `ipconfig` (or any harmless command) using `Run` with `waitOnReturn = true`, and prints the exit code.
3. **Capture and count.** Write a script that runs `dir C:\\` using `Exec`, captures the output, and prints the number of lines it produced.
4. **Special folder.** Write a script that uses `SpecialFolders` to print the path to the user's Desktop, then uses `FileSystemObject` to list the files there.
5. **Wrap a command.** Implement `runCmd(cmdline)` exactly as shown above. Test by calling it with a valid command (`echo hello`) and an invalid one (a bogus path).

---

## Lesson 3.7: Regular expressions

> **New words in this lesson**
>
> - **regular expression / regex** — a pattern that describes a set of strings
> - **pattern** — the rules a regex describes
> - **match** — a piece of text that fits a pattern
> - **character class** — a set of characters (like `\d` for any digit)
> - **quantifier** — how many times something must repeat (`+`, `*`, `?`, `{n}`)
> - **anchor** — a position match (`^` for start of string, `$` for end)
> - **group** — parenthesized subpattern
> - **capture group** — a group whose match can be referenced later
> - **alternation** — "OR" written as `|`
> - **flag** — modifier letter after the closing slash (`i`, `g`, `m`)
> - **greedy / lazy** — match as much / as little as possible
> - **non-capturing group** — `(?:...)`, groups without a capture slot

A **regular expression** (regex) is a pattern that describes a set of strings. Use them when you need to match, find, validate, or replace text based on shape rather than exact content.

If you've avoided regex up to now, you'll want to learn at least the basics. They're heavily used in our codebase — almost any text processing leans on them.

### Creating a regex

Two forms.

**Literal syntax** — pattern between forward slashes:

```js
const r1 = /hello/;
const r2 = /hello/i; // with flag
```

**Constructor syntax** — useful when the pattern is built dynamically:

```js
const word = "hello";
const r3 = new RegExp(word);
const r4 = new RegExp(word, "i");
```

Use the literal form by default. Reach for `RegExp()` only when you need to build the pattern from variables.

### Testing for a match

`.test(str)` returns `true` or `false`:

```js
const looksLikeEmail = /@/;

WScript.Echo(looksLikeEmail.test("sam@example.com")); // true
WScript.Echo(looksLikeEmail.test("not an email"));    // false
```

### The pattern language

The basics of what goes between the slashes.

#### Literal characters

Most characters match themselves. `/cat/` matches the substring `"cat"` anywhere in the input.

#### Character classes

| Pattern | Matches |
|---------|---------|
| `.` | Any single character (except newline by default) |
| `\d` | A digit (`0`–`9`) |
| `\D` | Not a digit |
| `\w` | A "word" character: letter, digit, or `_` |
| `\W` | Not a word character |
| `\s` | Whitespace (space, tab, newline, etc.) |
| `\S` | Not whitespace |
| `[abc]` | Any one of `a`, `b`, or `c` |
| `[a-z]` | Any one lowercase letter |
| `[^abc]` | Any single char *not* `a`, `b`, or `c` |

```js
WScript.Echo(/\d\d\d/.test("abc123"));  // true (3 digits)
WScript.Echo(/[A-Z]/.test("hello"));     // false (no uppercase)
WScript.Echo(/[A-Z]/.test("Hello"));     // true
WScript.Echo(/[^aeiou]/.test("aeiou"));  // false (all are vowels)
```

#### Quantifiers

| Pattern | Meaning |
|---------|---------|
| `*` | Zero or more of the preceding |
| `+` | One or more |
| `?` | Zero or one |
| `{n}` | Exactly `n` |
| `{n,m}` | Between `n` and `m` |
| `{n,}` | `n` or more |

```js
/a*/.test("");                  // true   (zero a's)
/a+/.test("");                  // false  (need at least one)
/\d{3}-\d{4}/.test("555-1234"); // true
```

Add `?` after a quantifier to make it lazy (match as little as possible):

```js
/a.+a/.exec("aXXXXaYYYa");  // matches "aXXXXaYYYa" (greedy, all the way to last 'a')
/a.+?a/.exec("aXXXXaYYYa"); // matches "aXXXXa"     (lazy, stops at first)
```

#### Anchors

| Pattern | Meaning |
|---------|---------|
| `^` | Start of string |
| `$` | End of string |
| `\b` | Word boundary |

```js
/^hello/.test("hello world");  // true
/^hello/.test("oh hello");     // false (doesn't start with hello)
/world$/.test("hello world");  // true
/\bcat\b/.test("the cat sat"); // true
/\bcat\b/.test("category");    // false (cat is part of a longer word)
```

#### Groups and alternation

Parentheses group subpatterns (and create capture groups — more in a moment):

```js
/(ab)+/.test("ababab"); // true
```

The pipe `|` is OR:

```js
/cat|dog/.test("a cat sat");   // true
/cat|dog/.test("a fish swam"); // false
```

#### Escaping special characters

To match a literal special character, prefix it with `\`:

```js
/\./.test("3.14");   // true (literal period)
/\?/.test("hello?"); // true (literal question mark)
```

Special characters that need escaping when used literally: `. * + ? ^ $ ( ) [ ] { } | \`. When in doubt, escape.

### Flags

The flag characters after the closing slash modify regex behavior.

| Flag | Meaning |
|------|---------|
| `i` | Case-insensitive |
| `g` | Global — find all matches, not just the first |
| `m` | Multi-line — `^` and `$` match at line breaks too |

```js
/hello/.test("HELLO");  // false
/hello/i.test("HELLO"); // true

"a a a".replace(/a/, "b");  // "b a a"  (only first)
"a a a".replace(/a/g, "b"); // "b b b"  (all)
```

### `.exec` — get the match

`regex.exec(str)` returns a match array (or `null` if no match):

```js
const r = /(\d+)-(\d+)/;
const m = r.exec("Code 123-456 found");

WScript.Echo(m[0]); // "123-456"  (full match)
WScript.Echo(m[1]); // "123"      (first group)
WScript.Echo(m[2]); // "456"      (second group)
```

The first element is the entire match. Subsequent elements are capture groups (parenthesized subpatterns).

### String methods that take regex

You've already met `.replace` with a string argument. With a regex:

```js
"hello world".replace(/o/g, "0"); // "hell0 w0rld"
```

Capture groups can be referenced in the replacement string with `$1`, `$2`, etc.:

```js
"Sam Jones".replace(/(\w+) (\w+)/, "$2, $1"); // "Jones, Sam"
```

`.match` returns the match array(s):

```js
"a1 b2 c3".match(/\w\d/g); // ["a1", "b2", "c3"]
```

`.split` accepts a regex separator:

```js
"a, b,c , d".split(/\s*,\s*/); // ["a", "b", "c", "d"]
```

That last one — split on commas with optional whitespace on either side — is far more useful than `.split(",")`.

### Common patterns

Things you'll find yourself writing:

```js
// non-empty whitespace
/\S+/

// trim leading/trailing whitespace (one match each side)
/^\s+|\s+$/g

// a JS-style identifier
/^[a-zA-Z_$][\w$]*$/

// a hex number with optional prefix
/^(?:0x)?[0-9a-fA-F]+$/

// a decimal number, possibly signed and with a decimal point
/^-?\d+(?:\.\d+)?$/

// CRLF or LF line ending
/\r?\n/
```

`(?:...)` is a **non-capturing group** — same grouping behavior as `(...)`, but doesn't create a capture slot. Use it when you need grouping without the capture overhead.

### Pitfalls and performance

A few things to know:

- **Construction cost.** A literal regex is parsed once. `new RegExp(...)` runs every time the constructor is called. If you build the same regex repeatedly inside a loop, lift it out:
  ```js
  const pattern = new RegExp(somethingDynamic, "g");

  scan: for(let i = 0; i < lines.length; ++i) {
      // use pattern, not new RegExp every iteration
  }
  ```
- **Greedy by default.** `/.+/` will eat as much as it can. If you want minimal matching, add `?` after the quantifier.
- **Stateful global regex.** A regex with the `g` flag, used with `.exec`, remembers its position between calls (`lastIndex`). Useful for iteration; surprising if you don't expect it.

### Exercises

1. **Email lite.** Write a regex `emailish` that matches strings of the form `<word>@<word>.<word>` (a very loose definition of "email"). Test with `"sam@example.com"`, `"hello"`, and `"a@b"`.
2. **Phone number.** Write a function `isPhone(s)` (with header comment) that returns `true` if `s` looks like a phone number in the form `XXX-XXX-XXXX` (10 digits with dashes in fixed positions). Test with `"555-1234567"` (false), `"555-123-4567"` (true), `"123-456-789"` (false).
3. **Extract numbers.** Given the string `"item-3, item-17, item-2"`, use `.match` with a global regex to produce an array of just the numbers as strings: `["3", "17", "2"]`.
4. **Trim whitespace.** Write a function `myTrim(s)` (with header comment) that returns `s` with leading and trailing whitespace removed. Use a single `.replace` with a regex.
5. **Swap names.** Given `"Doe, John"`, use `.replace` with a regex and `$1`/`$2` to produce `"John Doe"`. Generalize it into a function `flipName(s)`.
6. **Identifier check.** Write a function `isIdentifier(s)` that returns `true` if `s` is a valid JavaScript identifier (letters, digits, `_`, `$`, not starting with a digit).

---

## Lesson 3.8: Modules — splitting code across files

> **New words in this lesson**
>
> - **module** — a separate file with code that can be imported elsewhere
> - **`export`** — make something available from a file
> - **`import`** — pull something in from another file
> - **named export** — exported by name; imported with curly braces
> - **default export** — the file's "main" export; imported without curly braces
> - **namespace import** — `import * as X` to grab everything as one object
> - **re-export** — exposing something from another module without using it locally
> - **barrel file** — a file (often `index.js`) that re-exports content from sibling files
> - **circular import** — when two files import from each other (a problem to avoid)

As scripts grow past a few hundred lines, keeping everything in one file gets painful. **Modules** let you split code into separate files and import what you need.

In modern JavaScript — and in our baseline-flavored source — the syntax is `import`/`export`. Baseline transforms these into the right WSH-compatible mechanism behind the scenes.

### Named exports

To make something available from a file, prefix it with `export`:

```js
// file: helpers.js

export const MAX_RETRIES = 3;

export function pad(n, width) {
    let s = "" + n;

    pad_loop: while(s.length < width)
        s = "0" + s;

    return s;
}

export class Counter {
    constructor() { this.n = 0; }
    increment() { ++this.n; }
    get() { return this.n; }
}
```

Anything declared with `export` is part of the module's public interface.

### Named imports

In another file, pull in what you need by name:

```js
// file: main.js

import { MAX_RETRIES, pad, Counter } from "./helpers.js";

WScript.Echo(MAX_RETRIES); // 3
WScript.Echo(pad(7, 4));   // "0007"

const c = new Counter();
c.increment();
WScript.Echo(c.get()); // 1
```

The imported names must match what the module exports exactly (case-sensitive). The path is relative to the importing file. The `./` prefix means "in the same directory as me." `../helpers.js` means "one directory up." Always include the `.js` extension.

### Renaming on import

If two modules export the same name (or you just want a clearer local name):

```js
import { pad as zeroPad } from "./helpers.js";

WScript.Echo(zeroPad(5, 3)); // "005"
```

### Default exports

A module can have one **default** export — its "main" thing:

```js
// file: logger.js

export default function log(message) {
    WScript.Echo(`[${new Date().getTime()}] ${message}`);
}
```

Imported without curly braces, and you can name it anything:

```js
import log from "./logger.js";

log("Starting up...");
```

Default and named exports can coexist in the same file:

```js
// file: config.js

export const DEFAULT_TIMEOUT = 5000;

export default {
    timeout: DEFAULT_TIMEOUT,
    retries: 3,
    verbose: false
};
```

```js
import settings, { DEFAULT_TIMEOUT } from "./config.js";

WScript.Echo(settings.retries); // 3
WScript.Echo(DEFAULT_TIMEOUT);  // 5000
```

### Namespace imports

To grab everything as one object:

```js
import * as helpers from "./helpers.js";

WScript.Echo(helpers.pad(5, 3));   // "005"
WScript.Echo(helpers.MAX_RETRIES); // 3
```

Useful when you need many things from a module and want them grouped under a name.

### Re-exports

A module can re-export from another module without using the value itself:

```js
// file: index.js

export { pad, MAX_RETRIES } from "./helpers.js";
export { default as log } from "./logger.js";
```

This is the pattern for **barrel** files that consolidate a directory's public surface into one import target.

### What baseline does with modules

Under the hood, baseline transforms `import`/`export` into something WSH can run. The exact mechanism is an implementation detail; you generally don't need to think about it. Two practical consequences:

- The transformed output is **one combined script** suitable for a single `cscript` invocation. There's no separate module loader at runtime.
- **Circular imports** can produce surprising results. If `A` imports from `B` and `B` imports from `A`, the resolution order matters. The codebase avoids them; if you find one, restructure.

### Organizing a project

Common patterns you'll see:

- **One concept per file.** A `Counter` class lives in `counter.js`. A regex utility lives in `regex.js`. The file name matches what's exported.
- **A central `globals.js`** for shared constants — the `WINDOW` enum from Lesson 3.6 lives in something like this.
- **A barrel `index.js`** at the top of a folder, re-exporting the public stuff so consumers can write `import { X } from "./util/"` instead of remembering which sub-file `X` lives in.

Rule of thumb: if a function is used in only one place, leave it in that file. The moment a second file needs it, lift it out into a module.

### Legacy alternative — `.wsf` files

Pre-baseline WSH supports a different mechanism for combining scripts: the `.wsf` (Windows Script File) wrapper, which can include multiple `.js` files via `<script src="...">` tags inside an XML wrapper. You'll occasionally see these in older parts of the codebase.

For new work, use baseline modules. If you encounter a `.wsf` and need to extend it, the patterns are similar to HTML's old script tags — recognize them but don't reach for them.

### Exercises

1. **Split a script.** Take any script you wrote in this tier (the text-stats capstone is a good candidate). Move helper functions into a separate file (e.g., `textHelpers.js`) and import them back into the main script. Confirm identical output.
2. **Default vs named.** Create `mathExtra.js` with a default export `clamp(n, min, max)` and named exports `lerp(a, b, t)` and `randInt(min, max)`. In a main script, import all three using the appropriate syntaxes.
3. **Barrel.** Create `util/string.js` (with one or two string helpers), `util/array.js` (with one or two array helpers), and `util/index.js` that re-exports both. In a main script, import everything from `"./util/"`.
4. **Renamed import.** Create a module that exports a function named `parse`. In two separate consumer scripts, import it once as `parse` and once as `parseLine`. Confirm both work.

---

## Lesson 3.9: Argument parsing patterns

> **New words in this lesson**
>
> - **named argument** — a CLI argument written as `/key:value`
> - **positional argument** — an argument identified by where it appears, not by name
> - **flag** — a boolean argument that's just present or absent
> - **parsed-args object** — an object that holds the parsed command-line values
> - **validation** — checking that input is correct before using it

In Lesson 2.7 you met `WScript.Arguments` for reading positional command-line arguments. Real utility scripts often want **named** arguments too — switches like `/input:foo.txt` or flags like `/verbose`. WSH supports both, and they're accessed through separate collections.

### The two collections

`WScript.Arguments` is split into:

- **`WScript.Arguments.Named`** — arguments that started with `/`, like `/input:foo.txt` or `/verbose`.
- **`WScript.Arguments.Unnamed`** — positional arguments without a `/` prefix.

Run this:

```
cscript myScript.js /input:data.txt /verbose first second
```

And the script sees:

- `Named.Item("input")` → `"data.txt"`
- `Named.Item("verbose")` → `""` (just present, no value)
- `Unnamed.Item(0)` → `"first"`
- `Unnamed.Item(1)` → `"second"`

### Reading named arguments

```js
const args = WScript.Arguments.Named;

if(args.Exists("input"))
    WScript.Echo(`Input file: ${args.Item("input")}`);

if(args.Exists("verbose"))
    WScript.Echo("Verbose mode on.");
```

`.Exists(name)` returns `true` if the argument was passed, regardless of whether it had a value.

`.Item(name)` returns the value (a string), or fails if the argument wasn't passed. **Always check `.Exists` first** unless you're certain the argument is present.

### Reading positional arguments

`Unnamed` works just like the basic `Arguments` collection from Lesson 2.7:

```js
const positional = WScript.Arguments.Unnamed;

WScript.Echo(`Got ${positional.length} positional argument(s)`);

read_pos: for(let i = 0; i < positional.length; ++i)
    WScript.Echo(`[${i}] ${positional.Item(i)}`);
```

### Building a parsed-args object

In real scripts, the typical pattern is to read all arguments up front and produce a single object representing the parsed command line. This makes the rest of the script independent of `WScript.Arguments`.

```js
/*
 * parseArgs
 * Reads WScript.Arguments and produces a parsed-args object.
 *
 * Input:  none (reads WScript.Arguments)
 * Output: object with keys for known named args + a positional array
 */
function parseArgs() {
    const named = WScript.Arguments.Named;
    const unnamed = WScript.Arguments.Unnamed;

    const result = Object.create(null);

    result.input   = named.Exists("input")  ? named.Item("input")  : void null;
    result.output  = named.Exists("output") ? named.Item("output") : void null;
    result.verbose = named.Exists("verbose");
    result.quiet   = named.Exists("quiet");

    result.positional = [];

    collect_pos: for(let i = 0; i < unnamed.length; ++i)
        result.positional.push(unnamed.Item(i));

    return result;
}
```

Usage in the main script:

```js
const args = parseArgs();

if(args.input === void null) {
    WScript.Echo("Usage: cscript myScript.js /input:<file> [/output:<file>] [/verbose]");
    WScript.Quit(1);
}

if(args.verbose)
    WScript.Echo(`Reading from ${args.input}...`);
```

The pattern's value: the rest of your script reads from a clean object instead of poking at `WScript.Arguments` directly. Easier to test, easier to refactor, easier to mock if you ever need to.

### Validation in the parser

The parser is also a good place for validation — fail fast with a clear message if something's wrong:

```js
function parseArgs() {
    const named = WScript.Arguments.Named;
    const result = Object.create(null);

    if(!named.Exists("input"))
        throw new Error("Missing required argument: /input:<file>");

    result.input = named.Item("input");

    if(named.Exists("count")) {
        const n = parseInt(named.Item("count"), 10);

        if(isNaN(n) || n < 1)
            throw new Error(`Invalid /count value: ${named.Item("count")}`);

        result.count = n;
    } else {
        result.count = 10; // default
    }

    return result;
}
```

Wrap the call in a top-level `try/catch` so the user sees a friendly message instead of a stack trace:

```js
try {
    const args = parseArgs();
    main(args);
} catch(e) {
    WScript.Echo(`Error: ${e.message}`);
    WScript.Quit(1);
}
```

### Boolean flags vs values

A subtle point: if the user passes `/verbose` (no value), `Named.Item("verbose")` returns an **empty string**. If they pass `/verbose:true`, it returns `"true"` (a string). For boolean flags, just use `.Exists` — don't try to read the value:

```js
result.verbose = named.Exists("verbose"); // good

// not: result.verbose = named.Item("verbose") === "true";
```

### Exercises

1. **Mixed args.** Write a script that takes one named `/name:<value>` argument and any number of positional arguments. Print the name once, then print each positional with its index.
2. **Defaults.** Write a script with two named args: `/count` (defaults to `5`) and `/prefix` (defaults to `"item"`). It prints `<prefix>-1` through `<prefix>-N`. Test with no args, with just `/count:3`, and with both.
3. **Required + validate.** Write a script that requires `/file:<path>` and accepts optional `/maxLines:<n>`. If `/file` is missing, print usage and exit 1. If `/maxLines` is given but isn't a positive integer, throw a friendly error. Otherwise, print the first `maxLines` lines of the file (default 10).
4. **Build the parser.** Write `parseArgs()` as a reusable function (with header comment) returning an object with: `input` (string, required), `output` (string, optional, defaults to `input + ".out"`), `verbose` (boolean), `dryRun` (boolean). Test with several command-line combinations.

---

## Lesson 3.10: The rest of modern syntax

> **New words in this lesson**
>
> - **exponentiation** — raising a number to a power (`2 ** 10` is `1024`)
> - **`BigInt`** — a polyfilled type for integers too large for normal numbers
> - **`Symbol`** — a polyfilled type for unique identifiers
> - **`ES6.typeOf`** — a helper that returns proper type strings (`"bigint"`, `"symbol"`) for polyfilled types
> - **generator** — a function that can pause and resume (uses `function*` and `yield`)
> - **`yield`** — pause a generator and produce a value
> - **lazy sequence** — values produced on demand instead of all at once
> - **promise** — a placeholder for a value that will arrive later
> - **`async` / `await`** — syntax for writing promise-based code that reads top-to-bottom
> - **pipeline operator (`|>`)** — chains expressions left-to-right (non-standard, baseline-only)

You've now used most of baseline's modern syntax. This lesson cleans up the remaining features in roughly the order you're likely to encounter them.

### `**` — exponentiation

The `**` operator is shorthand for `Math.pow`:

```js
WScript.Echo(2 ** 10);  // 1024
WScript.Echo(3 ** 3);   // 27
WScript.Echo(2 ** 0.5); // 1.414... (square root)
```

The assignment form `**=` exists too:

```js
let n = 3;
n **= 2;
WScript.Echo(n); // 9
```

Use `**` over `Math.pow` in new code — it's clearer and the codebase prefers it.

### Empty `catch` blocks

When you don't care about the error object, baseline lets you omit it:

```js
try {
    // something that might throw
} catch {
    // no `(e)` — we don't need the error
}
```

This is the modern equivalent of `catch(e) { /* ignore */ }`. Use it when you genuinely want to swallow an error and continue. Most of the same caveats from Lesson 2.6 apply — silent swallowing is rarely the right answer.

### `BigInt` — arbitrary-precision integers

JScript's regular numbers are 64-bit floats, which means integer math loses precision past about 2^53. For larger integers, baseline pre-loads a `BigInt` polyfill — just use it:

```js
const big = BigInt("123456789012345678901234567890");
const doubled = BigInt.add(big, big);

WScript.Echo(doubled.toStringBase(10));
```

No import needed. `BigInt` is available everywhere in baseline source.

Two things to know:

- baseline transforms references so it behaves as if it were a native type. There's no `123n` literal syntax — construct with `BigInt(value)` or read from a string.
- It uses an explicit method API (`BigInt.add`, `BigInt.compare`, `BigInt.pow`, etc.) rather than overloaded `+`/`*` operators, since JScript can't actually overload operators.

Common methods:

- `BigInt.add(a, b)`, `BigInt.sub`, `BigInt.mul`, `BigInt.mod`, `BigInt.pow`, `BigInt.abs`
- `BigInt.compare(a, b)` — returns a sort-style number (`-1`, `0`, `1`)
- `.toStringBase(radix)`, `.isZero()`, `.negate()`, `.valueOf()` (clamped to `MAX_SAFE_INTEGER`)
- Constants: `BigInt.zero`, `BigInt.one`

One subtle point: plain `typeof big` returns `"object"`, not `"bigint"` — the underlying JScript engine doesn't know about the polyfill. If you need the proper ES6 type string, the separate `ES6` package transforms all `typeof` calls in your source into `ES6.typeOf(...)`, which returns `"bigint"`, `"symbol"`, etc., correctly. Pull in `ES6` when you specifically need that distinction; otherwise plain `typeof` is fine for everyday work.

### `Symbol` — unique identifiers

Same story — pre-loaded by baseline, no import needed:

```js
const KEY = Symbol("my-key");
const obj = Object.create(null);

obj[KEY] = "secret";
```

Symbols don't collide with string keys and don't appear in `for...in` or `Object.keys`. Useful for "private" properties on objects you don't fully control. Rarely needed in everyday code; recognize it when you see it.

`typeof` caveat is the same as for `BigInt` — plain `typeof` returns `"object"`, `ES6.typeOf` returns `"symbol"`.

### Generators — pauseable functions

A **generator function** can pause and resume mid-execution. Defined with `function*` and the `yield` keyword:

```js
function* counter() {
    let n = 0;

    forever: while(true) {
        yield n;
        ++n;
    }
}

const c = counter();

WScript.Echo(c.next().value); // 0
WScript.Echo(c.next().value); // 1
WScript.Echo(c.next().value); // 2
```

`yield` produces a value and pauses. `c.next()` resumes execution until the next `yield` (or `return`). The returned object has `.value` (the yielded value) and `.done` (boolean — `true` after the function returns).

Generators shine for **lazy sequences** — values produced on demand instead of all up front:

```js
function* take(gen, n) {
    take_loop: for(let i = 0; i < n; ++i) {
        const result = gen.next();

        if(result.done)
            return;

        yield result.value;
    }
}

const first5 = take(counter(), 5);

print_first5: for(let r = first5.next(); !r.done; r = first5.next())
    WScript.Echo(r.value);
// 0, 1, 2, 3, 4
```

`yield*` delegates to another generator, useful for composition:

```js
function* abc() {
    yield "a";
    yield "b";
    yield "c";
}

function* numbered() {
    yield* abc();
    yield "1";
    yield "2";
}
```

In our codebase, generators show up most often in transpiler pipelines — token streams, AST traversal — anywhere "produce on demand" is more efficient than "produce a giant array."

### `async` / `await`

`async` declares a function that returns a `Promise`. `await` pauses inside an async function until a promise resolves:

```js
async function fetchSomething(path) {
    const data = await readFileAsync(path);

    return data.toUpperCase();
}
```

In a typical Node.js or browser environment, this is the standard pattern for I/O. In WSH, you'll use it less — most WSH I/O is **synchronous** (FSO reads block; `Exec` blocks). When you do see `async`/`await` in our codebase, it's usually:

- Bridging into something that returns a promise (a polyfilled `Promise.resolve(value)` chain).
- Code shared with non-WSH targets.
- Generator-style flow control where promise semantics make the structure cleaner.

The mechanics:

- `await x` where `x` is a promise pauses until it resolves; the result is the resolved value.
- Throwing inside `async` rejects the returned promise. Use `try/catch` around `await` to handle errors.
- Calling an async function returns a promise; if you want the result, you `await` it (inside another async function) or chain with `.then()`.

Recognize `async`/`await` when you see it; reach for it sparingly in WSH-targeted scripts.

### The pipeline operator (`|>`)

A **non-standard** operator that chains expressions left-to-right. It makes nested function calls more readable:

```js
// Without pipeline:
const result = trim(uppercase(prefix("hello")));

// With pipeline:
const result = "hello" |> prefix |> uppercase |> trim;
```

The right side of `|>` must be a unary function — it receives the left side as its single argument.

The pipeline operator isn't part of the JavaScript standard; it's a baseline feature for our codebase. You'll see it in transformation chains where a value flows through several steps.

### Wrap

You've now seen the full vocabulary of modern JavaScript that baseline supports. From here it's mostly composition — combining what you know to solve real problems.

---

## Tier 3 wrap-up

You can now write scripts that:

- Compose modern JavaScript via baseline (templates, destructuring, spread, classes, etc.)
- Build and manipulate objects, including lookup tables and class instances
- Read, write, append, and list files via `FileSystemObject`
- Run external commands and capture their output via `WScript.Shell`
- Format dates reliably (manually, since the built-in formatters lie)
- Match, extract, and replace text via regex
- Split a project across multiple files with `import`/`export`
- Parse named and positional command-line arguments cleanly
- Use the rest of modern syntax — `**`, generators, `async`/`await`, `BigInt`, the pipeline operator

That's a complete utility-scripting toolkit. Most maintenance scripts in the codebase will use a subset of these.

### What's next

In **Tier 4: Confident** we'll cover what it takes to maintain production code without hand-holding:

- Reading other people's JScript: navigating an unfamiliar codebase efficiently
- Recognizing baseline's transformed output (so you can map runtime errors back to source)
- Debugging strategies without a real debugger
- Common gotchas in real production code
- Working with the project's journal and existing docs as a reference
- Project-specific patterns: how AutoIATE structures things, where to find what

Tier 4 is the bridge from "I can write a script" to "I can fix a bug in a script someone else wrote five years ago." You'll get there by reading more than writing.

### Capstone exercise: directory report

Write a script `dirReport.js` that produces a summary of a directory's contents. Usage:

```
cscript dirReport.js /path:<directory> [/output:<file>] [/extension:<ext>]
```

The script should:

1. Validate `/path` (required; must exist; must be a directory). Fail with a clear message if missing or invalid.
2. Walk the directory (top level only — no recursion required) and gather: total file count, total combined size, count by extension, most-recently-modified file, oldest file.
3. Format the output as a text report containing all of the above. Use the `formatDate` helper from Lesson 3.4 for any timestamps.
4. If `/output` is given, write the report to that path. Otherwise, `Echo` it to the console.
5. If `/extension` is given (e.g., `.txt`), filter the report to only files with that extension.

Requirements:

- Split the script across at least two files: a main entry script and a helpers module. Use `import`/`export`.
- Define a parsed-args object (`Object.create(null)`-based) at the top. The rest of the script should not touch `WScript.Arguments` directly.
- Include header comments on every function.
- Use labeled `for` loops everywhere iteration is required.
- Wrap the main work in a top-level `try/catch`. Print a clean error message and exit with code `1` on any failure.
- Use `Array.enumerate` to iterate the FSO collection.

Example output:

```
Directory: C:\Reports
Generated: 2025-04-12 14:30:00

Total files:  47
Total size:   12,345,678 bytes

Files by extension:
  .txt:  23
  .csv:  12
  .log:   8
  (other): 4

Most recent: report-2025-04-10.txt (2025-04-10 09:15:22)
Oldest:      archive-2023.zip       (2023-08-04 11:02:18)
```

When you can write this confidently, including using the codebase's conventions throughout, Tier 3 is complete.
