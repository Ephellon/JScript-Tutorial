# Style Guide

This document outlines the rules and conventions used in this project. All contributors must follow these to keep code consistent, reliable, and maintainable across all JScript-based systems.

Rules are grouped by concern. Where a rule references another, the section name is used rather than a number so cross-references survive reorganization.

---

## I. Environment

### Language Target
- Write explicitly for the intended runtime environment (e.g., WSH / legacy JScript engines or controlled ES6+ environments).
- When targeting legacy JScript (ES3/WSH), use `var` exclusively and avoid unsupported syntax.
- When working in modern environments or pre-transpiled codebases, `const`/`let` are preferred.
- Any generated or distributed code must match the capabilities of its lowest supported runtime.

### Null & Undefined
- Never use the bare identifier `undefined` — it is mutable and can be shadowed. Use `void null` instead.
- Prefer `x == null` to test for both `null` and `void null` at once. Use `x === void null` only when you specifically mean "undefined, not null."
- Use nullish coalescing patterns (`??`, `??=`) only when the runtime guarantees support; otherwise, use equivalent helper patterns.

---

## II. Names & Declarations

### Naming Conventions
- **Variables & functions**: `camelCase`
- **Semantic constants** (values that never change by design): `UPPERCASE`
- **Labels**: `snake_case`
- **Short names** (`$`, `_`, `k`, `v`, `i`, `j`) are acceptable inside tight, self-evident scopes. Verbosity is reserved for broader scopes.
- Temporary variables created during transformations must use consistent prefixes that reflect their role:
  - `tmpObj_` for captured objects, `tmpFn_` for captured functions, `optObj_` for optional-chaining intermediates, etc.
  - Append a `pseudo.randomNanoID` suffix to guarantee uniqueness.

### Variable Declarations
- `const` for stable bindings; `let` for bindings that are reassigned.
- Multiple declarations use comma-first style, one per line, with `const`/`let`/`var` on the first line only:
  ```javascript
  const result = ''
      , parts  = []
      , depth  = 0;
  ```
- Prefer prefix `++i` / `--i` everywhere **except** the step expression of a `for` loop header, where either form is acceptable.

### String Quote Convention
- **Single quotes** `'...'` for strings the codebase treats as constants — internal token type names, sentinel values, flags, and any string the program reasons about as a fixed symbol:
  ```javascript
  node.type = 'KEYWORD';
  const FLAG = 'binary';
  tokens.push({ type: 'operator', flag: 'COMMA' });
  ```
- **Double quotes** `"..."` for strings that are mutable by nature — user-facing messages, error text, and generated code fragments emitted into the transpiler output:
  ```javascript
  throw new Error("Unexpected token at line " + line);
  out += "var " + name + " = void null;";
  ```
- The distinction communicates intent: single quotes say "*fixed symbol or content*"; double quotes say "*content that could change*".
- Mixing is permitted only when the string itself contains the other quote character and escaping would harm readability.

### Type Annotation Comments
- Inline type comments use `/*:type*/` with **no spaces** inside.
- Nullable types use `/*:?type*/`.
- Return types follow the closing `)` of the parameter list: `) /*:type*/ {`.
  ```javascript
  function skipWS(s /*:string*/, i /*:number*/) /*:number*/ {
      while(i < s.length && /\s/.test(s.charAt(i)))
          ++i;
      return i;
  }
  ```

---

## III. Expressions

### Operator Spacing
- Put a single space before and after binary operators: `+`, `-`, `*`, `/`, `%`, `**`, `=`, `==`, `===`, `!=`, `!==`, `<`, `>`, `<=`, `>=`, `&&`, `||`, `??`, `|>`, `<<`, `>>`, `>>>`, `&`, `|`, `^`, and all compound-assignment forms (`+=`, `&&=`, `??=`, etc.).
- Unary operators (`!`, `~`, `-`, `+`, `++`, `--`, `typeof`, `void`, `delete`) are written flush against their operand — no space between operator and operand:
  ```javascript
  const inv  = !flag;
  const neg  = -value;
  const bits = ~mask;
  ++cursor;
  ```
- Ternary `?` and `:` are spaced like binary operators inline (`x ? y : z`). Multi-line layout is governed by the **Ternaries** rule below.
- The object literal colon is **not** an operator — spacing inside object literals is governed by the **Object & Array Literals** rule.
- Examples:
  ```javascript
  // Correct
  const result = a + b * c;
  const mask   = (2 << (Math.log(n - 1) / Math.LN2)) - 1;
  if(x == null || y !== 0) { ... }

  // Incorrect
  const result=a+b*c;
  if(x==null||y!==0) { ... }
  const inv = ! flag;
  const neg = - value;
  ```

### Object & Array Literal Spacing
- **Object literals** are padded with a single space inside the braces: `{ key: value }`.
  - Empty objects: `{}` — no space.
- **Array literals** are **not** padded: `[1, 2, 3]`.
  - Empty arrays: `[]` — no space.
- The distinction mirrors their semantics: an object is a named-member structure and benefits from visual breathing room; an array is an ordered sequence and reads naturally compact.
  ```javascript
  // Correct
  const node  = { type: 'KEYWORD', value: 'for' };
  const empty = {};
  const arr   = [1, 2, 3];
  const none  = [];
  const mixed = { keys: ['a', 'b'] };

  // Incorrect
  const node = {type: 'KEYWORD', value: 'for'};
  const arr  = [ 1, 2, 3 ];
  ```

### Conditionals
- Two or fewer conditions that can be collapsed into a shorthand test (`.includes()`, regex test, etc.) should be — prefer brevity:
  ```javascript
  if('dvimguys'.includes(char))
      flag += char;
  ```
- Three or more conditions that **cannot** be collapsed use the broken logical form, with the logical symbol **leading** each line, prefaced by its identity value:
  - `false` for `||` chains
  - `true` for `&&` chains
  - `null` for `??` chains
  ```javascript
  if(false
      || char === void null
      || char === null
      || char === '\0'
  )
      break tokenizer;
  ```
- When chains of different operators are nested, use parentheses to show grouping — indentation alone is not sufficient.

### Ternaries
- Multi-line ternaries break with `?` and `:` **leading** each new line, aligned under the condition:
  ```javascript
  const type = token.flag === 'paren'
      ? 'PARENTHESIS'
      : token.flag === 'brack'
          ? 'BRACKETS'
          : 'BRACES';
  ```
- Short ternaries that fit comfortably on one line may stay inline: `x ? y : z`.

---

## IV. Statements & Control Flow

### One Statement Per Line
- Each statement occupies its own line. Multiple statements on a single line are never permitted, even when short:
  ```javascript
  // Correct
  const item = items[i];

  if(!item)
      continue loop;

  // Incorrect
  const item = items[i]; if(!item) continue;
  ```

### One-liner Statement Bodies
- The body of a one-liner `if`, `while`, or `for` appears on its own line, indented, never inline with the condition:
  ```javascript
  // Correct
  while(i < s.length && /\s/.test(s.charAt(i)))
      ++i;

  // Incorrect
  while(i < s.length && /\s/.test(s.charAt(i))) ++i;
  ```

### Block Separation
- A blank line must separate unrelated blocks. The following count as distinct block types:
  - `if` / `else if` / `else` chains (the whole chain as one unit)
  - `while` loops
  - `for` loops
  - `var`, `let`, or `const` declaration groups
- Consecutive blocks of the **same type** performing the same logical operation (e.g., a run of `while` loops advancing a cursor) may remain unseparated.

### Labeled Blocks & Loops
- `for` loops and standalone blocks must be named with labels. `continue` and `break` always reference their label explicitly. Label names are `snake_case`:
  ```javascript
  tokenizer: while(cursor < length) {
      ...
      word: while(cursor < length) {
          char = input.charAt(++cursor);

          if(/[\w$]/.test(char)) {
              word += char;
          } else {
              tokens.push({ type: 'word', value: word, index });

              break word;
          }
      }

      continue tokenizer;
  } // :tokenizer
  ```
- A standalone `continue` or `break` (not the `} break;` switch form — see **Switch Statements**) must always be preceded by a blank line:
  ```javascript
  if(char === '\\') {
      current += char + text.charAt(scanIndex + 1);
      scanIndex += 2;

      continue scan;
  }
  ```

---

## V. Functions

### Function Declarations
- Named function declarations are preferred at outer scope.
- Functions declared inside `case` blocks are expressed as `const NAME = (...) => {}` to respect block scoping in ES6+.
- Every function must have a header comment describing what it does and its inputs and outputs at a high level.
- Private helpers used only within one transformation function are defined inside it. Reusable helpers belong in the top-level utility section.

### Regex Callbacks
- All positional parameters are always included, even unused ones (`$0, $1, $$, $_`):
  ```javascript
  .replace(/[018]/g, ($0, $$, $_) => {
      return ($0 ^ getRandomValues(new Array(1))[0] & 15 >> $0 / 4).toString(16);
  });
  ```
- Arrow functions are used for regex callbacks in ES6+.

---

## VI. Switch Statements

### Case Structure
- Each `case` body is wrapped in braces `{}`.
- The closing brace and `break` share a line: `} break;`.

### Sibling Block Consistency
- In an `if` / `else if` / `else` chain:
  - If **any** sibling has braces, **all** siblings must have braces.
  - If braces are present and **any** sibling contains more than one statement, all siblings use indented multi-line formatting.
- A braced block containing exactly one statement omits the trailing semicolon on that statement:
  ```javascript
  if(ch === '(') {
      ++depthParen
  } else if(ch === ')') {
      --depthParen
  }
  ```

---

## VII. Comments & Documentation

### Inline Comments
- Comments explain **why**, not just what. Reserve them for non-obvious logic — delimiter scanning, regex tricks, sentinel values, workarounds.

### Closing Brace Breadcrumbs
- Closing braces for labeled and nested blocks carry a trailing breadcrumb comment tracing the full nesting path **only when any of the following are true:**
  - The deepest loop or case is more than **three blocks deep**, OR
  - There is **any nested switch** (a switch not root to its parent function), OR
  - There are **more than three cases** in a switch, OR
  - The block is **more than sixty lines** brace-to-brace.
- The breadcrumb traces the full nesting path:
  ```javascript
  } // :label | switch syntax | default | switch char | default
  ```

---

## VIII. Code Architecture

### Code Organization
Code is grouped into logical sections appropriate to the project:
- Utilities / helpers
- Core logic
- Orchestration / entry points

### Transformation & Processing Rules (When Applicable)
- When writing code that transforms or processes other code/data:
  - **Masking**: Protect segments that must not be altered (e.g., strings, comments) before applying broad operations.
  - **Order matters**: Some operations depend on earlier normalization steps.
  - **Safe defaults**: Preserve invalid or ambiguous input unchanged when correctness cannot be guaranteed.
  - **Wrapping**: Use parentheses where needed to preserve evaluation order.
  - **Flattening**: Simplify nested constructs when it improves clarity without altering behavior.
- Functions that produce code or structured output must always return valid, usable results — never partial structures.

### Error Handling
- Functions that process or transform external input should fail gracefully when possible.
- If an operation cannot be completed safely, return the original input or a safe fallback.
- Critical system-level failures may throw explicit errors with clear messaging.

---

## IX. Formatting

### Indentation & Layout
- Indentation: **4 spaces**. No tabs.
- Inline comments sit flush after the code they annotate, on the same line where space permits.

---

## Example Function

The function below demonstrates all rules in combination.

```javascript
// EMPTY_STRING is the generated representation of an empty JS string literal.
// Single-quoted because it is a fixed constant the transpiler reasons about.
const EMPTY_STRING = '""';

/*
 * transformTemplateLiterals
 * Replaces ES6 template literals (`...`) with ES3 string concatenation.
 * Nested ${} expressions are preserved and wrapped in parentheses.
 * Returns the original code unchanged if no backtick is found, or if an
 * unmatched delimiter is encountered mid-scan.
 *
 * Input:  `Hello, ${name}!`
 * Output: ("Hello, " + (name) + "!")
 */
function transformTemplateLiterals(code /*:string*/) /*:string*/ {
    let result = ''
        , buffer = ''
        , parts  = []
        , depth  = 0
        , index  = 0;

    if(!code.includes('`'))
        return code;

    scanning: for(let char, next, syntax = void null; index < code.length; ++index) {
        char = code.charAt(index);
        next = code.charAt(index + 1);

        switch(syntax) {
            case 'template': {
                if(char == '`') {
                    if(buffer.length)
                        parts.push('"' + buffer + '"');

                    result += parts.length
                        ? '(' + parts.join(' + ') + ')'
                        : EMPTY_STRING;

                    buffer = '';
                    parts  = [];
                    syntax = void null;

                    continue scanning;
                }

                if(char == '$' && next == '{') {
                    if(buffer.length)
                        parts.push('"' + buffer + '"');

                    buffer = '';
                    depth  = 1;
                    syntax = 'expression';
                    ++index;

                    continue scanning;
                }

                // Escape sequences — rewrite special chars, pass others through
                if(char == '\\') {
                    buffer += next.replace(/^(["\\`])/g, ($0, $1, $$, $_) => '\\' + $1);
                    ++index;

                    continue scanning;
                }

                buffer += char == '"'
                    ? '\\"'
                    : char;

                continue scanning;
            } break;

            case 'expression': {
                if(char == '{') {
                    ++depth;
                    buffer += char;

                    continue scanning;
                }

                if(char == '}') {
                    if(--depth === 0) {
                        parts.push('(' + buffer + ')');
                        buffer = '';
                        syntax = 'template';
                    } else {
                        buffer += char;
                    }

                    continue scanning;
                }

                buffer += char;

                continue scanning;
            } break;

            default: {
                if(char == '`') {
                    result += buffer;
                    buffer = '';
                    syntax = 'template';

                    continue scanning;
                }

                buffer += char;
            } break;
        } // :scanning | switch syntax
    } // :scanning

    // Unmatched delimiter — return original rather than emitting broken output
    if(false
        || syntax === 'template'
        || syntax === 'expression'
    )
        return code;

    return result + buffer;
}
```
