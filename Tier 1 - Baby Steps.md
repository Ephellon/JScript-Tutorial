# JScript at Work — Tier 1: Baby Steps

Welcome. This is the first of four tutorial files that will take you from "what's a variable?" to confidently maintaining the JScript codebase you're inheriting. The four tiers are:

1. **Baby Steps** (you are here): What this is, how to run scripts, basic vocabulary.
2. **Beginner**: Real programming — control flow, functions, working with text and lists.
3. **Intermediate**: Talking to the system — files, processes, modern syntax via `baseline`.
4. **Semi-Advanced**: Reading and modifying production code, including `baseline` itself.

Take your time. Do the exercises. Ask questions when you're stuck — that's much faster than spinning your wheels.

> **A note on the environment:** the machines you'll be working on are air-gapped — no internet access by design. That means no Stack Overflow, no `npm install`, no live documentation on the dev box. This tutorial is written to be self-contained, but expect to keep notes (or use a separate connected machine) when you want to look something up. When you hit a wall, ask — that's faster than guessing in the dark anyway.

---

## What you'll learn in Tier 1

- What WSH and JScript actually are, and why we use them
- How to run a script
- Variables, basic types, simple operators
- Output (`WScript.Echo`)
- Comments
- Basic conditionals (`if` / `else`)

By the end you should be able to write a 10–20 line script that prints something based on a condition. We're not trying to make you productive yet — we're building a foundation.

---

## Lesson 1.1: What is WSH/JScript?

Let's get vocabulary straight first.

**JScript** is Microsoft's version of JavaScript. Yes, JavaScript — the same language family that runs in web browsers. But:

1. **It's old.** JScript implements roughly the JavaScript standard from 1999 (called *ES3*), plus a few features from 2009 (*ES5*). Most of the modern JavaScript you'll find online — `let`, `const`, arrow functions, classes — **does not work in plain JScript**. We get around this with a tool called `baseline`, which we'll meet in Tier 2.
2. **It's local.** JScript doesn't run in a browser. It runs on your Windows machine, with full access to files, processes, and Windows itself.

**WSH** stands for **Windows Script Host**. It's the program that actually executes JScript files. It comes pre-installed on every Windows machine — that universality is a big reason we use this stack. We can deploy our scripts anywhere without installing anything.

You'll hear "JScript" and "WSH" used somewhat interchangeably. Strictly: WSH is the runtime, JScript is the language. Comparable to Node.js vs JavaScript, but Microsoft.

**Why are we still using this?** Honestly: it works, it's universal on Windows, zero install footprint, and a lot of our automation predates better alternatives. When you're reading our code, treat the WSH/JScript constraint as permanent — we're not migrating off it.

---

## Lesson 1.2: Running your first script

Open whatever text editor is available on the machine. Notepad works fine; if Notepad++ is installed, that's nicer. Type this exactly:

```js
WScript.Echo("Hello, world!");
```

Save the file as `hello.js` somewhere you can find it — let's say `C:\jscript-tutorial\hello.js`.

Open `cmd`, navigate to that folder, and run:

```
cscript hello.js
```

You should see:

```
Microsoft (R) Windows Script Host Version 5.812
Copyright (C) Microsoft Corporation. All rights reserved.

Hello, world!
```

That banner at the top is annoying. To suppress it:

```
cscript //nologo hello.js
```

Now try this instead:

```
wscript hello.js
```

A dialog box pops up with "Hello, world!" and an OK button.

**So what's the difference?**

- **`cscript`** runs the script in console mode. `WScript.Echo` prints to the terminal. This is what we use during development and for anything called from a command prompt or batch file.
- **`wscript`** runs the script in window mode. `WScript.Echo` opens a dialog. This is what end users see when they double-click a `.js` file in Windows Explorer — which is why double-clicking a script can be surprising. It'll start popping up dialogs.

For our work you'll almost always use `cscript`. There's a way to make it the default for `.js` files; we'll cover that later. For now, just remember to type it.

### What just happened?

Two things worth understanding:

1. `WScript` is a built-in object the script host provides. Think of it as a toolbox that's just *there* — you didn't import it, you didn't define it.
2. `Echo` is a method (a function attached to an object) that prints text. The parentheses pass `"Hello, world!"` as the argument.

The semicolon at the end is technically optional in JavaScript, but **always include it**. The style guide requires it, and skipping semicolons leads to a class of bugs that don't even make sense to explain yet.

### Exercises

Save your work in `C:\jscript-tutorial\` or wherever feels right.

1. **Three lines.** Modify `hello.js` to print three different messages using three separate `WScript.Echo` calls. Run it with `cscript`.
2. **Two lines, one Echo.** Make a *single* `WScript.Echo` call produce two lines of output. Hint: the string `"\n"` is a newline character — figure out where it goes.
3. **GUI vs console.** Run your three-line version with both `cscript` and `wscript`. How does `wscript` handle three Echo calls? (You'll be clicking OK a lot. This is why we use `cscript`.)
4. **Wrong filename.** Try `cscript hellobutwrong.js`. Read the error message carefully. WSH errors are not friendly — getting comfortable reading them is a real skill, especially since you can't paste them into a search engine on the dev box.

---

## Lesson 1.3: Variables and types

A **variable** is a named container that holds a value. In JScript, you declare one with the keyword `var`:

```js
var name = "Sam";
var age = 34;
var isOnTheTeam = true;
```

Three variables. Each has a name, a value, and (implicitly) a type.

**Note on style:** Our codebase uses `var` only in the *output* — code that runs directly on WSH/JScript. The *source code* of our transpilers uses `let` and `const`, which get transformed into `var` for output. You're learning JScript-as-the-engine-runs-it first, so we're using `var`. Modern keywords come in Tier 2.

### Naming rules

A variable name:

- Can contain letters, digits, `$`, and `_`
- Cannot start with a digit
- Cannot be a reserved word (`if`, `var`, `function`, etc.)
- Is case-sensitive: `total` and `Total` are different variables

Conventions matter beyond the rules. Use meaningful names — `customerCount`, not `cc`. Style guide is `camelCase` for variables and functions, which means start lowercase and capitalize each subsequent word: `firstName`, `totalRevenue`.

### The basic types

We'll use three of JScript's primitive types in Tier 1.

**Strings** — text in quotes. The codebase uses both single and double quotes for different purposes; we'll cover that distinction in Tier 2. For now, use **double quotes** for the user-facing text we're writing.

```js
var greeting = "Hello, world!";
```

**Numbers** — JScript has one number type. No separate integer/float; `5` and `5.0` are the same value.

```js
var count = 42;
var price = 19.99;
```

**Booleans** — `true` or `false`. That's it.

```js
var isActive = true;
var isFinished = false;
```

Two more primitives worth knowing about:

- `undefined` — what a variable holds if you declared it but didn't assign anything: `var x;` leaves `x` as `undefined`. *Heads-up: in our codebase you'll see `void null` used instead of the bare word `undefined`. There's a real reason for this — we'll cover it in Tier 2.*
- `null` — an explicit "nothing here" value: `var y = null;`. The difference: `undefined` is the default emptiness, `null` is intentional emptiness.

### Inspecting types with `typeof`

The `typeof` operator returns a string telling you a value's type:

```js
WScript.Echo(typeof "hello");   // "string"
WScript.Echo(typeof 42);        // "number"
WScript.Echo(typeof true);      // "boolean"
WScript.Echo(typeof undefined); // "undefined"
```

**Gotcha:** `typeof null` returns `"object"`. This is a 25-year-old bug in the JavaScript standard that nobody can fix without breaking every program ever written. Memorize it as a quirk and move on.

### Reassignment

Variables can change:

```js
var score = 0;
WScript.Echo(score); // 0

score = 10;
WScript.Echo(score); // 10
```

You only use `var` once — when you first declare the variable. After that, plain `=` reassigns it.

### Exercises

1. **Three variables.** Declare three variables: your name (string), your age (number), and whether you've had coffee today (boolean). `Echo` each.
2. **Type inspection.** For each variable above, also `Echo` its `typeof`. Confirm you get `"string"`, `"number"`, `"boolean"`.
3. **The null gotcha.** Declare `var nothing = null;` and `var notSet;` (no value). `Echo` `typeof` for both. Confirm which one returns `"object"`.
4. **Reassignment across types.** Declare `var x = 1;`. `Echo` it. Reassign `x = "one";`. `Echo` again. JScript doesn't care that you changed the type — variables are not locked to their original type. (This flexibility is a foot-gun; we'll address it later.)

---

## Lesson 1.4: Operators

An **operator** combines or transforms values. You already know `+` from math; JScript has more.

### Arithmetic

```js
var sum       = 5 + 3;   // 8
var diff      = 10 - 4;  // 6
var product   = 6 * 7;   // 42
var quotient  = 20 / 4;  // 5
var remainder = 17 % 5;  // 2
```

`%` is *modulo* (or "remainder"). `17 % 5` is `2` because `17 / 5` is `3` with `2` left over. Useful for "every Nth thing" logic.

### String concatenation

`+` also joins strings:

```js
var first = "John";
var last  = "Doe";
var full  = first + " " + last; // "John Doe"
```

This dual purpose for `+` causes a lot of early confusion. If *either* side is a string, JScript converts the other side to a string and concatenates:

```js
WScript.Echo("Score: " + 5);     // "Score: 5"   (concat)
WScript.Echo(2 + 3);              // 5            (math)
WScript.Echo("2" + 3);            // "23"         (concat — "2" is a string)
WScript.Echo(2 + 3 + " points");  // "5 points"   (math first, then concat)
```

That last one is sneaky. JScript reads left to right: `2 + 3` becomes `5` (both numbers), then `5 + " points"` becomes `"5 points"`.

### Comparison

```js
5 > 3    // true
5 < 3    // false
5 >= 5   // true
5 <= 4   // false
```

Equality has two forms, and the difference is critical:

```js
5 == "5"    // true   — loose equality, converts types before comparing
5 === "5"   // false  — strict equality, types must match
```

**Always use `===` and `!==`.** The loose `==` and `!=` operators do "helpful" type conversions that are mostly unhelpful. You'll see `==` in legacy parts of our codebase; new code uses `===`.

### Logical

```js
true  && true    // true   — AND: both must be true
true  && false   // false
true  || false   // true   — OR:  at least one must be true
!true            // false  — NOT: flips the value
```

These are most useful inside conditionals, which we'll cover next lesson.

### Assignment shortcuts

```js
var x = 10;
x += 5;  // same as x = x + 5;  → 15
x -= 3;  // same as x = x - 3;  → 12
x *= 2;  // → 24
x /= 4;  // → 6
```

And `++` (add 1), `--` (subtract 1) — by style guide, prefer the **prefix** form (`++n`, `--n`):

```js
var n = 5;
++n;     // n is now 6
--n;     // n is back to 5
```

There's a postfix form (`n++`, `n--`) that you'll see occasionally — it has subtly different evaluation timing when used inside larger expressions. We'll get into that in a later tier. For now, prefix is the safer default.

### Exercises

1. **Math practice.** Declare two number variables and `Echo` their sum, difference, product, and quotient.
2. **The concatenation trap.** Predict the output of `WScript.Echo(1 + 2 + "3" + 4 + 5);` *before* running it. Then run it. Walk through left-to-right to understand why.
3. **Strict equality.** Echo the results of `5 == "5"`, `5 === "5"`, `0 == false`, and `0 === false`. Convince yourself why strict is the safer default.
4. **Compound assignment.** Start with `var n = 100;`. Apply `n -= 25;`, then `n /= 5;`, then `n *= 2;`. Echo `n` after each step. (Final answer: 30.)

---

## Lesson 1.5: Comments

Comments are notes for human readers. JScript ignores them.

```js
// This is a single-line comment.
var count = 10; // Comments can also follow code.

/*
   This is a block comment.
   It can span multiple lines.
   Useful for longer explanations.
*/
```

Use comments to explain *why* you're doing something, not *what* you're doing. The code itself shows what — your comment should add something the code can't say.

```js
// Bad: explains what the code obviously does
var total = 0; // set total to zero

// Good: explains why
var total = 0; // accumulator for the daily revenue loop below
```

You'll see a special form of block comment in our codebase: `/** ... */` (note the double asterisk on the opening). That's a **JSDoc** comment, used for documenting functions. We'll cover JSDoc properly in Tier 2; for now, just recognize it when you see it.

### Exercises

1. **Self-documenting script.** Take your three-variables script from Lesson 1.3 and add a comment above each variable explaining what it represents.
2. **Commenting out code.** Add a fourth `Echo` line to any script, then comment it out so it doesn't run. Confirm the output drops the fourth line.
3. **Block comment header.** Add a `/* ... */` block at the top of the script with a one-paragraph description of what the script does.

---

## Lesson 1.6: Conditionals — `if` / `else`

A **conditional** runs different code depending on whether something is true.

### `if`

```js
var temperature = 75;

if(temperature > 70)
    WScript.Echo("Warm enough.");
```

If the expression in the parentheses is `true`, the next line runs. If not, it's skipped. Two style points worth flagging up front:

- No space between `if` and `(` — it's `if(...)`, not `if (...)`. Same for `while`, `for`, `switch`.
- Single-statement bodies sit on their own line, indented one level. No braces needed.

### `else`

For "otherwise":

```js
var hour = 14;

if(hour < 12)
    WScript.Echo("Good morning.");
else
    WScript.Echo("Good afternoon.");
```

Exactly one of those `Echo`s runs — whichever branch matches.

### `else if`

For more than two branches:

```js
var score = 73;

if(score >= 90)
    WScript.Echo("A");
else if(score >= 80)
    WScript.Echo("B");
else if(score >= 70)
    WScript.Echo("C");
else if(score >= 60)
    WScript.Echo("D");
else
    WScript.Echo("F");
```

JScript checks conditions in order and runs the *first* matching branch. **Order matters** — if you wrote `score >= 60` first, every score above 60 would match it before reaching the higher thresholds.

### When you need braces

Single-statement bodies don't need braces. Multi-statement bodies do:

```js
if(temperature > 70) {
    WScript.Echo("Warm enough.");
    WScript.Echo("Going outside.");
}
```

One sibling-consistency rule from the style guide: if any branch in an `if/else if/else` chain has braces, *all* of them do. Don't mix — it reads badly and the guide rejects it.

### Truthy and falsy

The expression in `if(...)` doesn't have to literally be `true` or `false`. JScript *coerces* whatever you put there into a boolean. Values that coerce to `false` are called **falsy**:

- `false`
- `0` (the number zero)
- `""` (the empty string)
- `null`
- `undefined`
- `NaN` (a special "not a number" value — we'll meet it in Tier 2)

Every other value is **truthy**, including:

- Any non-zero number (positive *or* negative)
- Any non-empty string — even `"false"`, because it's a non-empty string
- Any object, array, or function, even empty ones (`{}`, `[]`)

This is convenient, but it's also a bug source:

```js
var count = 0;

if(count)
    WScript.Echo("We have items.");
else
    WScript.Echo("No items.");
```

That looks like it should say "No items" — and it does, because `0` is falsy. But what if `count` could legitimately be `0`, and we wanted to distinguish "we counted, found zero" from "uninitialized"? Then `if(count)` is wrong, and we'd want:

```js
if(count > 0)
    // ...
```

**Rule of thumb:** when in doubt, be explicit. `if(count > 0)` is clearer than `if(count)`, even when both work. Save the implicit version for when you genuinely mean "is this thing set / non-empty?"

### Combining conditions

Use `&&`, `||`, and `!` to combine:

```js
var age = 25;
var hasLicense = true;

if(age >= 18 && hasLicense)
    WScript.Echo("Can drive.");
```

Parentheses help readability when you mix operators:

```js
if((age >= 18 && hasLicense) || isPassenger)
    WScript.Echo("Can ride.");
```

### Exercises

1. **Even or odd.** Declare `var n = 7;`. Use `if`/`else` and the `%` operator to print `"even"` or `"odd"`. (Hint: `n % 2` is `0` for even numbers — use `===` to compare.)
2. **Three branches.** Declare `var temp = 65;`. Print `"cold"` if below 50, `"mild"` if 50–75, `"hot"` above 75.
3. **Truthy explorer.** For each of these values, write an `if`/`else` that prints `"truthy"` or `"falsy"`: `0`, `1`, `""`, `"hello"`, `null`, `"false"`. Confirm that `"false"` reports as truthy.
4. **The grade calculator.** Write the full grade script using the `else if` chain shown above. Test it with at least three different scores by editing and re-running.

---

## Tier 1 wrap-up

You now know enough JScript to write small scripts that:

- Print messages
- Store values in variables
- Do arithmetic and string manipulation
- Make decisions based on conditions

That's a real skill. Every script in our codebase is built from these same pieces — just with more layers on top.

### What's next

In **Tier 2: Beginner** we'll cover:

- Loops — making the script repeat work
- Functions — bundling code into reusable pieces
- Arrays — storing lists of things
- String methods — manipulating text properly
- Error handling — what to do when something goes wrong
- An introduction to `baseline` and the modern syntax we use day-to-day

### Capstone exercise

Write a single script that does all of this:

1. Declares a variable for someone's age (hardcode any value).
2. Prints the age.
3. Prints their life stage: `"minor"` (under 18), `"adult"` (18–64), or `"senior"` (65+).
4. Includes a comment block at the top describing what the script does.
5. Uses meaningful variable names and follows the style conventions you've learned: double quotes for user-facing strings, `if(...)` with no space, semicolons everywhere, prefix `++`/`--`.

It should land around 10–15 lines. When you can write this without referring back to the lessons, Tier 1 is complete.
