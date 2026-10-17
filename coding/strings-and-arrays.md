---
title: Strings and Arrays
parent: Coding Problems
nav_order: 2
---

# Strings and Arrays
{: .no_toc }

Common string manipulation programs.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Check whether two strings are anagrams.

Easy
{: .label .label-green }

**Answer**

```java
static boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < a.length(); i++) {
        count[a.charAt(i) - 'a']++;
        count[b.charAt(i) - 'a']--;
    }
    return Arrays.stream(count).allMatch(c -> c == 0);
}
```
Time O(n), space O(1) for lowercase letters.

---

## Find the first non-repeating character in a string.

Easy
{: .label .label-green }

**Answer**

```java
static Character firstUnique(String s) {
    Map<Character, Integer> freq = new LinkedHashMap<>();
    for (char c : s.toCharArray()) freq.merge(c, 1, Integer::sum);
    return freq.entrySet().stream()
            .filter(e -> e.getValue() == 1)
            .map(Map.Entry::getKey).findFirst().orElse(null);
}
```

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
