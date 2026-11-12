# The CSS Machine

**Euclid's GCD and LeetCode 704 (Binary Search), solved with only HTML and CSS. There is no JavaScript on the page.**

🔗 **Live demo:** https://sidhanth-mandal.github.io/CSS-DSA-Machine/

Press the button until it disappears, then view the source and look for a `<script>` tag. There isn't one.

---

## What's on the page

| # | Demo | What it shows |
| --- | --- | --- |
| 01 | [A while loop](https://sidhanth-mandal.github.io/CSS-DSA-Machine/#while) | `while (n > 0) n = n - 1;` counting down to zero |
| 02 | [An if / else](https://sidhanth-mandal.github.io/CSS-DSA-Machine/#ifelse) | `max = x > y ? x : y` using a comparator built from `clamp()` |
| 03 | [Euclid's GCD](https://sidhanth-mandal.github.io/CSS-DSA-Machine/#gcd) | The if/else nested inside the while loop, for two numbers from 1 to 30 |
| 04 | [Binary Search (LeetCode 704)](https://sidhanth-mandal.github.io/CSS-DSA-Machine/#binary-search) | `O(log n)` search over a sorted array of 10, with a three-way branch and `nums[mid]` lookup |

## How it works

CSS can read state and do arithmetic, but it can't change state on its own. The machine splits the work between the browser and you:

| Computer part | Built from |
| --- | --- |
| Memory | Hidden `<input type="radio">` groups. One checked radio = one stored value. |
| Registers | `@property` custom properties typed as `<integer>`, loaded with `#a-5:checked ~ .machine { --a: 5 }` |
| Arithmetic and comparisons | `calc()`, `clamp()`, `max()`, `round()`. For example, `clamp(0, a - b, 1)` is 1 when `a > b`, else 0. |
| Branching | `scale: var(--flag)`. An element scaled to 0 can't be seen or clicked. |
| Array indexing | A multiplexer: `nums[mid] = Σ (mid == i) × v[i]` |
| Write-back | A strip of `<label for>` elements behind a one-label window. CSS slides the label for the correct next value under your cursor. |
| Output | CSS counters (`counter-reset` + `content: counter()`) |
| Clock | **You.** One click is one step. |

The full explanations are in the two write-ups:

- [`ARTICLE.md`](ARTICLE.md) covers memory, the while loop, the if/else, and Euclid's GCD.
- [`BINARY-SEARCH.md`](BINARY-SEARCH.md) covers LeetCode 704: `mid` calculation, array lookup without arrays, the three-way branch, and how the test harness checked all 120 cases.

## Files

```
index.html          the page: all four demos
style.css           base styles plus the while, if/else and GCD machines
binary-search.css   the binary search machine
ARTICLE.md          write-up: while loop, if/else, GCD
BINARY-SEARCH.md    write-up: LeetCode 704
```

## Run it locally

No build step and no dependencies. Clone the repo and open `index.html` in a browser:

```bash
git clone https://github.com/Sidhanth-Mandal/CSS-DSA-Machine.git
cd CSS-DSA-Machine
```

## Browser support

Needs `@property`, `round()` and `color-mix()`: Chrome / Edge 125+, Safari 16.4+, Firefox 128+.

## Is CSS Turing complete?

With a footnote. CSS can run Rule 110, which is known to be Turing complete, but only if something outside the stylesheet advances the state. Here that's your click. The machine also has finite memory (30 values per register), which makes it a finite state machine. So is every real computer.
