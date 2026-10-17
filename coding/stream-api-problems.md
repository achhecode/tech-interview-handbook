---
title: Stream API Problems
parent: Coding Problems
nav_order: 1
---

# Stream API Problems
{: .no_toc }

Collections and employee-list problems using streams.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Find duplicates in an array using the Stream API.

Easy
{: .label .label-green }

**Answer**

```java
int[] arr = {1, 2, 3, 2, 4, 3, 5};

List<Integer> duplicates = Arrays.stream(arr).boxed()
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
        .entrySet().stream()
        .filter(e -> e.getValue() > 1)
        .map(Map.Entry::getKey)
        .toList();                       // [2, 3]
```

{: .note }
The shortcut `filter(n -> !seen.add(n))` with a `HashSet` also works, but it is a stateful lambda and is unsafe on parallel streams.

---

## Sort an employee list in ascending and descending order using streams.

Easy
{: .label .label-green }

**Answer**

```java
record Employee(int id, String name, String dept, double salary) {}

List<Employee> asc  = list.stream()
        .sorted(Comparator.comparingDouble(Employee::salary)).toList();

List<Employee> desc = list.stream()
        .sorted(Comparator.comparingDouble(Employee::salary).reversed()).toList();

// by department, then salary descending
list.stream().sorted(Comparator.comparing(Employee::dept)
        .thenComparing(Employee::salary, Comparator.reverseOrder())).toList();
```

---

## Find the highest-paid employee in each department, and the second highest salary overall.

Medium
{: .label .label-yellow }

**Answer**

```java
Map<String, Optional<Employee>> top = list.stream()
        .collect(Collectors.groupingBy(Employee::dept,
                 Collectors.maxBy(Comparator.comparingDouble(Employee::salary))));

Optional<Double> second = list.stream().map(Employee::salary)
        .distinct().sorted(Comparator.reverseOrder()).skip(1).findFirst();
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
