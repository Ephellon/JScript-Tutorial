# JScript at Work — Tier 4: Confident

Welcome to Tier 4. The shift from Tier 3 to Tier 4 is the shift from *writing* to *reading*. You can write a complete script. Now you need to read, understand, debug, and modify scripts that someone else wrote — possibly years ago, possibly under deadline, possibly without comments to spare.

What we'll cover:

- **Reading other people's JScript** — strategies for getting up to speed on unfamiliar code
- **The AutoIATE codebase** — file inventory, loading order, where to find what
- **Reading ES3 patterns** — recognizing how the codebase does what you learned to do in modern syntax
- **Debugging without a debugger** — print debugging, logging, bisecting, trace-by-hand
- **Production-code gotchas** — bugs you'll find in real code
- **The project journal** — what it is, how to use it, when to add to it

There's no traditional capstone for Tier 4. Instead, the final exercise is a maintenance task: take a real script from the codebase, find a real (or planted) bug, and fix it using the techniques you've learned. The goal is *documented self-sufficiency* — knowing where to look, who to ask, and how to verify a fix.

---

## Lesson 4.1: How to read JScript you didn't write

> **New words in this lesson**
>
> - **top-down** — read from the entry point down through the functions it calls
> - **bottom-up** — start from a symptom and trace back to its cause
> - **heuristic** — a quick mental rule of thumb, not a hard rule
> - **trace by hand** — walk through code on paper without running it
> - **ESX** — our shorthand for "modern JavaScript" (post-ES2015)
> - **`imports()`** — the lowercase polyfill function that loads ESX modules at runtime
> - **`debug`** — our project's mini-terminal for runtime inspection

You can write a complete utility script. That's a real skill. But reading code you didn't write is *harder*, not easier — and it's most of what you'll do as a maintainer.

### Why reading is harder than writing

When you write a script, you start with intent and produce code. When you read a script, you start with code and have to *infer* intent. The difference is:

- The author knew the names of variables. You have to learn them.
- The author knew the data shapes. You have to discover them.
- The author knew the control flow. You have to trace it.
- The author knew the bugs they were avoiding. You don't see those.

Plan for reading to take longer than you think. Don't beat yourself up when it does.

### Top-down strategy

For a single-script program, start at the bottom of the file and work up. The bottom usually has the entry point — the `main()` call or the top-level `try/catch` or the actual work. Read it first. Then trace into the functions it calls.

For a multi-file project, start at the entry-point file. In our codebase, that's `AutoIATE*` for the main program. Open it, find where execution actually starts (after the imports and preamble), and follow from there.

Don't read every file top-to-bottom. Read the entry point, then trace into the named functions when their behavior matters. Most files have plenty of code you'll never need to look at.

### Bottom-up strategy

When you're trying to understand *one specific behavior* — a bug, a feature you want to extend — start from the symptom and work backwards. The symptom might be:

- An error message → search the codebase for the literal text. Find the `throw new Error("...")` that produced it.
- A piece of output → search for the format string. Find the `Echo`/`WriteLine` that produced it.
- A file written somewhere → search for the path or the `OpenTextFile` call.

Once you've found the producing code, work outward: who calls this function? What state did they pass in? Where did *that* state come from?

This is the most useful strategy for bugs. The forward "trace from main" strategy is for understanding what something *does*; the backward "trace from symptom" strategy is for finding where it *broke*.

### Following imports

When a file imports from another, the import line tells you exactly what's coming in:

```js
import { formatDate, parseArgs } from "./helpers.js";
```

You know `formatDate` and `parseArgs` are both defined in `helpers.js`. To learn what they do, jump there. Don't assume from the name — names lie. Read at least the function header.

For namespace imports (`import * as helpers from "./helpers.js"`), you'll need to skim the exporting file's `export` statements to see what's in scope.

### Style-guide-conforming code as a quick filter

Style is signal. Code that follows our style guide tells you something about its quality:

- **Labeled loops** mean the author thought about which loop they were breaking out of.
- **Header comments** mean the author cared enough to document intent.
- **`Object.create(null)` for lookup tables** means the author understood the prototype gotcha.
- **`const` for stable bindings** means the author thought about mutability.

Conversely, code that *violates* the style guide — bare `break`, missing headers, `==` instead of `===`, magic numbers — is either old, hasty, or both. **Read it more carefully.** It's likelier to contain bugs.

This isn't a judgment about the original author. It's a heuristic about *attention*. When you see consistent style, you can read at speed and trust patterns. When you see inconsistent style, slow down.

### Most files are ES3, some are ESX

The bulk of the codebase is hand-written ES3 — `var`, `function`, prototype-based patterns, and the like. These files are the source *and* what the engine runs. There's no transformation step in between.

A handful of specific modules use modern syntax (ESX) and are routed through `baseline` at runtime via the `imports()` polyfill (lowercase, since `import` is reserved). The `debug` mini-terminal also runs through `baseline`. When you see modern syntax — `let`, `const`, arrow functions, template literals — in a project file, you're looking at one of those ESX modules or `debug`. Otherwise, you're looking at ES3 directly.

When tracking down a runtime error, line numbers point at the actual file you're reading. No source-vs-output mapping required.

### Reading without running

In our environment you can usually run the script to see what it does. But sometimes you can't — the script needs production data, or it has side effects, or it's part of a larger pipeline. In those cases you have to read by *trace-by-hand*: pick an example input, walk through the code, and predict what each statement does to your mental model of the state.

This sounds tedious but it's the most reliable way to understand control flow. It's also how you find subtle bugs — the kind that only fire on certain inputs.

A practical aid: keep a notepad open. As you trace, write down the values of variables at each step. Don't try to keep it all in your head.

### Search techniques

In an air-gapped environment, your dev box's filesystem search and grep tools are your best friends.

- **Search for literal strings.** Error messages, file paths, magic constants — all easier to find by their literal text than by name.
- **Search for symbol names.** Function names, class names, variable names. The first hit is usually the definition; later hits are uses.
- **Search for unusual patterns.** A weird-looking expression you don't understand might appear elsewhere in the codebase. Other instances may be commented or paired with surrounding context that explains it.
- **Search the journal.** Most surprising behavior has been documented before.

### Picking your battles

Some code can't be understood quickly. When you've spent an hour on a piece of code and you still don't get it, the right move is usually to:

1. **Document what you've learned.** Even partial understanding has value.
2. **Ask.** Don't burn a day silently.
3. **Limit your change.** If you have to modify code you don't fully understand, make the *smallest possible* change and verify it doesn't break anything obvious.

Confident maintenance isn't about understanding everything. It's about knowing what you understand, what you don't, and how to verify your changes regardless.

### Exercises

1. **Trace a script.** Pick a script from the codebase you haven't seen before. Without running it, write a one-paragraph description of what you think it does. Then run it (with safe inputs) and compare. How close were you?
2. **Reverse a search.** Find an error message that the codebase produces (look at any user-facing error). Search the codebase for the literal text. Find the file and line that throws it. From there, work outward: in what circumstances is the error thrown?
3. **Read a function header.** Pick any function in the codebase with a proper header comment. Without reading the body, write down what you expect the function to do based on the header alone. Then read the body. Did the comment match?
4. **Identify ES3 vs ESX.** Find one file that's clearly ES3 and one that's clearly ESX (modern syntax). Note the visual differences. What's the cleanest way to tell them apart at a glance?

---

## Lesson 4.2: The AutoIATE codebase — a tour

> **New words in this lesson**
>
> - **preamble** — a file that loads other files at startup
> - **polyfill** — code that adds missing features so an older runtime behaves like a newer one
> - **entry point** — the file the user actually runs to start the program
> - **mojibake** — garbled text caused by encoding mishaps (think `â€™` instead of `'`)
> - **AUTO** — our domain-specific language for writing automation scripts (separate from JScript)
> - **domain-specific language** — a small language built for a specific purpose
> - **transpiler** — a tool that translates code from one language form to another (`baseline` for ESX, `parseAuto` for AUTO)

This lesson is a map. By the end you should know what each file is for and roughly which file to open when you're chasing down a specific kind of behavior.

### The big picture

AutoIATE automates ATLAS systems — running tests, capturing output, handling data. It's structured in layers:

1. **Polyfills** at the bottom — things that ought to be in JScript but aren't.
2. **Tools and utilities** above the polyfills — JSON, beautifier, transpilers, formatting.
3. **Core logic** — non-polyfill program logic for loading tests, gathering input, generating UI.
4. **Automation logic** — the side that actually runs automation against ATLAS.
5. **Entry point** — `AutoIATE*`, where users start.

Files load roughly bottom-up: polyfills first, so the rest of the code can rely on them.

### File inventory

Each file has a defined role.

#### Preambles (loaders)

- **`auto`** — Preamble for automation modules. Loads everything below unless a script explicitly opts out.
- **`core`** — Preamble for main program logic. Loads everything below unless opted out, and **also sets up import logic and globals**. The shared globals you've already met (`WINDOW`, `Array.enumerate` injection, `Symbol`, `BigInt`) get wired up here.

`auto` and `core` are alternative starting preambles depending on what kind of script you're writing. Automation scripts use `auto`; main program code uses `core`. When you see a file that doesn't seem to import anything but uses things like `WINDOW` or `Array.enumerate` — it's because the preamble has already pulled them in.

#### Logic layers

- **`sub_auto`** — Automation-specific logic. Code that runs against ATLAS systems.
- **`sub_core`** — All non-polyfilled logic: loading test names, generating UI/UX, gathering input, etc. The "main program logic that isn't a polyfill."

#### Polyfills

- **`polyfills`** — The catch-all polyfill file. `Array.from`, `String.trim`, `Promise`, the various other ES5+/ES2015+ runtime methods that vanilla JScript doesn't provide. `Symbol` and `BigInt` are *not* here — they live in their own files.
- **`symbol`** — The `Symbol` polyfill, in its own file because it's substantial and self-contained.
- **`bigint`** — The `BigInt` polyfill, similarly.
- **`json3`** — A `JSON` polyfill (vanilla JScript's `JSON` support is patchy).
- **`jsonx`** — An add-on for `json3` that allows comments in JSON content. Useful for config files where you want both readability and machine-parseability.
- **`unorm`** — Unicode normalization polyfill. Handles **mojibake** (garbled text from encoding mishaps) and other Unicode edge cases. Important for any input that might have non-ASCII content.
- **`unormdata`** — JSON data file consumed by `unorm`. Pure data, no logic.

#### Tools

- **`baseline`** — The ESX transpiler. Translates modern JavaScript source to ES3 output that WSH can run. Used at runtime when an ESX module is loaded via `imports()`, and inside the `debug` mini-terminal.
- **`parseAuto`** — Handles the **AUTO** language: a domain-specific automation language used in our scripts. `parseAuto` is essentially a transpiler from AUTO to JScript, parallel to how `baseline` converts modern JS to ES3.
- **`beautify`** — A JS beautifier. Useful for inspecting transpiler output (since the output is technically valid but not human-friendly).
- **`figlet`** — Generates large ASCII text. Used for CLI banners and visible separators in output.

#### Entry point

- **`AutoIATE*`** — The main UI/UX entry point. This is the file users actually run when they invoke the program. The `*` covers variants (versions, configurations).

### Loading order in practice

When you run `AutoIATE.js`:

1. **`core`** is pulled in first (or `auto` if it's an automation context). This brings in globals and the import system.
2. **Polyfills** load next: `polyfills`, `symbol`, `bigint`, `json3`, `unorm` (which itself uses `unormdata`).
3. **Tools** load if they're needed: `baseline` if dynamic transpilation happens at runtime, `parseAuto` for AUTO input, `beautify` for debugging output, etc.
4. **Logic** layers load: `sub_core`, then `sub_auto` if relevant.
5. **`AutoIATE*`** runs as the entry point, calling into the loaded modules.

If you're tracking down "why doesn't this thing exist when my script runs?", the answer is usually load order — your code is running before its dependency was loaded. The fix is usually to import what you need explicitly, even if a preamble would have loaded it eventually.

### Where to look for things

A rough rule of thumb for "I need to find or modify X":

| If you're looking for... | Check |
|---|---|
| A polyfill or override of a built-in | `polyfills` (or `symbol` / `bigint` for those specifically) |
| Test loading, UI generation, input gathering | `sub_core` |
| Automation-specific logic | `sub_auto` |
| Shared constants or globals (`WINDOW`, etc.) | `core` |
| The user-facing entry point | `AutoIATE*` |
| Something AUTO-language related | `parseAuto` |
| Something baseline-transform related | `baseline` |
| Something Unicode/encoding related | `unorm` (data lives in `unormdata`) |
| Something formatting JSON | `json3`, or `jsonx` for commented JSON |
| Something formatting JS | `beautify` |
| ASCII art / banners | `figlet` |

### A note on AUTO

AUTO is its own language with its own syntax. If you're reading `parseAuto`'s source, or actual AUTO scripts, you're in *that* language's territory, not JScript's. The patterns and idioms are different.

For Tier 4, you don't need to learn AUTO. You need to recognize when you're looking at AUTO source vs `parseAuto`'s JScript implementation, and know that `parseAuto` itself is regular JScript that follows the same conventions as everything else.

If you ever need to modify AUTO behavior, that's a separate study. For now, treat `parseAuto` as a black box that converts AUTO to JScript, parallel to how `baseline` converts modern JS to ES3.

### Exercises

1. **File mapping.** Without opening the files, write down which file you'd look in for: (a) a fix to `Array.from`, (b) the `WINDOW` enum definition, (c) the entry-point `main()` call, (d) the JSON parser used to read config, (e) the AUTO-to-JScript transformation.
2. **Open and skim.** Pick one file from each layer (polyfill, tool, logic, entry) and skim its top 30 lines. What do the headers and imports tell you about its role? How does this match the inventory above?
3. **Find the globals.** Locate `core` and find where it defines or assigns globals. Write down three globals it sets up that you'd want to know exist.
4. **Trace an import.** Pick any non-trivial function in `sub_core` or `sub_auto`. Trace its imports back to their definitions. How many files away is the deepest dependency?

---

## Lesson 4.3: Reading ES3 patterns

> **New words in this lesson**
>
> - **prototype** — a shared object that holds the methods for all instances of a "class" (the ES3 way of doing inheritance)
> - **constructor function** — a regular function used with `new` to create objects (the ES3 "class")
> - **IIFE** — Immediately Invoked Function Expression: `(function(){...})()`, a function called the moment it's defined
> - **revealing module pattern** — an IIFE that returns an object exposing some "private" inner pieces; an ES3 module
> - **factory function** — a function that creates and returns an object (alternative to a constructor)
> - **`apply` / `call`** — methods for invoking a function with a specific `this` and a specific list of arguments
> - **`arguments`** — an automatic variable inside every function that holds all the arguments passed in
> - **bookkeeping** — small assignments that aren't the main point but are needed for correctness

You spent Tier 3 learning modern JavaScript syntax. Most files in the codebase don't use it — they're ES3, with idioms that pre-date `let`, `class`, arrow functions, and template literals. To read those files fluently, you need to recognize the ES3 way of doing the same things.

This lesson is structured as a translation guide: modern feature on the left, ES3 equivalent on the right.

### Constructor functions and prototypes (the ES3 "class")

Modern:

```js
class Counter {
    constructor() {
        this.count = 0;
    }

    increment() {
        ++this.count;
    }
}

const c = new Counter();
```

ES3 equivalent:

```js
function Counter() {
    this.count = 0;
}

Counter.prototype.increment = function() {
    ++this.count;
};

var c = new Counter();
```

Anatomy:

- The function `Counter` *is* the constructor — what `new Counter()` invokes. Initial state goes inside the function body.
- Methods get attached to `Counter.prototype`, not to individual instances. Every `Counter` instance shares the same prototype methods.
- `this` inside refers to the instance, same as in a `class`.

Inheritance:

```js
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    WScript.Echo(this.name + " makes a sound.");
};

function Dog(name) {
    Animal.call(this, name); // equivalent to super(name)
}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.speak = function() {
    WScript.Echo(this.name + " barks.");
};
```

The three lines you'll see repeatedly when ES3 simulates inheritance:

- `Animal.call(this, name)` — runs the parent constructor with the child's `this`. The `super(...)` of ES3.
- `Dog.prototype = Object.create(Animal.prototype)` — wires up the prototype chain. Now `Dog` instances inherit `Animal.prototype` methods.
- `Dog.prototype.constructor = Dog` — bookkeeping so `instance.constructor` points back at `Dog`. Easy to forget; mostly cosmetic but good practice.

When you see this trio of lines together, you're looking at "Dog extends Animal."

### IIFE for module-like scoping

Modern:

```js
import { helper } from "./helpers.js";
```

ES3 equivalent — the **revealing module pattern**:

```js
var helpers = (function() {
    function privateThing() {
        // not exported
    }

    function helper() {
        // can use privateThing freely
    }

    return {
        helper: helper
    };
})();
```

The IIFE (immediately-invoked function expression — `(function(){...})()`) creates a private scope. Anything declared inside is hidden unless explicitly returned. The returned object is the module's "public surface."

Variations:

- `var helpers = (function() { ... })();` — assigned to a name.
- `MyApp.helpers = (function() { ... })();` — attached to a namespace object.
- `(function() { ... })();` — bare IIFE for side-effect-only code (registering a handler, mutating a global).

### `apply` and `call` for invocation

Modern:

```js
const args = [1, 2, 3];
WScript.Echo(Math.max(...args));
```

ES3 equivalent:

```js
var args = [1, 2, 3];
WScript.Echo(Math.max.apply(null, args)); // first arg is "this" — null since Math.max ignores it
```

`apply(thisArg, arrayOfArgs)` — invokes the function with `thisArg` as `this`, spreading the array as positional arguments. `call(thisArg, arg1, arg2, ...)` — same but takes individual arguments instead of an array.

Together they're the ES3 way to:

- Pass an array as variadic arguments (`Math.max.apply(null, arr)`).
- Call a method with a different receiver (`obj.method.call(otherObj, ...)`).

Closely related: the `var self = this` pattern. Where modern code uses arrow functions to preserve `this` in callbacks, ES3 captures it explicitly:

```js
function MyClass() {
    var self = this;

    setTimeout(function() {
        self.update(); // 'this' would be wrong here; 'self' is the captured outer this
    }, 1000);
}
```

Recognize `var self = this;` — it always means "I need the outer `this` inside a nested function."

### Object literal methods (no shorthand)

Modern:

```js
const obj = {
    bark() { WScript.Echo("Woof!"); },
    fetch() { WScript.Echo("Got it!"); }
};
```

ES3 equivalent:

```js
var obj = {
    bark: function() { WScript.Echo("Woof!"); },
    fetch: function() { WScript.Echo("Got it!"); }
};
```

Functionally identical. Just longer.

### String building (no templates)

Modern:

```js
const msg = `Hello, ${name}! You have ${count} new messages.`;
```

ES3 equivalent:

```js
var msg = "Hello, " + name + "! You have " + count + " new messages.";
```

For multi-line content, the `Array.join` pattern from Tier 2:

```js
var block = [
    "line one",
    "line two",
    "line three"
].join("\n");
```

That's the codebase preference for any non-trivial multi-line string.

### `arguments` instead of rest

Modern:

```js
function sum(...numbers) {
    let total = 0;

    sum_loop: for(let i = 0; i < numbers.length; ++i)
        total = total + numbers[i];

    return total;
}
```

ES3 equivalent:

```js
function sum() {
    var total = 0;

    sum_loop: for(var i = 0; i < arguments.length; ++i)
        total = total + arguments[i];

    return total;
}
```

`arguments` is array-*like* (has `.length`, supports indexing) but isn't a real array. To get a real array in ES3:

```js
var args = Array.prototype.slice.call(arguments);
```

That `.slice.call(arguments)` trick is surprisingly common. Recognize it as "convert `arguments` to a real array."

### Defaulting

Modern:

```js
function greet(name) {
    name = name ?? "stranger";
    WScript.Echo(`Hello, ${name}!`);
}
```

ES3 equivalent:

```js
function greet(name) {
    name = name != null ? name : "stranger";
    WScript.Echo("Hello, " + name + "!");
}
```

The `!= null` (with **loose** `!=`) is the canonical ES3 way to test "is this null *or* undefined." It's one of the few times the style guide allows loose equality. The semantics: `x != null` is `false` only when `x` is `null` or `undefined` — perfect for "use default if unset."

Don't use the lazier `name = name || "stranger"` pattern — it trips on `0` and `""` (and other legitimately-falsy values).

You'll occasionally see the older idiom for "use default if not passed":

```js
function greet(name) {
    if(typeof name === "undefined")
        name = "stranger";
}
```

This works but only catches `undefined`, not `null`. The `!= null` form is the codebase preference because it handles both.

### Factory functions

Modern:

```js
class User {
    constructor(name) {
        this.name = name;
    }
}

const u = new User("Sam");
```

ES3 (factory function, an alternative to a constructor):

```js
function makeUser(name) {
    return {
        name: name
    };
}

var u = makeUser("Sam");
```

Not strictly equivalent — factories return plain objects, not "instances." But for simple data containers without inheritance, factories are often used in place of constructor functions. Recognize the pattern; choice between the two is taste.

### Recognizing rare ESX files

Most files in the codebase are ES3 — flat `var` declarations, `function` declarations, prototype assignments. A small number are ESX (modern syntax) and get transpiled by `baseline` at runtime when loaded via `imports()`.

Visual indicators that you're in an ESX file:

- `let` or `const` instead of `var`
- Arrow functions: `(x) => ...`
- Template literals (backticks)
- `class` keyword
- Static `import`/`export` keywords (note: lowercase `imports()` is the polyfill function call; uppercase `import` is the static syntax)
- Spread/rest `...`

If you don't see any of these, assume ES3 and read accordingly.

### Exercises

1. **Translate a class.** Take any class you wrote in Tier 3 (e.g., `Counter` or `Person`). Rewrite it in ES3 — constructor function plus prototype assignments. Confirm it behaves identically.
2. **Find an IIFE.** Look through the codebase for an IIFE-based module pattern. What's "private" inside it? What's exposed in the return value?
3. **Recognize `var self = this`.** Find a function that uses the `var self = this` pattern. What `this`-rebinding problem is it working around? What would the modern arrow-function version look like?
4. **Defaulting audit.** Find three different defaulting patterns in the codebase: `x != null ? x : default`, `x || default`, and `if(typeof x === "undefined")`. Note any cases where the wrong pattern is used (e.g., `||` on a value that could legitimately be `0` or `""`).
5. **`apply` in action.** Find an `.apply(...)` call somewhere in the codebase. What's the receiver (the `this` argument)? Why is `apply` used instead of a direct call?
6. **Spot the ESX.** Find one ESX file (one that uses modern syntax). What signals tell you it's not ES3? What would it look like translated to ES3?

---

## Lesson 4.4: Debugging without a debugger

> **New words in this lesson**
>
> - **print debugging** — adding `Echo` or `console.log` calls to inspect values mid-run
> - **bisect** — narrow a bug down by halving the search space at each step
> - **REPL** — Read-Eval-Print Loop, an interactive prompt where you type code and see the result
> - **stack trace** — the chain of function calls leading up to an error
> - **reproduce** — make a bug happen on demand from a known starting point
> - **regression test** — a test that protects against a bug coming back
> - **`console.log`** — a polyfilled function in our environment, behaves like in modern JS

In our environment you don't have a step-debugger. WSH supports the antique Microsoft Script Debugger, but it's unreliable. Your tools are: `Echo`, `console.log` (polyfilled), file-based logging, careful reading, and the `debug` mini-terminal.

That sounds limiting, and it is. But it forces good debugging discipline — narrowing the bug down by intentional steps, instead of stepping through code line by line and hoping to spot it.

### Print debugging — `Echo` and `console`

The simplest debugger is `WScript.Echo(thing)`. It prints to stdout under `cscript`.

```js
WScript.Echo("about to call processFile");
processFile(path);
WScript.Echo("done with processFile");
```

For more structured output, use `console.log` — polyfilled in our environment to behave like modern JS:

```js
console.log("user state:", user);
console.warn("expected value missing");
console.error("hit unexpected branch");
```

The `console` family supports multiple arguments and produces object inspection that's more readable than raw `Echo`. Use it when you want to see what an object actually contains — `console.log(complexObject)` is far more useful than `WScript.Echo(complexObject)`, which calls `.toString()` and gives you `"[object Object]"`.

### Object inspection with `JSON.stringify`

For a fully readable dump of an object's contents, `JSON.stringify` with indentation:

```js
WScript.Echo(JSON.stringify(obj, null, 2));
```

The `2` is the indentation level — produces formatted, multi-line output. Useful when:

- The object has nested structure
- You want a snapshot you can paste into a journal entry or share with someone

Caveats:

- Circular references throw. Wrap in `try/catch` if it might happen.
- Functions, `Symbol` keys, and `BigInt` polyfill objects don't serialize cleanly — they get omitted or stringify oddly.

### File-based logging

For long-running scripts, scripts running under `wscript` (where `Echo` opens dialogs), or scripts that produce output you want to compare across runs, log to a file:

```js
var fso = new ActiveXObject("Scripting.FileSystemObject");
var log = fso.OpenTextFile("C:\\debug.log", 8, true); // 8 = ForAppending, true = create

log.WriteLine("[" + (new Date()).getTime() + "] entered processFile");
// ...
log.Close();
```

Always close. Always include a timestamp. Don't leave logging code in committed scripts unless it's intentional and gated (e.g., behind a `/verbose` flag).

A useful wrapper:

```js
function dlog(msg) {
    var fso = new ActiveXObject("Scripting.FileSystemObject");
    var f = fso.OpenTextFile("C:\\debug.log", 8, true);
    f.WriteLine("[" + (new Date()).getTime() + "] " + msg);
    f.Close();
}

dlog("entered processFile with path=" + path);
```

Slow if called in a tight loop (opens the file every call). Fine for sparse high-level logging.

### Bisecting — narrowing down

When a script fails somewhere unknown, **bisect**:

1. Comment out the second half of the suspect code. Does the script still fail?
2. If yes, the bug is in the first half. Comment out the first half's second half. Repeat.
3. If no, the bug is in the second half. Repeat the bisect there.

After log₂(N) iterations, you've narrowed to the offending statement. Faster than trying to reason about every line.

You can also bisect with `Echo`:

```js
WScript.Echo("checkpoint A");
firstThing();
WScript.Echo("checkpoint B");
secondThing();
WScript.Echo("checkpoint C");
thirdThing();
WScript.Echo("checkpoint D");
```

If the script fails after `B` but before `C`, the bug is in `secondThing`.

### Reading runtime errors

A typical WSH error looks like:

```
C:\Scripts\myScript.js(42, 8) Microsoft JScript runtime error:
'something' is undefined
```

The pieces:

- **`(42, 8)`** — line 42, column 8 of the source file. Open the file and go there directly.
- **`Microsoft JScript runtime error`** — distinguishes from compile-time/syntax errors.
- **`'something' is undefined`** — the actual problem. In this case, you tried to access a property of something that's undefined.

Common error messages and what they mean:

- **`'X' is undefined`** — `X` doesn't exist. Either you typo'd the name, or you tried to access a property of `null`/`undefined` (like `obj.foo` where `obj` is undefined). Check the spelling and check the value.
- **`Object expected`** — you called something as if it were a function, or accessed a property of `undefined`. Often a missing `new` or a function that wasn't loaded.
- **`Object doesn't support this property or method`** — you called a method that doesn't exist on the object. Maybe a typo, maybe wrong type.
- **`Type mismatch`** — you passed a wrong-typed value to something that cares (often a COM call).
- **`File not found` / `Path not found`** — exactly what it says. Verify the path; check escaping (`\` vs `\\`).
- **`Permission denied`** — file is locked, in use, or you don't have rights.

When the error doesn't make sense, **check the journal first**.

### The `debug` mini-terminal

The codebase ships a `debug` function that opens an interactive REPL. Expressions you paste in are evaluated through `baseline`, so modern syntax works. Useful for:

- Inspecting a value mid-execution.
- Trying a fix idea before committing it to source.
- Probing the runtime state when an error doesn't tell you enough.

Drop a `debug()` call wherever you want to pause and explore. When the script reaches it, you get an interactive prompt with access to the surrounding scope.

This is the closest thing to a step-debugger we have. Lean on it.

### Trace by hand

For code you can't run — production-only paths, side-effect-heavy code, code that needs data you don't have — trace by hand. Pick an example input, walk through line by line, and predict each statement's effect on a notepad.

Slow but reliable. It's the technique that finds bugs nothing else finds — the "this can only happen on Tuesdays" kind that you'd never reproduce in testing.

### Reproducing minimally

When a real-world bug fires, the first instinct is often "fix it and move on." Resist. The right move:

1. **Reproduce it.** Find a minimal input that triggers the bug. If the input is sensitive, scrub or redact.
2. **Save the reproduction.** A few lines that always reproduce the bug, kept somewhere — a scratch script, a journal entry, a test file.
3. **Then fix.**
4. **Verify the fix against the reproduction.** Confirm the bug is gone *and* nothing nearby broke.

This pays off twice: the next time someone hits a similar bug, the reproduction is a head start; and you have a regression test you can re-run when something nearby changes later.

### Exercises

1. **Echo bisect.** Write a script with a deliberate bug (e.g., a typo'd variable or a missing function). Use `Echo` checkpoints to bisect to the failing line.
2. **JSON dump.** Take a script that operates on an object. Add `WScript.Echo(JSON.stringify(obj, null, 2))` at one point to dump its state. Read the output — does it match what you expected?
3. **Decode an error.** Take any error message you've encountered. Walk through it: what file, what line, what kind of error, what the error text means. If the error makes no sense, check the journal.
4. **Trace by hand.** Pick a non-trivial function from the codebase. Without running it, walk through its execution with a sample input. Note where you got stuck or surprised.
5. **Use `debug`.** Find a `debug()` call in the codebase (or add one to a test script). When it fires, try inspecting a few values. Note what's available and what's not.
6. **Reproduce a fix.** Find a closed-out journal entry describing a past bug. Write a minimal reproduction script for that bug as if you were filing it fresh. Confirm the current code no longer exhibits the bug.

---

## Lesson 4.5: Production-code gotchas

> **New words in this lesson**
>
> - **gotcha** — a non-obvious behavior that catches people off guard
> - **silent failure** — when something goes wrong without producing an error
> - **encoding** — how text is stored as bytes (UTF-8, UTF-16, ANSI, etc.)
> - **race condition** — a bug that depends on timing between two things happening
> - **resource leak** — failing to release something (a file handle, a COM object) when done
> - **dead code** — code that exists but never runs
> - **off-by-one** — being one step away from the right index or count

This lesson is a collected list of bugs you'll find in real code — including in our codebase. Most of them you've seen pieces of in earlier tiers. Here they're collected so you know what to look for.

### `==` in legacy code

You'll see plenty of `==` and `!=` in older parts of the codebase. Sometimes it's intentional (the canonical `x != null` defaulting check from Lesson 4.3). Sometimes it's just legacy.

The dangerous case: comparing a string to a number, or a value to `false`/`true`.

```js
if(value == 0) { ... }    // also true for "", false, [], "0", null...
if(loaded == true) { ... } // wonky for non-boolean truthy values
```

Whenever you see `==` (or `!=`) outside the `!= null` defaulting idiom, suspect it. Either change to `===` or convince yourself the loose semantics are actually wanted.

### Hoisting bites

Recall from Tier 2: `var` declarations are hoisted to the top of their function scope, but assignments are not. This means a variable can exist as `undefined` before its `var` line.

```js
function process() {
    if(condition) {
        var result = computeResult();
    }

    return result; // exists because of hoisting; might be undefined
}
```

Looks like `result` is local to the `if` block. It isn't — `var` is function-scoped, so `result` is visible throughout `process`. Whether it's `undefined` depends on whether `condition` was true.

Fix: either move the `var` to the top of the function, or convert to ESX with `let`/`const`.

### `this` rebinding

In ES3, `this` rebinds based on *how* a function is called, not where it was defined.

```js
function MyClass() {
    this.value = 42;

    setTimeout(function() {
        WScript.Echo(this.value); // undefined — `this` is the timer, not MyClass
    }, 1000);
}
```

The `var self = this` pattern from Lesson 4.3 fixes this. In ESX code, arrow functions inherit `this` from the surrounding scope and don't have this problem.

When you see weird `undefined` errors inside a callback, suspect `this` rebinding.

### Truthy/falsy traps

Repeat from earlier tiers because it bites in production:

```js
if(count) { ... }       // false when count is 0
if(name) { ... }        // false when name is ""
if(items) { ... }       // true even when items is [] (empty array is truthy)
```

Be deliberate. `if(count > 0)`, `if(name !== "")`, `if(items.length > 0)` say what you mean.

### Reference vs value confusion

Functions get references to objects, not copies. Mutating an argument mutates the caller's object too.

```js
function reset(config) {
    config.theme = "default"; // caller's config also changes
}
```

If a function is supposed to leave its argument alone but actually mutates it, that's a bug — usually subtle, often only noticed when the same object is reused.

When in doubt, copy the input first:

```js
function reset(config) {
    var local = JSON.parse(JSON.stringify(config)); // deep copy
    local.theme = "default";
    return local;
}
```

(Cheap-and-cheerful deep copy via JSON. Fine for plain data; fails on functions, dates, circular references.)

### Off-by-one in COM enumerations

When iterating `WScript.Arguments` or `Folder.Files`, the indexing is zero-based but the COM `Count` is one-based in some collections. Always test with both empty and single-item inputs to catch off-by-one bugs.

A safer pattern: convert with `Array.enumerate` first, then iterate the resulting array. Lesson 3.5 covers this.

### Date month is zero-indexed

From Lesson 3.4: `new Date(2024, 5, 15)` is **June 15**, not May. Re-stated here because it's caught hundreds of programmers in the wild.

If you're constructing dates from user input where the month is one-indexed, subtract one:

```js
var d = new Date(year, month - 1, day);
```

### Encoding and mojibake

Files saved in different encodings (UTF-8, UTF-16, Windows-1252) read back differently in WSH. Symptoms include:

- Strange characters appearing where punctuation should be (`â€™` instead of `'`)
- Comparisons silently failing because the bytes differ even when the visible characters match
- File reads producing garbled content

The `unorm` polyfill normalizes Unicode and helps with these issues. When something looks "almost right but wrong," check the encoding of the input file first.

### File handle leaks in error paths

```js
var stream = fso.OpenTextFile(path, 1);
var content = stream.ReadAll();
stream.Close();
```

Looks fine — but if `ReadAll()` throws for any reason, `Close()` never runs and the file stays locked. The defensive form (Lesson 3.5):

```js
var stream = fso.OpenTextFile(path, 1);

try {
    var content = stream.ReadAll();
    return content;
} finally {
    stream.Close();
}
```

`finally` runs whether or not the `try` succeeded.

### Silent failures in transformation functions

The style guide says transformation functions (the `transformX` family in our transpilers) must never throw — they return their input unchanged on failure. That's the right design choice, but it means a buggy transformation produces *quiet wrongness* instead of an error.

When a transpiler-related bug shows up, suspect transformations first. Add temporary logging inside each transform to see which one is changing the input in an unexpected way.

### `parseInt` without a radix

Always pass `10` as the second argument. Without it, JScript guesses the base from the input — `parseInt("010")` might return `8` on some engines (interpreting `0`-prefix as octal). With `10`, you get `10`.

You'll see `parseInt(s)` in legacy code. It's almost always a latent bug.

### Copy-paste bugs

The boring one. When code is copied from elsewhere and then partially edited:

- A loop variable still has its old name (`for(var i = 0; i < users.length; ++i) { posts[i].title = ... }` — using `users.length` to iterate `posts`)
- An error message refers to the original context (`throw new Error("Cannot load user");` inside a function that loads files)
- A comment describes the old code (`// fetch user from DB` above a function that reads from a file)

Read the comments and error messages. When they don't match what the code does, the code was probably copy-pasted and not fully edited. The semantic bug is often nearby.

### Exercises

1. **`==` audit.** Search the codebase for `==` (taking care to exclude `===`). For each hit, decide: intentional `!= null` idiom, or legacy `==` that should be `===`?
2. **Hoisting trace.** Write a function with a `var` declaration inside an `if` block. Trace by hand what happens when the `if` condition is false but the variable is referenced later.
3. **`this` callback.** Write a small `class` with a method that calls `setTimeout` from inside another method. Demonstrate the `this` rebinding bug. Fix it both ways: with `var self = this` and (in ESX) with an arrow function.
4. **Mutation surprise.** Write a function `clearTheme(config)` that mutates `config.theme`. Call it from a script, then `Echo` the original `config` to show it changed. Then write `clearedTheme(config)` that returns a new object instead.
5. **Encoding spelunk.** Find a file in the codebase that involves Unicode handling. What encoding edge cases does it guard against? How does it use `unorm`?
6. **Find a copy-paste bug.** Search for at least three error messages or comments in the codebase. Verify each one accurately describes the code below it. If you find a mismatch, that's a candidate copy-paste bug.

---

## Lesson 4.6: The project journal

> **New words in this lesson**
>
> - **journal** — `journal.txt`, the project's running record of session notes, findings, and decisions
> - **session transcript** — a record of what was done in a single working session
> - **closed-out** — an issue that's been resolved (often noted in the journal)
> - **breadcrumb** — a small note left for future readers (a pointer to the journal, a known-issue marker)

The codebase has a long history. Bugs have been found and fixed. Surprising behaviors have been documented. Decisions have been made for reasons that aren't obvious from the code. All of that lives in `journal.txt`.

This lesson covers what the journal is, how to use it, and when to add to it.

### What's in the journal

`journal.txt` is the project's running record. It's not a formal document — it's more like a working log. Typical entries include:

- **Session transcripts** — notes from a single working session: what was investigated, what was changed, what was learned.
- **Closed-out bug records** — when a tricky bug is fixed, the journal usually has the trail of investigation: how it was reproduced, what the actual cause was, what the fix was.
- **Known-issue notes** — bugs we know about but haven't fixed yet, with notes on how to work around them.
- **Decision records** — choices made for non-obvious reasons. "We're using `Array.prototype.slice.call(arguments)` here instead of `Array.from` because the latter wasn't polyfilled when this was written. Could be revisited."
- **Architecture notes** — high-level explanations of why something is structured the way it is.

It's not organized like a textbook. It's chronological with occasional thematic sections.

### Why search the journal first

When you encounter:

- An error message that doesn't make sense
- A function whose name says one thing but body does another
- A comment with `// known issue, see journal` or similar breadcrumb
- A line that looks like a hack or workaround
- Behavior that seems wrong but might be intentional

…search the journal before changing anything. Most weird things have been weird before, and the explanation is usually there.

### How to search it

In our air-gapped environment, you have local search tools (whatever `find`/`grep` equivalents are installed). Strategies:

- **Search the literal error text.** If the journal mentions the error, it'll usually quote it.
- **Search by function or file name.** Past bugs often referenced specific code.
- **Search by date.** If you know when something changed, the journal entries from around that time will mention it.
- **Search by keyword.** "encoding," "hoisting," "race," "leak," "crash," "transform" all turn up relevant entries.

If you don't find anything, that's also useful information. It means the issue is new, or hasn't been documented.

### When to add to it

Add a journal entry when:

1. **You fix a bug that took more than 30 minutes to diagnose.** Future you (or someone else) shouldn't have to re-do that detective work.
2. **You discover a non-obvious behavior** in the code or in WSH itself. Even if it's not a bug, document it so the next person doesn't waste time figuring it out.
3. **You make a non-trivial decision.** Why a particular function is structured a certain way. Why an alternative was rejected.
4. **You hit a known issue and work around it.** Even if you're not fixing the underlying bug, document the workaround.

You don't need to log every change. Routine fixes don't need entries. The bar is roughly: "would this save someone an hour of confusion later?"

### Writing a good entry

A useful journal entry usually has:

- **A date or timestamp.**
- **A brief title or summary.** What's this about?
- **Context.** What were you working on when this came up?
- **The investigation.** What did you try? What did you learn?
- **The resolution.** What did you change, or what's the conclusion?
- **Any pointers.** Files affected, related entries, things to watch for.

It doesn't have to be polished prose. Bullet points, code snippets, and informal language are all fine. The audience is future maintainers — usually yourself in three months.

### Breadcrumbs in code

Sometimes the journal entry is too detailed for in-code commentary, but readers still need to know there's something worth knowing. The convention is a breadcrumb comment pointing at the journal:

```js
// known issue with empty input — see journal entry 2024-08-15
if(!input)
    return defaultValue;
```

When you see one of these, look it up in the journal before "fixing" the apparent oddity. It's there for a reason.

### The journal as a learning resource

Apart from being a working record, the journal is one of the best ways to learn the codebase. Reading old entries — even ones not relevant to current work — teaches you:

- The history of the project (what's been built, what's been tried)
- Common bug patterns (what usually goes wrong)
- The reasoning behind current code (why things are the way they are)
- The team's working style (how people approach problems)

If you have downtime, browsing past journal entries is genuinely productive. Treat it as supplementary reading.

### Exercises

1. **Find a closed-out bug.** Search the journal for any entry describing a bug fix. Read the whole entry. Could you have reproduced the bug from the description alone?
2. **Find a breadcrumb.** Search the codebase for in-code references to the journal (look for "journal" in comments). Pick one and find the corresponding entry. Does the entry explain what the comment hinted at?
3. **Search by error text.** Pick any error message produced by the codebase. Search the journal for parts of that text. Are there past investigations of it?
4. **Write a mock entry.** Pick any past change you've made (real or hypothetical). Write a journal entry for it. Does it have the elements above (context, investigation, resolution, pointers)? Refine until it does.

---

## Tier 4 wrap-up

You can now:

- Read JScript you didn't write — by entry-point, by symptom, by following imports
- Navigate the AutoIATE codebase — knowing what each file does and where to look for things
- Recognize ES3 patterns — constructor functions, IIFEs, `apply`/`call`, `arguments`, the canonical defaulting idiom
- Debug without a step-debugger — `Echo`, `console.log`, file logs, `JSON.stringify`, bisecting, and the `debug` mini-terminal
- Spot common production-code gotchas — `==` traps, hoisting, `this` rebinding, encoding issues, file-handle leaks
- Use the journal — searching it, adding to it, reading it as a learning resource

That's confident maintenance. Not "I understand everything" — that's a higher tier and not the goal here. It's "I can find the answer, verify a fix, and document my work."

### Final exercise: the maintenance task

Instead of a synthetic capstone, the final exercise is a real maintenance task. Your manager will assign one — typically a known bug, a feature extension, or a refactoring task in an existing script.

For that task:

1. **Read first.** Use the strategies from Lesson 4.1. Don't write anything until you understand the relevant code.
2. **Search the journal.** Before changing anything, see if the area you're touching has prior history.
3. **Reproduce the issue** (if it's a bug) using the techniques from Lesson 4.4.
4. **Make the smallest change that fixes it.** No drive-by refactoring unless explicitly asked.
5. **Verify against your reproduction.** Confirm the bug is gone and nothing nearby broke.
6. **Add a journal entry** if anything you learned would save someone else time later.
7. **Submit for review.** Walk the reviewer through what you changed and why.

When you can do all of that without hand-holding — when the reviewer's questions are about the *change* rather than your *process* — Tier 4 is complete. From there, you're a maintainer.

### Beyond Tier 4

The project hierarchy goes further: **Expert** (able to polyfill missing capabilities) and **Virtuoso** (can create new concepts and provide documentation). Those are not part of this tutorial — they require deeper architectural knowledge of the codebase, including the transpilers (`baseline`, `parseAuto`) themselves.

The path there isn't more tutorials. It's reading, modifying, and gradually understanding more of the codebase as you maintain it. The journal will be your textbook.

Welcome to the team.
