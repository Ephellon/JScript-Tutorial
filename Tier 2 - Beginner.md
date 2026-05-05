# JScript at Work — Tier 2: Beginner

Welcome to Tier 2. By the end you'll be able to write small but genuinely useful scripts — file renamers, log filters, simple report generators. The pieces we'll add:

- **Loops** — making the script repeat work
- **Functions** — bundling code into reusable pieces
- **Arrays** — storing lists of things
- **String methods** — manipulating text properly
- **Math** — built-in numeric operations
- **Error handling** — what to do when something goes wrong
- **Script arguments** — reading values passed in from the command line
- **An introduction to `baseline`** and the modern syntax our codebase uses

The capstone for Tier 2 is a real utility script — small enough to write in one sitting, useful enough to keep around.

---

## Lesson 2.1: Loops

A **loop** runs a block of code repeatedly. Three kinds matter in JScript: `while`, `do...while`, and `for`.

### `while`

A `while` loop runs as long as a condition is true.

```js
var n = 0;

while(n < 5) {
    WScript.Echo(n);
    ++n;
}
```

That prints `0` through `4`. The condition is checked **before** each iteration. When `n` reaches `5`, the condition is false, and the loop exits.

If the condition is false on the first check, the body never runs. If you forget to update the variable that makes the condition eventually false, you have an **infinite loop** — the script runs forever (or until you Ctrl-C it). Spotting and avoiding this is a real skill.

### `do...while`

A close cousin: runs the body **first**, then checks the condition.

```js
var n = 0;

do {
    WScript.Echo(n);
    ++n;
} while(n < 5);
```

Same output. The difference matters when the condition starts false: a `do...while` still runs once. Not common in our codebase, but worth recognizing.

### `for`

The workhorse. A `for` loop has three parts in its header — initialization, condition, and step:

```js
for(var i = 0; i < 5; ++i) {
    WScript.Echo("Step " + i);
}
```

Read it as: *"start with `i = 0`; keep going while `i < 5`; after each iteration, increment `i`."* Same output structure as the `while` loop above, just more compact.

The three parts can each be empty:

```js
for(;;) {
    // infinite loop on purpose — break out from inside
}
```

A small note on style: the for-loop step expression is the **one place** where postfix `i++` is acceptable alongside prefix `++i`. Everywhere else, prefer prefix.

### Labeled loops — required style

Here's something specific to our codebase: **all `for` and `while` loops must be labeled.** A label is a name attached to the loop, used for `break` and `continue`. The style guide is firm on this.

Label names use `snake_case`:

```js
loop_steps: for(var i = 0; i < 5; ++i) {
    WScript.Echo("Step " + i);
}
```

Why force this? Because nested loops are common in our transpiler code, and an unlabeled `break` always exits the *innermost* loop — which is often not what you want once nesting gets thick. Requiring labels means there's never ambiguity about which loop is being affected, even in shallow code where it doesn't matter yet. Build the habit early.

The rules:

- Label name on the same line as the loop, followed by `:`, then the loop keyword.
- `break` and `continue` **always** name their target: `break loop_steps;` or `continue loop_steps;`.
- A bare `break;` or `continue;` is **not allowed** in our codebase.
- For long or deeply nested loops, add a closing breadcrumb comment after the closing brace: `} // :loop_steps`. The exact rule for when breadcrumbs are required is in Tier 3 territory; for now, follow whatever pattern the surrounding code uses.

Example with `break`:

```js
search: for(var i = 1; i < 100; ++i) {
    if(i % 13 === 0) {
        WScript.Echo("First multiple of 13: " + i);

        break search;
    }
}
```

`break` exits the named loop entirely. `continue` skips to the next iteration of the named loop:

```js
print_evens: for(var n = 0; n < 10; ++n) {
    if(n % 2 !== 0)
        continue print_evens;

    WScript.Echo(n);
}
```

That prints `0`, `2`, `4`, `6`, `8` — the `continue` skips the `Echo` for odd numbers.

### A blank-line rule for `break` and `continue`

The style guide requires a blank line **before** any standalone `break` or `continue` — you saw it in the two examples above. The reasoning: these are control-flow exits, and a little visual breathing room makes them harder to miss when scanning code.

```js
// Correct
if(i % 13 === 0) {
    WScript.Echo("Found");

    break search;
}

// Wrong — missing blank line before break
if(i % 13 === 0) {
    WScript.Echo("Found");
    break search;
}
```

### When to use which loop

- **`for`** — when you know the count, or have a clear iteration structure (index from 0 to N, character-by-character scan, etc.)
- **`while`** — when the loop continues "until something changes" and there's no clean counter
- **`do...while`** — rarely; only when the body must run at least once

In practice our codebase is mostly `for` loops, with the occasional `while` for character-by-character scanning in the transpilers.

### Exercises

1. **Count up.** Write a `for` loop labeled `count_up` that prints the numbers 1 through 10.
2. **Count down.** Write a `for` loop labeled `count_down` that prints from 10 down to 1. (Hint: start at 10, condition `>= 1`, decrement.)
3. **Sum the first hundred.** Write a labeled `for` loop that adds the numbers 1 through 100 into a variable `total`, then `Echo`s `total`. Expected output: `5050`.
4. **First match.** Write a labeled `for` loop that scans the numbers 1 through 50 looking for the first multiple of 7, prints it, and `break`s out of the loop. (The answer is 7. Make sure the loop stops there and doesn't keep iterating.)
5. **Skip the bad ones.** Write a labeled `for` loop over 1 through 20 that uses `continue` to skip multiples of 3 and `Echo`s the rest.

---

## Lesson 2.2: Functions

A **function** is a reusable, named block of code. You define it once and call it as many times as you need.

### Defining and calling a function

The simplest function:

```js
/*
 * sayHello
 * Prints a greeting to the console.
 *
 * Input:  none
 * Output: none (prints to console)
 */
function sayHello() {
    WScript.Echo("Hello!");
}

sayHello();
sayHello();
sayHello();
```

That prints `Hello!` three times. The function is **defined** once with `function sayHello() { ... }` and **called** three times with `sayHello();`. The parentheses are how JScript knows you're invoking the function rather than just referring to it.

### Header comments are required

Notice the comment block above the function. **Every function in our codebase must have a header comment** describing what it does and its inputs and outputs. The style guide is firm. The format:

```
/*
 * functionName
 * One- or two-line description of what it does.
 *
 * Input:  description of parameters
 * Output: description of return value, or a note like "none (prints to console)"
 */
```

You'll also see JSDoc-style block comments (`/** ... */`) in the codebase, mostly applied automatically by our tooling. Both formats convey the same information; follow whichever the surrounding code uses.

### Parameters and arguments

A function can take **parameters** — values passed in:

```js
/*
 * greet
 * Prints a personalized greeting.
 *
 * Input:  name (string) — who to greet
 * Output: none (prints to console)
 */
function greet(name) {
    WScript.Echo("Hello, " + name + "!");
}

greet("Sam");   // Hello, Sam!
greet("Alex");  // Hello, Alex!
```

Vocabulary: the thing in the parentheses of the *definition* (`name`) is the **parameter** — a placeholder. The thing in the parentheses of the *call* (`"Sam"`) is the **argument** — the actual value passed in. People mix these up constantly; getting them straight makes documentation easier to read.

Multiple parameters separated by commas:

```js
/*
 * sumAndPrint
 * Prints the sum of two numbers.
 *
 * Input:  a (number), b (number)
 * Output: none (prints to console)
 */
function sumAndPrint(a, b) {
    WScript.Echo(a + b);
}

sumAndPrint(3, 4); // 7
```

### Return values

Most useful functions don't just *do* something — they *compute and return* something. Use `return`:

```js
/*
 * square
 * Computes the square of a number.
 *
 * Input:  n (number)
 * Output: n * n (number)
 */
function square(n) {
    return n * n;
}

var result = square(5);
WScript.Echo(result); // 25
```

A `return` statement does two things: it sends a value back to the caller, and it exits the function immediately. Anything after the `return` (in the same branch) doesn't run.

Functions without an explicit `return` automatically return `undefined`. So `sayHello()` and `greet("Sam")` above both return `undefined` — they just don't use that fact.

### Scope basics

Variables declared with `var` inside a function are **local** to that function. Code outside can't see them:

```js
function doStuff() {
    var temp = 42;
    WScript.Echo(temp); // works — we're inside the function
}

doStuff();
WScript.Echo(temp); // ERROR — temp is not defined out here
```

Conversely, code inside a function can read variables declared outside:

```js
var globalCount = 0;

function increment() {
    ++globalCount; // reads and modifies the outer variable
}

increment();
increment();
WScript.Echo(globalCount); // 2
```

Using global variables this way works but is generally a poor practice — it makes code hard to reason about because anything can change them. Prefer passing values in and returning values out.

A subtle point: `var` is **function-scoped**, not block-scoped. A `var` declared inside an `if` or `for` block leaks out to the whole enclosing function. This is one of the things `let` and `const` (introduced via baseline) fix. We'll see them in Lesson 2.8.

### Hoisting — a real gotcha

JScript does something unusual: it "hoists" function declarations and `var` declarations to the top of their scope before executing. Practical effect:

```js
sayHi(); // works — function is hoisted

function sayHi() {
    WScript.Echo("Hi!");
}
```

That's surprising if you're used to other languages. It works because the entire function definition is hoisted.

But for `var`, only the *declaration* is hoisted, not the *assignment*:

```js
WScript.Echo(x); // undefined — x is declared but not yet assigned
var x = 10;
WScript.Echo(x); // 10
```

Mental model: JScript secretly rewrites your code as `var x; WScript.Echo(x); x = 10;` before running it. This is rarely something to rely on; mostly it's a footgun to recognize.

**Rule of thumb:** declare your variables at the top of the function, before you use them. Don't depend on hoisting to bail you out.

### The `arguments` object

Inside any function, JScript automatically creates a special variable called `arguments` that holds all the values passed in:

```js
function showAll() {
    WScript.Echo("Got " + arguments.length + " argument(s)");

    list_args: for(var i = 0; i < arguments.length; ++i)
        WScript.Echo("  " + i + ": " + arguments[i]);
}

showAll("a", "b", "c");
// Got 3 argument(s)
//   0: a
//   1: b
//   2: c
```

`arguments` is array-*like* — it has `.length` and supports indexing — but it's not actually an array. Mostly useful for *variadic* functions (functions that accept a variable number of arguments). You'll see it more in Tier 3.

### Exercises

1. **Add two numbers.** Write a function `add(a, b)` (with a header comment) that returns the sum of its two arguments. Test it by `Echo`ing the result of `add(3, 4)`.
2. **Even or odd.** Write a function `isEven(n)` that returns `true` if `n` is even, `false` otherwise. Test with several values.
3. **Largest of three.** Write a function `maxOfThree(a, b, c)` that returns the largest of three numbers.
4. **Mini FizzBuzz.** Write a function `fizzBuzz(n)` that returns:
   - `"FizzBuzz"` if `n` is divisible by both 3 and 5
   - `"Fizz"` if divisible by 3 only
   - `"Buzz"` if divisible by 5 only
   - the number itself (as a string — use `"" + n` to convert) otherwise

   Test by calling it with `15`, `9`, `10`, and `7`.
5. **Hoisting in action.** Write a script that calls a function `sayHi()` *before* defining it (it should still work). Then declare `var x` and `Echo` it *before* its `var x = 10;` line, then `Echo` again after. Note the `undefined` followed by the value.

---

## Lesson 2.3: Arrays

An **array** is an ordered list of values. JScript arrays can hold any types — strings, numbers, booleans, even other arrays — though in practice you usually use one type per array.

### Creating and reading arrays

The literal syntax (preferred):

```js
var fruits = ["apple", "banana", "cherry"];
var ages   = [34, 27, 41, 19];
var empty  = [];
```

**Style note:** array literals are **not** padded inside the brackets. `[1, 2, 3]`, never `[ 1, 2, 3 ]`. Empty arrays are `[]`. (Object literals are different — they *do* get padding. We'll cover those in Tier 3.)

Read elements by their **index**, starting from `0`:

```js
var fruits = ["apple", "banana", "cherry"];

WScript.Echo(fruits[0]); // apple
WScript.Echo(fruits[1]); // banana
WScript.Echo(fruits[2]); // cherry
```

Reading past the end gives `undefined`:

```js
WScript.Echo(fruits[99]); // undefined
```

### `.length`

Every array has a `.length` property — the number of elements:

```js
var fruits = ["apple", "banana", "cherry"];
WScript.Echo(fruits.length); // 3
```

`.length` is one greater than the highest valid index — that pattern shows up constantly in iteration code.

### Modifying arrays

Change an element by assigning to its index:

```js
var fruits = ["apple", "banana", "cherry"];
fruits[1] = "blueberry";
WScript.Echo(fruits[1]); // blueberry
```

Add or remove with built-in **methods**:

```js
var nums = [1, 2, 3];

nums.push(4);     // adds to end:        [1, 2, 3, 4]
nums.pop();       // removes from end:   [1, 2, 3], returns 4
nums.unshift(0);  // adds to start:      [0, 1, 2, 3]
nums.shift();     // removes from start: [1, 2, 3], returns 0
```

Mnemonic: **push/pop** work at the end; **shift/unshift** work at the start. Both **pop** and **shift** *return* the removed element so you can capture it.

### Slicing

`.slice(start, end)` returns a new array containing elements from `start` (inclusive) to `end` (exclusive):

```js
var nums = [10, 20, 30, 40, 50];
var middle = nums.slice(1, 4);
WScript.Echo(middle); // 20,30,40
```

The original array isn't modified — `.slice` is non-destructive. (When you `Echo` an array, JScript joins it with commas. That's just default formatting.)

### Iterating

The standard pattern is a labeled `for` loop driven by `.length`:

```js
var fruits = ["apple", "banana", "cherry"];

print_fruits: for(var i = 0; i < fruits.length; ++i)
    WScript.Echo(fruits[i]);
```

That prints each fruit on its own line. **Memorize this pattern** — you'll write it thousands of times.

### Searching

To find whether (and where) a value is in an array, scan with a labeled `for` loop:

```js
var fruits = ["apple", "banana", "cherry"];
var foundAt = -1;

search_fruits: for(var i = 0; i < fruits.length; ++i) {
    if(fruits[i] === "banana") {
        foundAt = i;

        break search_fruits;
    }
}

if(foundAt !== -1)
    WScript.Echo("Found banana at index " + foundAt);
else
    WScript.Echo("No banana.");
```

There's also an `.indexOf` shortcut — guaranteed available in our environment via polyfill — that you'll start seeing in Lesson 2.4. Knowing the manual loop pattern is still valuable for understanding what's happening under the hood.

### Exercises

1. **Print a list.** Create an array of 5 names. Use a labeled `for` loop to print each name on its own line.
2. **Sum the numbers.** Create an array of 6 numbers. Use a labeled `for` loop to compute and print their sum.
3. **Find the max.** Write a function `maxOf(arr)` (with header comment) that takes an array of numbers and returns the largest. Test it with `[3, 1, 4, 1, 5, 9, 2, 6]` (expected: 9).
4. **Count occurrences.** Write a function `countOf(arr, target)` that returns how many times `target` appears in `arr`. Test it with `["a", "b", "a", "c", "a"]` and target `"a"` (expected: 3).
5. **Reverse without `.reverse()`.** Write a function `reversed(arr)` that returns a *new* array with the elements in reverse order. Don't use the built-in `.reverse()`; build it yourself with a `for` loop and `.push`. Test with `[1, 2, 3, 4]` (expected: `[4, 3, 2, 1]`).

---

## Lesson 2.4: String methods

You already know how to make and concatenate strings. JScript provides a rich set of **methods** — built-in functions attached to every string — for working with text.

### `.length`

Like arrays, strings have a `.length`:

```js
var name = "Sam";
WScript.Echo(name.length); // 3
```

Counts characters, not bytes. Spaces and punctuation count: `"Hello!"` has length 6.

### Reading individual characters

`.charAt(i)` returns the character at position `i` (zero-indexed), or an empty string if `i` is out of range:

```js
var word = "hello";

WScript.Echo(word.charAt(0));  // "h"
WScript.Echo(word.charAt(4));  // "o"
WScript.Echo(word.charAt(99)); // "" (empty string)
```

### Searching

`.indexOf(needle)` returns the position of the first occurrence of `needle`, or `-1` if not found:

```js
var sentence = "the quick brown fox";

WScript.Echo(sentence.indexOf("quick")); // 4
WScript.Echo(sentence.indexOf("slow"));  // -1
WScript.Echo(sentence.indexOf("o"));     // 12 (first 'o', in 'brown')
```

`.lastIndexOf(needle)` does the same but starts from the end:

```js
WScript.Echo(sentence.lastIndexOf("o")); // 17 (the 'o' in 'fox')
```

Common pattern for "does this string contain X":

```js
if(sentence.indexOf("fox") !== -1)
    WScript.Echo("Yes, there's a fox.");
```

### Extracting substrings

`.slice(start, end)` returns the portion of the string from `start` (inclusive) to `end` (exclusive):

```js
var s = "JavaScript";

WScript.Echo(s.slice(0, 4)); // "Java"
WScript.Echo(s.slice(4));    // "Script"   (no end → goes to the end)
WScript.Echo(s.slice(-6));   // "Script"   (negative = count from end)
```

`.substring(start, end)` exists too but treats negative arguments as `0`. **Use `.slice`** in our codebase — it's more predictable.

### Case conversion

```js
WScript.Echo("hello".toUpperCase()); // "HELLO"
WScript.Echo("HELLO".toLowerCase()); // "hello"
```

These return *new* strings; the original isn't changed. Strings in JavaScript are **immutable** — you can never modify the characters of an existing string, only produce new ones from it.

### Splitting

`.split(separator)` chops a string into an array, breaking at every occurrence of `separator`:

```js
var csv = "apple,banana,cherry";
var parts = csv.split(",");
WScript.Echo(parts.length); // 3
WScript.Echo(parts[1]);     // "banana"
```

A common idiom — split on `"\n"` to break a multi-line string into lines:

```js
var doc = "line one\nline two\nline three";
var lines = doc.split("\n");
WScript.Echo(lines.length); // 3
```

### Replacing

`.replace(search, replacement)` returns a new string with the **first** occurrence of `search` replaced:

```js
var s = "Hello, world! Hello, again!";
WScript.Echo(s.replace("Hello", "Goodbye"));
// "Goodbye, world! Hello, again!"
```

To replace **all** occurrences, you'd use a regular expression with the `g` flag — that's Tier 3 material. As a quick workaround, `split` then `join` (an array method that concatenates with a separator):

```js
var s = "a-b-c-d";
WScript.Echo(s.split("-").join(",")); // "a,b,c,d"
```

### Trimming

`.trim()` removes whitespace from both ends of a string:

```js
var input = "   hello world   ";
WScript.Echo("[" + input.trim() + "]"); // "[hello world]"
```

Inner whitespace is preserved. Useful when parsing user input or file content with unpredictable spacing.

### Escape sequences

Some characters need special syntax inside a string literal:

| Sequence | Meaning |
|----------|---------|
| `\n`     | Newline |
| `\t`     | Tab |
| `\\`     | Literal backslash |
| `\"`     | Literal double quote (in a double-quoted string) |
| `\'`     | Literal single quote (in a single-quoted string) |

```js
WScript.Echo("Line one\nLine two");
// Line one
// Line two

WScript.Echo("She said, \"hi\".");
// She said, "hi".

WScript.Echo("C:\\Users\\Admin");
// C:\Users\Admin
```

Forgetting to escape backslashes in Windows paths is the most common mistake. `"C:\Users\new"` would try to interpret `\U` and `\n` as escape sequences (one of which doesn't exist, the other becomes a newline). Either escape with `"C:\\Users\\new"` or — when the API allows it — use forward slashes, which Windows usually accepts: `"C:/Users/new"`.

### Exercises

1. **Anatomy of a sentence.** Given `var sentence = "The quick brown fox jumps over the lazy dog";`, use string methods to print:
   - the total length,
   - the index of `"fox"`,
   - the substring from position 4 to position 9 (should be `"quick"`),
   - the whole sentence in uppercase.
2. **Word count.** Write a function `wordCount(s)` (with header comment) that returns the number of words in a string. Assume words are separated by single spaces. Hint: `.split(" ").length`.
3. **Reverse a string.** Write a function `reverseString(s)` that returns the input reversed. Hint: loop from the end and build a new string.
4. **Strip a prefix.** Write a function `stripPrefix(s, prefix)` that returns `s` with `prefix` removed from the start *if* it appears there; otherwise returns `s` unchanged. Example: `stripPrefix("xline.js", "x")` → `"line.js"`; `stripPrefix("hello", "x")` → `"hello"`.
5. **Path tail.** Write a function `lastPathSegment(path)` that returns the part after the last `\` in a Windows-style path. Example: `lastPathSegment("C:\\Users\\Admin\\file.txt")` → `"file.txt"`. Hint: `.lastIndexOf("\\")` and `.slice`.

---

## Lesson 2.5: Math and number conversion

JScript provides a `Math` object with number-related functions, plus a few global functions for converting between strings and numbers.

### Common `Math` methods

```js
WScript.Echo(Math.abs(-5));      // 5
WScript.Echo(Math.floor(3.7));   // 3   (round down)
WScript.Echo(Math.ceil(3.2));    // 4   (round up)
WScript.Echo(Math.round(3.5));   // 4   (round to nearest)
WScript.Echo(Math.min(1, 2, 3)); // 1
WScript.Echo(Math.max(1, 2, 3)); // 3
WScript.Echo(Math.sqrt(16));     // 4
WScript.Echo(Math.pow(2, 10));   // 1024  (2^10)
```

`Math.min` and `Math.max` accept any number of arguments. They don't take an array directly — passing `[1, 2, 3]` won't work the way you'd hope. Feeding an array into a variadic function requires the spread operator, which is Tier 3 material.

### Constants

```js
WScript.Echo(Math.PI); // 3.141592653589793
WScript.Echo(Math.E);  // 2.718281828459045
```

### Random numbers

`Math.random()` returns a random decimal between `0` (inclusive) and `1` (exclusive):

```js
WScript.Echo(Math.random()); // e.g., 0.4571092837
```

To get a random integer between `min` and `max` (inclusive):

```js
/*
 * randomInt
 * Returns a random integer in [min, max], inclusive on both ends.
 *
 * Input:  min (number), max (number)
 * Output: integer (number)
 */
function randomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

WScript.Echo(randomInt(1, 6)); // a die roll
```

Memorize that formula — random ranges come up often.

### Converting strings to numbers

JScript has three ways to turn a string into a number:

```js
WScript.Echo(parseInt("42"));     // 42
WScript.Echo(parseInt("42.7"));   // 42 (truncates)
WScript.Echo(parseFloat("42.7")); // 42.7
WScript.Echo(Number("42"));       // 42
WScript.Echo(Number("42.7"));     // 42.7
```

The differences matter:

- `parseInt(s)` reads digits from the start and stops at the first non-digit. `parseInt("42abc")` is `42`; `parseInt("abc42")` is `NaN`.
- `parseFloat(s)` works the same way but preserves a decimal point.
- `Number(s)` is **strict**: `Number("42abc")` is `NaN`, not `42`. The whole string must be a valid number.

**Important:** always pass a second argument to `parseInt` — the **radix** (number base):

```js
WScript.Echo(parseInt("42", 10)); // 42, definitely base 10
```

Without the radix, JScript guesses based on the string. `parseInt("0x10")` is `16` (hexadecimal). `parseInt("010")` *might* be `8` (octal) on older engines. Pass `10` and the ambiguity goes away.

### `NaN` — "Not a Number"

When a number conversion fails, the result is `NaN`:

```js
WScript.Echo(Number("hello")); // NaN
WScript.Echo(0 / 0);           // NaN
```

`NaN` has a frustrating property: it's not equal to anything, *including itself*.

```js
NaN === NaN; // false (!)
```

So you can't check for `NaN` with `===`. Use the global `isNaN` function instead:

```js
var n = Number("hello");

if(isNaN(n))
    WScript.Echo("Conversion failed");
```

### Converting numbers to strings

Most of the time, concatenation handles it: `"" + 42` becomes `"42"`. There's also `String(n)` and `n.toString()`:

```js
WScript.Echo(String(42));         // "42"
WScript.Echo((42).toString());    // "42"
WScript.Echo((255).toString(16)); // "ff"  (hex representation)
```

The parentheses around `(42)` and `(255)` are needed because `.toString` directly on a number literal would be ambiguous to the parser. `.toString(radix)` is occasionally useful for hex/binary output.

### Exercises

1. **Round trip.** Use `parseInt` to convert `"123"` to a number, add `7`, then use `String()` (or concatenation) to convert it back to a string. Echo each intermediate value.
2. **Validate a number.** Write a function `isValidNumber(s)` (with header comment) that returns `true` if `s` represents a valid number, `false` otherwise. Use `Number()` and `isNaN()`. Test with `"42"`, `"3.14"`, `"hello"`, and `""`. (Watch out: `Number("")` is `0`, not `NaN` — decide whether you want to count that as valid.)
3. **Hex viewer.** Write a function `toHex(n)` that returns the hexadecimal representation of a non-negative integer. Example: `toHex(255)` → `"ff"`, `toHex(4096)` → `"1000"`.
4. **Roll some dice.** Write a function `rollDice(sides)` that returns a random integer between `1` and `sides`. Use a labeled `for` loop to print 10 rolls of a 20-sided die.

---

## Lesson 2.6: Error handling — `try` / `catch`

When something goes wrong at runtime — a function gets bad input, a file doesn't exist, a number conversion fails — JScript "throws" an **error**. By default, an unhandled error halts the script and prints a message. Often you want to handle it instead.

### The basic structure

```js
try {
    // code that might fail
} catch(e) {
    // runs only if code in `try` throws
    WScript.Echo("Something went wrong: " + e.message);
}
```

Read it as: *"try this block; if it throws, jump to the catch with the error captured as `e`."*

A concrete example:

```js
try {
    var n = parseInt("not a number", 10);

    if(isNaN(n))
        throw new Error("Could not parse number");

    WScript.Echo("Got: " + n);
} catch(e) {
    WScript.Echo("Failed: " + e.message);
}
```

### The `Error` object

The variable in `catch(e)` (you can name it anything; `e` and `err` are common) is an **Error object** with two properties you'll use most:

- `e.message` — the description string passed to `new Error(...)`
- `e.name` — the type of error (e.g., `"TypeError"`, `"ReferenceError"`)

```js
try {
    var x = undefinedVariable;
} catch(e) {
    WScript.Echo(e.name);    // "ReferenceError"
    WScript.Echo(e.message); // "'undefinedVariable' is undefined"
}
```

### Throwing your own errors

`throw` raises an error from your code. By convention, throw an `Error`:

```js
function divide(a, b) {
    if(b === 0)
        throw new Error("Division by zero");

    return a / b;
}

try {
    var result = divide(10, 0);
} catch(e) {
    WScript.Echo("Caught: " + e.message);
}
```

The style guide has two rules worth knowing now:

1. **Transformation functions** (the `transformX` functions in our transpilers) must **never** throw. They return their input unchanged on failure. Other code may throw normally.
2. Error messages are constructed inline in the `throw`, not assigned to a variable first. For multi-line messages, build with `Array.join`:

```js
throw new Error([
    "Cannot process input.",
    "Expected a positive number; got: " + value,
    "Use Math.abs() to convert."
].join("\n"));
```

### `finally`

A third clause runs whether or not an error was thrown:

```js
try {
    // ...
} catch(e) {
    // ...
} finally {
    WScript.Echo("This always runs");
}
```

Used for cleanup (closing files, releasing resources). You'll see it occasionally in the codebase.

### When to catch — and when not to

The instinct as a beginner is to wrap everything in `try/catch` to "be safe." Resist it. A `try/catch` swallows information about what went wrong; if you catch every error and just log it, you've made the script harder to debug, not easier. Catch when:

- You can do something *meaningful* with the error — retry, fall back to a default, give the user a friendly message.
- You're at a boundary where errors should be reported rather than crashing — typically the top of a script.

Otherwise, let the error propagate. A loud crash is easier to fix than a silent failure.

### Exercises

1. **Catch and report.** Write a script that calls `divide(10, 0)` (the function above) inside a `try/catch` and prints the error message.
2. **Validate input.** Write a function `requirePositive(n)` (with header comment) that throws an `Error` if `n` is not a positive number, and returns `n` otherwise. Test inside a `try/catch` with both a valid value (`5`) and an invalid one (`-3`).
3. **Inspect the error.** Write a script that intentionally references an undefined variable inside a `try`. In the `catch`, print the error's `.name`, `.message`, and the result of `typeof e`.
4. **Multi-line error.** Write a function `parseAge(s)` that converts a string to a non-negative integer, throwing a multi-line error (use the `Array.join` pattern) if the input is invalid. Test with `"30"`, `"-5"`, and `"hello"`.

---

## Lesson 2.7: Reading script arguments

Most of our utility scripts take inputs from the command line — a filename, a flag, a number. JScript exposes these through `WScript.Arguments`.

### The `WScript.Arguments` collection

When you run `cscript myScript.js foo bar baz`, JScript exposes `"foo"`, `"bar"`, `"baz"` to your script via `WScript.Arguments`. It's a *collection* (a COM object), not a regular array, so the access pattern is slightly different:

```js
// arglist.js
WScript.Echo("Total arguments: " + WScript.Arguments.length);
WScript.Echo("First argument:  " + WScript.Arguments.Item(0));
WScript.Echo("Second argument: " + WScript.Arguments.Item(1));
```

Run as:

```
cscript arglist.js apple banana
```

Output:

```
Total arguments: 2
First argument:  apple
Second argument: banana
```

Two differences from a regular array:

- **Read with `.Item(i)`** — capital `I`, not `[i]`. The shortcut `WScript.Arguments(i)` (no `.Item`) also works, but `.Item(i)` is the explicit form and what you'll see in our code.
- **`.length` is a property**, just like an array. Loop with the same pattern.

### The standard iteration pattern

```js
read_args: for(var i = 0; i < WScript.Arguments.length; ++i)
    WScript.Echo("[" + i + "] " + WScript.Arguments.Item(i));
```

Run with `cscript script.js one two three` and you get:

```
[0] one
[1] two
[2] three
```

### A practical example

A script that greets each name passed in:

```js
// greet.js
if(WScript.Arguments.length === 0) {
    WScript.Echo("Usage: cscript greet.js <name1> <name2> ...");
    WScript.Quit(1);
}

greet_each: for(var i = 0; i < WScript.Arguments.length; ++i)
    WScript.Echo("Hello, " + WScript.Arguments.Item(i) + "!");
```

`WScript.Quit(n)` ends the script immediately with exit code `n`. By convention, `0` means success and any non-zero code means failure — useful for batch scripts that check whether your script succeeded.

### Named vs unnamed arguments

WSH actually splits arguments into two collections — `WScript.Arguments.Named` (for `/key:value` style) and `WScript.Arguments.Unnamed` (positional). Most of our scripts use only positional, so `.Item(i)` suffices for now. You'll meet the named flavor in Tier 3 when we cover argument parsing patterns properly.

### Exercises

1. **Echo each.** Write `echoEach.js` that prints each command-line argument on its own line, prefixed with its index.
2. **Sum numbers.** Write `sumArgs.js` that takes any number of numeric arguments, sums them, and prints the total. Use `parseFloat` and check for `NaN`. Example: `cscript sumArgs.js 3 4 5.5` should print `12.5`.
3. **Required argument.** Write `requireOne.js` that expects exactly one argument. If zero or more than one are given, print a usage message and `WScript.Quit(1)`. If exactly one is given, print it.
4. **Search args.** Write `findArg.js` that takes a target as the first argument and any number of additional arguments. It should print whether the target appears among the rest, and at which positions. Example: `cscript findArg.js apple banana apple cherry apple` → `Found "apple" at positions: 1, 3`.

---

## Lesson 2.8: Introducing `baseline`

You've been learning JScript-as-the-engine-runs-it: `var`, `function`, ES3 syntax. From this lesson on, the codebase you'll be reading uses **modern JavaScript** — `let`, `const`, arrow functions, template literals, destructuring, and more.

How? A tool called `baseline` translates the modern source code into the ES3 code that WSH actually runs. We write what's pleasant to write; the tool produces what the engine can run.

### What baseline does

You write this:

```js
const greet = (name) => `Hello, ${name}!`;
WScript.Echo(greet("Sam"));
```

Baseline turns it into something like this:

```js
var greet = function(name) { return "Hello, " + name + "!"; };
WScript.Echo(greet("Sam"));
```

Same behavior, different syntax. Your job as a maintainer is increasingly to read and modify the *modern* form, knowing the engine eats the converted output.

### Features you'll start using right away

The most common modern features in our codebase, in roughly the order they'll bite you:

#### `const` and `let` instead of `var`

```js
const PI = 3.14159; // never reassigned — use const
let counter = 0;    // will be reassigned — use let
counter = counter + 1;
```

Style guide rule: **`const` for stable bindings, `let` for ones that change.** Reach for `var` only in pure-output JScript files that bypass baseline.

There's also a meaningful behavior difference from `var`: `let` and `const` are **block-scoped**, not function-scoped. A `let` inside a `for` block doesn't leak out:

```js
for(let i = 0; i < 3; ++i) {
    // ...
}

WScript.Echo(i); // ReferenceError — i doesn't exist out here
```

That's the bug-class fix from Lesson 2.2.

#### Arrow functions

Compact function syntax, especially for short helpers:

```js
const square = (n) => n * n;
const greet  = (name) => "Hello, " + name + "!";

WScript.Echo(square(5));    // 25
WScript.Echo(greet("Sam")); // Hello, Sam!
```

Arrow functions have other behaviors that matter (around `this` in particular), but those land in Tier 3.

#### Template literals (backtick strings)

Strings with embedded expressions, written between backticks:

```js
const name = "Sam";
const age  = 34;

WScript.Echo(`Hello, ${name}. You are ${age} years old.`);
// Hello, Sam. You are 34 years old.
```

`${expression}` interpolates the value. Backtick strings can also span multiple lines without `\n`:

```js
const block = `Line one
Line two
Line three`;
```

In the codebase, you'll see template literals everywhere there's any kind of formatted string or generated code.

#### Default parameters

```js
function greet(name = "world") {
    WScript.Echo(`Hello, ${name}!`);
}

greet();      // Hello, world!
greet("Sam"); // Hello, Sam!
```

If the argument is omitted (or explicitly undefined), the default kicks in.

#### `void null` instead of `undefined`

A small but consistent rule: in baseline source, never write the bare word `undefined`. Use `void null` instead.

```js
let value = void null;             // declared as undefined
if(x === void null) { /* ... */ }  // explicit "is undefined" check
```

The reason: in older JavaScript, `undefined` was a *variable* that could be reassigned, which led to genuine bugs. `void null` is an expression that always produces the undefined value — defensively bulletproof. The convention persists in our code; you'll see it everywhere.

### The full list (preview)

Baseline supports most of modern JavaScript. The complete inventory we'll work through in Tier 3:

- `let`, `const`, arrow functions, template literals, default parameters, `void null` (above)
- Destructuring (assignment and reassignment)
- Spread (`...`) in arrays, objects, and function arguments
- Rest parameters
- Classes (transformed under the hood)
- Optional chaining (`?.`) and nullish coalescing (`??`, `??=`)
- `**` (exponentiation) and its assignment form
- Generators (`function*`, `yield`, `yield*`)
- `async` / `await`
- `import` / `export` (modules)
- Empty `catch` blocks (`catch { ... }`)
- `BigInt` and `Symbol` (as polyfilled packages)
- The pipeline operator (`|>`) — non-standard, but supported

You don't need to absorb that list now; just know the breadth is there. We'll work through them with proper context in Tier 3.

### What baseline doesn't do

A few honest limitations:

- Baseline transforms **syntax**. Runtime methods that don't exist in the target environment may need to be polyfilled separately (we covered this with `.indexOf` earlier).
- Some advanced ES features (deep `Reflect`, full `Proxy`) aren't covered. Tier 4 has the complete inventory.
- The transformed output is `var`-based ES3. If you're debugging the actual code the engine sees, you'll be reading `var` everywhere even though you wrote `const`.

### From this point forward

In Tier 3 onward, examples in this tutorial — and most of the code you read — will use modern syntax. The fundamentals you've learned still apply: a `const` is a `var` that's promised not to change. An arrow function is a function with a tighter syntax. Template literals are `+` concatenation in disguise. The mental model carries over.

### Exercises

1. **`var` to `const`/`let`.** Take any script you wrote in this tier (the FizzBuzz function is a good one) and rewrite it using `const` for things that don't change and `let` for things that do. Behavior should be identical.
2. **Template literal makeover.** Take any `Echo` from your earlier exercises that used `+` concatenation and rewrite it as a template literal. Compare readability.
3. **Arrow conversion.** Convert the `square`, `add`, and `isEven` functions from Lesson 2.2 into arrow form. Note: you can omit the braces and `return` for single-expression bodies — `const square = (n) => n * n;`.
4. **Defaults.** Rewrite `greet(name)` from Lesson 2.2 with a default value of `"stranger"`. Test calling it both with and without an argument.

---

## Tier 2 wrap-up

You can now write scripts that:

- Repeat work with loops
- Bundle logic into reusable, documented functions
- Handle ordered lists with arrays
- Manipulate text with string methods
- Do math and convert between strings and numbers
- React to errors without crashing
- Read arguments from the command line
- Use modern syntax (via baseline) for cleaner code

Most utility scripts in our codebase are built from exactly these pieces.

### What's next

In **Tier 3: Intermediate** we'll cover:

- Objects — key/value structures, the foundation of larger code
- The `FileSystemObject` for reading and writing files
- `WScript.Shell` for running commands and reading the environment
- Date and time
- Regular expressions
- Modular code — splitting a script across files via `import`/`export`
- The full baseline feature inventory in working examples
- Argument parsing patterns (named vs positional)

### Capstone exercise: text stats

Write a script `textStats.js` that takes a single string as a command-line argument and prints:

1. The total character count.
2. The character count excluding spaces.
3. The word count (split on spaces).
4. The number of vowels (a/e/i/o/u, case-insensitive).
5. The string reversed.

Requirements:

- Handle the case of zero arguments by printing a usage message and quitting with code `1`.
- Wrap the main work in a `try/catch` at the top level. If anything throws, print a clean error message and quit with code `2`.
- Define at least two helper functions, each with a proper header comment.
- Use labeled `for` loops for any iteration.
- Use double quotes for user-facing strings and follow the style conventions you've learned.

Example:

```
> cscript textStats.js "The quick brown fox"
Total characters:  19
Without spaces:    16
Word count:        4
Vowel count:       5
Reversed:          xof nworb kciuq ehT
```

When you can write this confidently, Tier 2 is complete. (You may keep the script in `var`/ES3 form, or experiment with the modern syntax from Lesson 2.8 — whichever feels right.)
