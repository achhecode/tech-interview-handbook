---
title: Asked In Companies
parent: Coding Problems
nav_order: 5
---

# Asked In Companies
{: .no_toc }

Questions generally asked in companies.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## How to Find Unique Characters in a Given String?

HCL
{: .label }
Medium
{: .label .label-yellow }


**Answer**

A unique character is a character that appears **exactly once** in a string.

For example, in `"programming"`, the characters `p`, `o`, and `a` appear only once.

### Java 8 Solution Using Streams

```java
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

public class UniqueCharacters {
    public static void main(String[] args) {
        String str = "programming";

        Map<Character, Long> frequencyMap = str.chars()
                .mapToObj(c -> (char) c)
                .collect(Collectors.groupingBy(
                        Function.identity(),
                        LinkedHashMap::new,
                        Collectors.counting()
                ));

        frequencyMap.entrySet().stream()
                .filter(entry -> entry.getValue() == 1)
                .forEach(entry -> System.out.print(entry.getKey() + " "));
    }
}
```

**Output**
```text
p o a
```

### Explanation

1. `str.chars()` converts the string into an `IntStream` of character values.
2. `mapToObj(c -> (char) c)` converts each value into a `Character`.
3. `Collectors.groupingBy()` groups identical characters together.
4. `Collectors.counting()` counts the occurrences of each character.
5. `LinkedHashMap::new` preserves the original order of characters.
6. `filter(entry -> entry.getValue() == 1)` selects characters that occur exactly once.
7. `forEach()` prints the unique characters.

### Alternative: Using a Frequency Array

For strings containing only standard ASCII characters, an array provides an efficient solution.

```java
public class UniqueCharactersArray {
    public static void main(String[] args) {
        String str = "programming";
        int[] frequency = new int[128];

        for (char ch : str.toCharArray()) {
            frequency[ch]++;
        }

        for (char ch : str.toCharArray()) {
            if (frequency[ch] == 1) {
                System.out.print(ch + " ");
            }
        }
    }
}
```

**Output**
```text
p o a
```

### Complexity Analysis

| Approach | Time Complexity | Space Complexity |
|---|---|---|
| Java Streams + Map | O(n) average | O(k) |
| Frequency array | O(n) | O(1) for ASCII |

Here, `n` is the string length and `k` is the number of distinct characters. The frequency array uses constant auxiliary space because its size is fixed at 128.

**Key points**
- Clarify whether *unique characters* means characters appearing exactly once or distinct characters appearing at least once.
- Use `LinkedHashMap` when insertion order matters.
- Use `Collectors.counting()` to count character frequencies with Java Streams.
- For Unicode text, consider code points (`str.codePoints()`) instead of `char`, because some characters require two UTF-16 code units.
- The frequency-array approach assumes ASCII input and can fail for characters outside its supported range.

**Follow-up questions**
- Find the first non-repeating character in a string.
- Find duplicate characters and their frequencies.
- Find all distinct characters in a string.
- Find the first repeating character using Java 8 Streams.
- Solve the problem using `HashMap` without Streams.
- Find unique characters while ignoring case and spaces.

---

## How to Find the Minimum Value in a List?

HCL
{: .label }

Easy
{: .label .label-green }

**Answer**

The minimum value in a list is the smallest element present in that list. In Java 8, we can find it using Streams and the `min()` method.

### Java 8 Solution Using Streams

```java
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

public class MinimumValue {
    public static void main(String[] args) {
        List<Integer> numbers =
                Arrays.asList(45, 12, 78, 3, 56, 9);

        Optional<Integer> min = numbers.stream()
                .min(Integer::compareTo);

        min.ifPresent(System.out::println);
    }
}
```

**Output**
```text
3
```

### Explanation

1. `numbers.stream()` creates a stream from the list.
2. `min(Integer::compareTo)` compares elements and returns the smallest value.
3. The result is an `Optional<Integer>` because the list might be empty.
4. `ifPresent(System.out::println)` prints the result if a minimum exists.

### Alternative: Using `mapToInt()`

```java
int min = numbers.stream()
        .mapToInt(Integer::intValue)
        .min()
        .orElseThrow(() -> new IllegalArgumentException(
                "List cannot be empty"
        ));

System.out.println(min);
```

**Output**
```text
3
```

### Alternative: Using Collections

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class MinimumUsingCollections {
    public static void main(String[] args) {
        List<Integer> numbers =
                Arrays.asList(45, 12, 78, 3, 56, 9);

        System.out.println(Collections.min(numbers));
    }
}
```

**Output**
```text
3
```

### Complexity Analysis

| Approach | Time Complexity | Space Complexity |
|---|---|---|
| Streams `min()` | O(n) | O(1) auxiliary |
| `Collections.min()` | O(n) | O(1) auxiliary |

**Key points**
- `min()` is a terminal stream operation.
- `Optional<Integer>` handles the possibility of an empty stream.
- `orElseThrow()` lets you explicitly handle an empty list.
- `Collections.min()` is a simple alternative when Streams are not required.
- For a list of custom objects, use a comparator based on the field you want to minimize.

**Follow-up questions**
- Find the maximum value in a list.
- Find the second-smallest element in a list.
- Find the minimum value without using Streams.
- Find the minimum value in a list of employee salaries.
- Find the minimum and maximum values in a single traversal.

---

## String Challenge — Minimum Binary Digit Changes

Coderbyte
{: .label}

Medium
{: .label .label-yellow }

**Answer**

The goal is to convert the first element (a decimal number) into its binary representation and compare it with the second element (a binary string).

Count how many bits must be flipped (`0` to `1` or `1` to `0`) to make the binary string represent the decimal number correctly.

### Java Solution

```java
public class StringChallenge {

    public static int stringChallenge(String[] strArr) {
        long decimal = Long.parseLong(strArr[0]);

        String expectedBinary = Long.toBinaryString(decimal);
        String givenBinary = strArr[1];

        // Binary strings must have equal lengths.
        int maxLength = Math.max(
                expectedBinary.length(),
                givenBinary.length()
        );

        expectedBinary = String.format(
                "%" + maxLength + "s", expectedBinary
        ).replace(' ', '0');

        givenBinary = String.format(
                "%" + maxLength + "s", givenBinary
        ).replace(' ', '0');

        int changes = 0;

        for (int i = 0; i < maxLength; i++) {
            if (expectedBinary.charAt(i) != givenBinary.charAt(i)) {
                changes++;
            }
        }

        return changes;
    }

    public static void main(String[] args) {
        System.out.println(
                stringChallenge(new String[]{"56", "011000"})
        );

        System.out.println(
                stringChallenge(new String[]{"5624", "0010111111001"})
        );

        System.out.println(
                stringChallenge(new String[]{"44", "111111"})
        );
    }
}
```

**Output**
```text
1
2
3
```

### Explanation

**Example 1:** `["56", "011000"]`

Step 1: Convert decimal `56` to binary.

```text
56 = 111000
```

Step 2: Compare the binary strings.

```text
Expected: 111000
Given:    011000
          ^
```

Step 3: Only the first bit differs (`0` → `1`).

Result: `1`

**Example 2:** `["5624", "0010111111001"]`

```text
Expected: 1010111111000
Given:    0010111111001
          ^           ^
```

Two bits differ, so the result is `2`.

**Example 3:** `["44", "111111"]`

```text
Expected: 101100
Given:    111111
           ^ ^^
```

Three bits differ, so the result is `3`.

### Complexity Analysis

Let `n` be the maximum length of the two binary strings.

- **Time Complexity:** O(n), because each bit is compared once.
- **Space Complexity:** O(n), because the binary strings are padded to equal lengths.

**Key points**
- `Long.toBinaryString()` converts a decimal number to binary.
- Binary strings must be aligned from the right, because leading zeros do not change a number's value.
- Each differing bit requires exactly one flip.
- `Long.parseLong()` supports decimal inputs up to the `long` range. For larger inputs, use `BigInteger`.

**Follow-up questions**
- Solve the problem using `BigInteger`.
- Find the Hamming distance between two binary strings.
- Count the number of set bits (`1`s) in a number.
- Solve the problem without using `String.format()`.


---


## Find All Unique Characters and Their Frequency in a String

Medium
{: .label .label-yellow }

**Answer**

Given a string, find each distinct character and count how many times it appears. Print each character along with its frequency.

### Java 8 Solution Using Streams

```java
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

public class CharacterFrequency {
    public static void main(String[] args) {
        String str = "abcad";

        Map<Character, Long> frequencyMap = str.chars()
                .mapToObj(c -> (char) c)
                .collect(Collectors.groupingBy(
                        Function.identity(),
                        LinkedHashMap::new,
                        Collectors.counting()
                ));

        frequencyMap.forEach((character, frequency) ->
                System.out.print(character + ":" + frequency + ", ")
        );
    }
}
```

**Output**
```text
a:2, b:1, c:1, d:1,
```

### Example 2

**Input**
```text
daabaccsfgb
```

**Output**
```text
d:1, a:3, b:2, c:2, s:1, f:1, g:1
```

### Explanation

1. `str.chars()` converts the string into a stream of character values.
2. `mapToObj(c -> (char) c)` converts each value to a `Character`.
3. `groupingBy()` groups identical characters.
4. `LinkedHashMap::new` preserves the order in which characters first appear.
5. `Collectors.counting()` counts the occurrences of each character.
6. `forEach()` prints each character and its frequency.

### Alternative: Using a HashMap

```java
import java.util.HashMap;
import java.util.Map;

public class CharacterFrequency {
    public static void main(String[] args) {
        String str = "daabaccsfgb";

        Map<Character, Integer> map = new HashMap<>();

        for (char ch : str.toCharArray()) {
            map.put(ch, map.getOrDefault(ch, 0) + 1);
        }

        map.forEach((ch, count) ->
                System.out.println(ch + ":" + count)
        );
    }
}
```

**Note:** `HashMap` does not guarantee iteration order. Use `LinkedHashMap` if you need the characters in their original order.

### Complexity Analysis

- **Time Complexity:** O(n) on average, where `n` is the string length.
- **Space Complexity:** O(k), where `k` is the number of distinct characters.

**Key points**
- *Distinct characters* means each character is listed once, along with its total frequency.
- `getOrDefault(ch, 0) + 1` increments a character's count.
- `LinkedHashMap` preserves insertion order.
- `Collectors.counting()` returns a `Long`, while the manual `HashMap` example uses `Integer`.

**Follow-up questions**
- Find only characters that appear exactly once.
- Find duplicate characters in a string.
- Find the first non-repeating character.
- Sort characters by descending frequency.
- Find character frequencies using Java 8 Streams without `groupingBy()`.


---


## Longest Substring Without Repeating Characters

Medium
{: .label .label-yellow }

**Answer**

Given a string, find all longest contiguous substrings that contain no repeated characters. If multiple substrings have the same maximum length, return all of them.

### Java Solution

```java
import java.util.ArrayList;
import java.util.List;

public class LongestSubstring {

    public static List<String> findLongestSubstrings(String s) {
        List<String> result = new ArrayList<>();

        int maxLength = 0;

        for (int i = 0; i < s.length(); i++) {
            boolean[] seen = new boolean[65536];

            for (int j = i; j < s.length(); j++) {
                char ch = s.charAt(j);

                if (seen[ch]) {
                    break;
                }

                seen[ch] = true;
                int length = j - i + 1;

                if (length > maxLength) {
                    maxLength = length;
                    result.clear();
                    result.add(s.substring(i, j + 1));
                } else if (length == maxLength) {
                    result.add(s.substring(i, j + 1));
                }
            }
        }

        return result;
    }

    public static void main(String[] args) {
        System.out.println(findLongestSubstrings("daabaccsfgb"));
        System.out.println(findLongestSubstrings("ababc"));
    }
}
```

**Output**

```text
[bac, acsf, csfg]
[abc, babc]
```

### Explanation

The algorithm checks substrings starting at each index and stops extending a substring when a repeated character is found.

- `seen[ch]` checks whether a character has already appeared in the current substring.
- `maxLength` stores the longest length found so far.
- When a longer substring is found, `result.clear()` removes shorter answers.
- When another substring of the same maximum length is found, it is added to the result.

### Complexity Analysis

- **Time Complexity:** O(n²) in the worst case.
- **Space Complexity:** O(n + k), where `k` is the total size of the returned substrings, excluding the fixed character lookup array.

**Key points**
- A substring must contain contiguous characters.
- A subsequence does not have to contain contiguous characters.
- This solution assumes the intended condition is that characters within each substring must be distinct.

**Follow-up questions**
- Solve this problem using a sliding window in O(n) time.
- Find the longest substring containing only one repeated character.
- Find all longest palindromic substrings.


---


## Count the Number of Words in a String

Easy
{: .label .label-green }

**Answer**

Given a string, count the number of words present. A word is a sequence of letters or digits, and special characters such as commas, hyphens, and exclamation marks do not count as words.

### Java Solution Using Regular Expressions

```java
public class WordCount {

    public static int countWords(String str) {
        String trimmed = str.trim();

        if (trimmed.isEmpty()) {
            return 0;
        }

        String[] words = trimmed.split("[^a-zA-Z0-9]+");
        return words.length;
    }

    public static void main(String[] args) {
        System.out.println(countWords("Hey, A R"));
        System.out.println(countWords("My, name is - Danish!"));
    }
}
```

**Output**
```text
3
4
```

### Explanation

The regular expression `[^a-zA-Z0-9]+` matches one or more consecutive characters that are **not** English letters or digits.

For example:

Input: `"My, name is - Danish!"`

The string is split into:

```text
My
name
is
Danish
```

Therefore, the word count is `4`.

### Alternative: Using Java 8 Streams

```java
import java.util.Arrays;

public class WordCount {
    public static void main(String[] args) {
        String str = "My, name is - Danish!";

        long count = Arrays.stream(str.split("[^a-zA-Z0-9]+"))
                .filter(word -> !word.isEmpty())
                .count();

        System.out.println(count);
    }
}
```

**Output**
```text
4
```

### Complexity Analysis

- **Time Complexity:** O(n), where `n` is the length of the string.
- **Space Complexity:** O(n), because splitting creates an array of words.

**Key points**
- `split()` divides the string using a regular expression.
- `[^a-zA-Z0-9]+` treats punctuation and spaces as separators.
- `trim()` handles leading and trailing whitespace in the first solution.
- The Streams solution filters empty strings to handle leading or trailing separators.
- These solutions count alphanumeric sequences as words; they assume English letters and digits.

**Follow-up questions**
- Count words without using `split()`.
- Count words using Java 8 Streams.
- Count words while ignoring numbers.
- Handle Unicode letters, such as Hindi or Arabic words.


---


## Check Whether One Array Is a Subset of Another in the Same Order

Medium
{: .label .label-yellow }

**Answer**

Given two arrays, `A1` and `A2`, determine whether `A2` is a subsequence of `A1`.

`A2` is valid if all its elements appear in `A1` in the same relative order. The elements do not need to be contiguous.

### Java Solution Using Two Pointers

```java
public class ArraySubsetCheck {

    public static boolean isSubset(int[] a1, int[] a2) {
        int i = 0;
        int j = 0;

        while (i < a1.length && j < a2.length) {
            if (a1[i] == a2[j]) {
                j++;
            }
            i++;
        }

        return j == a2.length;
    }

    public static void main(String[] args) {
        int[] a1 = {7, 8, 2, 6, 22, 23};

        int[] a2 = {8, 22, 23};
        System.out.println(isSubset(a1, a2) ? "Valid" : "Invalid");

        int[] a3 = {7, 6, 8};
        System.out.println(isSubset(a1, a3) ? "Valid" : "Invalid");

        int[] a4 = {2, 6, 9, 8};
        int[] a5 = {2, 3, 5, 6, 8, 9};
        System.out.println(isSubset(a5, a4) ? "Valid" : "Invalid");
    }
}
```

**Output**

```text
Valid
Invalid
Invalid
```

### Explanation

**Example 1**

```text
A1 = [7, 8, 2, 6, 22, 23]
A2 = [8, 22, 23]
```

- Find `8` in `A1`.
- Continue searching after `8` and find `22`.
- Continue searching after `22` and find `23`.

All elements appear in the correct order, so the result is `Valid`.

**Example 2**

```text
A1 = [7, 8, 2, 6, 22, 23]
A2 = [7, 6, 8]
```

- Find `7`.
- Find `6` after `7`.
- The next element to find is `8`, but it appears before `6`, not after it.

Therefore, the result is `Invalid`.

### How the Two Pointers Work

- `i` traverses array `A1`.
- `j` tracks the next element we need to find in `A2`.
- When `A1[i] == A2[j]`, increment `j`.
- Increment `i` on every iteration.
- If `j == a2.length`, every element of `A2` was found in order.

### Complexity Analysis

- **Time Complexity:** O(n), where `n` is the length of `A1`.
- **Space Complexity:** O(1), because only two pointers are used.

**Key points**
- This problem checks for a **subsequence**, not a conventional set subset.
- Elements must appear in the same relative order, but need not be adjacent.
- Duplicate elements are handled correctly because each match advances the search.
- An empty `A2` is considered valid because an empty sequence is a subsequence of every array.

**Follow-up questions**
- Solve this problem using Java 8 Streams.
- Check whether `A2` is a contiguous subarray of `A1`.
- Find the longest common subsequence of two arrays.
- Handle arrays containing duplicate elements.


---


## Find the Equilibrium Index in an Array

Medium
{: .label .label-yellow }

**Answer**

Given an array, find the index where the sum of all elements on the left equals the sum of all elements on the right.

This index is called the **equilibrium index**. The element at that index is excluded from both sums.

### Java Solution — Optimal Approach

```java
public class EquilibriumIndex {

    public static int findEquilibriumIndex(int[] arr) {
        int totalSum = 0;

        for (int num : arr) {
            totalSum += num;
        }

        int leftSum = 0;

        for (int i = 0; i < arr.length; i++) {
            int rightSum = totalSum - leftSum - arr[i];

            if (leftSum == rightSum) {
                return i;
            }

            leftSum += arr[i];
        }

        return -1;
    }

    public static void main(String[] args) {
        int[] arr1 = {2, 3, 5, 4, 100, 6, 2, 3, 3};
        int[] arr2 = {2, 3, 4, 1, 4, 5};

        System.out.println(findEquilibriumIndex(arr1));
        System.out.println(findEquilibriumIndex(arr2));
    }
}
```

**Output**

```text
4
2
```

### Important: Your Expected Outputs Need Correction

For the first input:

```text
[2, 3, 5, 4, 100, 6, 2, 3, 3]
```

At index `4`, the value is `100`.

- Left sum = `2 + 3 + 5 + 4 = 14`
- Right sum = `6 + 2 + 3 + 3 = 14`

Therefore, the equilibrium index is **4**, not 5. If you mean the position using 1-based indexing, it is 5.

For the second input:

```text
[2, 3, 4, 1, 4, 5]
```

At index `2`, the value is `4`.

- Left sum = `2 + 3 = 5`
- Right sum = `1 + 4 + 5 = 10`

So index `2` is not an equilibrium index. In fact, this array has **no equilibrium index**, and the method returns `-1`.

### Explanation

1. Calculate the total sum of the array.
2. Initialize `leftSum = 0`.
3. For each index, calculate the right sum:

   `rightSum = totalSum - leftSum - arr[i]`

4. If `leftSum == rightSum`, return the current index.
5. Otherwise, add the current element to `leftSum` and continue.
6. If no equilibrium index exists, return `-1`.

### Complexity Analysis

- **Time Complexity:** O(n) — two traversals of the array.
- **Space Complexity:** O(1) — only a few variables are used.

**Key points**
- The equilibrium index is zero-based in Java.
- The current element is excluded from both left and right sums.
- The algorithm finds the first equilibrium index if multiple exist.
- The `-1` return value indicates that no equilibrium index exists.

**Follow-up questions**
- Find all equilibrium indices in an array.
- Solve the problem using Java 8 Streams.
- Find the equilibrium index without calculating the total sum first.
- Handle arrays containing negative numbers.


---

## Replace Each Element with the Next Greater Element

Medium
{: .label .label-yellow }

**Answer**

Given an array, replace each element with the **first greater element to its right**. If no greater element exists to its right, replace it with `-1`.

### Java Solution — Brute Force

```java
import java.util.Arrays;

public class NextGreaterElement {

    public static int[] findNextGreater(int[] arr) {
        int n = arr.length;
        int[] result = new int[n];

        for (int i = 0; i < n; i++) {
            result[i] = -1;

            for (int j = i + 1; j < n; j++) {
                if (arr[j] > arr[i]) {
                    result[i] = arr[j];
                    break;
                }
            }
        }

        return result;
    }

    public static void main(String[] args) {
        int[] arr1 = {4, 5, 2, 25};
        int[] arr2 = {11, 13, 21, 3};

        System.out.println(Arrays.toString(findNextGreater(arr1)));
        System.out.println(Arrays.toString(findNextGreater(arr2)));
    }
}
```

**Output**

```text
[5, 25, 25, -1]
[13, 21, -1, -1]
```

### Explanation

**Test Case 1:** `[4, 5, 2, 25]`

| Element | First greater element to the right |
|---:|---:|
| 4 | 5 |
| 5 | 25 |
| 2 | 25 |
| 25 | -1 |

Result: `[5, 25, 25, -1]`

**Test Case 2:** `[11, 13, 21, 3]`

| Element | First greater element to the right |
|---:|---:|
| 11 | 13 |
| 13 | 21 |
| 21 | -1 |
| 3 | -1 |

Result: `[13, 21, -1, -1]`

### Optimal Java Solution — Using a Monotonic Stack

The brute-force solution takes O(n²) time. We can optimize it to O(n) using a stack.

```java
import java.util.Arrays;
import java.util.ArrayDeque;
import java.util.Deque;

public class NextGreaterElementOptimal {

    public static int[] findNextGreater(int[] arr) {
        int n = arr.length;
        int[] result = new int[n];
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = n - 1; i >= 0; i--) {
            while (!stack.isEmpty() && stack.peek() <= arr[i]) {
                stack.pop();
            }

            result[i] = stack.isEmpty() ? -1 : stack.peek();
            stack.push(arr[i]);
        }

        return result;
    }

    public static void main(String[] args) {
        int[] arr = {4, 5, 2, 25};
        System.out.println(Arrays.toString(findNextGreater(arr)));
    }
}
```

**Output**

```text
[5, 25, 25, -1]
```

### How the Stack Approach Works

1. Traverse the array from right to left.
2. Remove stack elements that are less than or equal to the current element, because they cannot be its next greater element.
3. If the stack is empty, assign `-1`. Otherwise, the top of the stack is the next greater element.
4. Push the current element onto the stack.

### Complexity Analysis

| Approach | Time Complexity | Space Complexity |
|---|---|---|
| Brute force | O(n²) | O(n) for result |
| Monotonic stack | O(n) | O(n) for stack and result |

**Key points**
- The next greater element must appear to the right.
- It must be strictly greater, not equal.
- The stack solution processes each element at most once when pushing and once when popping.
- In Java, `ArrayDeque` is generally preferred over the legacy `Stack` class for stack operations.

**Follow-up questions**
- Find the next smaller element.
- Find the previous greater element.
- Solve the circular next greater element problem.
- Find the next greater element using Java 8 Streams.


---

## Search Insert Position — Binary Search

Medium
{: .label .label-yellow }

**Answer**

Given a sorted array of distinct integers and a target value, return the index where the target exists. If it does not exist, return the index where it should be inserted to maintain sorted order.

The solution must have **O(log n) time complexity**, so we use Binary Search.

### Java Solution

```java
public class SearchInsertPosition {

    public static int searchInsert(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }

        return left;
    }

    public static void main(String[] args) {
        System.out.println(searchInsert(
                new int[]{1, 3, 5, 6}, 5)); // 2

        System.out.println(searchInsert(
                new int[]{1, 3, 5, 6}, 2)); // 1

        System.out.println(searchInsert(
                new int[]{1, 3, 5, 6}, 7)); // 4
    }
}
```

**Output**

```text
2
1
4
```

### Explanation

**Example 1:** `nums = [1, 3, 5, 6]`, `target = 5`

- `left = 0`, `right = 3`
- `mid = 1`, so `nums[mid] = 3`
- Since `3 < 5`, update `left = 2`
- `mid = 2`, so `nums[mid] = 5`
- Target found. Return `2`.

**Example 2:** `nums = [1, 3, 5, 6]`, `target = 2`

- `mid = 1`, value `3` is greater than `2`, so update `right = 0`.
- `mid = 0`, value `1` is less than `2`, so update `left = 1`.
- Now `left > right`. Return `left`, which is `1`.

The target belongs between `1` and `3`.

**Example 3:** `nums = [1, 3, 5, 6]`, `target = 7`

- Every element is smaller than `7`.
- Eventually, `left` becomes `4`, which is the array length.
- Return `4`, the position immediately after the last element.

### Why return `left`?

When the loop ends without finding the target, `left` identifies the first position where the target can be inserted without breaking the sorted order.

### Complexity Analysis

- **Time Complexity:** O(log n) — each iteration eliminates approximately half of the remaining search range.
- **Space Complexity:** O(1) — only a few variables are used.

**Key points**
- Binary Search requires a sorted array.
- Use `left + (right - left) / 2` to calculate the midpoint safely.
- Return `mid` when the target is found.
- Return `left` when the target is not found.
- The same approach works for an empty array, returning index `0`.

**Follow-up questions**
- Implement Binary Search recursively.
- Find the first and last occurrence of a target in a sorted array.
- Find the lower bound and upper bound of a target.
- Search for an element in a rotated sorted array.


---


## Find the Number That Appears Only Once

Easy
{: .label .label-green }

**Answer**

Given an array where every number appears exactly twice except one number, find the number that appears only once.

Example:

```text
Input:  [1, 1, 2, 2, 3, 4, 4]
Output: 3
```

### Approach 1: Using XOR (Optimal Solution)

The XOR operator (`^`) has three important properties:

- `a ^ a = 0`
- `a ^ 0 = a`
- XOR is commutative and associative.

Therefore, duplicate numbers cancel each other out, leaving only the unique number.

```java
public class SingleNumber {

    public static int findSingleNumber(int[] nums) {
        int result = 0;

        for (int num : nums) {
            result ^= num;
        }

        return result;
    }

    public static void main(String[] args) {
        int[] arr1 = {1, 1, 2, 2, 3, 4, 4};
        int[] arr2 = {1, 1, 4, 3, 3};

        System.out.println(findSingleNumber(arr1));
        System.out.println(findSingleNumber(arr2));
    }
}
```

**Output**

```text
3
4
```

**Note:** For the second array `[1, 1, 4, 3, 3]`, the unique number is `4`, not `3`, because `3` appears twice.

### How XOR Works

For `[1, 1, 2, 2, 3, 4, 4]`:

```text
0 ^ 1 ^ 1 ^ 2 ^ 2 ^ 3 ^ 4 ^ 4

= (1 ^ 1) ^ (2 ^ 2) ^ 3 ^ (4 ^ 4)

= 0 ^ 0 ^ 3 ^ 0

= 3
```

### Approach 2: Using HashSet

```java
import java.util.HashSet;
import java.util.Set;

public class SingleNumberSet {

    public static int findSingleNumber(int[] nums) {
        Set<Integer> set = new HashSet<>();

        for (int num : nums) {
            if (!set.add(num)) {
                set.remove(num);
            }
        }

        return set.iterator().next();
    }

    public static void main(String[] args) {
        int[] nums = {1, 1, 2, 2, 3, 4, 4};

        System.out.println(findSingleNumber(nums));
    }
}
```

**Output**
```text
3
```

### Complexity Analysis

| Approach | Time Complexity | Space Complexity |
|---|---|---|
| XOR | O(n) | O(1) |
| HashSet | O(n) average | O(n) |

**Key points**
- XOR is the optimal solution when every other number appears exactly twice.
- XOR uses constant extra space, unlike a HashSet.
- The input can be unsorted; XOR still works.
- An empty array or an array violating the problem's assumptions should be handled separately if invalid inputs are possible.

**Follow-up questions**
- Find the unique number when every other number appears three times.
- Find two numbers that appear only once.
- Solve the problem using a frequency map.
- Explain how XOR works at the bit level.


---



## Travelling Expenses

Medium
{: .label .label-yellow }

**Problem Statement**

You are given a string of 365 characters representing the days of a year.

- `1` indicates a day you travelled.
- `0` indicates a day you did not travel.

Calculate the total travelling expenses based on consecutive travelling days:

- For every 7 consecutive travelling days, the cost is 5.
- For remaining travelling days, the cost is 2 per day.
- When a travelling streak ends, calculate its cost and reset the counter.
- Calculate the cost of the final streak even if it ends on the last day of the year.

Return the total travelling expenses.

### Java Solution

```java
import java.util.Scanner;

public class TravellingExpenses {

    public static int calculateExpenses(String inp) {
        int count = 0;
        int cost = 0;

        for (int i = 0; i < inp.length(); i++) {
            if (inp.charAt(i) == '0') {
                cost += (count / 7) * 5;
                cost += (count % 7) * 2;
                count = 0;
            } else {
                count++;
            }
        }

        // Calculate expenses for the final travelling streak
        cost += (count / 7) * 5;
        cost += (count % 7) * 2;

        return cost;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        String inp = sc.next();
        System.out.println(calculateExpenses(inp));

        sc.close();
    }
}
```

### Explanation

The solution counts consecutive travelling days. Whenever a non-travelling day (`0`) appears, it calculates the cost of the completed streak:

- `count / 7` gives the number of complete groups of 7 days.
- Each complete group costs 5.
- `count % 7` gives the remaining days.
- Each remaining day costs 2.

After processing the string, the same calculation is performed for the final streak.

### Example

**Input**
```text
1111111101111111
```

**Calculation**

- First streak: 8 days → one group of 7 costs 5, plus 1 remaining day costs 2.
- Second streak: 7 days → one group of 7 costs 5, plus no remaining days.

Total cost = `5 + 2 + 5 = 12`.

**Output**
```text
12
```

### Complexity Analysis

- **Time Complexity:** O(n), where `n` is the length of the input string.
- **Space Complexity:** O(1), because only a few variables are used.

**Key points**
- Consecutive travelling days are counted as a single streak.
- Each streak is calculated independently.
- The final streak must be processed separately after the loop.
- The implementation uses `inp.length()` instead of assuming the input always contains exactly 365 characters.
- This solution assumes the cost rule is 5 per complete group of 7 consecutive travelling days and 2 per remaining day.


---


## Longest Substring Without Repeating Characters

Medium
{: .label .label-yellow }

**Problem Statement**

Given a string `s`, find the length of the longest substring that contains no repeating characters.

A substring must contain consecutive characters from the original string.

### Java Solution — Sliding Window

```java
import java.util.HashMap;
import java.util.Map;

public class LongestSubstring {

    public static int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> lastSeen = new HashMap<>();

        int left = 0;
        int maxLength = 0;

        for (int right = 0; right < s.length(); right++) {
            char ch = s.charAt(right);

            if (lastSeen.containsKey(ch)
                    && lastSeen.get(ch) >= left) {
                left = lastSeen.get(ch) + 1;
            }

            lastSeen.put(ch, right);

            maxLength = Math.max(maxLength, right - left + 1);
        }

        return maxLength;
    }

    public static void main(String[] args) {
        System.out.println(lengthOfLongestSubstring("abcabcbb"));
        System.out.println(lengthOfLongestSubstring("bbbbb"));
        System.out.println(lengthOfLongestSubstring("pwwkew"));
    }
}
```

**Output**

```text
3
1
3
```

### Explanation

We use a **sliding window** represented by two pointers, `left` and `right`.

- `left` marks the beginning of the current substring.
- `right` moves through the string one character at a time.
- `lastSeen` stores the most recent index of each character.
- If a repeated character appears inside the current window, move `left` past its previous occurrence.
- Update `maxLength` using `right - left + 1`.

**Example 1:** `s = "abcabcbb"`

| Character | Current window | Maximum length |
|---|---|---:|
| a | a | 1 |
| b | ab | 2 |
| c | abc | 3 |
| a | bca | 3 |
| b | cab | 3 |
| c | abc | 3 |

The longest substring has length `3`, such as `"abc"`.

**Example 2:** `s = "bbbbb"`

Every character repeats immediately, so the longest substring without repeating characters is `"b"`, with length `1`.

**Example 3:** `s = "pwwkew"`

The longest valid substring is `"wke"` or `"kew"`, both with length `3`.

### Complexity Analysis

- **Time Complexity:** O(n) — each character is processed once by the right pointer, and the left pointer only moves forward.
- **Space Complexity:** O(k), where `k` is the number of distinct characters stored in the map.

**Key points**
- Use a sliding window to avoid checking every possible substring.
- A `HashMap` tracks each character's most recent index.
- Update `left` only when the previous occurrence lies inside the current window.
- The problem asks for the length, not the substring itself.
- An empty string returns `0`.

**Follow-up questions**
- Return the actual longest substring instead of its length.
- Solve the problem using an integer array instead of a `HashMap`.
- Find the longest substring containing at most `K` distinct characters.
- Explain why the sliding-window approach runs in O(n) time.



---


## Implement a HashMap from Scratch

Medium
{: .label .label-yellow }

**Problem Statement**

Implement a simplified version of a HashMap using an array and a hashing technique, without using Java's built-in `HashMap`.

A HashMap stores data as key-value pairs and supports operations such as:

- `put(key, value)` — insert or update a key-value pair.
- `get(key)` — retrieve the value associated with a key.
- `remove(key)` — remove a key-value pair.
- `containsKey(key)` — check whether a key exists.

### Approach

A HashMap uses an array internally. A hash function converts each key into an array index.

For example, if the array capacity is `10` and the key is `25`:

```text
index = hash(25) % 10
      = 25 % 10
      = 5
```

The key-value pair is stored at index `5`.

**What happens if two keys produce the same index?**

This is called a *collision*. We can handle collisions using **separate chaining**, where each array slot holds a linked list of entries.

### Java Solution

```java
public class MyHashMap<K, V> {

    private static final int CAPACITY = 16;

    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> next;

        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }

    private final Node<K, V>[] buckets;
    private int size;

    @SuppressWarnings("unchecked")
    public MyHashMap() {
        buckets = (Node<K, V>[]) new Node[CAPACITY];
    }

    private int getIndex(K key) {
        return (key == null ? 0 : key.hashCode() & 0x7fffffff)
                % buckets.length;
    }

    public void put(K key, V value) {
        int index = getIndex(key);
        Node<K, V> current = buckets[index];

        while (current != null) {
            if (java.util.Objects.equals(current.key, key)) {
                current.value = value;
                return;
            }
            current = current.next;
        }

        Node<K, V> node = new Node<>(key, value);
        node.next = buckets[index];
        buckets[index] = node;
        size++;
    }

    public V get(K key) {
        int index = getIndex(key);
        Node<K, V> current = buckets[index];

        while (current != null) {
            if (java.util.Objects.equals(current.key, key)) {
                return current.value;
            }
            current = current.next;
        }

        return null;
    }

    public boolean containsKey(K key) {
        int index = getIndex(key);
        Node<K, V> current = buckets[index];

        while (current != null) {
            if (java.util.Objects.equals(current.key, key)) {
                return true;
            }
            current = current.next;
        }

        return false;
    }

    public void remove(K key) {
        int index = getIndex(key);
        Node<K, V> current = buckets[index];
        Node<K, V> previous = null;

        while (current != null) {
            if (java.util.Objects.equals(current.key, key)) {
                if (previous == null) {
                    buckets[index] = current.next;
                } else {
                    previous.next = current.next;
                }
                size--;
                return;
            }

            previous = current;
            current = current.next;
        }
    }

    public int size() {
        return size;
    }

    public static void main(String[] args) {
        MyHashMap<String, Integer> map = new MyHashMap<>();

        map.put("Apple", 10);
        map.put("Banana", 20);
        map.put("Orange", 30);

        System.out.println(map.get("Apple"));
        System.out.println(map.get("Banana"));

        map.put("Apple", 100);
        System.out.println(map.get("Apple"));

        System.out.println(map.containsKey("Orange"));

        map.remove("Banana");
        System.out.println(map.get("Banana"));
        System.out.println(map.size());
    }
}
```

**Output**

```text
10
20
100
true
null
2
```

### Explanation

**1. Array of buckets**

```java
private final Node<K, V>[] buckets;
```

Each array slot stores the head of a linked list. Multiple entries can occupy the same bucket when collisions occur.

**2. Hash function**

```java
private int getIndex(K key) {
    return (key == null ? 0 : key.hashCode() & 0x7fffffff)
            % buckets.length;
}
```

The key's hash code is converted into a non-negative number and mapped to a valid array index. This implementation places `null` keys in bucket zero.

**3. Insert or update**

`put()` searches the linked list at the calculated index. If the key already exists, its value is updated. Otherwise, a new node is inserted.

**4. Retrieve a value**

`get()` calculates the bucket and searches its linked list for the matching key.

**5. Handle collisions**

Different keys may map to the same index. Separate chaining stores these entries in the same bucket's linked list.

**6. Remove a key**

`remove()` locates the entry and unlinks its node from the linked list.

### Complexity Analysis

Let `n` be the number of key-value pairs and `b` the number of buckets.

| Operation | Average | Worst case |
|---|---|---|
| `put()` | O(1) | O(n) |
| `get()` | O(1) | O(n) |
| `remove()` | O(1) | O(n) |
| `containsKey()` | O(1) | O(n) |

The average assumes a reasonably distributed hash function and a manageable load factor. This simplified implementation uses a fixed-size array and does not resize automatically, so performance may degrade as entries accumulate.

**Key points**
- A hash function maps a key to a bucket index.
- A collision occurs when multiple keys map to the same bucket.
- Separate chaining handles collisions with linked lists.
- Equal keys must have equal hash codes.
- Updating an existing key should not increase the map's size.
- This implementation is educational, not a replacement for production `java.util.HashMap`.

**Follow-up questions**
- How does Java's `HashMap` handle collisions internally?
- What are `hashCode()` and `equals()`, and why must they agree?
- What is a load factor, and why does a HashMap resize?
- How does open addressing differ from separate chaining?
- Why is `HashMap` not thread-safe?


---


## Maximum Sum Root-to-Leaf Path in a Binary Tree

Medium
{: .label .label-yellow }

**Problem Statement**

Given a binary tree in which each node contains an integer value, find the maximum sum of any path that starts at the root and ends at a leaf node.

A leaf node is a node that has no left or right child.

The path must follow the parent-child relationship from the root to a leaf. The goal is to return the maximum sum among all such paths.

**Example 1**

```text
        10
       /  \
      2    10
     / \     \
    20  1     -25
               \
                4
```

**Output**

```text
32
```

**Explanation**

The maximum-sum root-to-leaf path is:

```text
10 -> 2 -> 20
```

Sum = 10 + 2 + 20 = 32.

**Example 2**

```text
       5
      / \
     4   8
    /   / \
   11  13  4
  /  \       \
 7    2       1
```

**Output**

```text
26
```

**Explanation**

The maximum-sum root-to-leaf path is:

```text
5 -> 4 -> 11 -> 7
```

Sum = 5 + 4 + 11 + 7 = 27.

Therefore, the correct output for this tree is **27**.

### Java Solution

```java
public class MaximumRootToLeafSum {

    static class Node {
        int data;
        Node left;
        Node right;

        Node(int data) {
            this.data = data;
        }
    }

    public static int maxRootToLeafSum(Node root) {
        if (root == null) {
            throw new IllegalArgumentException(
                    "The tree must not be empty");
        }

        // A leaf node ends the path.
        if (root.left == null && root.right == null) {
            return root.data;
        }

        // If only one child exists, the path must use that child.
        if (root.left == null) {
            return root.data + maxRootToLeafSum(root.right);
        }

        if (root.right == null) {
            return root.data + maxRootToLeafSum(root.left);
        }

        // Both children exist; choose the higher-sum path.
        return root.data + Math.max(
                maxRootToLeafSum(root.left),
                maxRootToLeafSum(root.right)
        );
    }

    public static void main(String[] args) {
        Node root = new Node(10);
        root.left = new Node(2);
        root.right = new Node(10);

        root.left.left = new Node(20);
        root.left.right = new Node(1);

        root.right.right = new Node(-25);
        root.right.right.right = new Node(4);

        System.out.println(maxRootToLeafSum(root));
    }
}
```

**Output**

```text
32
```

### Explanation

The algorithm uses **recursion** to calculate the maximum sum from each node down to a leaf.

1. **Base case:** If the node is `null`, it is not treated as a valid path. The public method rejects an empty tree.
2. **Leaf node:** If both children are `null`, return the node's own value.
3. **Only one child exists:** Continue through that child. This ensures the path always ends at a leaf.
4. **Both children exist:** Recursively calculate the maximum sum from the left and right children, then add the larger sum to the current node's value.

For the first example:

```text
maxSum(20) = 20
maxSum(1)  = 1
maxSum(2)  = 2 + max(20, 1) = 22

maxSum(4)  = 4
maxSum(-25) = -25 + 4 = -21
maxSum(10 on right) = 10 + (-21) = -11

maxSum(root) = 10 + max(22, -11)
             = 32
```

**Important:** We must handle nodes with only one child separately. Otherwise, treating a missing child as having a sum of `0` can incorrectly select a path that ends before reaching a leaf, especially when node values are negative.

### Complexity Analysis

- **Time Complexity:** \(O(n)\), where \(n\) is the number of nodes. Each node is visited once.
- **Space Complexity:** \(O(h)\) auxiliary space for the recursion stack, where \(h\) is the height of the tree. In a balanced tree, this is \(O(\log n)\); in a skewed tree, it is \(O(n)\).

### Key Points

- The path must start at the root and end at a leaf.
- Negative node values are valid and must be handled correctly.
- Always choose between the existing child paths, not a nonexistent child.
- The recursive approach is efficient because it visits every node once.

### Follow-up Questions

1. How would you return the actual maximum-sum path, not just its sum?
2. How would you solve this problem iteratively using a stack?
3. How would the solution change if the path could start and end at any two nodes?
4. What is the difference between the maximum root-to-leaf path sum and the maximum path sum in a binary tree?

---


## Path Sum — Root-to-Leaf Path Equals Target

Easy
{: .label .label-green }

**Problem Statement**

Given the root of a binary tree and an integer `targetSum`, return `true` if there exists a root-to-leaf path such that the sum of all node values along that path equals `targetSum`. Otherwise, return `false`.

A leaf is a node that has no left or right child.

The path must start at the root and end at a leaf. A path that ends at an internal node does not count.

### Example 1

**Input**

```text
root = [5,4,8,11,null,13,4,7,2,null,null,null,1]
targetSum = 22
```

**Output**

```text
true
```

**Explanation**

The path `5 → 4 → 11 → 2` has a sum of:

```text
5 + 4 + 11 + 2 = 22
```

Since this path ends at a leaf and equals the target sum, return `true`.

### Example 2

**Input**

```text
root = [1,2,3]
targetSum = 5
```

**Output**

```text
false
```

**Explanation**

- Path `1 → 2`: sum = 3.
- Path `1 → 3`: sum = 4.

Neither root-to-leaf path has a sum of 5.

### Example 3

**Input**

```text
root = []
targetSum = 0
```

**Output**

```text
false
```

**Explanation**

An empty tree has no root-to-leaf path.

### Java Solution — Recursion

```java
public class PathSum {

    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int val) {
            this.val = val;
        }
    }

    public static boolean hasPathSum(
            TreeNode root, int targetSum) {

        // An empty tree has no valid path.
        if (root == null) {
            return false;
        }

        // Check whether the current node is a leaf.
        if (root.left == null && root.right == null) {
            return root.val == targetSum;
        }

        // Subtract the current value and search either subtree.
        int remainingSum = targetSum - root.val;

        return hasPathSum(root.left, remainingSum)
                || hasPathSum(root.right, remainingSum);
    }

    public static void main(String[] args) {
        TreeNode root = new TreeNode(5);

        root.left = new TreeNode(4);
        root.right = new TreeNode(8);

        root.left.left = new TreeNode(11);
        root.left.left.left = new TreeNode(7);
        root.left.left.right = new TreeNode(2);

        root.right.left = new TreeNode(13);
        root.right.right = new TreeNode(4);
        root.right.right.right = new TreeNode(1);

        System.out.println(hasPathSum(root, 22));
        System.out.println(hasPathSum(root, 26));
        System.out.println(hasPathSum(null, 0));
    }
}
```

**Output**

```text
true
true
false
```

### Explanation

The solution uses **Depth-First Search (DFS)** with recursion.

**Step 1: Handle an empty tree**

```java
if (root == null) {
    return false;
}
```

If there is no node, there cannot be a valid root-to-leaf path.

**Step 2: Check the leaf node**

```java
if (root.left == null && root.right == null) {
    return root.val == targetSum;
}
```

At a leaf, the remaining target must equal the leaf's value. We only return `true` when the entire path matches the target.

**Step 3: Update the remaining sum**

```java
int remainingSum = targetSum - root.val;
```

Instead of calculating the complete sum at every step, subtract each visited node's value from the target.

For Example 1:

| Node visited | Remaining sum |
|---|---:|
| 5 | 22 − 5 = 17 |
| 4 | 17 − 4 = 13 |
| 11 | 13 − 11 = 2 |
| 2 | 2 − 2 = 0 |

At node `2`, we reach a leaf and the remaining sum is `0`. Therefore, the path is valid.

**Step 4: Explore both subtrees**

```java
return hasPathSum(root.left, remainingSum)
        || hasPathSum(root.right, remainingSum);
```

If either subtree contains a valid path, the result is `true`. The `||` operator short-circuits when the left subtree already returns `true`.

### Complexity Analysis

- **Time Complexity:** \(O(n)\) in the worst case, where \(n\) is the number of nodes. Each node may be visited once.
- **Space Complexity:** \(O(h)\) for the recursive call stack, where \(h\) is the height of the tree. This is \(O(\log n)\) for a balanced tree and \(O(n)\) for a skewed tree.

### Key Points

- The path must begin at the root and end at a leaf.
- A matching sum at an internal node is not sufficient.
- Negative node values are supported.
- The empty tree always returns `false`, even when `targetSum` is `0`.
- Recursion naturally explores each possible root-to-leaf path.

### Follow-up Questions

1. How can you solve this problem iteratively using a stack?
2. How would you return the actual path instead of a boolean?
3. How would you find all root-to-leaf paths whose sums equal `targetSum`?
4. What changes would be needed to count the number of matching paths?

---


## Count Anagrams of a Pattern in a String

Medium
{: .label .label-yellow }

**Problem Statement**

Given two strings, `text` and `pattern`, count how many substrings of `text` are anagrams of `pattern`.

Two strings are anagrams if they contain exactly the same characters with the same frequencies, but the characters may appear in a different order.

Each substring must be contiguous and have the same length as `pattern`.

### Example 1

**Input**

```text
text = "cbaebabacd"
pattern = "abc"
```

**Output**

```text
2
```

**Explanation**

The substrings that are anagrams of `"abc"` are:

- `"cba"` — indices 0 to 2
- `"bac"` — indices 6 to 8

Therefore, the count is `2`.

### Example 2

**Input**

```text
text = "abab"
pattern = "ab"
```

**Output**

```text
3
```

**Explanation**

The matching substrings are `"ab"`, `"ba"`, and `"ab"`.

### Java Solution — Sliding Window

```java
public class CountAnagrams {

    public static int countAnagrams(String text, String pattern) {
        if (text == null || pattern == null
                || pattern.isEmpty()
                || pattern.length() > text.length()) {
            return 0;
        }

        int[] patternFreq = new int[256];
        int[] windowFreq = new int[256];

        int windowSize = pattern.length();

        // Count characters in the pattern.
        for (char ch : pattern.toCharArray()) {
            if (ch >= 256) {
                throw new IllegalArgumentException(
                        "Only ASCII characters are supported");
            }
            patternFreq[ch]++;
        }

        int count = 0;

        for (int right = 0; right < text.length(); right++) {
            char incoming = text.charAt(right);

            if (incoming >= 256) {
                throw new IllegalArgumentException(
                        "Only ASCII characters are supported");
            }

            // Add the new character to the window.
            windowFreq[incoming]++;

            // Remove the character that falls outside the window.
            if (right >= windowSize) {
                char outgoing = text.charAt(right - windowSize);
                windowFreq[outgoing]--;
            }

            // Compare frequencies when the window reaches full size.
            if (right >= windowSize - 1
                    && areEqual(patternFreq, windowFreq)) {
                count++;
            }
        }

        return count;
    }

    private static boolean areEqual(int[] a, int[] b) {
        for (int i = 0; i < 256; i++) {
            if (a[i] != b[i]) {
                return false;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        System.out.println(
                countAnagrams("cbaebabacd", "abc"));

        System.out.println(
                countAnagrams("abab", "ab"));

        System.out.println(
                countAnagrams("abcdef", "xyz"));
    }
}
```

**Output**

```text
2
3
0
```

### Explanation

The solution uses the **sliding window technique**. Because every anagram must have the same character frequencies and length as the pattern, we only need to examine windows of that fixed length.

**Step 1: Count the pattern's characters**

For `pattern = "abc"`:

```text
a → 1
b → 1
c → 1
```

**Step 2: Create a window of the same length**

For `text = "cbaebabacd"`, the first window is `"cba"`.

Its character frequencies match the pattern, so increment the count.

**Step 3: Slide the window**

Move one position to the right each time:

- Add the incoming character.
- Remove the outgoing character.
- Compare the window's frequency array with the pattern's frequency array.

For example:

```text
Pattern: abc
Window:  cba  → match
Window:  bae  → no match
Window:  aeb  → no match
...
Window:  bac  → match
```

**Step 4: Return the count**

The total number of matching windows is `2`.

### Complexity Analysis

Let \(n\) be the length of `text` and \(m\) the length of `pattern`.

- **Time Complexity:** \(O(n \times 256)\), which simplifies to \(O(n)\) because the alphabet size is fixed.
- **Space Complexity:** \(O(256)\), which is \(O(1)\) for the fixed-size frequency arrays.

### Key Points

- Anagrams must have identical character frequencies.
- The sliding window avoids generating every substring separately.
- Overlapping matches are counted independently.
- This implementation supports ASCII characters. For general Unicode text, use a map-based frequency approach or an appropriately defined character model.

### Follow-up Questions

1. How would you count anagrams using a `HashMap<Character, Integer>`?
2. How can you optimize the solution to avoid comparing all 256 frequency entries for every window?
3. How would you return the starting indices of all matching anagrams?
4. What is the difference between an anagram substring and a subsequence?


---


## Interleaving Strings

Medium
{: .label .label-yellow }

**Problem Statement**

Given three strings `s1`, `s2`, and `s3`, determine whether `s3` can be formed by interleaving `s1` and `s2`.

An interleaving preserves the original character order of both `s1` and `s2`. At each step, we choose the next character from either string to build `s3`.

Characters cannot be skipped, rearranged within either source string, or added from anywhere else.

**Important condition:**

```text
s3.length() == s1.length() + s2.length()
```

If this condition is not satisfied, return `false`.

### Example 1

**Input**

```text
s1 = "aabcc"
s2 = "dbbca"
s3 = "aadbbcbcac"
```

**Output**

```text
true
```

**Explanation**

We can construct `s3` by taking characters alternately from `s1` and `s2`, while preserving the order of characters within each string.

One valid grouping is:

```text
s1 = "aa" + "bc" + "c"
s2 = "dbbc" + "a"

s3 = "aa" + "dbbc" + "bc" + "a" + "c"
   = "aadbbcbcac"
```

Therefore, return `true`.

### Example 2

**Input**

```text
s1 = "aabcc"
s2 = "dbbca"
s3 = "aadbbbaccc"
```

**Output**

```text
false
```

**Explanation**

Although the lengths match, the characters cannot be selected in a way that preserves the order of both source strings.

### Java Solution — Dynamic Programming

```java
public class InterleavingString {

    public static boolean isInterleave(
            String s1, String s2, String s3) {

        if (s1.length() + s2.length() != s3.length()) {
            return false;
        }

        int m = s1.length();
        int n = s2.length();

        // dp[i][j] indicates whether the first i characters
        // of s1 and first j characters of s2 form s3's
        // first i + j characters.
        boolean[][] dp = new boolean[m + 1][n + 1];

        dp[0][0] = true;

        // Use characters from s1 only.
        for (int i = 1; i <= m; i++) {
            dp[i][0] = dp[i - 1][0]
                    && s1.charAt(i - 1) == s3.charAt(i - 1);
        }

        // Use characters from s2 only.
        for (int j = 1; j <= n; j++) {
            dp[0][j] = dp[0][j - 1]
                    && s2.charAt(j - 1) == s3.charAt(j - 1);
        }

        // Combine characters from both strings.
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {

                char target = s3.charAt(i + j - 1);

                boolean fromS1 =
                        dp[i - 1][j]
                        && s1.charAt(i - 1) == target;

                boolean fromS2 =
                        dp[i][j - 1]
                        && s2.charAt(j - 1) == target;

                dp[i][j] = fromS1 || fromS2;
            }
        }

        return dp[m][n];
    }

    public static void main(String[] args) {
        System.out.println(
                isInterleave("aabcc", "dbbca", "aadbbcbcac"));

        System.out.println(
                isInterleave("aabcc", "dbbca", "aadbbbaccc"));

        System.out.println(
                isInterleave("", "", ""));
    }
}
```

**Output**

```text
true
false
true
```

### Explanation

We use a two-dimensional boolean DP table to explore whether each prefix of `s1` and each prefix of `s2` can form the corresponding prefix of `s3`.

**1. Define the DP state**

```java
dp[i][j]
```

This is `true` if the first `i` characters of `s1` and first `j` characters of `s2` can form the first `i + j` characters of `s3`.

**2. Initialize the base case**

```java
dp[0][0] = true;
```

Two empty strings can form an empty string.

We then initialize the first row and first column to handle cases where characters come entirely from one source string.

**3. Calculate each state**

The next required character in `s3` is:

```java
char target = s3.charAt(i + j - 1);
```

There are two possibilities:

- **Take from `s1`:** The previous state `dp[i - 1][j]` must be valid, and the next character of `s1` must match `target`.
- **Take from `s2`:** The previous state `dp[i][j - 1]` must be valid, and the next character of `s2` must match `target`.

Therefore:

```java
dp[i][j] =
        (dp[i - 1][j] && s1.charAt(i - 1) == target)
        || (dp[i][j - 1] && s2.charAt(j - 1) == target);
```

**4. Return the final state**

```java
return dp[m][n];
```

If this state is `true`, the entire strings `s1` and `s2` can be interleaved to form `s3`.

### Complexity Analysis

Let \(m = |s1|\) and \(n = |s2|\).

- **Time Complexity:** \(O(m \times n)\), because each DP state is calculated once.
- **Space Complexity:** \(O(m \times n)\), for the two-dimensional DP table.

### Key Points

- The relative order of characters in both source strings must be preserved.
- Matching character frequencies alone does not prove that an interleaving is possible.
- The length check quickly rejects impossible cases.
- Dynamic programming avoids repeatedly exploring the same combinations.
- An iterative DP solution avoids the potential stack overflow of deep recursion.

### Follow-up Questions

1. How can you optimize the space complexity to \(O(\min(m,n))\)?
2. Can you solve this problem using recursion and memoization?
3. Why is a greedy approach unreliable when characters repeat?
4. What is the difference between interleaving strings and merging two sorted arrays?


---


## Find the First Duplicate in an Array

Easy
{: .label .label-green }

**Problem Statement**

Given an array of integers, write a Java function to find the first duplicate element.

For this problem, the **first duplicate** means the element whose second occurrence appears earliest while traversing the array from left to right.

If no duplicate exists, return `-1`.

### Example 1

**Input**

```text
arr = [2, 1, 3, 2, 3, 4, 5]
```

**Output**

```text
2
```

**Explanation**

- `2` appears first at index `0` and again at index `3`.
- `3` appears first at index `2` and again at index `4`.

The second occurrence of `2` comes before the second occurrence of `3`. Therefore, the answer is `2`.

### Java Solution — HashSet

```java
import java.util.HashSet;
import java.util.Set;

public class FirstDuplicate {

    public static int findFirstDuplicate(int[] arr) {
        Set<Integer> seen = new HashSet<>();

        for (int num : arr) {
            if (!seen.add(num)) {
                return num;
            }
        }

        return -1;
    }

    public static void main(String[] args) {
        int[] arr1 = {2, 1, 3, 2, 3, 4, 5};
        int[] arr2 = {1, 2, 3, 4, 5};
        int[] arr3 = {5, 4, 3, 4, 5};

        System.out.println(findFirstDuplicate(arr1));
        System.out.println(findFirstDuplicate(arr2));
        System.out.println(findFirstDuplicate(arr3));
    }
}
```

**Output**

```text
2
-1
4
```

### Explanation

The solution uses a `HashSet` to track elements already encountered.

**Step 1: Create a set**

```java
Set<Integer> seen = new HashSet<>();
```

A set stores unique values and allows us to check whether an element has appeared before.

**Step 2: Traverse the array**

```java
for (int num : arr)
```

Process elements from left to right.

**Step 3: Detect the first duplicate**

```java
if (!seen.add(num)) {
    return num;
}
```

The `add()` method returns:

- `true` if the element is new and was added successfully.
- `false` if the element already exists in the set.

As soon as `add()` returns `false`, we return that element. This guarantees that the duplicate's second occurrence is the earliest one encountered.

**Step 4: Handle no duplicates**

```java
return -1;
```

If the loop finishes without finding a duplicate, return `-1`.

### Complexity Analysis

- **Time Complexity:** \(O(n)\) average, where \(n\) is the array length.
- **Space Complexity:** \(O(n)\) in the worst case for the `HashSet`.

### Alternative: Using a HashMap

A `HashMap<Integer, Integer>` can track the frequency of each number. Return the first number whose frequency reaches `2`. However, the `HashSet` solution is simpler when only duplicate detection is needed.

### Key Points

- Traverse the array from left to right.
- Return immediately when an element is encountered for the second time.
- A `HashSet` avoids nested loops and provides average constant-time membership checks.
- This interpretation finds the earliest **second occurrence**, not necessarily the duplicate with the smallest first index under every possible definition.

### Follow-up Questions

1. How would you find all duplicate elements?
2. How would you find the first duplicate without using extra space?
3. How would you find the element that repeats most frequently?
4. What is the difference between `HashSet` and `HashMap`?

---
