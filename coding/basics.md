---
title: Basics
parent: Coding Problems
nav_order: 1
---

# Basic Java Problems
{: .no_toc }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }

  1. TOC
  {:toc}
</details>

---


## Check Armstrong Number

Easy
{: .label .label-green }

**Problem Statement**

Given a non-negative integer `n`, determine whether it is an Armstrong number.

An Armstrong number is a number equal to the sum of its digits, where each digit is raised to the power of the total number of digits in the number.

**Formula**

For a number with \(k\) digits:

\[
n = d_1^k + d_2^k + \cdots + d_k^k
\]

### Example 1

**Input**

```text
n = 153
```

**Output**

```text
true
```

**Explanation**

153 has 3 digits.

\[
1^3 + 5^3 + 3^3
= 1 + 125 + 27
= 153
\]

Since the sum equals the original number, `153` is an Armstrong number.

### Example 2

**Input**

```text
n = 123
```

**Output**

```text
false
```

**Explanation**

\[
1^3 + 2^3 + 3^3 = 36
\]

Since 36 is not equal to 123, it is not an Armstrong number.

### Java Solution

```java
public class ArmstrongNumber {

    public static boolean isArmstrong(int n) {
        if (n < 0) {
            return false;
        }

        int digits = String.valueOf(n).length();
        int original = n;
        long sum = 0;

        do {
            int digit = n % 10;
            sum += (long) Math.pow(digit, digits);
            n /= 10;
        } while (n > 0);

        return sum == original;
    }

    public static void main(String[] args) {
        System.out.println(isArmstrong(153));
        System.out.println(isArmstrong(370));
        System.out.println(isArmstrong(9474));
        System.out.println(isArmstrong(123));
        System.out.println(isArmstrong(0));
    }
}
```

**Output**

```text
true
true
true
false
true
```

### Explanation

1. **Count the digits:** `String.valueOf(n).length()` gives the number of digits.
2. **Extract the last digit:** `n % 10` retrieves the last digit.
3. **Raise it to the required power:** Each digit is raised to the total number of digits.
4. **Add the result:** Accumulate the powered digits in `sum`.
5. **Remove the last digit:** `n /= 10` removes the digit just processed.
6. **Compare:** If `sum` equals the original number, return `true`.

The `do-while` loop ensures that `0` is processed as a one-digit number.

### Complexity Analysis

- **Time Complexity:** \(O(d)\) digit-processing iterations, where \(d\) is the number of digits. Counting digits via string conversion also takes \(O(d)\).
- **Space Complexity:** \(O(d)\) for converting the number to a string to count its digits; the arithmetic loop itself uses \(O(1)\) extra space.

### Key Points

- All single-digit non-negative integers are Armstrong numbers.
- Negative integers are not Armstrong numbers under this definition.
- Use `long` for the sum to reduce the risk of intermediate integer overflow.
- `Math.pow()` returns a `double`; for strict integer arithmetic, an integer-power helper can be used instead.

### Follow-up Questions

1. How would you check an Armstrong number without converting the number to a string?
2. How would you find all Armstrong numbers between 1 and 10,000?
3. What is the difference between an Armstrong number and a palindrome number?
4. How would you implement the digit power calculation without using `Math.pow()`?


---


## Calculate Average of Numbers

Easy
{: .label .label-green }

**Problem Statement**

Given an array of numbers, calculate and return their arithmetic average.

The average is calculated by dividing the sum of all numbers by the total number of elements.

### Example 1

**Input**

```text
numbers = [10, 20, 30, 40, 50]
```

**Output**

```text
30.0
```

**Explanation**

Sum = 10 + 20 + 30 + 40 + 50 = 150

Number of elements = 5

Average = 150 / 5 = 30.0

### Java Solution — Using a Loop

```java
public class CalculateAverage {

    public static double calculateAverage(int[] numbers) {
        if (numbers == null || numbers.length == 0) {
            throw new IllegalArgumentException(
                    "Array must not be null or empty");
        }

        int sum = 0;

        for (int number : numbers) {
            sum += number;
        }

        return (double) sum / numbers.length;
    }

    public static void main(String[] args) {
        int[] numbers = {10, 20, 30, 40, 50};

        System.out.println(calculateAverage(numbers));
    }
}
```

**Output**

```text
30.0
```

### Java 8 Solution — Using Streams

```java
import java.util.Arrays;

public class CalculateAverage {

    public static double calculateAverage(int[] numbers) {
        if (numbers == null || numbers.length == 0) {
            throw new IllegalArgumentException(
                    "Array must not be null or empty");
        }

        return Arrays.stream(numbers)
                .average()
                .orElse(0.0);
    }

    public static void main(String[] args) {
        int[] numbers = {10, 20, 30, 40, 50};

        System.out.println(calculateAverage(numbers));
    }
}
```

**Output**

```text
30.0
```

### Explanation

**Using a loop**

1. Initialize `sum` to zero.
2. Iterate through every number and add it to `sum`.
3. Divide the sum by the array length.
4. Cast the sum to `double` before division to preserve decimal values.

**Using Java 8 Streams**

- `Arrays.stream(numbers)` creates an `IntStream`.
- `.average()` calculates the average and returns an `OptionalDouble`.
- `.orElse(0.0)` supplies a fallback if the stream is empty. The validation above rejects empty arrays first.

### Example 2 — Average With a Decimal Result

**Input**

```text
numbers = [10, 20, 25]
```

**Output**

```text
18.333333333333332
```

The sum is `55`, and the number of elements is `3`. Therefore, the average is \(55 / 3 \approx 18.33\).

### Complexity Analysis

- **Time Complexity:** \(O(n)\), because every element is processed once.
- **Space Complexity:** \(O(1)\) auxiliary space for the loop solution. The stream solution also uses constant auxiliary space for this calculation.

### Key Points

- Return `double` to support fractional averages.
- Handle null or empty arrays explicitly.
- Avoid integer division when a decimal result is required.
- For very large integer values, use a `long` accumulator or a suitable numeric strategy to avoid overflow.

### Follow-up Questions

1. How would you calculate the average of a `List<Integer>` using Java 8 Streams?
2. How would you calculate the average of student marks using `mapToInt()`?
3. How would you find the average without using a loop or streams?
4. What is the difference between `map()` and `mapToInt()` in Java 8?


---


## Bubble Sort in Java

Easy
{: .label .label-green }

**Problem Statement**

Given an array of integers, sort the elements in ascending order using the Bubble Sort algorithm.

Bubble Sort repeatedly compares adjacent elements and swaps them if they are in the wrong order. After each pass, the largest unsorted element moves to its correct position at the end of the array.

### Example

**Input**

```text
arr = [5, 1, 4, 2, 8]
```

**Output**

```text
[1, 2, 4, 5, 8]
```

### Java Solution

```java
import java.util.Arrays;

public class BubbleSort {

    public static void bubbleSort(int[] arr) {
        int n = arr.length;

        for (int i = 0; i < n - 1; i++) {
            boolean swapped = false;

            for (int j = 0; j < n - 1 - i; j++) {

                if (arr[j] > arr[j + 1]) {
                    // Swap adjacent elements
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;

                    swapped = true;
                }
            }

            // Stop early if the array is already sorted
            if (!swapped) {
                break;
            }
        }
    }

    public static void main(String[] args) {
        int[] arr = {5, 1, 4, 2, 8};

        bubbleSort(arr);

        System.out.println(Arrays.toString(arr));
    }
}
```

**Output**

```text
[1, 2, 4, 5, 8]
```

### Explanation

**Pass 1**

Starting array: `[5, 1, 4, 2, 8]`

- Compare 5 and 1 → swap: `[1, 5, 4, 2, 8]`
- Compare 5 and 4 → swap: `[1, 4, 5, 2, 8]`
- Compare 5 and 2 → swap: `[1, 4, 2, 5, 8]`
- Compare 5 and 8 → no swap.

The largest element, `8`, is already at the end.

**Pass 2**

- Compare 1 and 4 → no swap.
- Compare 4 and 2 → swap: `[1, 2, 4, 5, 8]`
- Compare 4 and 5 → no swap.

The remaining passes continue until the array is sorted or a complete pass makes no swaps.

### Complexity Analysis

| Case | Time Complexity |
|---|---|
| Best case (already sorted) | \(O(n)\) |
| Average case | \(O(n^2)\) |
| Worst case (reverse sorted) | \(O(n^2)\) |

- **Space Complexity:** \(O(1)\) auxiliary space because sorting is performed in place.
- **Stable:** Yes. Equal elements are not swapped when using `>`.
- **Optimized:** The `swapped` flag allows early termination when the array is already sorted.

### Key Points

- Bubble Sort compares adjacent elements.
- The inner loop becomes shorter after each pass.
- The `swapped` flag improves best-case performance.
- Bubble Sort is simple to understand but inefficient for large arrays.

### Follow-up Questions

1. How would you sort the array in descending order?
2. Why does the inner loop use `n - 1 - i`?
3. What is the difference between Bubble Sort, Selection Sort, and Insertion Sort?
4. Why is Bubble Sort stable?


---



## Check Whether Two Strings Are Anagrams

Easy
{: .label .label-green }

**Problem Statement**

Given two strings `s1` and `s2`, determine whether they are anagrams of each other.

Two strings are anagrams if they contain exactly the same characters with the same frequencies, but the characters may appear in a different order.

For this solution, the comparison is case-sensitive and includes spaces and punctuation.

### Example 1

**Input**

```text
s1 = "listen"
s2 = "silent"
```

**Output**

```text
true
```

**Explanation**

Both strings contain the same characters with the same frequencies.

### Example 2

**Input**

```text
s1 = "hello"
s2 = "world"
```

**Output**

```text
false
```

**Explanation**

The strings do not contain the same characters with the same frequencies.

### Java Solution 1 — Using Sorting

```java
import java.util.Arrays;

public class AnagramCheck {

    public static boolean areAnagrams(String s1, String s2) {
        if (s1 == null || s2 == null) {
            return s1 == s2;
        }

        if (s1.length() != s2.length()) {
            return false;
        }

        char[] arr1 = s1.toCharArray();
        char[] arr2 = s2.toCharArray();

        Arrays.sort(arr1);
        Arrays.sort(arr2);

        return Arrays.equals(arr1, arr2);
    }

    public static void main(String[] args) {
        System.out.println(areAnagrams("listen", "silent"));
        System.out.println(areAnagrams("hello", "world"));
        System.out.println(areAnagrams("aabb", "baba"));
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

1. Check whether either string is `null`.
2. If their lengths differ, they cannot be anagrams.
3. Convert both strings into character arrays.
4. Sort both arrays.
5. If the sorted arrays are equal, the strings are anagrams.

For example:

```text
"listen" → ['e', 'i', 'l', 'n', 's', 't']
"silent" → ['e', 'i', 'l', 'n', 's', 't']
```

Both sorted arrays are identical, so the result is `true`.

### Java Solution 2 — Using Character Frequency

This approach counts each character in the first string and subtracts the counts using the second string.

```java
public class AnagramCheck {

    public static boolean areAnagrams(String s1, String s2) {
        if (s1 == null || s2 == null) {
            return s1 == s2;
        }

        if (s1.length() != s2.length()) {
            return false;
        }

        int[] frequency = new int[Character.MAX_VALUE + 1];

        for (int i = 0; i < s1.length(); i++) {
            frequency[s1.charAt(i)]++;
            frequency[s2.charAt(i)]--;
        }

        for (int count : frequency) {
            if (count != 0) {
                return false;
            }
        }

        return true;
    }

    public static void main(String[] args) {
        System.out.println(areAnagrams("listen", "silent"));
        System.out.println(areAnagrams("race", "care"));
        System.out.println(areAnagrams("aab", "abb"));
    }
}
```

**Output**

```text
true
true
false
```

### Explanation of the Frequency Approach

For `s1 = "aab"` and `s2 = "abb"`:

| Character | Frequency in `s1` | Frequency in `s2` |
|---|---:|---:|
| `a` | 2 | 1 |
| `b` | 1 | 2 |

The frequencies differ, so the method returns `false`.

### Complexity Analysis

Let \(n\) be the length of each string.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Sorting | \(O(n \log n)\) | \(O(n)\) |
| Frequency array | \(O(n + 65536)\) | \(O(1)\) for the fixed-size array |

The frequency-array solution is linear in the input length when the character set is fixed. It counts Java UTF-16 `char` values, not full Unicode code points or grapheme clusters.

### Key Points

- Anagrams must have equal lengths and identical character frequencies.
- Sorting is straightforward and easy to explain in interviews.
- Frequency counting avoids sorting.
- Decide whether case, whitespace, and punctuation should matter based on the problem requirements.

### Follow-up Questions

1. How would you check anagrams using Java 8 Streams?
2. How would you make the comparison case-insensitive?
3. How would you ignore spaces and punctuation?
4. How would you check anagrams for Unicode code points?


---


## Check Whether Two Strings Are Anagrams

Easy
{: .label .label-green }

**Problem Statement**

Given two strings `s1` and `s2`, determine whether they are anagrams of each other.

Two strings are anagrams if they contain exactly the same characters with the same frequencies, but the characters may appear in a different order.

For this solution, the comparison is case-sensitive and includes spaces and punctuation.

### Example 1

**Input**

```text
s1 = "listen"
s2 = "silent"
```

**Output**

```text
true
```

**Explanation**

Both strings contain the same characters with the same frequencies.

### Example 2

**Input**

```text
s1 = "hello"
s2 = "world"
```

**Output**

```text
false
```

**Explanation**

The strings do not contain the same characters with the same frequencies.

### Java Solution 1 — Using Sorting

```java
import java.util.Arrays;

public class AnagramCheck {

    public static boolean areAnagrams(String s1, String s2) {
        if (s1 == null || s2 == null) {
            return s1 == s2;
        }

        if (s1.length() != s2.length()) {
            return false;
        }

        char[] arr1 = s1.toCharArray();
        char[] arr2 = s2.toCharArray();

        Arrays.sort(arr1);
        Arrays.sort(arr2);

        return Arrays.equals(arr1, arr2);
    }

    public static void main(String[] args) {
        System.out.println(areAnagrams("listen", "silent"));
        System.out.println(areAnagrams("hello", "world"));
        System.out.println(areAnagrams("aabb", "baba"));
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

1. Check whether either string is `null`.
2. If their lengths differ, they cannot be anagrams.
3. Convert both strings into character arrays.
4. Sort both arrays.
5. If the sorted arrays are equal, the strings are anagrams.

For example:

```text
"listen" → ['e', 'i', 'l', 'n', 's', 't']
"silent" → ['e', 'i', 'l', 'n', 's', 't']
```

Both sorted arrays are identical, so the result is `true`.

### Java Solution 2 — Using Character Frequency

This approach counts each character in the first string and subtracts the counts using the second string.

```java
public class AnagramCheck {

    public static boolean areAnagrams(String s1, String s2) {
        if (s1 == null || s2 == null) {
            return s1 == s2;
        }

        if (s1.length() != s2.length()) {
            return false;
        }

        int[] frequency = new int[Character.MAX_VALUE + 1];

        for (int i = 0; i < s1.length(); i++) {
            frequency[s1.charAt(i)]++;
            frequency[s2.charAt(i)]--;
        }

        for (int count : frequency) {
            if (count != 0) {
                return false;
            }
        }

        return true;
    }

    public static void main(String[] args) {
        System.out.println(areAnagrams("listen", "silent"));
        System.out.println(areAnagrams("race", "care"));
        System.out.println(areAnagrams("aab", "abb"));
    }
}
```

**Output**

```text
true
true
false
```

### Explanation of the Frequency Approach

For `s1 = "aab"` and `s2 = "abb"`:

| Character | Frequency in `s1` | Frequency in `s2` |
|---|---:|---:|
| `a` | 2 | 1 |
| `b` | 1 | 2 |

The frequencies differ, so the method returns `false`.

### Complexity Analysis

Let \(n\) be the length of each string.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Sorting | \(O(n \log n)\) | \(O(n)\) |
| Frequency array | \(O(n + 65536)\) | \(O(1)\) for the fixed-size array |

The frequency-array solution is linear in the input length when the character set is fixed. It counts Java UTF-16 `char` values, not full Unicode code points or grapheme clusters.

### Key Points

- Anagrams must have equal lengths and identical character frequencies.
- Sorting is straightforward and easy to explain in interviews.
- Frequency counting avoids sorting.
- Decide whether case, whitespace, and punctuation should matter based on the problem requirements.

### Follow-up Questions

1. How would you check anagrams using Java 8 Streams?
2. How would you make the comparison case-insensitive?
3. How would you ignore spaces and punctuation?
4. How would you check anagrams for Unicode code points?


---


## Factorial of a Number — Iterative and Recursive Approaches

Easy
{: .label .label-green }

**Problem Statement**

Given a non-negative integer `n`, calculate its factorial.

The factorial of a number is the product of all positive integers from `1` to `n`.

\[
n! = n \times (n-1) \times \cdots \times 2 \times 1
\]

Special case:

\[
0! = 1
\]

### Example 1

**Input**
```text
n = 5
```

**Output**
```text
120
```

**Explanation**

\[
5! = 5 \times 4 \times 3 \times 2 \times 1 = 120
\]

### Java Solution 1 — Using a Loop (Iterative)

```java
public class Factorial {

    public static long factorial(int n) {
        if (n < 0) {
            throw new IllegalArgumentException(
                "Factorial is not defined for negative integers"
            );
        }

        long result = 1;

        for (int i = 2; i <= n; i++) {
            result *= i;
        }

        return result;
    }

    public static void main(String[] args) {
        System.out.println(factorial(5));
        System.out.println(factorial(0));
    }
}
```

**Output**
```text
120
1
```

### Explanation

For `n = 5`:

| Iteration | Calculation | Result |
|---|---|---:|
| Initial | `result = 1` | 1 |
| 1 | `1 × 2` | 2 |
| 2 | `2 × 3` | 6 |
| 3 | `6 × 4` | 24 |
| 4 | `24 × 5` | 120 |

The loop multiplies the numbers from `2` through `n`. For `n = 0` or `n = 1`, the loop does not execute, so the result remains `1`.

### Java Solution 2 — Using Recursion

```java
public class Factorial {

    public static long factorial(int n) {
        if (n < 0) {
            throw new IllegalArgumentException(
                "Factorial is not defined for negative integers"
            );
        }

        if (n == 0 || n == 1) {
            return 1;
        }

        return n * factorial(n - 1);
    }

    public static void main(String[] args) {
        System.out.println(factorial(5));
        System.out.println(factorial(0));
    }
}
```

**Output**
```text
120
1
```

### Explanation of Recursion

Recursion occurs when a method calls itself to solve a smaller version of the same problem.

The formula is:

\[
n! = n \times (n-1)!
\]

The base case is:

\[
0! = 1,\quad 1! = 1
\]

For `factorial(5)`, the calls proceed as follows:

```text
factorial(5)
    5 × factorial(4)
        4 × factorial(3)
            3 × factorial(2)
                2 × factorial(1)
                    1  ← Base case
```

When the base case returns, the calls resolve in reverse order:

```text
factorial(1) = 1
factorial(2) = 2 × 1   = 2
factorial(3) = 3 × 2   = 6
factorial(4) = 4 × 6   = 24
factorial(5) = 5 × 24  = 120
```

### Complexity Analysis

Let \(n\) be the input number.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Iterative | \(O(n)\) | \(O(1)\) |
| Recursive | \(O(n)\) | \(O(n)\) call stack |

### Important Interview Points

- **Base case:** Stops recursion and prevents infinite calls.
- **Recursive case:** Reduces the problem from `n` to `n - 1`.
- **Overflow:** Java's `long` cannot store factorials beyond `20!`. For larger values, use `BigInteger`.
- **Which is better?** The iterative solution uses constant extra space and avoids stack overflow from deep recursion.

### Follow-up Question — Handle Large Factorials

Use `BigInteger` when the factorial may exceed the range of `long`.

```java
import java.math.BigInteger;

public class FactorialBigInteger {

    public static BigInteger factorial(int n) {
        if (n < 0) {
            throw new IllegalArgumentException("n must be non-negative");
        }

        BigInteger result = BigInteger.ONE;

        for (int i = 2; i <= n; i++) {
            result = result.multiply(BigInteger.valueOf(i));
        }

        return result;
    }

    public static void main(String[] args) {
        System.out.println(factorial(25));
    }
}
```

**Output**
```text
15511210043330985984000000
```


---


## Fibonacci Series up to a Number — Iterative and Recursive Approaches

Easy
{: .label .label-green }

**Problem Statement**

Given a non-negative integer `limit`, print all Fibonacci numbers that are less than or equal to the limit.

The Fibonacci series starts with `0` and `1`. Each subsequent number is the sum of the previous two numbers.

\[
F(n) = F(n-1) + F(n-2)
\]

Base cases:

\[
F(0)=0,\quad F(1)=1
\]

### Example

**Input**
```text
limit = 20
```

**Output**
```text
0 1 1 2 3 5 8 13
```

**Explanation**

Each number is the sum of the previous two numbers. The next number after `13` is `21`, which exceeds the limit of `20`.

### Java Solution 1 — Using a Loop (Iterative)

```java
public class FibonacciSeries {

    public static void printFibonacci(int limit) {
        if (limit < 0) {
            throw new IllegalArgumentException(
                "Limit must be non-negative"
            );
        }

        long first = 0;
        long second = 1;

        while (first <= limit) {
            System.out.print(first + " ");

            long next = first + second;
            first = second;
            second = next;
        }

        System.out.println();
    }

    public static void main(String[] args) {
        printFibonacci(20);
    }
}
```

**Output**
```text
0 1 1 2 3 5 8 13
```

### Explanation

Initially:

```text
first = 0
second = 1
```

In each iteration:

1. Print `first` if it is within the limit.
2. Calculate `next = first + second`.
3. Move `second` into `first`.
4. Move `next` into `second`.

| Iteration | First | Second | Next |
|---|---:|---:|---:|
| 1 | 0 | 1 | 1 |
| 2 | 1 | 1 | 2 |
| 3 | 1 | 2 | 3 |
| 4 | 2 | 3 | 5 |
| 5 | 3 | 5 | 8 |
| 6 | 5 | 8 | 13 |
| 7 | 8 | 13 | 21 |
| 8 | 13 | 21 | 34 |

The loop stops when `first` becomes greater than `20`.

### Java Solution 2 — Using Recursion

In this approach, a recursive method calculates each Fibonacci number. Another method prints the numbers while they remain within the limit.

```java
public class FibonacciRecursion {

    public static long fibonacci(int n) {
        if (n == 0) {
            return 0;
        }

        if (n == 1) {
            return 1;
        }

        return fibonacci(n - 1) + fibonacci(n - 2);
    }

    public static void printFibonacci(int limit) {
        if (limit < 0) {
            throw new IllegalArgumentException(
                "Limit must be non-negative"
            );
        }

        int n = 0;

        while (true) {
            long number = fibonacci(n);

            if (number > limit) {
                break;
            }

            System.out.print(number + " ");
            n++;
        }

        System.out.println();
    }

    public static void main(String[] args) {
        printFibonacci(20);
    }
}
```

**Output**
```text
0 1 1 2 3 5 8 13
```

### Explanation of Recursion

For example, `fibonacci(5)` calculates:

```text
fibonacci(5)
    = fibonacci(4) + fibonacci(3)
    = 3 + 2
    = 5
```

The recursive method uses these base cases:

- `fibonacci(0)` returns `0`.
- `fibonacci(1)` returns `1`.

For all other values, it returns the sum of the two preceding Fibonacci numbers.

**Important:** This basic recursive implementation recalculates the same Fibonacci values many times, making it inefficient for large inputs.

### Complexity Analysis

Let \(k\) be the number of Fibonacci terms printed and \(n\) the index of the largest term.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Iterative series | \(O(k)\) | \(O(1)\) |
| Naive recursion for one term | \(O(2^n)\) | \(O(n)\) call stack |
| Recursive series printing | Exponential overall | \(O(n)\) call stack |

The iterative approach is generally the best choice for printing a Fibonacci series up to a limit.

### Key Interview Points

- **Iterative approach:** Efficient and uses constant extra space.
- **Recursive approach:** Demonstrates recursion and base cases, but repeats calculations.
- **Limit condition:** Print only numbers less than or equal to the given limit.
- **Integer overflow:** Fibonacci numbers eventually exceed the range of `long`; for very large limits, use `BigInteger`.

### Follow-up Questions

1. Print the first `n` Fibonacci numbers instead of using a numeric limit.
2. Find only the `n`th Fibonacci number.
3. Optimize recursion using memoization.
4. Generate Fibonacci numbers using Java 8 Streams.


---


## Find the Largest Number in an Array — Normal and Java 8 Stream API

Easy
{: .label .label-green }

**Problem Statement**

Given an array of integers, find the largest number in the array using:

1. A normal `for` loop.
2. Java 8 Stream API.

### Example

**Input**
```text
arr = [10, 25, 7, 90, 45, 90, 12]
```

**Output**
```text
Largest number: 90
```

### Java Solution 1 — Using a Normal Loop

```java
public class LargestNumber {

    public static int findLargest(int[] arr) {
        if (arr == null || arr.length == 0) {
            throw new IllegalArgumentException(
                "Array must not be null or empty"
            );
        }

        int largest = arr[0];

        for (int i = 1; i < arr.length; i++) {
            if (arr[i] > largest) {
                largest = arr[i];
            }
        }

        return largest;
    }

    public static void main(String[] args) {
        int[] arr = {10, 25, 7, 90, 45, 90, 12};

        System.out.println("Largest number: " + findLargest(arr));
    }
}
```

**Output**
```text
Largest number: 90
```

### Explanation

1. Assume the first element is the largest.
2. Iterate through the remaining elements.
3. If the current element is greater than `largest`, update `largest`.
4. After the loop, return the largest value.

For the array `[10, 25, 7, 90, 45]`:

| Current number | Largest so far |
|---:|---:|
| 10 | 10 |
| 25 | 25 |
| 7 | 25 |
| 90 | 90 |
| 45 | 90 |

### Java Solution 2 — Using Java 8 Stream API

```java
import java.util.Arrays;

public class LargestNumberStream {

    public static int findLargest(int[] arr) {
        if (arr == null || arr.length == 0) {
            throw new IllegalArgumentException(
                "Array must not be null or empty"
            );
        }

        return Arrays.stream(arr)
                     .max()
                     .getAsInt();
    }

    public static void main(String[] args) {
        int[] arr = {10, 25, 7, 90, 45, 90, 12};

        System.out.println("Largest number: " + findLargest(arr));
    }
}
```

**Output**
```text
Largest number: 90
```

### Explanation of the Stream API

```java
Arrays.stream(arr)
      .max()
      .getAsInt();
```

- `Arrays.stream(arr)` converts the `int[]` into an `IntStream`.
- `.max()` finds the maximum element and returns an `OptionalInt`.
- `.getAsInt()` extracts the maximum value. It is safe here because the method checks that the array is not empty.

**Alternative using `orElseThrow()`**

```java
return Arrays.stream(arr)
             .max()
             .orElseThrow(() ->
                 new IllegalArgumentException("Array must not be empty")
             );
```

This is useful when working directly with a stream that might be empty.

### Complexity Analysis

Let \(n\) be the number of elements in the array.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Normal loop | \(O(n)\) | \(O(1)\) |
| Stream API | \(O(n)\) | \(O(1)\) auxiliary space for this operation |

Both approaches examine all elements to determine the maximum.

### Key Interview Points

- Initialize `largest` with `arr[0]`, not `0`, so negative arrays work correctly.
- Handle `null` and empty arrays.
- For an `int[]`, use `Arrays.stream(arr).max()`.
- For a `List<Integer>`, use `list.stream().max(Integer::compareTo)`.
- The Stream API is concise; the normal loop is straightforward and makes the comparison logic explicit.

### Follow-up Questions

1. Find the smallest number in an array.
2. Find the second-largest distinct number.
3. Find the largest number in a `List<Integer>` using Java 8 Streams.
4. Find the maximum number using `reduce()`.


---



## Calculate HCF (GCD) and LCM of Two Numbers

Easy
{: .label .label-green }

**Problem Statement**

Given two integers, calculate:

- **HCF (Highest Common Factor)** or **GCD (Greatest Common Divisor):** The largest positive integer that divides both numbers without leaving a remainder.
- **LCM (Least Common Multiple):** The smallest positive integer that is divisible by both numbers.

### Example

**Input**
```text
a = 12
b = 18
```

**Output**
```text
HCF / GCD = 6
LCM = 36
```

**Explanation**

Factors of `12`: `1, 2, 3, 4, 6, 12`

Factors of `18`: `1, 2, 3, 6, 9, 18`

The greatest common factor is `6`.

Multiples of `12`: `12, 24, 36, 48, ...`

Multiples of `18`: `18, 36, 54, ...`

The least common multiple is `36`.

### Java Solution 1 — Using a Normal Loop

```java
public class HCFLCM {

    public static int findHCF(int a, int b) {
        a = Math.abs(a);
        b = Math.abs(b);

        int hcf = 1;
        int min = Math.min(a, b);

        for (int i = 1; i <= min; i++) {
            if (a % i == 0 && b % i == 0) {
                hcf = i;
            }
        }

        return hcf;
    }

    public static long findLCM(int a, int b) {
        if (a == 0 || b == 0) {
            return 0;
        }

        return Math.abs((long) a / findHCF(a, b) * b);
    }

    public static void main(String[] args) {
        int a = 12;
        int b = 18;

        System.out.println("HCF / GCD = " + findHCF(a, b));
        System.out.println("LCM = " + findLCM(a, b));
    }
}
```

**Output**
```text
HCF / GCD = 6
LCM = 36
```

### Explanation

The `findHCF()` method checks every number from `1` to the smaller input. Whenever a number divides both inputs, it updates `hcf`.

The `findLCM()` method uses the relationship between HCF and LCM:

\[
\operatorname{LCM}(a,b)
=\frac{|a \times b|}{\operatorname{GCD}(a,b)}
\]

Dividing before multiplying helps reduce the chance of integer overflow.

**Complexity:** HCF using this loop takes \(O(\min(a,b))\) time for positive inputs. Auxiliary space is \(O(1)\).

### Java Solution 2 — Using the Euclidean Algorithm

The Euclidean algorithm is more efficient than checking every possible divisor.

The key property is:

\[
\operatorname{GCD}(a,b)=\operatorname{GCD}(b,a\bmod b)
\]

Repeat until the second number becomes zero. The first number is then the GCD.

```java 
public class HCFLCM {

    public static long findGCD(long a, long b) {
        a = Math.abs(a);
        b = Math.abs(b);

        while (b != 0) {
            long remainder = a % b;
            a = b;
            b = remainder;
        }

        return a;
    }

    public static long findLCM(long a, long b) {
        if (a == 0 || b == 0) {
            return 0;
        }

        long gcd = findGCD(a, b);

        return Math.abs((a / gcd) * b);
    }

    public static void main(String[] args) {
        long a = 12;
        long b = 18;

        System.out.println("HCF / GCD = " + findGCD(a, b));
        System.out.println("LCM = " + findLCM(a, b));
    }
}
```

**Output**
```text
HCF / GCD = 6
LCM = 36
```

### Dry Run of the Euclidean Algorithm

For `a = 12` and `b = 18`:

| Step | `a` | `b` | `a % b` |
|---|---:|---:|---:|
| 1 | 12 | 18 | 12 |
| 2 | 18 | 12 | 6 |
| 3 | 12 | 6 | 0 |

When `b = 0`, the algorithm returns `a`, which is `6`.

### Complexity Analysis

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Normal HCF loop | \(O(\min(a,b))\) | \(O(1)\) |
| Euclidean algorithm | \(O(\log(\min(a,b)))\) for positive inputs | \(O(1)\) |
| LCM after GCD | Dominated by GCD calculation | \(O(1)\) |

### Important Interview Points

- HCF and GCD mean the same thing.
- LCM is calculated efficiently using the GCD.
- If either input is zero, the LCM is zero.
- The GCD of `a` and `0` is `|a|`.
- For two zero inputs, this implementation returns GCD `0` and LCM `0`.
- Extremely large inputs can overflow even a `long` when calculating the LCM; use `BigInteger` if the full range must be supported.

### Follow-up Questions

1. Find the HCF of three numbers.
2. Calculate GCD using recursion.
3. Find the LCM without using the GCD formula.
4. Explain why the Euclidean algorithm is faster than checking all divisors.


---



## Program to Check Whether a Year Is a Leap Year

Easy
{: .label .label-green }

**Problem Statement**

Write a Java program to determine whether a given year is a leap year.

A year is a leap year if:

1. It is divisible by `400`, or
2. It is divisible by `4` but **not** divisible by `100`.

Otherwise, it is not a leap year.

### Examples

| Input Year | Output | Explanation |
|---:|---|---|
| 2024 | Leap year | Divisible by 4, not by 100 |
| 2023 | Not a leap year | Not divisible by 4 |
| 1900 | Not a leap year | Divisible by 100, not by 400 |
| 2000 | Leap year | Divisible by 400 |

### Java Solution 1 — Using If-Else

```java
public class LeapYear {

    public static boolean isLeapYear(int year) {
        if (year % 400 == 0) {
            return true;
        } else if (year % 100 == 0) {
            return false;
        } else if (year % 4 == 0) {
            return true;
        } else {
            return false;
        }
    }

    public static void main(String[] args) {
        int year = 2024;

        if (isLeapYear(year)) {
            System.out.println(year + " is a leap year");
        } else {
            System.out.println(year + " is not a leap year");
        }
    }
}
```

**Output**
```text
2024 is a leap year
```

### Java Solution 2 — Using a Single Condition

```java
public class LeapYear {

    public static boolean isLeapYear(int year) {
        return (year % 400 == 0)
                || (year % 4 == 0 && year % 100 != 0);
    }

    public static void main(String[] args) {
        int[] years = {2024, 2023, 1900, 2000};

        for (int year : years) {
            System.out.println(
                year + ": " +
                (isLeapYear(year) ? "Leap year" : "Not a leap year")
            );
        }
    }
}
```

**Output**
```text
2024: Leap year
2023: Not a leap year
1900: Not a leap year
2000: Leap year
```

### Explanation

The condition is:

```java
(year % 400 == 0) || (year % 4 == 0 && year % 100 != 0)
```

- `year % 400 == 0`: Century years divisible by 400 are leap years.
- `year % 4 == 0`: The year must be divisible by 4.
- `year % 100 != 0`: It must not be a century year unless divisible by 400.

**Remember the order:** Check divisibility by `400` first, then `100`, then `4` when using `if-else`.

### Complexity Analysis

- **Time Complexity:** \(O(1)\) — a fixed number of arithmetic operations.
- **Auxiliary Space:** \(O(1)\) — no additional data structures are required.

### Follow-up Questions

1. Why is 1900 not a leap year, but 2000 is?
2. Can you solve this using a ternary operator?
3. How would you check leap years for a list of years using Java 8 Streams?


---


## Calculate Power of a Number — Normal Loop and Recursion

Easy
{: .label .label-green }

**Problem Statement**

Given two integers, `base` and `exponent`, calculate the power of the base raised to the exponent.

For example:

- `(2, 2) = 2² = 4`
- `(3, 4) = 3⁴ = 81`
- `(5, 3) = 5³ = 125`

### Java Solution 1 — Using a Normal Loop

```java
public class PowerOfNumber {

    public static long power(int base, int exponent) {
        if (exponent < 0) {
            throw new IllegalArgumentException(
                "Exponent must be non-negative"
            );
        }

        long result = 1;

        for (int i = 1; i <= exponent; i++) {
            result *= base;
        }

        return result;
    }

    public static void main(String[] args) {
        System.out.println(power(2, 2));
        System.out.println(power(3, 4));
        System.out.println(power(5, 3));
        System.out.println(power(7, 0));
    }
}
```

**Output**

```text
4
81
125
1
```

### Explanation

For `power(3, 4)`:

| Iteration | Calculation | Result |
|---|---|---:|
| Initial | `result = 1` | 1 |
| 1 | `1 × 3` | 3 |
| 2 | `3 × 3` | 9 |
| 3 | `9 × 3` | 27 |
| 4 | `27 × 3` | 81 |

The loop multiplies the base by itself `exponent` times.

### Java Solution 2 — Using Recursion

```java
public class PowerRecursion {

    public static long power(int base, int exponent) {
        if (exponent < 0) {
            throw new IllegalArgumentException(
                "Exponent must be non-negative"
            );
        }

        if (exponent == 0) {
            return 1;
        }

        return base * power(base, exponent - 1);
    }

    public static void main(String[] args) {
        System.out.println(power(2, 2));
        System.out.println(power(3, 4));
        System.out.println(power(5, 3));
    }
}
```

**Output**

```text 
4
81
125
```

### Explanation of Recursion

The formula is:

\[
a^n = a \times a^{n-1}
\]

The base case is:

\[
a^0 = 1
\]

For `power(3, 4)`:

```text 
power(3, 4)
= 3 × power(3, 3)
= 3 × 3 × power(3, 2)
= 3 × 3 × 3 × power(3, 1)
= 3 × 3 × 3 × 3 × power(3, 0)
= 3 × 3 × 3 × 3 × 1
= 81
```

### Complexity Analysis

Let \(n\) be the exponent.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Normal loop | \(O(n)\) | \(O(1)\) |
| Recursion | \(O(n)\) | \(O(n)\) call stack |

### Important Interview Points

- Any nonzero number raised to the power `0` is `1`.
- `0⁰` is treated as `1` by these implementations, following the common programming convention.
- These methods support only non-negative exponents.
- A `long` can overflow for large powers; Java integer arithmetic does not automatically detect overflow.

### Follow-up Question — Can We Make It Faster?

Yes. **Exponentiation by squaring** calculates the power in \(O(\log n)\) time instead of \(O(n)\).

```java
public static long fastPower(long base, int exponent) {
    if (exponent < 0) {
        throw new IllegalArgumentException(
            "Exponent must be non-negative"
        );
    }

    long result = 1;

    while (exponent > 0) {
        if (exponent % 2 == 1) {
            result *= base;
        }

        base *= base;
        exponent /= 2;
    }

    return result;
}
```

For example, `fastPower(3, 4)` returns `81`. This approach is useful when the exponent is very large, provided the result fits within the `long` range.


---



## Find All Prime Numbers in a Given Range

Easy
{: .label .label-green }

**Problem Statement**

Given two integers `start` and `end`, print all prime numbers within the range, including the boundaries.

A **prime number** is a number greater than `1` that has exactly two positive divisors: `1` and itself.

Examples of prime numbers: `2, 3, 5, 7, 11, 13, 17`.

### Example

**Input**
```text
start = 10
end = 30
```

**Output**
```text
11 13 17 19 23 29
```

### Java Solution 1 — Using Normal Loops

```java
public class PrimeNumbers {

    public static boolean isPrime(int n) {
        if (n < 2) {
            return false;
        }

        for (int i = 2; i <= n / i; i++) {
            if (n % i == 0) {
                return false;
            }
        }

        return true;
    }

    public static void printPrimes(int start, int end) {
        for (int number = Math.max(2, start);
             number <= end;
             number++) {

            if (isPrime(number)) {
                System.out.print(number + " ");
            }
        }

        System.out.println();
    }

    public static void main(String[] args) {
        printPrimes(10, 30);
    }
}
```

**Output**
```text
11 13 17 19 23 29
```

### Explanation

1. Start from the given range's lower bound, but never below `2`.
2. Check each number using `isPrime()`.
3. If the number is less than `2`, it is not prime.
4. Check whether any integer from `2` through the square root of the number divides it evenly.
5. If a divisor is found, the number is not prime; otherwise, it is prime.

**Why check only up to the square root?**

If a number \(n\) has a divisor greater than \(\sqrt n\), it must also have a corresponding divisor smaller than \(\sqrt n\). Therefore, checking beyond the square root is unnecessary.

The condition `i <= n / i` avoids the potential integer overflow of `i * i`.

### Java Solution 2 — Using Java 8 Stream API

```java
import java.util.stream.IntStream;

public class PrimeNumbersStream {

    public static boolean isPrime(int n) {
        if (n < 2) {
            return false;
        }

        return IntStream.rangeClosed(2, n / 2)
                .noneMatch(i -> n % i == 0);
    }

    public static void main(String[] args) {
        int start = 10;
        int end = 30;

        IntStream.rangeClosed(Math.max(2, start), end)
                .filter(PrimeNumbersStream::isPrime)
                .forEach(n -> System.out.print(n + " "));

        System.out.println();
    }
}
```

**Output**
```text
11 13 17 19 23 29
```

This Stream API version is easy to read, but its primality check can be improved by checking divisors only up to the square root.

### Optimized Java 8 Stream API Solution

```java
import java.util.stream.IntStream;

public class PrimeNumbersStream {

    public static boolean isPrime(int n) {
        if (n < 2) {
            return false;
        }

        return IntStream.rangeClosed(2, (int) Math.sqrt(n))
                .noneMatch(i -> n % i == 0);
    }

    public static void main(String[] args) {
        int start = 10;
        int end = 30;

        IntStream.rangeClosed(Math.max(2, start), end)
                .filter(PrimeNumbersStream::isPrime)
                .forEach(n -> System.out.print(n + " "));

        System.out.println();
    }
}
```

### Complexity Analysis

Let \(R = \text{end} - \text{start} + 1\) be the number of values in the range, and let \(E\) be the largest positive number examined.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Normal loops, optimized divisor check | \(O(R\sqrt E)\) | \(O(1)\) |
| Java 8 Stream, optimized divisor check | \(O(R\sqrt E)\) | \(O(1)\) auxiliary space, excluding implementation overhead |

### Key Interview Points

- `1` is not a prime number.
- `2` is the smallest prime number and the only even prime.
- Use `i <= n / i` or check up to `Math.sqrt(n)` to avoid unnecessary divisions.
- For very large ranges, the **Sieve of Eratosthenes** is generally more efficient than checking each number individually.

### Follow-up Questions

1. Check whether a single number is prime.
2. Find the first `n` prime numbers.
3. Count the prime numbers in a range.
4. Find all prime numbers using the Sieve of Eratosthenes.


---



## Find All Prime Numbers in a Given Range

Easy
{: .label .label-green }

**Problem Statement**

Given two integers `start` and `end`, print all prime numbers within the range, including the boundaries.

A **prime number** is a number greater than `1` that has exactly two positive divisors: `1` and itself.

Examples of prime numbers: `2, 3, 5, 7, 11, 13, 17`.

### Example

**Input**
```text
start = 10
end = 30
```

**Output**
```text
11 13 17 19 23 29
```

### Java Solution 1 — Using Normal Loops

```java
public class PrimeNumbers {

    public static boolean isPrime(int n) {
        if (n < 2) {
            return false;
        }

        for (int i = 2; i <= n / i; i++) {
            if (n % i == 0) {
                return false;
            }
        }

        return true;
    }

    public static void printPrimes(int start, int end) {
        for (int number = Math.max(2, start);
             number <= end;
             number++) {

            if (isPrime(number)) {
                System.out.print(number + " ");
            }
        }

        System.out.println();
    }

    public static void main(String[] args) {
        printPrimes(10, 30);
    }
}
```

**Output**
```text
11 13 17 19 23 29
```

### Explanation

1. Start from the given range's lower bound, but never below `2`.
2. Check each number using `isPrime()`.
3. If the number is less than `2`, it is not prime.
4. Check whether any integer from `2` through the square root of the number divides it evenly.
5. If a divisor is found, the number is not prime; otherwise, it is prime.

**Why check only up to the square root?**

If a number \(n\) has a divisor greater than \(\sqrt n\), it must also have a corresponding divisor smaller than \(\sqrt n\). Therefore, checking beyond the square root is unnecessary.

The condition `i <= n / i` avoids the potential integer overflow of `i * i`.

### Java Solution 2 — Using Java 8 Stream API

```java
import java.util.stream.IntStream;

public class PrimeNumbersStream {

    public static boolean isPrime(int n) {
        if (n < 2) {
            return false;
        }

        return IntStream.rangeClosed(2, n / 2)
                .noneMatch(i -> n % i == 0);
    }

    public static void main(String[] args) {
        int start = 10;
        int end = 30;

        IntStream.rangeClosed(Math.max(2, start), end)
                .filter(PrimeNumbersStream::isPrime)
                .forEach(n -> System.out.print(n + " "));

        System.out.println();
    }
}
```

**Output**
```text
11 13 17 19 23 29
```

This Stream API version is easy to read, but its primality check can be improved by checking divisors only up to the square root.

### Optimized Java 8 Stream API Solution

```java
import java.util.stream.IntStream;

public class PrimeNumbersStream {

    public static boolean isPrime(int n) {
        if (n < 2) {
            return false;
        }

        return IntStream.rangeClosed(2, (int) Math.sqrt(n))
                .noneMatch(i -> n % i == 0);
    }

    public static void main(String[] args) {
        int start = 10;
        int end = 30;

        IntStream.rangeClosed(Math.max(2, start), end)
                .filter(PrimeNumbersStream::isPrime)
                .forEach(n -> System.out.print(n + " "));

        System.out.println();
    }
}
```

### Complexity Analysis

Let \(R = \text{end} - \text{start} + 1\) be the number of values in the range, and let \(E\) be the largest positive number examined.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Normal loops, optimized divisor check | \(O(R\sqrt E)\) | \(O(1)\) |
| Java 8 Stream, optimized divisor check | \(O(R\sqrt E)\) | \(O(1)\) auxiliary space, excluding implementation overhead |

### Key Interview Points

- `1` is not a prime number.
- `2` is the smallest prime number and the only even prime.
- Use `i <= n / i` or check up to `Math.sqrt(n)` to avoid unnecessary divisions.
- For very large ranges, the **Sieve of Eratosthenes** is generally more efficient than checking each number individually.

### Follow-up Questions

1. Check whether a single number is prime.
2. Find the first `n` prime numbers.
3. Count the prime numbers in a range.
4. Find all prime numbers using the Sieve of Eratosthenes.


---



## Reverse a Number

Easy
{: .label .label-green }

**Problem Statement**

Given an integer, reverse its digits and return the reversed number.

### Example 1

**Input**
```text
number = 12345
```

**Output**
```text
54321
```

### Example 2

**Input**
```text
number = 1200
```

**Output**
```text
21
```

### Java Solution 1 — Using a While Loop

```java
public class ReverseNumber {

    public static long reverse(int number) {
        long reversed = 0;
        long n = number;

        while (n != 0) {
            long digit = n % 10;
            reversed = reversed * 10 + digit;
            n /= 10;
        }

        return reversed;
    }

    public static void main(String[] args) {
        System.out.println(reverse(12345));
        System.out.println(reverse(1200));
        System.out.println(reverse(-123));
        System.out.println(reverse(0));
    }
}
```

**Output**
```text
54321
21
-321
0
```

### Explanation

For `number = 12345`:

| Step | `n % 10` (digit) | Reversed number | `n / 10` |
|---|---:|---:|---:|
| 1 | 5 | 5 | 1234 |
| 2 | 4 | 54 | 123 |
| 3 | 3 | 543 | 12 |
| 4 | 2 | 5432 | 1 |
| 5 | 1 | 54321 | 0 |

The algorithm uses two operations:

- `n % 10` extracts the last digit.
- `n / 10` removes the last digit.

The reversed number is built by multiplying the current result by `10` and adding the extracted digit.

### Java Solution 2 — Using StringBuilder

```java
public class ReverseNumber {

    public static long reverse(int number) {
        String reversed = new StringBuilder(
            String.valueOf(Math.abs((long) number))
        ).reverse().toString();

        long result = Long.parseLong(reversed);

        return number < 0 ? -result : result;
    }

    public static void main(String[] args) {
        System.out.println(reverse(12345));
        System.out.println(reverse(-123));
        System.out.println(reverse(1200));
    }
}
```

**Output**
```text
54321
-321
21
```

### Complexity Analysis

Let \(d\) be the number of digits in the input.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| While loop | \(O(d)\) | \(O(1)\) |
| StringBuilder | \(O(d)\) | \(O(d)\) |

### Key Interview Points

- `% 10` extracts the last digit.
- `/ 10` removes the last digit.
- Trailing zeros become leading zeros when reversed, and numeric values do not preserve leading zeros.
- The loop approach handles negative integers naturally.
- Using `long` for the intermediate value helps avoid overflow when reversing an `int`.

### Follow-up Questions

1. Check whether a number is a palindrome.
2. Find the sum of digits of a number.
3. Count the digits in a number.
4. Reverse a number without using multiplication.


---


## Reverse a String — Normal and Java 8 Stream API

Easy
{: .label .label-green }

**Problem Statement**

Given a string, reverse its characters and return the reversed string.

### Example

**Input**
```text
s = "Hello World"
```

**Output**
```text
dlroW olleH
```

### Java Solution 1 — Using a For Loop

```java
public class ReverseString {

    public static String reverse(String s) {
        if (s == null) {
            return null;
        }

        StringBuilder reversed = new StringBuilder();

        for (int i = s.length() - 1; i >= 0; i--) {
            reversed.append(s.charAt(i));
        }

        return reversed.toString();
    }

    public static void main(String[] args) {
        System.out.println(reverse("Hello World"));
        System.out.println(reverse("Java"));
    }
}
```

**Output**
```text
dlroW olleH
avaJ
```

### Explanation

For `s = "Java"`:

1. Start from the last index, `3`.
2. Append each character while moving backward.
3. Convert the `StringBuilder` into a string.

```text
Original: J a v a
Index:    0 1 2 3

Read:     a v a J
Reversed: avaJ
```

### Java Solution 2 — Using StringBuilder

Java provides a built-in `reverse()` method in `StringBuilder`.

```java
public class ReverseString {

    public static String reverse(String s) {
        if (s == null) {
            return null;
        }

        return new StringBuilder(s).reverse().toString();
    }

    public static void main(String[] args) {
        System.out.println(reverse("Hello World"));
    }
}
```

**Output**
```text
dlroW olleH
```

### Java Solution 3 — Using Java 8 Streams

```java
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class ReverseStringStream {

    public static String reverse(String s) {
        if (s == null) {
            return null;
        }

        return IntStream.range(0, s.length())
                .map(i -> s.length() - 1 - i)
                .mapToObj(s::charAt)
                .map(String::valueOf)
                .collect(Collectors.joining());
    }

    public static void main(String[] args) {
        System.out.println(reverse("Hello World"));
    }
}
```

**Output**
```text
dlroW olleH
```

### Explanation of the Stream API

- `IntStream.range(0, s.length())` generates indices from `0` to `length - 1`.
- `.map(i -> s.length() - 1 - i)` converts each index into its reverse position.
- `.mapToObj(s::charAt)` retrieves the character at that position.
- `.map(String::valueOf)` converts each character into a string.
- `.collect(Collectors.joining())` joins the characters into one string.

### Complexity Analysis

Let \(n\) be the length of the string.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| For loop | \(O(n)\) | \(O(n)\) |
| StringBuilder | \(O(n)\) | \(O(n)\) |
| Java 8 Streams | \(O(n)\) | \(O(n)\) |

### Key Interview Points

- `String` is immutable in Java.
- `StringBuilder` is mutable and provides a convenient `reverse()` method.
- The loop approach demonstrates basic string manipulation.
- Java 8 Streams offer a functional approach but are more verbose for this task.
- These examples reverse Java UTF-16 `char` units. Strings containing supplementary Unicode characters or combined emoji may need code-point- or grapheme-aware handling.

### Follow-up Questions

1. Reverse a string without using `StringBuilder.reverse()`.
2. Reverse each word in a sentence while keeping word order unchanged.
3. Reverse the order of words in a sentence.
4. Check whether a string is a palindrome.


---


## Calculate the Square Root of a Number

Easy
{: .label .label-green }

**Problem Statement**

Given a non-negative number, calculate its square root using Java.

For example:

- Square root of `25` is `5`.
- Square root of `49` is `7`.
- Square root of `2` is approximately `1.4142`.

### Java Solution 1 — Using `Math.sqrt()`

Java provides the built-in `Math.sqrt()` method to calculate the square root.

```java
public class SquareRoot {

    public static void main(String[] args) {
        double number = 25;

        double result = Math.sqrt(number);

        System.out.println("Square root: " + result);
    }
}
```

**Output**
```text
Square root: 5.0
```

### Java Solution 2 — Using a Normal Loop (Approximation)

This approach uses binary search to approximate the square root without calling `Math.sqrt()`.

```java
public class SquareRoot {

    public static double squareRoot(double number) {
        if (number < 0 || Double.isNaN(number)) {
            throw new IllegalArgumentException(
                "Number must be non-negative"
            );
        }

        if (number == 0 || Double.isInfinite(number)) {
            return number;
        }

        double low = 0;
        double high = Math.max(1, number);
        double epsilon = 1e-10;

        while (high - low > epsilon * Math.max(1, high)) {
            double mid = low + (high - low) / 2;

            if (mid > number / mid) {
                high = mid;
            } else {
                low = mid;
            }
        }

        return low + (high - low) / 2;
    }

    public static void main(String[] args) {
        System.out.println(squareRoot(25));
        System.out.println(squareRoot(2));
        System.out.println(squareRoot(49));
    }
}
```

**Output (approximately)**
```text
5.0000000000
1.4142135624
7.0000000000
```

### Explanation of Binary Search

To calculate the square root of `25`:

1. Set `low = 0` and `high = 25`.
2. Find the midpoint.
3. If the midpoint is greater than the required square root, reduce the upper bound.
4. Otherwise, increase the lower bound.
5. Repeat until the interval is sufficiently small.

For example, the square root of `25` lies between `4` and `6`, and the algorithm keeps narrowing the interval toward `5`.

The comparison `mid > number / mid` avoids directly calculating `mid * mid`, which could overflow for very large values.

### Java Solution 3 — Using Newton's Method

Newton's method repeatedly improves an estimate of the square root.

```java
public class SquareRootNewton {

    public static double squareRoot(double number) {
        if (number < 0 || Double.isNaN(number)) {
            throw new IllegalArgumentException(
                "Number must be non-negative"
            );
        }

        if (number == 0 || Double.isInfinite(number)) {
            return number;
        }

        double guess = number >= 1 ? number : 1;

        for (int i = 0; i < 100; i++) {
            double next = 0.5 * (guess + number / guess);

            if (next == guess) {
                break;
            }

            guess = next;
        }

        return guess;
    }

    public static void main(String[] args) {
        System.out.println(squareRoot(25));
        System.out.println(squareRoot(2));
    }
}
```

**Output (approximately)**
```text
5.0
1.414213562373095
```

### Complexity Analysis

Let \(N\) be the input number and \(\varepsilon\) the desired absolute precision.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| `Math.sqrt()` | Implementation-dependent; typically highly optimized | \(O(1)\) |
| Binary search | \(O(\log(N/\varepsilon))\) for \(N \geq 1\) | \(O(1)\) |
| Newton's method | Quadratic convergence near the answer | \(O(1)\) |

### Key Interview Points

- Use `Math.sqrt(number)` for ordinary Java applications.
- Use binary search when demonstrating a search-based solution.
- Newton's method converges quickly when the initial estimate is reasonable.
- Square roots of negative real numbers are not real numbers, so this implementation rejects them.

### Follow-up Questions

1. Find the integer square root without using `Math.sqrt()`.
2. Check whether a number is a perfect square.
3. Calculate square roots to a specified number of decimal places.
4. Explain the difference between binary search and Newton's method.


---


## Calculate the Sum of Digits in a Given Number

Easy
{: .label .label-green }

**Problem Statement**

Given an integer, calculate the sum of all its digits.

### Example 1

**Input**
```text
number = 12345
```

**Output**
```text
Sum of digits = 15
```

**Explanation**

\[
1 + 2 + 3 + 4 + 5 = 15
\]

### Java Solution 1 — Using a While Loop

```java
public class SumOfDigits {

    public static int sumDigits(int number) {
        long n = Math.abs((long) number);
        int sum = 0;

        while (n > 0) {
            int digit = (int) (n % 10);
            sum += digit;
            n /= 10;
        }

        return sum;
    }

    public static void main(String[] args) {
        System.out.println(sumDigits(12345));
        System.out.println(sumDigits(908));
        System.out.println(sumDigits(-123));
        System.out.println(sumDigits(0));
    }
}
```

**Output**
```text
15
17
6
0
```

### Explanation

For `number = 12345`:

| Step | `n % 10` (digit) | Sum | `n / 10` |
|---|---:|---:|---:|
| 1 | 5 | 5 | 1234 |
| 2 | 4 | 9 | 123 |
| 3 | 3 | 12 | 12 |
| 4 | 2 | 14 | 1 |
| 5 | 1 | 15 | 0 |

The algorithm uses:

- `n % 10` to extract the last digit.
- `n / 10` to remove the last digit.
- `sum += digit` to accumulate the total.

The loop stops when the number becomes zero.

### Java Solution 2 — Using Java 8 Streams

```java
import java.util.stream.IntStream;

public class SumOfDigitsStream {

    public static int sumDigits(int number) {
        String digits = String.valueOf(Math.abs((long) number));

        return digits.chars()
                .map(Character::getNumericValue)
                .sum();
    }

    public static void main(String[] args) {
        System.out.println(sumDigits(12345));
        System.out.println(sumDigits(908));
        System.out.println(sumDigits(-123));
    }
}
```

**Output**
```text
15
17
6
```

### Explanation of the Stream API

- `String.valueOf(Math.abs((long) number))` converts the number into a string and safely handles `Integer.MIN_VALUE`.
- `.chars()` creates an `IntStream` of the string's UTF-16 character values.
- `.map(Character::getNumericValue)` converts each digit character into its numeric value.
- `.sum()` calculates the total.

Since the string contains decimal digits, another valid mapping is `.map(c -> c - '0')`.

### Complexity Analysis

Let \(d\) be the number of digits.

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| While loop | \(O(d)\) | \(O(1)\) |
| Java 8 Streams | \(O(d)\) | \(O(d)\) for the string representation |

### Key Interview Points

- `% 10` extracts the last digit.
- `/ 10` removes the last digit.
- The sum of digits of `0` is `0`.
- The examples treat negative numbers by summing the digits of their absolute value.
- The loop is generally the preferred approach when asked to solve this problem without converting the number to a string.

### Follow-up Questions

1. Count the number of digits in an integer.
2. Reverse a number.
3. Find the product of all digits.
4. Check whether a number is an Armstrong number.


---


## Calculate the Sum of Natural Numbers up to N

Easy
{: .label .label-green }

**Problem Statement**

Given a positive integer `n`, calculate the sum of all natural numbers from `1` to `n`.

### Example 1

**Input**
```text
n = 5
```

**Output**
```text
Sum = 15
```

**Explanation**

\[
1 + 2 + 3 + 4 + 5 = 15
\]

### Java Solution 1 — Using a For Loop

```java
public class SumUptoN {

    public static long sum(int n) {
        long sum = 0;

        for (int i = 1; i <= n; i++) {
            sum += i;
        }

        return sum;
    }

    public static void main(String[] args) {
        System.out.println(sum(5));
        System.out.println(sum(10));
        System.out.println(sum(100));
    }
}
```

**Output**
```text
15
55
5050
```

### Explanation

For `n = 5`:

| Iteration | `i` | `sum` |
|---|---:|---:|
| 1 | 1 | 1 |
| 2 | 2 | 3 |
| 3 | 3 | 6 |
| 4 | 4 | 10 |
| 5 | 5 | 15 |

The loop adds each number from `1` through `n` to the running total.

### Java Solution 2 — Using a Mathematical Formula

The sum of the first `n` natural numbers is:

\[
S = \frac{n(n+1)}{2}
\]

```java
public class SumUptoNFormula {

    public static long sum(int n) {
        return (long) n * (n + 1) / 2;
    }

    public static void main(String[] args) {
        System.out.println(sum(5));
        System.out.println(sum(10));
        System.out.println(sum(100));
    }
}
```

**Output**
```text
15
55
5050
```

Casting `n` to `long` before multiplication helps avoid integer overflow in the intermediate calculation for larger values of `n`.

### Java Solution 3 — Using Java 8 Streams

```java
import java.util.stream.IntStream;

public class SumUptoNStream {

    public static long sum(int n) {
        return IntStream.rangeClosed(1, n)
                .asLongStream()
                .sum();
    }

    public static void main(String[] args) {
        System.out.println(sum(5));
        System.out.println(sum(10));
        System.out.println(sum(100));
    }
}
```

**Output**
```text
15
55
5050
```

`IntStream.rangeClosed(1, n)` generates numbers from `1` to `n`, including `n`. The stream is converted to a `LongStream` before summing.

### Complexity Analysis

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| For loop | \(O(n)\) | \(O(1)\) |
| Mathematical formula | \(O(1)\) | \(O(1)\) |
| Java 8 Streams | \(O(n)\) | \(O(1)\) auxiliary space for this pipeline |

### Key Interview Points

- The mathematical formula is the most efficient approach for a single sum.
- The loop demonstrates basic iteration and accumulation.
- Java 8 Streams provide a functional programming alternative.
- If `n = 0`, the sum is `0`.
- The examples assume `n` is non-negative. For negative inputs, decide whether to reject the input or return `0`, depending on the problem requirements.
- For very large values of `n`, even `long` can overflow; validate input bounds when necessary.

### Follow-up Questions

1. Calculate the sum of even numbers from `1` to `n`.
2. Calculate the sum of odd numbers from `1` to `n`.
3. Calculate the sum of numbers from `a` to `b`.
4. Calculate the sum of squares: \(1^2 + 2^2 + \cdots + n^2\).



---


## Swap Two Numbers

Easy
{: .label .label-green }

**Problem Statement**

Given two numbers, swap their values so that the first variable contains the second number's original value and vice versa.

### Example

**Input**
```text
a = 10
b = 20
```

**Output**
```text
Before swapping: a = 10, b = 20
After swapping:  a = 20, b = 10
```

### Java Solution 1 — Using a Temporary Variable

```java
public class SwapNumbers {

    public static void main(String[] args) {
        int a = 10;
        int b = 20;

        System.out.println("Before swapping: a = " + a + ", b = " + b);

        int temp = a;
        a = b;
        b = temp;

        System.out.println("After swapping: a = " + a + ", b = " + b);
    }
}
```

**Output**
```text
Before swapping: a = 10, b = 20
After swapping: a = 20, b = 10
```

### Explanation

Initially, `a = 10` and `b = 20`.

1. `temp = a` stores `10` in the temporary variable.
2. `a = b` assigns `20` to `a`.
3. `b = temp` assigns the original value of `a`, which is `10`, to `b`.

The values are swapped successfully.

### Java Solution 2 — Without a Temporary Variable (Using Arithmetic)

```java
public class SwapWithoutTemp {

    public static void main(String[] args) {
        int a = 10;
        int b = 20;

        System.out.println("Before swapping: a = " + a + ", b = " + b);

        a = a + b;
        b = a - b;
        a = a - b;

        System.out.println("After swapping: a = " + a + ", b = " + b);
    }
}
```

**Output**
```text
Before swapping: a = 10, b = 20
After swapping: a = 20, b = 10
```

### Explanation of Arithmetic Swapping

Initially, `a = 10` and `b = 20`.

| Step | Statement | `a` | `b` |
|---|---|---:|---:|
| Initial | — | 10 | 20 |
| 1 | `a = a + b` | 30 | 20 |
| 2 | `b = a - b` | 30 | 10 |
| 3 | `a = a - b` | 20 | 10 |

**Important:** This approach can cause integer overflow when `a + b` exceeds the `int` range. It is mainly useful for interview exercises rather than production code.

### Java Solution 3 — Without a Temporary Variable (Using XOR)

```java
public class SwapUsingXOR {

    public static void main(String[] args) {
        int a = 10;
        int b = 20;

        System.out.println("Before swapping: a = " + a + ", b = " + b);

        a = a ^ b;
        b = a ^ b;
        a = a ^ b;

        System.out.println("After swapping: a = " + a + ", b = " + b);
    }
}
```

**Output**
```text
Before swapping: a = 10, b = 20
After swapping: a = 20, b = 10
```

### Explanation of XOR Swapping

The XOR (`^`) operator works on the binary representation of integers.

It uses these properties:

- `x ^ x = 0`
- `x ^ 0 = x`
- XOR is commutative and associative.

The first statement stores the combined XOR information in `a`. The next two statements recover the original values in swapped order.

Unlike arithmetic swapping, XOR swapping does not overflow. However, it is less readable and offers no practical advantage for ordinary Java variables.

### Complexity Analysis

| Approach | Time Complexity | Auxiliary Space |
|---|---|---|
| Temporary variable | \(O(1)\) | \(O(1)\) |
| Arithmetic | \(O(1)\) | \(O(1)\) |
| XOR | \(O(1)\) | \(O(1)\) |

### Key Interview Points

- The temporary-variable method is the clearest and most commonly recommended approach.
- Arithmetic swapping can overflow.
- XOR swapping works for integer types but is harder to understand.
- Java passes primitive values by value; swapping two local primitive variables does not swap the caller's variables.
- Java does not support a built-in assignment syntax that swaps two variables directly.

### Follow-up Questions

1. Swap two numbers without using a third variable.
2. Swap two strings in Java.
3. Swap two array elements.
4. Explain the difference between `^` (XOR) and `&&` (logical AND).



