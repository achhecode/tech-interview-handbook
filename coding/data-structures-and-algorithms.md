---
title: Data Structures and Algorithms
parent: Coding Problems
nav_order: 3
---

# Data Structures and Algorithms
{: .no_toc }

Stack, hashing and design-style problems.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Write a program to check valid parentheses.

Easy
{: .label .label-green }

**Answer**

```java
static boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        switch (c) {
            case '(', '{', '[' -> stack.push(c);
            case ')' -> { if (stack.isEmpty() || stack.pop() != '(') return false; }
            case '}' -> { if (stack.isEmpty() || stack.pop() != '{') return false; }
            case ']' -> { if (stack.isEmpty() || stack.pop() != '[') return false; }
            default -> { }
        }
    }
    return stack.isEmpty();
}
```
Time O(n), space O(n).

---

## Two Sum: find indices of two numbers that add up to a target.

Easy
{: .label .label-green }

**Answer**

```java
static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        Integer j = seen.get(target - nums[i]);
        if (j != null) return new int[] { j, i };
        seen.put(nums[i], i);
    }
    return new int[0];
}
```

---

## Implement an LRU cache.

Medium
{: .label .label-yellow }

**Answer**

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    LRUCache(int capacity) {
        super(capacity, 0.75f, true);        // true = access order
        this.capacity = capacity;
    }
    @Override protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```
Be ready to explain the HashMap plus doubly linked list version, which gives O(1) get and put.

---

<!--
ADD NEW QUESTIONS ABOVE THIS COMMENT. Copy the block below, then replace the text.
Levels: Easy (label-green), Medium (label-yellow), Hard (label-red).

## Your question here?

Medium
{: .label .label-yellow }

**Answer**

Write the answer here. Use tables, lists and ```java code blocks.

**Key points**
- Point one

**Follow-up questions**
- Follow-up one

Separate questions with a horizontal rule (three dashes on their own line).
-->
