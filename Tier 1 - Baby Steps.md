# JScript at Work — Tier 1: Baby Steps

The first of four tutorial files. By the end of Tier 1 you can write a small script that prints something based on a condition.

Do the exercises. When you're stuck, ask — that's faster than guessing.

> **Known Caveat: air-gapped environment.** The dev machines have no internet. No Stack Overflow, no live docs. This tutorial is self-contained; for anything outside it, use a connected machine and bring notes.

---

## What you'll learn in Tier 1

- What WSH and JScript are
- How to run a script
- Variables and basic types
- Output (`WScript.Echo`)
- Comments
- Operators
- Basic conditionals (`if` / `else`)

---

## Lesson 1.1: What is WSH/JScript?

> **New words in this lesson**
>
> - **JScript** — Microsoft's version of JavaScript
> - **WSH** — Windows Script Host, the program that runs JScript files
> - **runtime** — the program that executes your code (here, WSH)

**JScript** is Microsoft's version of JavaScript. Two things to know:

1. **It's old.** JScript follows an old version of the language. Newer features from online tutorials may not work — stick to this tutorial.
2. **It's local.** JScript runs on your Windows machine with full access to files, processes, and Windows itself.

**WSH** is **Windows Script Host** — the program that runs JScript files. It's pre-installed on every Windows machine, which is the reason we use it: zero install footprint, deploys anywhere.

> **Nice to Know.** "JScript" and "WSH" get used interchangeably. Strictly: WSH is the runtime, JScript is the language. We're not migrating off this stack — treat it as permanent.

---

## Lesson 1.2: Running your first script

> **New words in this lesson**
>
> - **script** — a file of code that gets run from top to bottom
> - **console** — the text-based command-line window (`cmd`)
> - **method** — a function attached to an object (like `Echo` on `WScript`)
> - **argument** — a value you pass into a function or method when you call it

Open a text editor (Notepad works; Notepad++ is nicer if available). Type:

```js
WScript.Echo("Hello, world!");
```

Save as `hello.js` in `C:\Users\B1-IATE\Documents\Tutorial\`.

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

To suppress the banner:

```
cscript //nologo hello.js
```

Now try:

```
wscript hello.js
```

A dialog box pops up with "Hello, world!" and an OK button.

### `cscript` vs `wscript`

- **`cscript`** — console mode. `WScript.Echo` prints to the terminal. Use this for development and anything called from a batch file.
- **`wscript`** — window mode. `WScript.Echo` opens a dialog. This is what end users see when they double-click a `.js` file in Explorer.

You'll almost always use `cscript`.

### What just happened?

- `WScript` is a built-in object the script host provides. You didn't define it — it's just *there*.
- `Echo` is a method that prints text. The parentheses pass `"Hello, world!"` as the argument.

> **Known Caveat: semicolons.** They're technically optional in JavaScript. **Always include them.** The style guide requires it, and skipping them causes bugs that aren't worth explaining.

### Exercises

1. **Three lines.** Modify `hello.js` to print three messages using three `WScript.Echo` calls. Run it with `cscript`.
2. **Two lines, one Echo.** Make a *single* `WScript.Echo` call produce two lines. Hint: the string `"\n"` is a newline character.
3. **GUI vs console.** Run your three-line version with both `cscript` and `wscript`. Note how `wscript` handles three Echo calls.
4. **Wrong filename.** Try `cscript hellobutwrong.js`. Read the error carefully — getting comfortable with WSH's unhelpful errors is a real skill.

---

## Lesson 1.3: Variables and types

> **New words in this lesson**
>
> - **variable** — a named place to store a value
> - **declare** — to introduce a new variable for the first time
> - **value** — the actual data: a number, some text, true/false, etc.
> - **type** — what kind of value something is
> - **literal** — a value written directly in code, like `5` or `"hello"`
> - **string** — text
> - **boolean** — a value that's either `true` or `false`
> - **camelCase** — a naming style: lowercase first word, capital first letter on each subsequent word

A **variable** is a named container that holds a value. Declare one with `var`:

```js
var name = "Sam";
var age = 34;
var isOnTheTeam = true;
```

### Naming rules

A variable name:

- Can contain letters, digits, `$`, and `_`
- Cannot start with a digit
- Cannot be a reserved word (`if`, `var`, `function`, etc.)
- Is case-sensitive: `total` and `Total` are different

Use meaningful names — `customerCount`, not `cc`. Style is `camelCase`: `firstName`, `totalRevenue`.

### Types and literals

A **value** has a **type**. When you write a value directly in code, that's a **literal**. There are visual rules for writing literals of different types.

**Strings** — text — go inside double quotes.

```js
var greeting = "Hello, world!";
var name = "Sam";
```

**Numbers** — go without quotes. JScript has one number type for whole numbers and decimals.

```js
var count = 42;
var price = 19.99;
```

**Booleans** — the bare words `true` and `false`. No quotes.

```js
var isActive = true;
var isFinished = false;
```

Two more bare-word values:

- `undefined` — what a variable holds if you declared it but didn't assign anything: `var x;` leaves `x` as `undefined`.
- `null` — an explicit "nothing here" value: `var y = null;`.

### Quotes vs no quotes — why it matters

The quoting rule isn't decoration. It's how JScript tells what you mean.

```js
"hello"   // a string — the five letters h, e, l, l, o
hello     // a name — JScript looks for a variable called "hello"
42        // a number — the value forty-two
"42"      // a string — the two characters 4 and 2
true      // a boolean — the value true
"true"    // a string — the four letters t, r, u, e
```

If you write `"True"` when you mean `true`, you have a string, not a boolean. They behave differently.

If you write `john` when you mean `"John"`, JScript will look for a variable called `john` and complain that it doesn't exist.

Common mistakes:

- **Forgetting quotes around a string.** `var name = John;` should be `var name = "John";`
- **Adding quotes around a boolean.** `var isActive = "true";` is a string, not a boolean.
- **Adding quotes around a number when you want math.** `"5" + "3"` is `"53"` (a string), not `8`.

When in doubt, ask: "Is this *text* I want to display, or a *name* I want JScript to look up, or a *number* I want to compute with?" Quotes go around text. Names don't get quotes. Numbers don't get quotes.

### Inspecting types with `typeof`

The `typeof` operator returns a string telling you a value's type:

```js
WScript.Echo(typeof "hello");   // Output is text: "string"
WScript.Echo(typeof 42);        // Output is text: "number"
WScript.Echo(typeof true);      // Output is text: "boolean"
WScript.Echo(typeof undefined); // Output is text: "undefined"
```

> **Known Caveat: `typeof null`.** Returns `"object"`. This is a 25-year-old bug in the JavaScript standard that can't be fixed without breaking every program ever written. Memorize and move on.

### Reassignment

Variables can change:

```js
var score = 0;
WScript.Echo(score); // Output is a number: 0

score = 10;
WScript.Echo(score); // Output is a number: 10
```

Use `var` only when you first declare. After that, plain `=` reassigns.

### Exercises

1. **Three variables.** Declare three variables: your name (string), your age (number), and whether you've had coffee today (boolean). `Echo` each.
2. **Type inspection.** For each variable above, also `Echo` its `typeof`. Confirm `"string"`, `"number"`, `"boolean"`.
3. **Quote-quiz.** Predict the output of each *before* running. Then run them and check.
   ```js
   WScript.Echo(typeof "42");
   WScript.Echo(typeof 42);
   WScript.Echo(typeof "true");
   WScript.Echo(typeof true);
   ```
4. **The null gotcha.** Declare `var nothing = null;` and `var notSet;`. `Echo` `typeof` for both. Confirm which returns `"object"`.
5. **Reassignment across types.** Declare `var x = 1;`. `Echo` it. Reassign `x = "one";`. `Echo` again. JScript doesn't lock variables to their original type.

---

## Lesson 1.4: Comments

> **New words in this lesson**
>
> - **comment** — a note for humans that JScript ignores
> - **block comment** — a multi-line comment using `/* ... */`

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

Use comments to explain *why*, not *what*. The code itself shows what.

```js
// Bad: explains what the code obviously does
var total = 0; // set total to zero

// Good: explains why
var total = 0; // accumulator for the daily revenue loop below
```

### Exercises

1. **Self-documenting script.** Take your three-variables script from Lesson 1.3 and add a comment above each variable explaining what it represents.
2. **Commenting out code.** Add a fourth `Echo` line, then comment it out. Confirm the output drops the fourth line.
3. **Block comment header.** Add a `/* ... */` block at the top of any script with a one-paragraph description of what it does.

---

## Lesson 1.5: Operators

> **New words in this lesson**
>
> - **operator** — a symbol that combines or transforms values, like `+` or `*`
> - **arithmetic** — math
> - **concatenation** — joining two strings (text values) together
> - **modulo** — the remainder after division
> - **strict equality** — checking equality without converting types

An **operator** combines or transforms values.

### Arithmetic

```js
var sum       = 5 + 3;   // 8
var diff      = 10 - 4;  // 6
var product   = 6 * 7;   // 42
var quotient  = 20 / 4;  // 5
var remainder = 17 % 5;  // 2
```

`%` is *modulo* — the remainder after division. `17 % 5` is `2` because `17 / 5` is `3` with `2` left over. Useful for "every Nth thing" logic.

### String concatenation (joining text)

`+` also joins strings — this is **concatenation**:

```js
var first = "John";
var last  = "Doe";
var full  = first + " " + last; // "John Doe"
```

If *either* side is a string, JScript converts the other side to a string and concatenates:

```js
WScript.Echo("Score: " + 5);     // Output is text: "Score: 5"
WScript.Echo(2 + 3);              // Output is a number: 5
WScript.Echo("2" + 3);            // Output is text: "23"
WScript.Echo(2 + 3 + " points");  // Output is text: "5 points"
```

That last one is sneaky. JScript reads left to right: `2 + 3` is `5` (both numbers), then `5 + " points"` is `"5 points"`.

### Comparison

```js
5 > 3    // true
5 < 3    // false
5 >= 5   // true
5 <= 4   // false
```

Equality has two forms:

```js
5 == "5"    // true   — loose equality, converts types
5 === "5"   // false  — strict equality, types must match
```

**Always use `===` and `!==`.** The loose `==` and `!=` do unhelpful type conversions.

### Logical

```js
true  && true    // true   — AND: both must be true
true  && false   // false
true  || false   // true   — OR:  at least one must be true
!true            // false  — NOT: flips the value
```

Most useful inside conditionals (next lesson).

### Exercises

1. **Math practice.** Declare two number variables and `Echo` their sum, difference, product, and quotient.
2. **The concatenation trap.** Predict the output of `WScript.Echo(1 + 2 + "3" + 4 + 5);` *before* running. Then run it. Walk through left-to-right.
3. **Strict equality.** Echo `5 == "5"`, `5 === "5"`, `0 == false`, and `0 === false`.
4. **Build a sentence.** Declare three variables: a string for someone's name, a number for their age, a boolean for whether they're a student. Build and `Echo` a single sentence using all three values via concatenation, like `"Sam is 34 years old. Student: true."`

---

## Lesson 1.6: Conditionals — `if` / `else`

> **New words in this lesson**
>
> - **conditional** — code that runs only when a condition is true
> - **expression** — anything that produces a value
> - **branch** — one of the possible paths an `if` / `else if` / `else` chain can take
> - **truthy** — a value that counts as `true` in a conditional
> - **falsy** — a value that counts as `false` in a conditional
> - **coerce** — to automatically convert one type to another

A **conditional** runs different code depending on whether something is true.

### `if`

```js
var temperature = 75;

if(temperature > 70)
    WScript.Echo("Warm enough.");
```

If the expression in the parentheses is `true`, the next line runs. Two style points:

- No space between `if` and `(` — it's `if(...)`, not `if (...)`. Same for `while`, `for`, `switch`.
- Single-statement bodies sit on their own line, indented one level. No braces.

### `else`

```js
var hour = 14;

if(hour < 12)
    WScript.Echo("Good morning.");
else
    WScript.Echo("Good afternoon.");
```

Exactly one branch runs.

### `else if`

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

JScript checks conditions in order and runs the *first* match. **Order matters** — `score >= 60` first would match every score above 60 before reaching the higher thresholds.

### When you need braces

Single-statement bodies don't need braces. Multi-statement bodies do:

```js
if(temperature > 70) {
    WScript.Echo("Warm enough.");
    WScript.Echo("Going outside.");
}
```

> **Known Caveat: sibling consistency.** If any branch in an `if/else if/else` chain has braces, *all* of them do. Don't mix.

### Truthy and falsy

The expression in `if(...)` doesn't have to literally be `true` or `false`. JScript **coerces** whatever you put there into a boolean. Values that coerce to `false` are **falsy**:

- `false`
- `0` (the number zero)
- `""` (the empty string)
- `null`
- `undefined`

Every other value is **truthy**, including:

- Any non-zero number (positive *or* negative)
- Any non-empty string — even `"false"`, because it's a non-empty string

This is convenient *and* a bug source:

```js
var count = 0;

if(count)
    WScript.Echo("We have items.");
else
    WScript.Echo("No items.");
```

Prints "No items" because `0` is falsy. But what if `count` could legitimately be `0`, and we wanted to distinguish "we counted, found zero" from "uninitialized"? Then `if(count)` is wrong:

```js
if(count > 0)
    // ...
```

**Rule of thumb:** when in doubt, be explicit. `if(count > 0)` is clearer than `if(count)` even when both work.

### Combining conditions

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

1. **Even or odd.** Declare `var n = 7;`. Use `if`/`else` and `%` to print `"even"` or `"odd"`. (Hint: `n % 2` is `0` for even.)
2. **Three branches.** Declare `var temp = 65;`. Print `"cold"` below 50, `"mild"` 50–75, `"hot"` above 75.
3. **Truthy explorer.** For each value, write an `if`/`else` that prints `"truthy"` or `"falsy"`: `0`, `1`, `""`, `"hello"`, `null`, `"false"`. Confirm `"false"` reports as truthy.
4. **The grade calculator.** Write the full grade script from above. Test with at least three different scores.

---

## Tier 1 wrap-up

You can now write small scripts that:

- Print messages
- Store values in variables
- Do arithmetic and string manipulation
- Make decisions based on conditions

Every script in our codebase is built from these same pieces.

### Capstone exercise

Write a single script that:

1. Declares a variable for someone's age (any value).
2. Prints the age.
3. Prints their life stage: `"minor"` (under 18), `"adult"` (18–64), or `"senior"` (65+).
4. Has a comment block at the top describing what it does.
5. Uses meaningful variable names and the style conventions from this tier: double quotes for user-facing strings, `if(...)` with no space, semicolons everywhere.

Around 10–15 lines. When you can write this without referring back, Tier 1 is complete.
