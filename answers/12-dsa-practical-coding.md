---
title: DSA & Practical Coding — Answers
topic: dsa
tags: [interview, dsa, javascript]
related: ["[[00-javascript]]"]
---

# DSA & Practical Coding — Answers

> [!abstract] How to use this note
> Two tracks: **Track A** (classic DSA warm-ups) and **Track B** (practical JS machine-coding). Each item embeds the **question**, a plain-English **explanation**, working **code**, and **Pros / Cons** (including time/space complexity for DSA). Numbers match [[questions/12-dsa-practical-coding|the questions file]].

---

## Track A: DSA fundamentals

### Beginner

### 1. Reverse a string / check palindrome

> [!question] Q1
> Reverse a string / check if a string is a palindrome.

To **reverse** a string, walk it from end to start (or split into an array, reverse, and join). To check a **palindrome**, compare characters from both ends moving inward — if every pair matches, the string reads the same forwards and backwards.

> [!example]
> ```js
> // Reverse — O(n) time, O(n) space (array copy)
> const reverse = (s) => [...s].reverse().join('');
>
> // Palindrome — two pointers, O(n) time, O(1) space
> function isPalindrome(s) {
>   for (let i = 0, j = s.length - 1; i < j; i++, j--) {
>     if (s[i] !== s[j]) return false;
>   }
>   return true;
> }
>
> isPalindrome('racecar'); // true
> isPalindrome('hello');   // false
> reverse('hello');        // 'olleh'
> ```

> [!success] Pros / Cons
> **Two-pointer palindrome:** O(n) time, O(1) space — best for interviews.
> **Array reverse:** O(n) time, O(n) space — concise in JS but uses extra memory.
> **Alternative:** recursive palindrome check — elegant but O(n) stack space.
> **Edge cases:** empty string (palindrome), single char, case sensitivity (`toLowerCase()` if needed), ignore non-alphanumeric for "real" palindrome problems.

> [!info] Further study
> - [LeetCode 125: Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)
> - [LeetCode 344: Reverse String](https://leetcode.com/problems/reverse-string/)

### 2. Find maximum / minimum in an array

> [!question] Q2
> Find the maximum/minimum in an array. What is time complexity?

Scan the array once, keeping track of the best value seen so far. Every element is compared exactly once, so this is **O(n) time** and **O(1) extra space**. You cannot do better than O(n) without extra structure because you must look at every element at least once.

> [!example]
> ```js
> function maxInArray(arr) {
>   if (arr.length === 0) return undefined;
>   let max = arr[0];
>   for (let i = 1; i < arr.length; i++) {
>     if (arr[i] > max) max = arr[i];
>   }
>   return max;
> }
>
> function minInArray(arr) {
>   let min = arr[0];
>   for (const n of arr) if (n < min) min = n;
>   return min;
> }
>
> // Works but risky on huge arrays (spread blows the call stack):
> // Math.max(...arr)
> ```

> [!success] Pros / Cons
> **Single-pass loop:** O(n) time, O(1) space — safe for any array size.
> **`Math.max(...arr)`:** O(n) time but can hit **stack limits** on very large arrays; fine for small inputs.
> **Sorting first:** O(n log n) — overkill when you only need min/max.
> **Edge cases:** empty array, all equal values, `NaN` (comparisons behave oddly — validate if needed).

### 3. Check if two strings are anagrams

> [!question] Q3
> Check if two strings are anagrams.

Two strings are **anagrams** if they contain exactly the same characters with the same frequencies (order does not matter). Quick reject: different lengths cannot be anagrams. Then either **sort both strings** and compare, or **count characters** in a frequency map.

> [!example]
> ```js
> // Map approach — O(n) time, O(k) space (k = unique chars)
> function isAnagram(a, b) {
>   if (a.length !== b.length) return false;
>   const counts = {};
>   for (const c of a) counts[c] = (counts[c] || 0) + 1;
>   for (const c of b) {
>     if (!counts[c]) return false;
>     counts[c]--;
>   }
>   return true;
> }
>
> // Sort approach — O(n log n) time, O(n) space
> const isAnagramSort = (a, b) =>
>   a.length === b.length && [...a].sort().join('') === [...b].sort().join('');
>
> isAnagram('listen', 'silent'); // true
> ```

> [!success] Pros / Cons
> **Frequency map:** O(n) time — preferred for interviews.
> **Sorting:** O(n log n) — simpler code, slower on long strings.
> **Edge cases:** empty strings (anagrams of each other), Unicode/code points (for production, normalize or use `Intl`).

> [!info] Further study
> - [LeetCode 242: Valid Anagram](https://leetcode.com/problems/valid-anagram/)
> - [LeetCode 49: Group Anagrams](https://leetcode.com/problems/group-anagrams/) (Q9)

### 4. Remove duplicates from an array

> [!question] Q4
> Remove duplicates from an array.

Keep only the **first occurrence** of each value while preserving order. A `Set` tracks what you have seen; filter or spread to build the result.

> [!example]
> ```js
> // Preserves order — O(n) time, O(n) space
> const removeDuplicates = (arr) => [...new Set(arr)];
>
> // Manual version (shows the logic)
> function removeDuplicatesManual(arr) {
>   const seen = new Set();
>   const out = [];
>   for (const x of arr) {
>     if (!seen.has(x)) { seen.add(x); out.push(x); }
>   }
>   return out;
> }
>
> removeDuplicates([1, 2, 2, 3, 1, 4]); // [1, 2, 3, 4]
> ```

> [!success] Pros / Cons
> **`Set` spread:** O(n) time, O(n) space — idiomatic JS, preserves insertion order (ES2015+).
> **Sort + filter:** O(n log n) — only if you already need sorted output.
> **In-place (sorted array):** two-pointer dedup — O(n) but requires sorted input.
> **Objects vs primitives:** `Set` uses SameValueZero equality; duplicate objects `{a:1}` and `{a:1}` are **not** deduped.

### 5. FizzBuzz

> [!question] Q5
> FizzBuzz.

Classic warm-up: for each number 1..n, print the number unless it is divisible by 3 ("Fizz"), 5 ("Buzz"), or both 15 ("FizzBuzz"). **Check 15 first** — otherwise 3 and 5 catch it incorrectly.

> [!example]
> ```js
> function fizzBuzz(n) {
>   const out = [];
>   for (let i = 1; i <= n; i++) {
>     if (i % 15 === 0) out.push('FizzBuzz');
>     else if (i % 3 === 0) out.push('Fizz');
>     else if (i % 5 === 0) out.push('Buzz');
>     else out.push(String(i));
>   }
>   return out;
> }
>
> fizzBuzz(15);
> // ['1','2','Fizz','4','Buzz','Fizz','7','8','Fizz','Buzz','11','Fizz','13','14','FizzBuzz']
> ```

> [!success] Pros / Cons
> **Modulo chain:** O(n) time, O(n) space for output — clear and correct.
> **String-building trick** (`i%3?'':'Fizz'`) — fewer branches, harder to read.
> **Why interviewers ask:** tests control flow, edge awareness, and whether you over-engineer a simple problem.

### 6. Big-O notation

> [!question] Q6
> What is Big-O notation? Give examples of O(1), O(n), O(log n), O(n²).

Big-O describes how **runtime or memory grows** as input size `n` increases — the **worst-case** dominant term, ignoring constants. It answers: "If I double the input, what happens to cost?"

> [!example]
> ```js
> // O(1) — hash map lookup
> const map = new Map([['key', 42]]);
> map.get('key');
>
> // O(log n) — binary search on sorted array
> function binarySearch(arr, target) {
>   let lo = 0, hi = arr.length - 1;
>   while (lo <= hi) {
>     const mid = lo + ((hi - lo) >> 1);
>     if (arr[mid] === target) return mid;
>     if (arr[mid] < target) lo = mid + 1; else hi = mid - 1;
>   }
>   return -1;
> }
>
> // O(n) — single loop
> function sum(arr) { let s = 0; for (const x of arr) s += x; return s; }
>
> // O(n²) — nested loops
> function hasDuplicatePair(arr) {
>   for (let i = 0; i < arr.length; i++)
>     for (let j = i + 1; j < arr.length; j++)
>       if (arr[i] === arr[j]) return true;
>   return false;
> }
> ```

> [!success] Pros / Cons
> **Common complexities:** O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ).
> **Drop constants:** O(2n) → O(n); keep the **fastest-growing** term only.
> **Space vs time:** an O(n) hash map trades memory for speed — always mention both when asked.
> **Interview tip:** state complexity **and** why (e.g. "one pass + map lookup = O(n)").

> [!info] Further study
> - [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
> - [LeetCode Explore: Time Complexity](https://leetcode.com/explore/learn/card/time-complexity/)

---

### Intermediate

### 7. Two Sum

> [!question] Q7
> Two Sum — find two numbers that add to a target. (extremely common — Amazon, Google)

For each number, ask: "Have I already seen `target - num`?" Store each value and its index in a **hash map**. One pass gives the answer in O(n) time instead of checking every pair (O(n²)).

> [!example]
> ```js
> function twoSum(nums, target) {
>   const seen = new Map(); // value -> index
>   for (let i = 0; i < nums.length; i++) {
>     const need = target - nums[i];
>     if (seen.has(need)) return [seen.get(need), i];
>     seen.set(nums[i], i);
>   }
>   return null;
> }
>
> twoSum([2, 7, 11, 15], 9); // [0, 1]
> ```

> [!success] Pros / Cons
> **Hash map:** O(n) time, O(n) space — industry standard.
> **Nested loops:** O(n²) time, O(1) space — fine for tiny arrays, fails interview follow-ups.
> **Two-pointer (sorted):** O(n log n) if you sort first — use when input is already sorted.
> **Edge cases:** no solution, duplicate values, same element used twice (store after check).

> [!info] Further study
> - [LeetCode 1: Two Sum](https://leetcode.com/problems/two-sum/) — #1 most common warm-up

### 8. First non-repeating character

> [!question] Q8
> Find the first non-repeating character in a string.

**Pass 1:** count every character in a map. **Pass 2:** scan the string left-to-right and return the first character with count 1. Two passes, linear time.

> [!example]
> ```js
> function firstNonRepeating(s) {
>   const counts = new Map();
>   for (const c of s) counts.set(c, (counts.get(c) || 0) + 1);
>   for (const c of s) {
>     if (counts.get(c) === 1) return c;
>   }
>   return null;
> }
>
> firstNonRepeating('leetcode');  // 'l'
> firstNonRepeating('aabb');      // null
> ```

> [!success] Pros / Cons
> **Two-pass map:** O(n) time, O(k) space (k = unique chars) — clear and optimal.
> **One-pass with index map:** track first index + count — same complexity, slightly more code.
> **Sort + scan:** O(n log n) — avoid unless required.
> **Edge cases:** empty string, all repeating chars.

> [!info] Further study
> - [LeetCode 387: First Unique Character in a String](https://leetcode.com/problems/first-unique-character-in-a-string/)

### 9. Group anagrams

> [!question] Q9
> Group anagrams together.

Anagrams share the same **signature** — sorted letters or a character-count tuple. Use the signature as a hash key and push each word into its bucket.

> [!example]
> ```js
> function groupAnagrams(words) {
>   const map = {};
>   for (const w of words) {
>     const key = [...w].sort().join('');
>     (map[key] ||= []).push(w);
>   }
>   return Object.values(map);
> }
>
> groupAnagrams(['eat', 'tea', 'tan', 'ate', 'nat', 'bat']);
> // [['eat','tea','ate'], ['tan','nat'], ['bat']]
> ```

> [!success] Pros / Cons
> **Sort as key:** O(n · k log k) where k = max word length — simple.
> **Char-count key** (26-letter tally): O(n · k) — faster for long words / large alphabets.
> **Space:** O(n · k) for storing all strings in buckets.
> **Edge cases:** empty strings, single-char words, duplicate words in input.

> [!info] Further study
> - [LeetCode 49: Group Anagrams](https://leetcode.com/problems/group-anagrams/)

### 10. Valid parentheses (stack)

> [!question] Q10
> Valid parentheses (balanced brackets) using a stack.

Walk the string. On an **opening** bracket, push it. On a **closing** bracket, pop and verify it matches. Valid only if the stack is **empty** at the end (every open was closed in correct order).

> [!example]
> ```js
> function isValid(s) {
>   const stack = [];
>   const pairs = { ')': '(', ']': '[', '}': '{' };
>   for (const c of s) {
>     if ('({['.includes(c)) stack.push(c);
>     else {
>       if (stack.pop() !== pairs[c]) return false;
>     }
>   }
>   return stack.length === 0;
> }
>
> isValid('()[]{}');  // true
> isValid('([)]');    // false
> ```

> [!success] Pros / Cons
> **Stack:** O(n) time, O(n) space — canonical solution.
> **Counter-only (single bracket type):** O(1) space — does not generalize to mixed types.
> **Edge cases:** empty string (valid), odd length (invalid), closing before opening.

> [!info] Further study
> - [LeetCode 20: Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

### 11. Merge two sorted arrays / linked lists

> [!question] Q11
> Merge two sorted arrays / linked lists.

Use **two pointers**, one per list. At each step, take the smaller head element and advance that pointer. Append leftovers when one list is exhausted.

> [!example]
> ```js
> // Merge sorted arrays — O(n + m) time, O(n + m) space
> function mergeSorted(a, b) {
>   const out = [];
>   let i = 0, j = 0;
>   while (i < a.length && j < b.length) {
>     if (a[i] <= b[j]) out.push(a[i++]);
>     else out.push(b[j++]);
>   }
>   return out.concat(a.slice(i), b.slice(j));
> }
>
> // Merge sorted linked lists — O(n + m) time, O(1) extra space
> function mergeLists(l1, l2) {
>   const dummy = { next: null };
>   let tail = dummy;
>   while (l1 && l2) {
>     if (l1.val <= l2.val) { tail.next = l1; l1 = l1.next; }
>     else { tail.next = l2; l2 = l2.next; }
>     tail = tail.next;
>   }
>   tail.next = l1 || l2;
>   return dummy.next;
> }
>
> mergeSorted([1, 3, 5], [2, 4, 6]); // [1,2,3,4,5,6]
> ```

> [!success] Pros / Cons
> **Two-pointer merge:** O(n + m) time — optimal; cannot beat linear since every element is visited once.
> **Arrays:** O(n + m) extra space for output (or in-place from the end for "merge into nums1" variants).
> **Linked lists:** O(1) extra space — rewire pointers, no new nodes.
> **Alternative:** recursive merge — same complexity, O(n) call stack.

> [!info] Further study
> - [LeetCode 21: Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)
> - [LeetCode 88: Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/)

### 12. Maximum subarray sum (Kadane's)

> [!question] Q12
> Maximum subarray sum (Kadane's algorithm).

Track a **running sum** of the current subarray. If the running sum goes negative, drop it and start fresh at the current element — a negative prefix can never help a future maximum. Keep the best sum seen.

> [!example]
> ```js
> function maxSubArray(nums) {
>   let best = nums[0];
>   let current = nums[0];
>   for (let i = 1; i < nums.length; i++) {
>     current = Math.max(nums[i], current + nums[i]);
>     best = Math.max(best, current);
>   }
>   return best;
> }
>
> maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]); // 6  (subarray [4,-1,2,1])
> ```

> [!success] Pros / Cons
> **Kadane's:** O(n) time, O(1) space — optimal.
> **Brute force (all subarrays):** O(n²) or O(n³) — too slow for large inputs.
> **Divide & conquer:** O(n log n) — educational, not needed in interviews.
> **Edge cases:** all negative numbers (returns the least negative single element).

> [!info] Further study
> - [LeetCode 53: Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)

### 13. Move all zeros to the end (two pointers)

> [!question] Q13
> Move all zeros to the end of an array (two pointers).

Use a **write pointer** for the next non-zero slot. Scan with a read pointer; swap non-zeros forward. Zeros naturally end up at the tail. In-place, single pass.

> [!example]
> ```js
> function moveZeroes(nums) {
>   let write = 0;
>   for (let read = 0; read < nums.length; read++) {
>     if (nums[read] !== 0) {
>       [nums[write], nums[read]] = [nums[read], nums[write]];
>       write++;
>     }
>   }
>   return nums;
> }
>
> moveZeroes([0, 1, 0, 3, 12]); // [1, 3, 12, 0, 0]
> ```

> [!success] Pros / Cons
> **Two-pointer in-place:** O(n) time, O(1) space — preferred.
> **Filter + concat:** O(n) time, O(n) space — `[...nonZeros, ...zeros]` — simpler but not in-place.
> **Minimize swaps:** only swap when `write !== read`.
> **Edge cases:** no zeros, all zeros, already sorted.

> [!info] Further study
> - [LeetCode 283: Move Zeroes](https://leetcode.com/problems/move-zeroes/)

### 14. Find duplicates in O(n) time

> [!question] Q14
> Find duplicates in an array in O(n) time.

Use a **Set** (or Map) to track seen values. On second sight of a value, it is a duplicate. One pass, linear time.

> [!example]
> ```js
> function findDuplicates(arr) {
>   const seen = new Set();
>   const dupes = new Set();
>   for (const x of arr) {
>     if (seen.has(x)) dupes.add(x);
>     else seen.add(x);
>   }
>   return [...dupes];
> }
>
> findDuplicates([1, 2, 3, 2, 4, 5, 4]); // [2, 4]
> ```

> [!success] Pros / Cons
> **Set:** O(n) time, O(n) space — general purpose.
> **Floyd's cycle (1..n integers):** O(n) time, O(1) space — classic follow-up when values are in range [1, n].
> **Sort + scan:** O(n log n) time, O(1) space if in-place sort allowed.
> **Nested loops:** O(n²) — avoid.

> [!info] Further study
> - [LeetCode 287: Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) (O(1) space variant)

---

### Advanced

### 15. Longest substring without repeating characters

> [!question] Q15
> Longest substring without repeating characters (sliding window). (commonly asked at Amazon)

Maintain a **sliding window** `[start, i]` with a map of each character's **last seen index**. When you see a repeat inside the window, jump `start` to `lastIndex + 1`. Track the maximum window length.

> [!example]
> ```js
> function lengthOfLongestSubstring(s) {
>   const last = new Map();
>   let start = 0, max = 0;
>   for (let i = 0; i < s.length; i++) {
>     if (last.has(s[i]) && last.get(s[i]) >= start) {
>       start = last.get(s[i]) + 1;
>     }
>     last.set(s[i], i);
>     max = Math.max(max, i - start + 1);
>   }
>   return max;
> }
>
> lengthOfLongestSubstring('abcabcbb'); // 3 ('abc')
> lengthOfLongestSubstring('bbbbb');    // 1
> ```

> [!success] Pros / Cons
> **Sliding window + map:** O(n) time, O(min(n, alphabet)) space — optimal.
> **Set + shrink window:** O(n) time — move `start` one step at a time; easier to code, same big-O.
> **Brute force (all substrings):** O(n²) or O(n³) — too slow.
> **Edge cases:** empty string (0), all unique (length = n).

> [!info] Further study
> - [LeetCode 3: Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

### 16. Debounced typeahead search

> [!question] Q16
> Explain and implement a debounce-based typeahead search.

**Typeahead** suggests results as the user types. Without control, every keystroke fires an API call — slow, expensive, and prone to **race conditions** (stale responses overwrite fresh ones). **Debounce** waits until the user **pauses** typing, then fires one request. Combine with **AbortController** to cancel in-flight requests when a newer query starts.

> [!example]
> ```js
> function debounce(fn, delay) {
>   let timer;
>   return function (...args) {
>     clearTimeout(timer);
>     timer = setTimeout(() => fn.apply(this, args), delay);
>   };
> }
>
> function createTypeahead({ fetchSuggestions, onResults, delay = 300 }) {
>   let controller = null;
>
>   const search = debounce(async (query) => {
>     controller?.abort();
>     controller = new AbortController();
>     try {
>       const results = await fetchSuggestions(query, { signal: controller.signal });
>       onResults(results);
>     } catch (err) {
>       if (err.name !== 'AbortError') throw err;
>     }
>   }, delay);
>
>   return (query) => {
>     if (!query.trim()) { onResults([]); return; }
>     search(query);
>   };
> }
>
> // Usage
> const typeahead = createTypeahead({
>   fetchSuggestions: (q, opts) =>
>     fetch(`/api/search?q=${encodeURIComponent(q)}`, opts).then(r => r.json()),
>   onResults: (items) => console.log(items),
> });
> ```

> [!success] Pros / Cons
> **Debounce + abort:** cuts API load, prevents stale UI — production pattern.
> **Debounce alone:** still better than raw input, but races possible without cancellation.
> **Throttle instead:** fires regularly during typing — smoother UI updates, more requests than debounce.
> **Complexity:** debounce wrapper O(1) per keystroke; network dominates real cost.
> **See also:** [[00-javascript#31 Implement debounce|debounce with leading/trailing]] in JavaScript notes.

> [!tip] Interview talking points
> Mention loading states, empty query handling, minimum character threshold, and caching recent queries.

### 17. Binary search

> [!question] Q17
> Binary search on a sorted array; discuss edge cases.

Repeatedly compare `target` to the **middle** element. If not equal, discard the half that cannot contain the target. Each step halves the search space → **O(log n)**.

> [!example]
> ```js
> function binarySearch(arr, target) {
>   let lo = 0, hi = arr.length - 1;
>   while (lo <= hi) {
>     const mid = lo + Math.floor((hi - lo) / 2); // avoids overflow in other langs
>     if (arr[mid] === target) return mid;
>     if (arr[mid] < target) lo = mid + 1;
>     else hi = mid - 1;
>   }
>   return -1;
> }
>
> binarySearch([1, 3, 5, 7, 9], 7); // 3
> binarySearch([1, 3, 5, 7, 9], 4); // -1
> ```

> [!success] Pros / Cons
> **Standard loop:** O(log n) time, O(1) space.
> **Edge cases:** empty array, single element, target not present, **duplicates** (need leftmost/rightmost variant), integer overflow (`mid = lo + (hi-lo)/2`).
> **Prerequisite:** array must be sorted — O(n log n) to sort first if not.
> **vs linear scan:** O(n) — binary search wins on large sorted data.

> [!info] Further study
> - [LeetCode 704: Binary Search](https://leetcode.com/problems/binary-search/)
> - [LeetCode 34: Find First and Last Position](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) (duplicate handling)

### 18. LRU cache

> [!question] Q18
> LRU cache — design and implement. (commonly asked at Amazon, Meta)

An **LRU (Least Recently Used) cache** evicts the stalest entry when capacity is exceeded. Requirements: **O(1) get** and **O(1) put**. Classic design: **hash map** (key → node) + **doubly linked list** (recency order). In modern JS interviews, `Map` insertion order is often accepted as a shortcut.

> [!example]
> ```js
> class LRUCache {
>   constructor(capacity) {
>     this.cap = capacity;
>     this.map = new Map(); // oldest at front, newest at back
>   }
>
>   get(key) {
>     if (!this.map.has(key)) return -1;
>     const val = this.map.get(key);
>     this.map.delete(key);
>     this.map.set(key, val); // move to most-recent
>     return val;
>   }
>
>   put(key, val) {
>     if (this.map.has(key)) this.map.delete(key);
>     else if (this.map.size >= this.cap) {
>       const oldest = this.map.keys().next().value;
>       this.map.delete(oldest);
>     }
>     this.map.set(key, val);
>   }
> }
>
> const cache = new LRUCache(2);
> cache.put(1, 1); cache.put(2, 2);
> cache.get(1);    // 1
> cache.put(3, 3); // evicts key 2
> cache.get(2);    // -1
> ```

> [!success] Pros / Cons
> **Map shortcut:** O(1) amortized get/put — good for JS interviews; `delete` + `set` reorders.
> **Map + DLL:** true O(1) with explicit nodes — what interviewers expect in Java/C++.
> **vs FIFO cache:** LRU keeps "hot" keys; FIFO is simpler but worse hit rate.
> **Space:** O(capacity).

> [!info] Further study
> - [LeetCode 146: LRU Cache](https://leetcode.com/problems/lru-cache/)

### 19. Topological sort (task dependencies)

> [!question] Q19
> Given an array of tasks with dependencies, determine execution order (topological sort).

Model tasks as a **DAG** (directed acyclic graph): edge `A → B` means "A must finish before B". **Topological sort** produces a valid execution order. **Kahn's algorithm** (BFS): compute in-degrees, repeatedly pick nodes with in-degree 0. If not all nodes are processed, there is a **cycle** (impossible schedule).

> [!example]
> ```js
> function topologicalSort(numTasks, edges) {
>   const adj = Array.from({ length: numTasks }, () => []);
>   const indegree = Array(numTasks).fill(0);
>
>   for (const [from, to] of edges) {
>     adj[from].push(to);
>     indegree[to]++;
>   }
>
>   const queue = [];
>   for (let i = 0; i < numTasks; i++) if (indegree[i] === 0) queue.push(i);
>
>   const order = [];
>   while (queue.length) {
>     const node = queue.shift();
>     order.push(node);
>     for (const next of adj[node]) {
>       if (--indegree[next] === 0) queue.push(next);
>     }
>   }
>   return order.length === numTasks ? order : null; // null = cycle
> }
>
> // tasks 0..3; edges: 0→1, 0→2, 1→3, 2→3
> topologicalSort(4, [[0,1],[0,2],[1,3],[2,3]]); // [0,1,2,3] or [0,2,1,3]
> ```

> [!success] Pros / Cons
> **Kahn's BFS:** O(V + E) time and space — intuitive, detects cycles.
> **DFS post-order:** O(V + E) — reverse finish order; needs cycle detection (gray/black nodes).
> **Use cases:** build systems, course prerequisites, job schedulers, package installs.
> **Edge cases:** disconnected components, multiple valid orderings, cyclic dependencies.

> [!info] Further study
> - [LeetCode 207: Course Schedule](https://leetcode.com/problems/course-schedule/)
> - [LeetCode 210: Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)

### 20. Two-pointer: container with most water / trapping rain water

> [!question] Q20
> Two-pointer: container with most water / trapping rain water (conceptual).

Both are classic **two-pointer** problems on height arrays.

**Container With Most Water:** pointers at both ends. Area = `min(height[l], height[r]) × width`. Move the **shorter** side inward (only a taller bar there could improve area). Track max. **O(n) time, O(1) space.**

**Trapping Rain Water:** for each position, water trapped = `min(leftMax, rightMax) - height[i]` (if positive). Two-pointer tracks running left/right maxes as pointers converge. **O(n) time, O(1) space.** Alternative: prefix max arrays — O(n) space, easier to explain.

> [!example]
> ```js
> // Container With Most Water
> function maxArea(height) {
>   let l = 0, r = height.length - 1, best = 0;
>   while (l < r) {
>     best = Math.max(best, Math.min(height[l], height[r]) * (r - l));
>     if (height[l] < height[r]) l++; else r--;
>   }
>   return best;
> }
>
> // Trapping Rain Water — two-pointer
> function trap(height) {
>   let l = 0, r = height.length - 1;
>   let leftMax = 0, rightMax = 0, water = 0;
>   while (l < r) {
>     if (height[l] < height[r]) {
>       leftMax = Math.max(leftMax, height[l]);
>       water += leftMax - height[l];
>       l++;
>     } else {
>       rightMax = Math.max(rightMax, height[r]);
>       water += rightMax - height[r];
>       r--;
>     }
>   }
>   return water;
> }
>
> maxArea([1,8,6,2,5,4,8,3,7]); // 49
> trap([0,1,0,2,1,0,1,3,2,1,2,1]); // 6
> ```

> [!success] Pros / Cons
> **Two-pointer:** O(n) time, O(1) space — optimal for both problems.
> **Brute force (container):** O(n²) — check every pair.
> **Prefix arrays (trap):** O(n) time, O(n) space — simpler logic, extra memory.
> **Stack (trap):** O(n) — alternative approach, good to mention.

> [!info] Further study
> - [LeetCode 11: Container With Most Water](https://leetcode.com/problems/container-with-most-water/)
> - [LeetCode 42: Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

---

## Track B: Practical JS machine-coding

### 21. Implement debounce

> [!question] Q21
> Implement `debounce`. (your CV: relevant to request queueing)

Debounce wraps a function so it runs only after **calls stop** for `delay` milliseconds. Each new call resets the timer. Preserves `this` and arguments via `apply`.

> [!example]
> ```js
> function debounce(fn, delay, { leading = false, trailing = true } = {}) {
>   let timer = null;
>   return function (...args) {
>     const callNow = leading && !timer;
>     clearTimeout(timer);
>     timer = setTimeout(() => {
>       timer = null;
>       if (trailing && !callNow) fn.apply(this, args);
>     }, delay);
>     if (callNow) fn.apply(this, args);
>   };
> }
>
> const log = debounce((x) => console.log(x), 300);
> log('a'); log('b'); log('c'); // prints 'c' once, ~300ms after last call
> ```

> [!success] Pros / Cons
> **Trailing debounce:** fewest executions — ideal for search-as-you-type, resize handlers.
> **Leading debounce:** immediate first call — good for button double-click prevention.
> **Complexity:** O(1) per invocation; actual work deferred.
> **vs throttle:** debounce waits for silence; throttle fires on a fixed interval during activity.
> **See also:** [[00-javascript#31 Implement debounce|Q31 in JavaScript notes]].

> [!info] Further study
> - [Lodash debounce docs](https://lodash.com/docs/#debounce)

### 22. Implement throttle

> [!question] Q22
> Implement `throttle`.

Throttle ensures a function runs **at most once per `limit` ms** while events keep firing — unlike debounce, which waits for a pause. Useful for scroll, mousemove, and rate-limited API polling.

> [!example]
> ```js
> function throttle(fn, limit) {
>   let inThrottle = false;
>   let lastArgs = null;
>   return function (...args) {
>     if (!inThrottle) {
>       fn.apply(this, args);
>       inThrottle = true;
>       setTimeout(() => {
>         inThrottle = false;
>         if (lastArgs) { fn.apply(this, lastArgs); lastArgs = null; }
>       }, limit);
>     } else {
>       lastArgs = args; // optional: trailing call with latest args
>     }
>   };
> }
>
> const onScroll = throttle(() => console.log(window.scrollY), 200);
> ```

> [!success] Pros / Cons
> **Throttle:** guarantees regular updates during continuous events — smooth UI feedback.
> **Debounce:** fewer calls but delayed until activity stops.
> **Complexity:** O(1) per call.
> **Production:** consider `requestAnimationFrame` for visual updates instead of raw throttle.

> [!info] Further study
> - [[00-javascript#22 Debounce vs throttle|Debounce vs throttle in JavaScript notes]]

### 23. Implement deep clone

> [!question] Q23
> Implement a deep clone.

A **deep clone** recursively copies every nested level so the copy is fully independent. Handle **circular references** (WeakMap), and built-ins: `Date`, `RegExp`, `Map`, `Set`, arrays, plain objects.

> [!example]
> ```js
> function deepClone(value, seen = new WeakMap()) {
>   if (value === null || typeof value !== 'object') return value;
>   if (value instanceof Date) return new Date(value);
>   if (value instanceof RegExp) return new RegExp(value);
>   if (seen.has(value)) return seen.get(value);
>
>   if (value instanceof Map) {
>     const m = new Map(); seen.set(value, m);
>     value.forEach((v, k) => m.set(deepClone(k, seen), deepClone(v, seen)));
>     return m;
>   }
>   if (value instanceof Set) {
>     const s = new Set(); seen.set(value, s);
>     value.forEach(v => s.add(deepClone(v, seen)));
>     return s;
>   }
>
>   const out = Array.isArray(value) ? [] : {};
>   seen.set(value, out);
>   for (const key of Reflect.ownKeys(value)) {
>     out[key] = deepClone(value[key], seen);
>   }
>   return out;
> }
>
> const obj = { a: 1, b: { c: 2 } };
> const copy = deepClone(obj);
> copy.b.c = 99;
> obj.b.c; // 2 — independent
> ```

> [!success] Pros / Cons
> **Custom deepClone:** handles cycles and special types — interview-ready.
> **`structuredClone(obj)`:** built-in (modern runtimes) — prefer in production when sufficient.
> **`JSON.parse(JSON.stringify(x))`:** fast but loses `Date`, `Map`, `undefined`, functions, and breaks on cycles.
> **Complexity:** O(n) time and space for n total properties/nodes.

> [!info] Further study
> - [[00-javascript#32 Deep clone|Deep clone in JavaScript notes]]

### 24. Implement Promise.all from scratch

> [!question] Q24
> Implement `Promise.all` from scratch.

Return a new promise that resolves with an **array of all results in order** once every input settles successfully. **Reject immediately** on the first failure (fail-fast).

> [!example]
> ```js
> function promiseAll(promises) {
>   return new Promise((resolve, reject) => {
>     const results = [];
>     let remaining = promises.length;
>     if (remaining === 0) return resolve(results);
>
>     promises.forEach((p, i) => {
>       Promise.resolve(p).then(
>         (value) => {
>           results[i] = value; // preserve input order
>           if (--remaining === 0) resolve(results);
>         },
>         reject // first rejection wins
>       );
>     });
>   });
> }
>
> promiseAll([
>   Promise.resolve(1),
>   Promise.resolve(2),
> ]).then(console.log); // [1, 2]
> ```

> [!success] Pros / Cons
> **Custom all:** parallel execution, ordered results — O(n) callbacks.
> **Native `Promise.all`:** same semantics — use in production.
> **Fail-fast con:** one failure rejects everything — use `Promise.allSettled` when you need every outcome.
> **Edge cases:** empty array (resolves `[]`), non-promise values (wrap with `Promise.resolve`).

> [!info] Further study
> - [[00-javascript#33 Implement Promise.all|Promise.all in JavaScript notes]]

### 25. Promise-based concurrency limiter / promise pool

> [!question] Q25
> Implement a promise-based concurrency limiter / promise pool. (your CV)

Run at most **N async tasks in parallel**. When one finishes, start the next. Prevents overwhelming APIs, DBs, or memory — the same pattern as bounded request queueing on your affiliate platform.

> [!example]
> ```js
> async function promisePool(tasks, limit) {
>   const results = [];
>   const executing = new Set();
>
>   for (let i = 0; i < tasks.length; i++) {
>     const p = Promise.resolve().then(() => tasks[i]());
>     results.push(p);
>     executing.add(p);
>     p.finally(() => executing.delete(p));
>
>     if (executing.size >= limit) {
>       await Promise.race(executing); // wait for a slot
>     }
>   }
>   return Promise.all(results);
> }
>
> // Example: fetch 100 URLs, max 5 at a time
> const tasks = urls.map(url => () => fetch(url).then(r => r.json()));
> await promisePool(tasks, 5);
> ```

> [!success] Pros / Cons
> **Promise pool:** O(n) tasks, bounded concurrency — protects downstream services.
> **`Promise.all` unbounded:** fastest wall-clock time but can overload servers / exhaust sockets.
> **Sequential `for await`:** safest load, slowest throughput.
> **Tuning `limit`:** needs load testing; too low = slow, too high = rate limits / OOM.

> [!tip] CV tie-in
> This is your **"Promise-based request queueing"** pattern — cap concurrent affiliate/API calls and drain as slots free up.

> [!info] Further study
> - [[00-javascript#34 Promise pool / concurrency limiter|Promise pool in JavaScript notes]]
> - [[06-system-design#12 Design a rate limiter|Rate limiter system design]]

### 26. Implement EventEmitter

> [!question] Q26
> Implement an EventEmitter (on/off/emit/once).

Pub/sub pattern: listeners register for named **events**; **emit** calls all listeners synchronously. **once** auto-unsubscribes after the first fire. Node's `EventEmitter` and browser `addEventListener` follow this model.

> [!example]
> ```js
> class EventEmitter {
>   constructor() { this.events = {}; }
>
>   on(event, fn) {
>     (this.events[event] ||= []).push(fn);
>     return this;
>   }
>
>   off(event, fn) {
>     this.events[event] = (this.events[event] || []).filter(f => f !== fn);
>     return this;
>   }
>
>   emit(event, ...args) {
>     (this.events[event] || []).forEach(fn => fn(...args));
>     return this;
>   }
>
>   once(event, fn) {
>     const wrapper = (...args) => { fn(...args); this.off(event, wrapper); };
>     return this.on(event, wrapper);
>   }
> }
>
> const bus = new EventEmitter();
> bus.on('data', (x) => console.log('got', x));
> bus.emit('data', 42); // got 42
> ```

> [!success] Pros / Cons
> **Pros:** decouples producers/consumers, familiar Node pattern, easy to test.
> **Cons:** no built-in error isolation (one listener throw breaks others — wrap in try/catch in production), memory leaks if you forget `off`.
> **Complexity:** O(L) per emit where L = listeners for that event.

### 27. Implement flatten

> [!question] Q27
> Implement `flatten` for a deeply nested array.

Recursively walk the array. If an element is an array, flatten it; otherwise append the value. Base case: non-array values.

> [!example]
> ```js
> function flatten(arr) {
>   return arr.reduce(
>     (acc, x) => acc.concat(Array.isArray(x) ? flatten(x) : x),
>     []
>   );
> }
>
> // Iterative (avoids deep recursion stack)
> function flattenIterative(arr) {
>   const stack = [...arr];
>   const out = [];
>   while (stack.length) {
>     const x = stack.pop();
>     if (Array.isArray(x)) stack.push(...x);
>     else out.push(x);
>   }
>   return out.reverse();
> }
>
> flatten([1, [2, [3, [4]]], 5]); // [1, 2, 3, 4, 5]
> // Native: [1, [2, [3]]].flat(Infinity)
> ```

> [!success] Pros / Cons
> **Recursive reduce:** concise — O(n) total elements; O(depth) stack space.
> **Iterative stack:** O(n) time, avoids stack overflow on very deep nesting.
> **`arr.flat(Infinity)`:** production default in modern JS.
> **Edge cases:** empty arrays, mixed types, sparse arrays.

### 28. Implement memoize

> [!question] Q28
> Implement `memoize`.

Cache a function's **return value** keyed by its arguments. Repeated calls with the same inputs skip recomputation — classic optimization for expensive pure functions (Fibonacci, API-less transforms).

> [!example]
> ```js
> function memoize(fn, resolver = (...args) => JSON.stringify(args)) {
>   const cache = new Map();
>   return function (...args) {
>     const key = resolver(...args);
>     if (cache.has(key)) return cache.get(key);
>     const result = fn.apply(this, args);
>     cache.set(key, result);
>     return result;
>   };
> }
>
> const slowSquare = (n) => { /* imagine expensive */ return n * n; };
> const fastSquare = memoize(slowSquare);
> fastSquare(5); // computes
> fastSquare(5); // cached
> ```

> [!success] Pros / Cons
> **Pros:** huge speedup for repeated inputs; easy to add to pure functions.
> **Cons:** memory grows with unique inputs; wrong for functions with side effects or time-dependent results; `JSON.stringify` keys fail on object key order / functions — use a custom `resolver`.
> **Complexity:** O(1) lookup per call (Map); first call pays full function cost.

> [!info] Further study
> - [[00-javascript#40 memoize with key resolver|Memoize in JavaScript notes]]

### 29. Implement curry

> [!question] Q29
> Implement `curry`.

**Currying** transforms `f(a, b, c)` into `f(a)(b)(c)` — each call fixes one argument until the function has enough args to run. Useful for partial application and functional composition.

> [!example]
> ```js
> function curry(fn) {
>   return function curried(...args) {
>     if (args.length >= fn.length) {
>       return fn.apply(this, args);
>     }
>     return (...more) => curried.apply(this, [...args, ...more]);
>   };
> }
>
> const add = (a, b, c) => a + b + c;
> const curriedAdd = curry(add);
> curriedAdd(1)(2)(3);   // 6
> curriedAdd(1, 2)(3);   // 6 — flexible arity grouping
> ```

> [!success] Pros / Cons
> **Pros:** reusable partial functions, cleaner composition (`compose`, pipelines).
> **Cons:** more call overhead, harder debugging, less idiomatic in imperative JS.
> **Complexity:** O(1) per curried call until enough args collected.
> **Note:** `fn.length` ignores rest parameters — mention if asked.

### 30. Implement retry with exponential backoff

> [!question] Q30
> Implement `retry` with exponential backoff for a failing async function. (your CV: webhooks/retries)

On failure, wait and retry with **doubling delay** (500ms → 1s → 2s …). Handles transient network blips without hammering a struggling service. Essential for webhook delivery and queue workers.

> [!example]
> ```js
> async function retry(fn, { retries = 3, delay = 500, factor = 2 } = {}) {
>   try {
>     return await fn();
>   } catch (err) {
>     if (retries <= 0) throw err;
>     await new Promise(r => setTimeout(r, delay));
>     return retry(fn, { retries: retries - 1, delay: delay * factor, factor });
>   }
> }
>
> // With jitter (production best practice)
> async function retryWithJitter(fn, opts) {
>   const jitter = opts.delay * (0.5 + Math.random() * 0.5);
>   await new Promise(r => setTimeout(r, jitter));
>   return retry(fn, opts);
> }
>
> await retry(() => fetch('/api/webhook'), { retries: 5, delay: 500 });
> ```

> [!success] Pros / Cons
> **Pros:** survives transient 5xx / timeouts; backoff reduces thundering herd.
> **Cons:** non-idempotent ops can **duplicate** side effects — use idempotency keys; cap max delay; add jitter so clients don't retry in sync.
> **vs fixed retry:** exponential gives struggling services time to recover.

> [!tip] CV tie-in
> Same resilience pattern for **webhook/queue processing** — see [[04-nestjs]] and [[06-system-design]].

> [!info] Further study
> - [[00-javascript#P20 Retry with exponential backoff|Retry in JavaScript notes]]

### 31. Implement groupBy

> [!question] Q31
> Implement a function that groups an array of objects by a key (`groupBy`).

Reduce the array into an object (or Map) where each key maps to an array of items sharing that property. Accept a string key or a function for computed grouping.

> [!example]
> ```js
> function groupBy(arr, key) {
>   return arr.reduce((acc, item) => {
>     const k = typeof key === 'function' ? key(item) : item[key];
>     (acc[k] ||= []).push(item);
>     return acc;
>   }, {});
> }
>
> const users = [
>   { name: 'Ana', role: 'admin' },
>   { name: 'Bob', role: 'user' },
>   { name: 'Cal', role: 'admin' },
> ];
> groupBy(users, 'role');
> // { admin: [{...}, {...}], user: [{...}] }
>
> // Native (ES2024): Object.groupBy(users, u => u.role)
> ```

> [!success] Pros / Cons
> **Reduce + object:** O(n) time, O(n) space — standard interview answer.
> **`Map` variant:** better keys when grouping by objects/non-strings.
> **`Object.groupBy`:** native when available.
> **Edge cases:** missing keys (`undefined` bucket), empty array.

### 32. Implement Array.prototype.map / reduce polyfills

> [!question] Q32
> Implement `Array.prototype.map` / `reduce` polyfills.

Understand how built-ins iterate: respect **holes** in sparse arrays (`if (i in this)`), pass `(element, index, array)` to the callback, and honour `thisArg`.

> [!example]
> ```js
> Array.prototype.myMap = function (cb, thisArg) {
>   const out = [];
>   for (let i = 0; i < this.length; i++) {
>     if (i in this) out[i] = cb.call(thisArg, this[i], i, this);
>   }
>   return out;
> };
>
> Array.prototype.myReduce = function (cb, initial) {
>   let acc = initial;
>   let start = 0;
>   if (acc === undefined) {
>     if (this.length === 0) throw new TypeError('Reduce of empty array with no initial value');
>     acc = this[0];
>     start = 1;
>   }
>   for (let i = start; i < this.length; i++) {
>     if (i in this) acc = cb(acc, this[i], i, this);
>   }
>   return acc;
> };
>
> [1, 2, 3].myMap(x => x * 2);           // [2, 4, 6]
> [1, 2, 3].myReduce((a, b) => a + b, 0); // 6
> ```

> [!success] Pros / Cons
> **Polyfills:** demonstrate iteration mechanics — common in JS interviews.
> **Native methods:** faster, handle edge cases (typed arrays, iterators) — always use in production.
> **Complexity:** O(n) per call for both.
> **Edge cases:** sparse arrays, no initial value on empty array (reduce throws).

> [!info] Further study
> - [MDN: Array.prototype.map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
> - [MDN: Array.prototype.reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

---

## Related notes
- [[00-javascript]] — deeper versions of debounce, deep clone, `Promise.all`, promise pool, memoize, retry, and async patterns.
- [[01-react]] — debounce/throttle in search inputs, memoization with `useMemo`/`useCallback`.
- [[06-system-design]] — rate limiting, LRU caching at scale, webhook retry architecture.
- [[07-cv-deep-dive]] — tie machine-coding answers to affiliate request queueing and webhook resilience.

## References & Further Study
- [LeetCode Explore Cards](https://leetcode.com/explore/) — Two Sum, Sliding Window, Stack, Binary Search patterns map directly to Track A.
- [NeetCode Roadmap](https://neetcode.io/roadmap) — curated path for the most common interview DSA topics.
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/) — quick complexity reference.
- [MDN: JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) — arrays, Map/Set, Promises for Track B.
- [You Don't Know JS Yet](https://github.com/getify/You-Dont-Know-JS) — closures, `this`, and async depth for machine-coding rounds.
