# Your Stylesheet Can Do Euclid

### Running a real algorithm (GCD) with a while loop and an if/else, in pure HTML + CSS

> Open `index.html` in this folder. Press the green button four times. You just watched a stylesheet
> work out that the greatest common divisor of 21 and 6 is 3. There is no `<script>` tag on the page.

---

## 1. The claim, and the fine print

People like to say "HTML + CSS is Turing complete" as a party trick. It is true, with a footnote.

The usual evidence is **Rule 110**, a one-dimensional cellular automaton. Each cell is on or off, and its next state depends only on itself and its two neighbours. In 2004 Matthew Cook published a proof that Rule 110 can simulate any Turing machine. A few years later, people started building Rule 110 out of checkboxes and CSS selectors. If CSS can run Rule 110, CSS can, in principle, compute anything.

The footnote is that **CSS cannot move time forward by itself**. A stylesheet is a function from "current page state" to "how the page looks". It never changes the state. Something outside has to do that, and in every pure-CSS computer that something is a human clicking. So the honest framing is:

| Computer part | What plays it here |
| --- | --- |
| Memory | Hidden `<input type="radio">` elements |
| Registers | `@property` custom properties typed as `<integer>` |
| ALU (arithmetic and logic) | `calc()` and `clamp()` |
| Output | CSS counters (`counter-reset` + `content: counter()`) |
| Write-back | `<label for="...">` |
| Clock | **You.** One click is one clock tick. |

A second footnote: real Turing machines have infinite tape. Our machine has 30 cells per number. That makes it a finite state machine, which is also true of the laptop you are reading this on. Nobody minds.

With that settled, let's build a computer.

---

## 2. The one rule that makes everything work

CSS can read state but cannot write it. Clicking a `<label for="x">` checks the input with id `x`. That is the only write operation we have.

So here is the trick that drives the whole machine:

> **CSS does the computing by deciding which label is under your cursor. Your click carries out the write.**

You never need to know which label you are clicking. You just click "the button". CSS has already slid the correct label into that spot.

Keep that in mind and every block below falls out naturally.

---

## 3. Memory: radio buttons that hold a number

A radio group allows exactly one checked input. Ten radios in one group therefore store one digit from 0 to 9.

```html
<form>
  <input class="mem" type="radio" name="n" id="w-0">
  <input class="mem" type="radio" name="n" id="w-1">
  <!-- ... -->
  <input class="mem" type="radio" name="n" id="w-7" checked>  <!-- n = 7 -->
  <!-- ... -->
  <div class="machine m-while"> ... </div>
</form>
```

```css
.mem { display: none; }  /* invisible, but labels can still check them */
```

Hiding radios with `display: none` is fine. Clicking a `<label for>` still checks its input.

### Loading memory into a register

We need the stored value as a number CSS can do arithmetic on. The general sibling combinator `~` lets a checked radio style the machine that comes after it:

```css
#w-0:checked ~ .machine { --n: 0; }
#w-1:checked ~ .machine { --n: 1; }
/* ... one rule per possible value ... */
#w-9:checked ~ .machine { --n: 9; }
```

This is the only place in the machine where we write one rule per value. Everything after this is real arithmetic.

### Making the register a real integer

An unregistered custom property is just a string of tokens. We want the browser to compute it into a number once and pass that number to everything downstream. `@property` does exactly that:

```css
@property --n {
  syntax: '<integer>';
  inherits: true;
  initial-value: 0;
}
```

Now `--n` behaves like a typed variable. If you write `--max: calc(var(--gt) * var(--x) + ...)` and `--max` is registered, `--max` holds the finished integer instead of a pile of unevaluated math.

### Printing a number

CSS has exactly one way to turn a number into visible text: counters.

```css
.val::after {
  counter-reset: v var(--show);
  content: counter(v);
}
.m-while .val-n { --show: var(--n); }
```

---

## 4. Block one: the `while` loop

Here is the program:

```js
while (n > 0) {
  n = n - 1;
}
```

A while loop has two parts: a **condition** checked before each pass, and a **body** that runs if the condition holds. We build each part separately.

### 4.1 The body: `n = n - 1`

We need a button that, when clicked, writes `n - 1` into memory. That means clicking the label `for="w-6"` when `n` is 7, `for="w-5"` when `n` is 6, and so on.

So we make **all** of those labels, stack them in a tall strip, and look at the strip through a window exactly one label high:

```html
<div class="step">            <!-- the window: 1 label tall, overflow hidden -->
  <div class="strip">         <!-- the tape of possible writes -->
    <label for="w-0">▶ step · n ← 0</label>
    <label for="w-1">▶ step · n ← 1</label>
    <!-- ... -->
    <label for="w-9">▶ step · n ← 9</label>
  </div>
</div>
```

```text
             strip                 window
          ┌──────────┐
 index 0  │  n ← 0   │
 index 1  │  n ← 1   │
   ...    │   ...    │
 index 6  │  n ← 6   │  ◀──── ┌──────────┐  when n = 7, slide the strip
 index 7  │  n ← 7   │        │  n ← 6   │  up by 6 rows so "n ← 6" is
   ...    │   ...    │        └──────────┘  the only label you can hit
          └──────────┘
```

The subtraction happens in a single `translate`:

```css
.step {
  height: var(--h);
  overflow: hidden;    /* clipped labels are not clickable either */
}

.m-while .strip {
  translate: 0 calc((var(--n) - 1) * var(--h) * -1);
}
```

When `n` is 7, the strip moves up 6 rows and the label `n ← 6` fills the window. You click, `#w-6` becomes checked, `--n` becomes 6, the strip moves up 5 rows, and `n ← 5` is waiting under your cursor. The button text even shows you the answer CSS computed before you commit it.

### 4.2 The condition: `n > 0`

CSS has no comparison operators. It does have `clamp(min, value, max)`, and for integers that is enough:

```css
.m-while {
  --run: clamp(0, var(--n), 1);
}
```

| n | clamp(0, n, 1) | meaning |
| --- | --- | --- |
| 0 | 0 | false |
| 1 | 1 | true |
| 7 | 1 | true |

We now have the condition as a 0 or 1. Next we need it to control whether the body can run at all.

### 4.3 The gate: turning a bit into "this button exists"

`display` cannot take a number. `scale` can:

```css
.step { scale: var(--run); }
```

When `--run` is 1, the window is normal size. When `--run` is 0, the window shrinks to a point. It is invisible and has no area to click. **The loop has exited** because its body is physically unreachable.

The "done" message uses the opposite gate:

```css
.done { scale: calc(1 - var(--run)); }
```

That's a while loop. The condition is a `clamp`, the body is a sliding label, and the repetition is your finger. Each press runs one iteration, and the loop refuses to run an iteration the condition does not allow.

---

## 5. Block two: the `if / else` statement

```js
if (x > y) {
  max = x;
} else {
  max = y;
}
```

### 5.1 Two ways to branch in CSS

**Selector branching** is the obvious approach:

```css
#x-7:checked ~ .machine .lucky { display: block; }   /* if (x == 7) */
```

It works well for "is this variable equal to a constant". It does not scale to comparing two variables. Testing `x > y` for two digits this way needs a rule for every one of the 100 combinations, and two 30-value numbers need 900.

**Arithmetic branching** treats the condition as a number. It needs one rule no matter how big the inputs get. This is the approach the machine uses.

### 5.2 The comparator

```css
.m-if {
  --gt: clamp(0, var(--x) - var(--y), 1);
}
```

Since `x` and `y` are integers, `x - y` is at least 1 whenever `x > y`. Clamping to `[0, 1]` squashes every positive difference to 1 and every zero or negative difference to 0.

| x | y | x − y | gt |
| --- | --- | --- | --- |
| 7 | 4 | 3 | 1 |
| 4 | 4 | 0 | 0 |
| 2 | 4 | −2 | 0 |

### 5.3 The select (a multiplexer)

With the condition as a 0/1 number, choosing a value is plain algebra:

```css
.m-if {
  --max: calc(var(--gt) * var(--x) + (1 - var(--gt)) * var(--y));
}
```

If `gt` is 1, the right-hand term vanishes and you get `x`. If `gt` is 0, the left-hand term vanishes and you get `y`. GPU shader programmers and constant-time crypto code use the same pattern, because it computes both sides and picks one instead of jumping.

### 5.4 Gating side effects

The value is chosen, but an `if` statement also decides **which code runs**. In our machine "code that runs" means "elements that exist", so we use gates again:

```css
.when-gt { scale: var(--gt); }          /* only exists in the if-branch   */
.when-le { scale: calc(1 - var(--gt)); } /* only exists in the else-branch */

.lit-gt  { --lit: var(--gt); }           /* highlight the line being taken */
```

In the demo, the taken line of the program listing lights up and the other dims. That happens because each line's opacity is `calc(.42 + .58 * var(--lit))`.

> Chromium has recently started shipping a native CSS `if()` function. The machine avoids it so it works in every current browser, and because `clamp` is more fun.

---

## 6. Putting it together: Euclid's GCD

Around 300 BC, Book VII of Euclid's *Elements* described how to find the greatest common divisor of two numbers by repeatedly subtracting the smaller from the larger. It is still one of the first algorithms in any DSA course, and it is exactly a while loop wrapped around an if/else:

```js
function gcd(a, b) {
  while (a !== b) {
    if (a > b) a = a - b;
    else       b = b - a;
  }
  return a;
}
```

Why does it work? Any number that divides both `a` and `b` also divides `a - b`. So replacing the larger number with the difference never changes the set of common divisors. The numbers keep shrinking, and when they become equal, the shared value is the largest common divisor.

### 6.1 Registers

Two numbers from 1 to 30, so two radio groups of 30 and 60 loading rules:

```css
#a-21:checked ~ .machine { --a: 21; }
#b-6:checked  ~ .machine { --b: 6; }
/* ... */
```

### 6.2 Three conditions from two comparators

```css
.m-gcd {
  --gt:  clamp(0, var(--a) - var(--b), 1);   /* a > b              */
  --lt:  clamp(0, var(--b) - var(--a), 1);   /* a < b              */
  --run: calc(var(--gt) + var(--lt));        /* a != b  (the loop) */
}
```

At most one of `gt` and `lt` can be 1, so their sum is 1 exactly when the numbers differ. That gives us the loop condition for free.

### 6.3 Two bodies in one window

The window now holds **two** strips, one per branch:

```css
/* loop guard: same as before */
.step { scale: var(--run); }

/* if (a > b)  a = a - b; */
.m-gcd .strip-a {
  scale: var(--gt);
  translate: 0 calc((var(--a) - var(--b) - 1) * var(--h) * -1);
}

/* else  b = b - a; */
.m-gcd .strip-b {
  scale: var(--lt);
  translate: 0 calc((var(--b) - var(--a) - 1) * var(--h) * -1);
}
```

The `- 1` exists because the strips start at value 1, not 0. The label for value `k` sits at row `k - 1`.

Read it as a program and it is the JavaScript above, line for line:

- The outer gate on `.step` is the `while`.
- The `scale: var(--gt)` and `scale: var(--lt)` gates are the `if` and the `else`.
- The two `translate`s are the two subtractions.

### 6.4 A trace

Here is what happens when you press the button with the default inputs, 21 and 6:

| click | a | b | branch CSS picks | label in the window |
| --- | --- | --- | --- | --- |
| 1 | 21 | 6 | `a > b` | `a ← a − b = 15` |
| 2 | 15 | 6 | `a > b` | `a ← a − b = 9` |
| 3 | 9 | 6 | `a > b` | `a ← a − b = 3` |
| 4 | 3 | 6 | `a < b` | `b ← b − a = 3` |
| – | 3 | 3 | `a = b` | button gone, **gcd = 3** |

I checked this on the real page by programmatically clicking whatever element sat at the centre of the button. Every run stopped on the right answer: gcd(21, 6) = 3, gcd(24, 18) = 6, gcd(30, 7) = 1, gcd(27, 18) = 9. The test harness used JavaScript to press the button. The page itself has none.

### 6.5 Complexity, measured in sore fingers

Subtraction-based Euclid takes up to `max(a, b) - 1` steps. Try `a = 1, b = 30` in the demo: you will click 29 times. The modern version replaces subtraction with remainder, `(a, b) → (b, a mod b)`, which needs O(log min(a, b)) steps.

CSS now has `mod()`, so the arithmetic is easy. The hard part is that the modulo version updates **both** registers in one step, and a click can only check one radio. Making it work would mean splitting each step into two ticks or packing both numbers into a single radio group of 900 states. That's a good exercise if you want one.

---

## 7. Bugs I hit so you don't have to

**Don't animate the strips.** A `transition` on `translate` looks lovely, but for 200 ms the wrong label is sliding past your cursor, and a fast double-click writes garbage into memory. The window and the "done" box animate their `scale`. The strips snap.

**`~` only looks forward.** `#a-5:checked ~ .machine` works only if every radio comes before `.machine` in the same parent. Put all the memory at the top of the `<form>`. (`:has()` removes this restriction if you prefer: `.machine:has(#a-5:checked)` with the inputs inside.)

**Register your properties.** Without `@property`, a value like `calc(var(--a) - var(--b))` is passed around as unevaluated text. Registered `<integer>` properties are computed once, at the element that declares them, and every consumer gets a plain number.

**One rule per cell is optional.** The memory tapes highlight the current value without 30 `:checked` rules. Each cell carries its own index, `style="--i: 17"`, and computes "am I the current value?" with two clamps:

```css
.cell {
  --hit: calc(1 - clamp(0, var(--v) - var(--i), 1)
                 - clamp(0, var(--i) - var(--v), 1));   /* 1 iff i == v */
  background: color-mix(in oklab, var(--tint) calc(var(--hit) * 88%), var(--panel-2));
}
```

That's an equality test built from two greater-than tests, which is how hardware does it too.

**A free reboot.** `<button type="reset">` inside the form restores every radio to its original `checked` state. It is a power cycle with zero JavaScript.

---

## 8. So, is CSS a programming language?

It depends where you put the line. CSS has state (via HTML), arithmetic, comparison, selection, and conditional execution. With a human supplying clock ticks, it can run Rule 110, so it can in principle simulate any machine. What it lacks is a way to advance its own state, and that missing piece is what makes the experiment interesting. You end up acting as the clock, and you see each tick happen.

Writing an algorithm this way also makes you think the way a hardware designer does: every `if` is a multiplexer, every loop guard is an enable line, and every variable is a bank of flip-flops. Euclid would probably have liked it.

---

## 9. Try it yourself

Open `index.html` in a recent browser (Chrome/Edge 111+, Safari 16.4+, Firefox 128+). Then try extending the machine:

1. **Multiplication by repeated addition.** `while (b > 0) { acc = acc + a; b = b - 1; }`. Note that each step needs two writes. How do you schedule them with one click per tick?
2. **Collatz.** `while (n != 1) { if (n is even) n = n / 2; else n = 3n + 1; }`. You'll need `mod()` for the parity test and a much longer tape.
3. **Max of an array.** Store five values in five radio groups and fold `max` across them with a chain of the multiplexers from section 5.
4. **Rule 110.** Build the original proof of concept: a row of checkboxes, one rule per neighbourhood, one click per generation.

Every one of these uses the same three pieces: radios for memory, `clamp` for decisions, and a sliding label for writing.
