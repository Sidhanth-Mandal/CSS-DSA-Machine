My junior said he couldn't do DSA because he only knew HTML and CSS. So I solved a LeetCode problem using only HTML and CSS.

It started with a simple question.
I asked one of my juniors, "Are you doing DSA yet?"
He said, "Not yet. I only know HTML and CSS."

That stuck with me. Everyone treats HTML and CSS as the "not real programming" step before you learn a proper language. So I wondered: is it actually possible to solve a DSA question with nothing but HTML and CSS?

No JavaScript. No <script> tag. Not one line.

Turns out it is. Here's how it went.

🧱 Problem 1: CSS can't store anything.
Then I remembered radio buttons. In a group, only one can be checked at a time. So a group of 10 hidden radios is really a variable that holds one number from 0 to 9. HTML became my memory.

🔢 Problem 2: CSS can't do logic.
But it can do math. A rule like #a-5:checked ~ .machine { --a: 5 } loads memory into a CSS variable, and @property makes that variable a real integer.
CSS has no ">" operator, so I built one: clamp(0, a − b, 1) gives 1 when a > b and 0 otherwise. One bit. And an element with scale: 0 can't be seen or clicked, so setting scale to that bit gave me an if statement.

🔁 Problem 3: CSS can't change its own memory.
It can work out the next value, but it can't save it. So I stacked a column of <label> buttons behind a window one button tall. CSS slides the label for the correct next value under your cursor, you click it, and the radio changes.
CSS is the processor, and you are the clock. When the loop ends, the button simply disappears.

With that, I had a while loop. Then an if/else. Then Euclid's GCD algorithm from ~300 BC. Then the real goal: LeetCode 704, Binary Search.

🧩 Problem 4: CSS has no arrays.
Binary search needs nums[mid]. I borrowed a trick from hardware called a multiplexer. For every index i, CSS computes a bit that is 1 only when mid == i, then adds up bit × value across all 10 slots. Only one term survives, and that's nums[mid].

Then I tested it properly: all 120 cases (3 arrays × targets 1 to 40) in headless Chrome, checked against a normal solution. Every case passed, and none took more than 4 clicks, which is exactly ⌊log₂ 10⌋ + 1. That's O(log n), happening in a stylesheet.

Will this ever go to production? Never. But building it taught me more about how a CPU makes decisions, and why binary search works, than any tutorial I've read.

So to my junior, and to anyone who says "I only know HTML and CSS": you already know enough to start thinking like a programmer. DSA is about the logic, not the language.

Try it yourself. Press the button until it disappears, then view source and look for the JavaScript 👇
🔗 https://sidhanth-mandal.github.io/css-machine/

#CSS #HTML #WebDevelopment #LeetCode #DSA #ComputerScience #LearningInPublic
