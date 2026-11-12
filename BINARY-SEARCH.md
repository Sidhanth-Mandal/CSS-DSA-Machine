# LeetCode 704 in a Stylesheet: Binary Search with Zero JavaScript

> Open `binary-search.html`. Pick a target. Press the button at most four times.
> CSS finds the index, or proves the number isn't there, without a single `<script>` tag.

This is the sequel to `ARTICLE.md`, where we built a while loop, an if/else, and Euclid's GCD out of radio buttons and `calc()`. If you haven't read it, the short version is:

| Part | Built from |
| --- | --- |
| Memory | hidden radio buttons, one group per variable |
| Registers | `@property` integers loaded by `#x-5:checked ~ .machine { --x: 5 }` |
| Arithmetic and comparisons | `calc()`, `clamp()`, `max()`, `round()` |
| Write-back | a strip of `<label for>` elements slid into a one-label window |
| Clock | your click |

Now we point that machine at a real interview question.

---

## 1. The problem

**[704. Binary Search](https://leetcode.com/problems/binary-search/)** (Easy)

> Given an array of integers `nums` sorted in ascending order and an integer `target`, return the index of `target`, or `-1` if it does not exist. Your algorithm must run in `O(log n)` time.

The standard solution:

```java
public int search(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

### Why this problem fits a CSS machine

Our machine has a hard limit. **One click checks one radio button, so each step can write one variable.**

Plenty of classic problems break that rule. Fibonacci / Climbing Stairs updates two variables per step (`a, b = b, a + b`). Two Sum with a hash map needs a growing data structure. The fast version of Euclid updates both registers at once.

Binary search writes exactly one variable per iteration: `lo` **or** `hi`, never both. `mid` is never stored at all, because it can be recomputed from `lo` and `hi`. That makes it a perfect fit, and it still has everything that makes it a good DSA problem: a loop invariant, an off-by-one trap, a three-way branch, and logarithmic running time you can count on your fingers.

---

## 2. Memory

Four radio groups:

| Variable | Range | Why that range |
| --- | --- | --- |
| `nums` | 3 preset datasets | sorted input, length 10 |
| `target` | 1 to 40 | covers every value in the datasets, plus plenty that are missing |
| `lo` | 0 to 10 | `lo` can run one past the last index |
| `hi` | −1 to 9 | `hi` can run one before the first index |

The extra values on `lo` and `hi` are the whole point of `while (lo <= hi)`. When the target is missing, the pointers cross, and the machine must be able to store that crossed state. A radio with id `hi-m1` holds the value −1.

### Two forms, one machine

The inputs (`nums`, `target`) and the search state (`lo`, `hi`) belong to **different forms**, using the HTML `form` attribute:

```html
<input type="radio" form="bs-cfg" name="t"  id="t-23" checked>
<input type="radio" form="bs-run" name="lo" id="lo-0" checked>
...
<form id="bs-cfg"></form>
<form id="bs-run"></form>

<button type="reset" form="bs-run">↺ restart search</button>
```

All the radios still sit next to `.machine` so the `~` selector works. But "restart search" resets only `lo` and `hi`, and your chosen target survives. Resetting part of the state with no JavaScript is a nice trick to keep around.

### The dataset is a register bank

Each preset loads ten registers at once:

```css
#ds-1:checked ~ .machine {
  --v0: 2; --v1: 5; --v2: 8; --v3: 12; --v4: 16;
  --v5: 23; --v6: 27; --v7: 31; --v8: 38; --v9: 40;
}
```

---

## 3. The loop condition: `while (lo <= hi)`

Same trick as last time. Integers plus `clamp` give a boolean:

```css
--active: clamp(0, var(--hi) - var(--lo) + 1, 1);
```

`hi - lo + 1` is the size of the search space. It is at least 1 while there's anything left to search and 0 or less once the pointers cross.

---

## 4. `mid = lo + (hi - lo) / 2`

Java's `/` on integers rounds toward zero. CSS has had a rounding function for a while now:

```css
--mid: calc(var(--lo) + round(down, (var(--hi) - var(--lo)) / 2, 1));
```

`round(down, x, 1)` is `floor(x)`. Since `hi - lo` is never negative inside the loop, floor and truncation agree.

> **About the `lo + (hi - lo) / 2` form.** It exists in Java because `(lo + hi) / 2` can overflow when both are near `2³¹`. Our registers top out at 10, so overflow can't happen here. The machine keeps the safe form anyway so it matches the code you'd write in an interview.

---

## 5. `nums[mid]`: indexing without indexing

This is the most interesting part. CSS has no arrays and no way to say "the custom property whose name is `--v` followed by the value of `--mid`".

So we build what hardware builds: a **multiplexer**. For each slot `i` we compute a selector bit that is 1 only when `mid == i`:

```css
--k3: max(0, 1 - max(var(--mid) - 3, 3 - var(--mid)));   /* max(0, 1 - |mid - 3|) */
```

| mid | \|mid − 3\| | 1 − that | k3 |
| --- | --- | --- | --- |
| 2 | 1 | 0 | 0 |
| 3 | 0 | 1 | **1** |
| 4 | 1 | 0 | 0 |

`max(a - b, b - a)` is the absolute value, built from a function that every browser has supported for years. Then a weighted sum picks out one element:

```css
--m: calc(
    var(--k0) * var(--v0) + var(--k1) * var(--v1) + var(--k2) * var(--v2)
  + ...
  + var(--k9) * var(--v9));
```

Exactly one `k` is 1, so `--m` equals `nums[mid]`. Reading an array element costs ten multiplications. That's an `O(n)` lookup per step, but it happens in parallel inside the style engine, the same way a RAM chip's address decoder checks every row at once.

---

## 6. The three-way branch

```css
--lt: clamp(0, var(--t) - var(--m), 1);    /* nums[mid] < target  */
--gt: clamp(0, var(--m) - var(--t), 1);    /* nums[mid] > target  */
--eq: calc(1 - var(--lt) - var(--gt));     /* nums[mid] == target */
```

At most one of `lt` and `gt` can be 1, so `eq` is 1 exactly when neither is. Combine them with the loop condition and you get one flag per path through the code:

```css
--found:   calc(var(--active) * var(--eq));   /* if       → return mid */
--right:   calc(var(--active) * var(--lt));   /* else if  → lo = mid+1 */
--left:    calc(var(--active) * var(--gt));   /* else     → hi = mid-1 */
--missing: calc(1 - var(--active));           /* loop exit → return -1 */
```

Multiplying two 0/1 flags is a logical AND. These four flags are mutually exclusive, and exactly one is always 1. Every gate on the page reads one of them: the highlighted line of code, the "next instruction" text, the step strips, and the two "done" boxes.

The step button exists only while there's a write to perform:

```css
--run: calc(var(--right) + var(--left));
.step { scale: var(--run); }
```

`return mid` and `return -1` aren't writes, so they don't need a button. They just switch on the matching result box.

---

## 7. Write-back: `lo = mid + 1` or `hi = mid - 1`

Two label strips share the step window, and only the strip for the live branch has non-zero scale:

```css
.strip-lo {
  scale: var(--right);
  translate: 0 calc((var(--mid) + 1) * var(--h) * -1);
}
.strip-hi {
  scale: var(--left);
  translate: 0 calc(var(--mid) * var(--h) * -1);
}
```

The `lo` strip lists the labels for values 0, 1, 2, …, so value `mid + 1` sits at row `mid + 1`. The `hi` strip starts at −1, so value `mid - 1` sits at row `mid`. That's the only off-by-one you have to think about, and it lives in the layout, not the logic.

---

## 8. The picture: every cell works out its own role

The array view has no per-state rules either. Each slot knows its own index (`style="--i: 6"`) and computes everything else:

```css
.slot {
  --inr:   calc(clamp(0, var(--i) - var(--lo) + 1, 1)
              * clamp(0, var(--hi) - var(--i) + 1, 1));      /* lo <= i <= hi */
  --isMid: calc(max(0, 1 - max(var(--mid) - var(--i), var(--i) - var(--mid)))
              * var(--active));
}
.slot .box { opacity: calc(.22 + .78 * var(--inr)); }       /* discarded cells fade */
.p-mid    { scale: var(--isMid); }                          /* the "mid" pointer   */
```

When you click, half of the remaining cells fade out at once. That fade is the `O(log n)` behaviour, shown directly.

The input panel also locks itself once the search starts:

```css
--busy: clamp(0, var(--lo) + 9 - var(--hi), 1);   /* 0 only at lo = 0, hi = 9 */
.lock { scale: var(--busy); }                      /* overlay covers the inputs */
```

Changing the target halfway through a search would break the loop invariant. Real code protects that with encapsulation. The machine protects it by covering the inputs with an overlay until you restart.

---

## 9. A trace

Dataset A, `nums = [2, 5, 8, 12, 16, 23, 27, 31, 38, 40]`, target **24** (not present):

| click | lo | hi | mid | nums[mid] | branch | write |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 0 | 9 | 4 | 16 | 16 < 24 | lo ← 5 |
| 2 | 5 | 9 | 7 | 31 | 31 > 24 | hi ← 6 |
| 3 | 5 | 6 | 5 | 23 | 23 < 24 | lo ← 6 |
| 4 | 6 | 6 | 6 | 27 | 27 > 24 | hi ← 5 |
| – | 6 | 5 | – | – | lo > hi | **return −1** |

Target **23** finishes after two clicks: at `lo = 5, hi = 6`, `mid = 5` and `nums[5] == 23`, so the machine shows `return 5` with no third click.

### Verification

I tested the page in headless Chrome with a harness that, for **all 120 cases** (3 datasets × targets 1 to 40), reset the machine and then repeatedly clicked whatever element sat at the centre of the step button until the button disappeared. For every case, it compared three things against a reference JavaScript implementation:

- the returned index (or −1), read from the computed `--found`, `--missing` and `--mid` properties
- the number of clicks, which must equal the number of `lo`/`hi` updates the reference performs
- whether the correct result box was visible

All 120 cases passed. No case needed more than 4 clicks, which matches `⌊log₂ 10⌋ + 1 = 4`. As before, the test used JavaScript to press the button. The page itself has none.

---

## 10. Things worth noticing

**`mid` is derived, not stored.** It's a pure function of `lo` and `hi`, so it lives in CSS and costs no memory and no clicks. Recomputing a value instead of storing it is the same choice functional programmers make.

**The invariant shows on screen.** Binary search is correct because "if the target exists, it is inside `[lo, hi]`" stays true after every step. On the page, `[lo, hi]` is the set of cells that are still lit. You can watch the invariant hold.

**Termination shows too.** `hi − lo + 1` has its own register and drops by at least half (rounded down) on every click. A value that keeps shrinking can't shrink forever, so the button has to disappear.

**`O(n)` hardware, `O(log n)` algorithm.** The multiplexer touches all ten slots on every step, but the *number of steps* is logarithmic. Hardware cost and algorithmic cost are different things, and this machine shows both side by side.

---

## 11. Challenges

1. **LeetCode 35, Search Insert Position.** Same loop, but return `lo` instead of −1. That's a one-line change to the "missing" box: show `lo`.
2. **LeetCode 278, First Bad Version.** Replace `nums` with a 0/1 array and the three-way branch with a two-way one (`hi = mid` vs `lo = mid + 1`). Watch out: the loop condition becomes `lo < hi`.
3. **LeetCode 69, Sqrt(x).** Replace `nums[mid]` with `mid * mid`. That's easier than the original, because no multiplexer is needed.
4. **Bigger arrays.** Going to 20 elements means 20 `--k` lines and 21-value `lo`/`hi` groups. How many clicks does the worst case take now?

Browser support: needs `@property`, `round()`, and `color-mix()`: Chrome/Edge 125+, Safari 16.4+, Firefox 128+.
