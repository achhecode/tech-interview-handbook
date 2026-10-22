---
title: Stream API Problems
parent: Coding Problems
nav_order: 2
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

## Find the frequency of each character in a string using the Stream API.

Easy
{: .label .label-green }

**Answer**

```java
String str = "programming";

Map<Character, Long> freq = str.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(Function.identity(),
                 LinkedHashMap::new, Collectors.counting()));
// {p=1, r=2, o=1, g=2, a=1, m=2, i=1, n=1}

// only repeated characters
freq.entrySet().stream()
    .filter(e -> e.getValue() > 1)
    .forEach(e -> System.out.println(e.getKey() + " -> " + e.getValue()));
```

{: .note }
`LinkedHashMap::new` preserves the order of first appearance. Without it, `groupingBy` returns a `HashMap` with no guaranteed order. For strings with emoji or other supplementary characters, use `codePoints()` instead of `chars()`.

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

## Find the first non-repeating character in a string.

Easy
{: .label .label-green }

**Answer**

```java
String str = "swiss";

Optional<Character> first = str.chars().mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(Function.identity(),
                 LinkedHashMap::new, Collectors.counting()))
        .entrySet().stream()
        .filter(e -> e.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst();                    // Optional[w]
```

---

## Separate even and odd numbers from a list.

Easy
{: .label .label-green }

**Answer**

```java
List<Integer> nums = List.of(1, 2, 3, 4, 5, 6, 7, 8);

Map<Boolean, List<Integer>> parts = nums.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));

parts.get(true);   // [2, 4, 6, 8]
parts.get(false);  // [1, 3, 5, 7]
```

---

## Find the first three highest-paid employees.

Easy
{: .label .label-green }

**Answer**

```java
List<Employee> top3 = list.stream()
        .sorted(Comparator.comparingDouble(Employee::salary).reversed())
        .limit(3)
        .toList();
```

---

## Count employees and find the average salary in each department.

Easy
{: .label .label-green }

**Answer**

```java
Map<String, Long> countByDept = list.stream()
        .collect(Collectors.groupingBy(Employee::dept, Collectors.counting()));

Map<String, Double> avgByDept = list.stream()
        .collect(Collectors.groupingBy(Employee::dept,
                 Collectors.averagingDouble(Employee::salary)));
```

---

## Flatten a list of lists into a single list.

Easy
{: .label .label-green }

**Answer**

```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4), List.of(5));

List<Integer> flat = nested.stream()
        .flatMap(List::stream)
        .toList();                       // [1, 2, 3, 4, 5]
```

---

## Check whether a string is a palindrome using streams.

Easy
{: .label .label-green }

**Answer**

```java
static boolean isPalindrome(String s) {
    String clean = s.replaceAll("[^A-Za-z0-9]", "").toLowerCase();
    return IntStream.range(0, clean.length() / 2)
            .allMatch(i -> clean.charAt(i) == clean.charAt(clean.length() - 1 - i));
}

isPalindrome("A man, a plan, a canal: Panama");   // true
```

---

## Reverse each word in a sentence.

Easy
{: .label .label-green }

**Answer**

```java
String s = "Java streams are fun";

String result = Arrays.stream(s.split(" "))
        .map(w -> new StringBuilder(w).reverse().toString())
        .collect(Collectors.joining(" "));
// avaJ smaerts era nuf
```

---

## Find the missing number in an array containing 1 to n.

Easy
{: .label .label-green }

**Answer**

```java
int[] arr = {1, 2, 4, 5, 6};
int n = 6;

int missing = IntStream.rangeClosed(1, n).sum() - Arrays.stream(arr).sum();  // 3
```

---

## Find the Nth highest salary.

Medium
{: .label .label-yellow }

**Answer**

```java
int n = 3;

Optional<Double> nth = list.stream()
        .map(Employee::salary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(n - 1)
        .findFirst();

// the employees earning that salary
nth.ifPresent(sal -> list.stream()
        .filter(e -> e.salary() == sal)
        .forEach(System.out::println));
```

---

## Get salary statistics (min, max, average, sum, count) in one pass.

Medium
{: .label .label-yellow }

**Answer**

```java
DoubleSummaryStatistics stats = list.stream()
        .mapToDouble(Employee::salary)
        .summaryStatistics();

stats.getMin();  stats.getMax();  stats.getAverage();
stats.getSum();  stats.getCount();
```

---

## Find the total salary of each department and the department with the highest total.

Medium
{: .label .label-yellow }

**Answer**

```java
Map<String, Double> totalByDept = list.stream()
        .collect(Collectors.groupingBy(Employee::dept,
                 Collectors.summingDouble(Employee::salary)));

Optional<Map.Entry<String, Double>> highest = totalByDept.entrySet().stream()
        .max(Map.Entry.comparingByValue());
```

---

## Convert a list of employees into a map of name to salary, handling duplicate keys.

Medium
{: .label .label-yellow }

**Answer**

```java
Map<String, Double> map = list.stream()
        .collect(Collectors.toMap(Employee::name, Employee::salary,
                 Double::sum,            // merge function for duplicate keys
                 LinkedHashMap::new));
```

{: .note }
Without the merge function, `toMap` throws `IllegalStateException` on duplicate keys.

---

## Check whether two strings are anagrams using streams.

Medium
{: .label .label-yellow }

**Answer**

```java
static boolean isAnagram(String a, String b) {
    Function<String, String> normalize = s -> s.replaceAll("\\s", "").toLowerCase()
            .chars().sorted()
            .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
            .toString();
    return normalize.apply(a).equals(normalize.apply(b));
}

isAnagram("Listen", "Silent");   // true
```

---

# Student Data Problems

This resource solves 31 common Stream API problems on a `Student` dataset. It covers filtering, grouping, counting, sorting, averages, min/max, partitioning, and `Optional` handling.

## Student Class

```java
package com.example.demo;

public class Student {

    private int id;
    private String name;
    private int age;
    private String branch;
    private String city;
    private double marks;
    private String gender;

    public Student(int id, String name, int age, String branch,
                   String city, double marks, String gender) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.branch = branch;
        this.city = city;
        this.marks = marks;
        this.gender = gender;
    }

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    public String getBranch() { return branch; }
    public void setBranch(String branch) { this.branch = branch; }

    public String getCity() { return city; }
    public void setCity(String city) { this.city = city; }

    public double getMarks() { return marks; }
    public void setMarks(double marks) { this.marks = marks; }

    public String getGender() { return gender; }
    public void setGender(String gender) { this.gender = gender; }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", branch='" + branch + '\'' +
                ", city='" + city + '\'' +
                ", marks=" + marks +
                ", gender='" + gender + '\'' +
                '}';
    }
}
```

## Sample Data

| ID | Name | Age | Branch | City | Marks | Gender |
|---|---|---|---|---|---|---|
| 1 | Aarav | 20 | CSE | Delhi | 88.5 | Male |
| 2 | Anjali | 22 | ECE | Mumbai | 76.0 | Female |
| 3 | Bhavna | 19 | CSE | Delhi | 92.0 | Female |
| 4 | Chirag | 23 | MECH | Pune | 64.5 | Male |
| 5 | Danish | 21 | ECE | Delhi | 81.0 | Male |
| 6 | Esha | 20 | CIVIL | Patna | 70.5 | Female |
| 7 | Farhan | 24 | MECH | Mumbai | 58.5 | Male |
| 8 | Gauri | 22 | CSE | Pune | 85.0 | Female |
| 9 | Harsh | 27 | CIVIL | Delhi | 67.0 | Male |
| 10 | Aditi | 19 | ECE | Patna | 90.5 | Female |
| 11 | Imran | 23 | CSE | Mumbai | 79.5 | Male |
| 12 | Kavya | 21 | MECH | Delhi | 73.5 | Female |

```java
import java.util.Arrays;
import java.util.List;

List<Student> students = Arrays.asList(
        new Student(1,  "Aarav",  20, "CSE",   "Delhi",  88.5, "Male"),
        new Student(2,  "Anjali", 22, "ECE",   "Mumbai", 76.0, "Female"),
        new Student(3,  "Bhavna", 19, "CSE",   "Delhi",  92.0, "Female"),
        new Student(4,  "Chirag", 23, "MECH",  "Pune",   64.5, "Male"),
        new Student(5,  "Danish", 21, "ECE",   "Delhi",  81.0, "Male"),
        new Student(6,  "Esha",   20, "CIVIL", "Patna",  70.5, "Female"),
        new Student(7,  "Farhan", 24, "MECH",  "Mumbai", 58.5, "Male"),
        new Student(8,  "Gauri",  22, "CSE",   "Pune",   85.0, "Female"),
        new Student(9,  "Harsh",  27, "CIVIL", "Delhi",  67.0, "Male"),
        new Student(10, "Aditi",  19, "ECE",   "Patna",  90.5, "Female"),
        new Student(11, "Imran",  23, "CSE",   "Mumbai", 79.5, "Male"),
        new Student(12, "Kavya",  21, "MECH",  "Delhi",  73.5, "Female")
);
```

All snippets below use this `students` list. Most examples map results to names to keep the output short. Remove `.map(Student::getName)` to get full `Student` objects. Grouping examples use `TreeMap::new` so the output order is predictable.

## Problems and Solutions

### Q1. Find the list of students whose name starts with the letter A

```java
List<String> result = students.stream()
        .filter(s -> s.getName().startsWith("A"))
        .map(Student::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Aarav, Anjali, Aditi]
```

### Q2. Group the students by branch

```java
Map<String, List<String>> byBranch = students.stream()
        .collect(Collectors.groupingBy(
                Student::getBranch,
                TreeMap::new,
                Collectors.mapping(Student::getName, Collectors.toList())));

System.out.println(byBranch);
```

Output:

```text
{CIVIL=[Esha, Harsh], CSE=[Aarav, Bhavna, Gauri, Imran], ECE=[Anjali, Danish, Aditi], MECH=[Chirag, Farhan, Kavya]}
```

### Q3. Find the total count of students

```java
long total = students.stream().count();

System.out.println(total);
```

Output:

```text
12
```

### Q4. Find the maximum age

```java
int maxAge = students.stream()
        .mapToInt(Student::getAge)
        .max()
        .getAsInt();

System.out.println(maxAge);
```

Output:

```text
27
```

### Q5. Find all branch names

```java
List<String> branches = students.stream()
        .map(Student::getBranch)
        .distinct()
        .collect(Collectors.toList());

System.out.println(branches);
```

Output:

```text
[CSE, ECE, MECH, CIVIL]
```

### Q6. Find the count of students in each branch

```java
Map<String, Long> countByBranch = students.stream()
        .collect(Collectors.groupingBy(
                Student::getBranch,
                TreeMap::new,
                Collectors.counting()));

System.out.println(countByBranch);
```

Output:

```text
{CIVIL=2, CSE=4, ECE=3, MECH=3}
```

### Q7. Find the students whose age is less than 21

```java
List<String> result = students.stream()
        .filter(s -> s.getAge() < 21)
        .map(Student::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Aarav, Bhavna, Esha, Aditi]
```

### Q8. Find the students whose age is between 21 and 23 (inclusive)

```java
List<String> result = students.stream()
        .filter(s -> s.getAge() >= 21 && s.getAge() <= 23)
        .map(Student::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Anjali, Chirag, Danish, Gauri, Imran, Kavya]
```

### Q9. Find the average age of male and female students

```java
Map<String, Double> avgAgeByGender = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGender,
                TreeMap::new,
                Collectors.averagingInt(Student::getAge)));

System.out.println(avgAgeByGender);
```

Output:

```text
{Female=20.5, Male=23.0}
```

### Q10. Find the branch with the maximum number of students

```java
Optional<Map.Entry<String, Long>> maxBranch = students.stream()
        .collect(Collectors.groupingBy(Student::getBranch, Collectors.counting()))
        .entrySet()
        .stream()
        .max(Map.Entry.comparingByValue());

maxBranch.ifPresent(e ->
        System.out.println(e.getKey() + " (" + e.getValue() + " students)"));
```

Output:

```text
CSE (4 students)
```

### Q11. Find the students who live in Delhi and sort them by name

```java
List<String> result = students.stream()
        .filter(s -> s.getCity().equalsIgnoreCase("Delhi"))
        .sorted(Comparator.comparing(Student::getName))
        .map(Student::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Aarav, Bhavna, Danish, Harsh, Kavya]
```

### Q12. Find the average marks in each branch

```java
Map<String, Double> avgMarks = students.stream()
        .collect(Collectors.groupingBy(
                Student::getBranch,
                TreeMap::new,
                Collectors.averagingDouble(Student::getMarks)));

System.out.println(avgMarks);
```

Output:

```text
{CIVIL=68.75, CSE=86.25, ECE=82.5, MECH=65.5}
```

### Q13. Find the highest marks in each branch

```java
Map<String, Double> highestMarks = students.stream()
        .collect(Collectors.groupingBy(
                Student::getBranch,
                TreeMap::new,
                Collectors.collectingAndThen(
                        Collectors.maxBy(Comparator.comparingDouble(Student::getMarks)),
                        opt -> opt.get().getMarks())));

System.out.println(highestMarks);
```

Output:

```text
{CIVIL=70.5, CSE=92.0, ECE=90.5, MECH=73.5}
```

### Q14. Find the list of students sorted by marks

```java
List<String> ascending = students.stream()
        .sorted(Comparator.comparingDouble(Student::getMarks))
        .map(s -> s.getName() + " (" + s.getMarks() + ")")
        .collect(Collectors.toList());

List<String> descending = students.stream()
        .sorted(Comparator.comparingDouble(Student::getMarks).reversed())
        .map(s -> s.getName() + " (" + s.getMarks() + ")")
        .collect(Collectors.toList());

System.out.println(ascending);
System.out.println(descending);
```

Output:

```text
[Farhan (58.5), Chirag (64.5), Harsh (67.0), Esha (70.5), Kavya (73.5), Anjali (76.0), Imran (79.5), Danish (81.0), Gauri (85.0), Aarav (88.5), Aditi (90.5), Bhavna (92.0)]
[Bhavna (92.0), Aditi (90.5), Aarav (88.5), Gauri (85.0), Danish (81.0), Imran (79.5), Anjali (76.0), Kavya (73.5), Esha (70.5), Harsh (67.0), Chirag (64.5), Farhan (58.5)]
```

### Q15. Find the student who has the second lowest marks

```java
Optional<Student> secondLowest = students.stream()
        .sorted(Comparator.comparingDouble(Student::getMarks))
        .skip(1)
        .findFirst();

secondLowest.ifPresent(s ->
        System.out.println(s.getName() + " (" + s.getMarks() + ")"));

// Safe version when several students can share the same marks
double secondLowestMarks = students.stream()
        .map(Student::getMarks)
        .distinct()
        .sorted()
        .skip(1)
        .findFirst()
        .orElse(0.0);

System.out.println(secondLowestMarks);
```

Output:

```text
Chirag (64.5)
64.5
```

### Q16. Find the student who has the second highest marks

```java
Optional<Student> secondHighest = students.stream()
        .sorted(Comparator.comparingDouble(Student::getMarks).reversed())
        .skip(1)
        .findFirst();

secondHighest.ifPresent(s ->
        System.out.println(s.getName() + " (" + s.getMarks() + ")"));
```

Output:

```text
Aditi (90.5)
```

### Q17. Find the students who scored more than 80 marks

```java
List<String> result = students.stream()
        .filter(s -> s.getMarks() > 80)
        .map(Student::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Aarav, Bhavna, Danish, Gauri, Aditi]
```

### Q18. Split the students into pass and fail groups (pass marks = 70)

```java
Map<Boolean, List<String>> passFail = students.stream()
        .collect(Collectors.partitioningBy(
                s -> s.getMarks() >= 70,
                Collectors.mapping(Student::getName, Collectors.toList())));

System.out.println(passFail);
```

Output:

```text
{false=[Chirag, Farhan, Harsh], true=[Aarav, Anjali, Bhavna, Danish, Esha, Gauri, Aditi, Imran, Kavya]}
```

### Q19. Find the top 3 students by marks

```java
List<String> top3 = students.stream()
        .sorted(Comparator.comparingDouble(Student::getMarks).reversed())
        .limit(3)
        .map(Student::getName)
        .collect(Collectors.toList());

System.out.println(top3);
```

Output:

```text
[Bhavna, Aditi, Aarav]
```

### Q20. Join all student names in alphabetical order

```java
String names = students.stream()
        .map(Student::getName)
        .sorted()
        .collect(Collectors.joining(", "));

System.out.println(names);
```

Output:

```text
Aarav, Aditi, Anjali, Bhavna, Chirag, Danish, Esha, Farhan, Gauri, Harsh, Imran, Kavya
```

### Q21. Find the count of students in each city

```java
Map<String, Long> countByCity = students.stream()
        .collect(Collectors.groupingBy(
                Student::getCity,
                TreeMap::new,
                Collectors.counting()));

System.out.println(countByCity);
```

Output:

```text
{Delhi=5, Mumbai=3, Patna=2, Pune=2}
```

### Q22. Find the count of male and female students in each branch (nested grouping)

```java
Map<String, Map<String, Long>> nested = students.stream()
        .collect(Collectors.groupingBy(
                Student::getBranch,
                TreeMap::new,
                Collectors.groupingBy(
                        Student::getGender,
                        TreeMap::new,
                        Collectors.counting())));

System.out.println(nested);
```

Output:

```text
{CIVIL={Female=1, Male=1}, CSE={Female=2, Male=2}, ECE={Female=2, Male=1}, MECH={Female=1, Male=2}}
```

### Q23. Find the topper (highest marks) in each branch

```java
Map<String, String> topperByBranch = students.stream()
        .collect(Collectors.groupingBy(
                Student::getBranch,
                TreeMap::new,
                Collectors.collectingAndThen(
                        Collectors.maxBy(Comparator.comparingDouble(Student::getMarks)),
                        opt -> opt.get().getName())));

System.out.println(topperByBranch);
```

Output:

```text
{CIVIL=Esha, CSE=Bhavna, ECE=Aditi, MECH=Kavya}
```

### Q24. Find the branch with the highest average marks

```java
students.stream()
        .collect(Collectors.groupingBy(
                Student::getBranch,
                Collectors.averagingDouble(Student::getMarks)))
        .entrySet()
        .stream()
        .max(Map.Entry.comparingByValue())
        .ifPresent(e -> System.out.println(e.getKey() + " -> " + e.getValue()));
```

Output:

```text
CSE -> 86.25
```

### Q25. Find summary statistics of marks (count, sum, min, average, max)

```java
DoubleSummaryStatistics stats = students.stream()
        .mapToDouble(Student::getMarks)
        .summaryStatistics();

System.out.println(stats);
```

Output:

```text
DoubleSummaryStatistics{count=12, sum=926.500000, min=58.500000, average=77.208333, max=92.000000}
```

### Q26. Find the total marks in each branch

```java
Map<String, Double> totalMarks = students.stream()
        .collect(Collectors.groupingBy(
                Student::getBranch,
                TreeMap::new,
                Collectors.summingDouble(Student::getMarks)));

System.out.println(totalMarks);
```

Output:

```text
{CIVIL=137.5, CSE=345.0, ECE=247.5, MECH=196.5}
```

### Q27. Group the students by grade (A: 90+, B: 75 to 89.99, C: 60 to 74.99, D: below 60)

```java
static String grade(double marks) {
    if (marks >= 90) return "A";
    if (marks >= 75) return "B";
    if (marks >= 60) return "C";
    return "D";
}

Map<String, List<String>> byGrade = students.stream()
        .collect(Collectors.groupingBy(
                s -> grade(s.getMarks()),
                TreeMap::new,
                Collectors.mapping(Student::getName, Collectors.toList())));

System.out.println(byGrade);
```

Output:

```text
{A=[Bhavna, Aditi], B=[Aarav, Anjali, Danish, Gauri, Imran], C=[Chirag, Esha, Harsh, Kavya], D=[Farhan]}
```

### Q28. Check conditions using `anyMatch`, `allMatch`, and `noneMatch`

```java
boolean anyAbove90 = students.stream().anyMatch(s -> s.getMarks() > 90);
boolean allAdults = students.stream().allMatch(s -> s.getAge() >= 18);
boolean noneBelow50 = students.stream().noneMatch(s -> s.getMarks() < 50);

System.out.println(anyAbove90 + ", " + allAdults + ", " + noneBelow50);
```

Output:

```text
true, true, true
```

### Q29. Find a student by id using `Optional`

```java
String found = students.stream()
        .filter(s -> s.getId() == 5)
        .findFirst()
        .map(Student::getName)
        .orElse("Not found");

String missing = students.stream()
        .filter(s -> s.getId() == 99)
        .findFirst()
        .map(Student::getName)
        .orElse("Not found");

System.out.println(found);
System.out.println(missing);
```

Output:

```text
Danish
Not found
```

### Q30. Convert the list into a map of id to name

```java
Map<Integer, String> idToName = students.stream()
        .collect(Collectors.toMap(
                Student::getId,
                Student::getName,
                (first, second) -> first,
                TreeMap::new));

System.out.println(idToName);
```

Output:

```text
{1=Aarav, 2=Anjali, 3=Bhavna, 4=Chirag, 5=Danish, 6=Esha, 7=Farhan, 8=Gauri, 9=Harsh, 10=Aditi, 11=Imran, 12=Kavya}
```

### Q31. Find the youngest student

```java
students.stream()
        .min(Comparator.comparingInt(Student::getAge))
        .ifPresent(s -> System.out.println(s.getName() + " (" + s.getAge() + ")"));
```

Output:

```text
Bhavna (19)
```

If several students share the minimum age, `min()` returns the first one in encounter order. Here Aditi is also 19, but Bhavna appears earlier in the list.

## Complete Runnable Program

```java
package com.example.demo;

import java.util.Arrays;
import java.util.Comparator;
import java.util.DoubleSummaryStatistics;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class StudentStreamDemo {

    static void print(String label, Object value) {
        System.out.println(label + value);
    }

    static String grade(double marks) {
        if (marks >= 90) return "A";
        if (marks >= 75) return "B";
        if (marks >= 60) return "C";
        return "D";
    }

    public static void main(String[] args) {

        List<Student> students = Arrays.asList(
                new Student(1,  "Aarav",  20, "CSE",   "Delhi",  88.5, "Male"),
                new Student(2,  "Anjali", 22, "ECE",   "Mumbai", 76.0, "Female"),
                new Student(3,  "Bhavna", 19, "CSE",   "Delhi",  92.0, "Female"),
                new Student(4,  "Chirag", 23, "MECH",  "Pune",   64.5, "Male"),
                new Student(5,  "Danish", 21, "ECE",   "Delhi",  81.0, "Male"),
                new Student(6,  "Esha",   20, "CIVIL", "Patna",  70.5, "Female"),
                new Student(7,  "Farhan", 24, "MECH",  "Mumbai", 58.5, "Male"),
                new Student(8,  "Gauri",  22, "CSE",   "Pune",   85.0, "Female"),
                new Student(9,  "Harsh",  27, "CIVIL", "Delhi",  67.0, "Male"),
                new Student(10, "Aditi",  19, "ECE",   "Patna",  90.5, "Female"),
                new Student(11, "Imran",  23, "CSE",   "Mumbai", 79.5, "Male"),
                new Student(12, "Kavya",  21, "MECH",  "Delhi",  73.5, "Female")
        );

        print("Q1  Name starts with A : ", students.stream()
                .filter(s -> s.getName().startsWith("A"))
                .map(Student::getName)
                .collect(Collectors.toList()));

        print("Q2  Group by branch    : ", students.stream()
                .collect(Collectors.groupingBy(Student::getBranch, TreeMap::new,
                        Collectors.mapping(Student::getName, Collectors.toList()))));

        print("Q3  Total count        : ", students.stream().count());

        print("Q4  Max age            : ", students.stream()
                .mapToInt(Student::getAge).max().getAsInt());

        print("Q5  Branch names       : ", students.stream()
                .map(Student::getBranch).distinct().collect(Collectors.toList()));

        print("Q6  Count per branch   : ", students.stream()
                .collect(Collectors.groupingBy(Student::getBranch, TreeMap::new,
                        Collectors.counting())));

        print("Q7  Age < 21           : ", students.stream()
                .filter(s -> s.getAge() < 21)
                .map(Student::getName).collect(Collectors.toList()));

        print("Q8  Age 21 to 23       : ", students.stream()
                .filter(s -> s.getAge() >= 21 && s.getAge() <= 23)
                .map(Student::getName).collect(Collectors.toList()));

        print("Q9  Avg age by gender  : ", students.stream()
                .collect(Collectors.groupingBy(Student::getGender, TreeMap::new,
                        Collectors.averagingInt(Student::getAge))));

        Optional<Map.Entry<String, Long>> maxBranch = students.stream()
                .collect(Collectors.groupingBy(Student::getBranch, Collectors.counting()))
                .entrySet().stream()
                .max(Map.Entry.comparingByValue());
        maxBranch.ifPresent(e ->
                print("Q10 Largest branch     : ", e.getKey() + " (" + e.getValue() + " students)"));

        print("Q11 Delhi, sorted      : ", students.stream()
                .filter(s -> s.getCity().equalsIgnoreCase("Delhi"))
                .sorted(Comparator.comparing(Student::getName))
                .map(Student::getName).collect(Collectors.toList()));

        print("Q12 Avg marks/branch   : ", students.stream()
                .collect(Collectors.groupingBy(Student::getBranch, TreeMap::new,
                        Collectors.averagingDouble(Student::getMarks))));

        print("Q13 Highest marks      : ", students.stream()
                .collect(Collectors.groupingBy(Student::getBranch, TreeMap::new,
                        Collectors.collectingAndThen(
                                Collectors.maxBy(Comparator.comparingDouble(Student::getMarks)),
                                opt -> opt.get().getMarks()))));

        print("Q14 Marks ascending    : ", students.stream()
                .sorted(Comparator.comparingDouble(Student::getMarks))
                .map(s -> s.getName() + " (" + s.getMarks() + ")")
                .collect(Collectors.toList()));

        print("Q14 Marks descending   : ", students.stream()
                .sorted(Comparator.comparingDouble(Student::getMarks).reversed())
                .map(s -> s.getName() + " (" + s.getMarks() + ")")
                .collect(Collectors.toList()));

        students.stream()
                .sorted(Comparator.comparingDouble(Student::getMarks))
                .skip(1).findFirst()
                .ifPresent(s -> print("Q15 Second lowest      : ",
                        s.getName() + " (" + s.getMarks() + ")"));

        students.stream()
                .sorted(Comparator.comparingDouble(Student::getMarks).reversed())
                .skip(1).findFirst()
                .ifPresent(s -> print("Q16 Second highest     : ",
                        s.getName() + " (" + s.getMarks() + ")"));

        print("Q17 Marks > 80         : ", students.stream()
                .filter(s -> s.getMarks() > 80)
                .map(Student::getName).collect(Collectors.toList()));

        print("Q18 Pass / Fail        : ", students.stream()
                .collect(Collectors.partitioningBy(s -> s.getMarks() >= 70,
                        Collectors.mapping(Student::getName, Collectors.toList()))));

        print("Q19 Top 3              : ", students.stream()
                .sorted(Comparator.comparingDouble(Student::getMarks).reversed())
                .limit(3).map(Student::getName).collect(Collectors.toList()));

        print("Q20 Joined names       : ", students.stream()
                .map(Student::getName).sorted().collect(Collectors.joining(", ")));

        print("Q21 Count per city     : ", students.stream()
                .collect(Collectors.groupingBy(Student::getCity, TreeMap::new,
                        Collectors.counting())));

        print("Q22 Gender per branch  : ", students.stream()
                .collect(Collectors.groupingBy(Student::getBranch, TreeMap::new,
                        Collectors.groupingBy(Student::getGender, TreeMap::new,
                                Collectors.counting()))));

        print("Q23 Topper per branch  : ", students.stream()
                .collect(Collectors.groupingBy(Student::getBranch, TreeMap::new,
                        Collectors.collectingAndThen(
                                Collectors.maxBy(Comparator.comparingDouble(Student::getMarks)),
                                opt -> opt.get().getName()))));

        students.stream()
                .collect(Collectors.groupingBy(Student::getBranch,
                        Collectors.averagingDouble(Student::getMarks)))
                .entrySet().stream()
                .max(Map.Entry.comparingByValue())
                .ifPresent(e -> print("Q24 Best avg branch    : ", e.getKey() + " -> " + e.getValue()));

        DoubleSummaryStatistics stats = students.stream()
                .mapToDouble(Student::getMarks).summaryStatistics();
        print("Q25 Marks statistics   : ", stats);

        print("Q26 Total marks/branch : ", students.stream()
                .collect(Collectors.groupingBy(Student::getBranch, TreeMap::new,
                        Collectors.summingDouble(Student::getMarks))));

        print("Q27 Group by grade     : ", students.stream()
                .collect(Collectors.groupingBy(s -> grade(s.getMarks()), TreeMap::new,
                        Collectors.mapping(Student::getName, Collectors.toList()))));

        print("Q28 any/all/none       : ",
                students.stream().anyMatch(s -> s.getMarks() > 90) + ", "
                + students.stream().allMatch(s -> s.getAge() >= 18) + ", "
                + students.stream().noneMatch(s -> s.getMarks() < 50));

        print("Q29 Find id 5          : ", students.stream()
                .filter(s -> s.getId() == 5).findFirst()
                .map(Student::getName).orElse("Not found"));

        print("Q29 Find id 99         : ", students.stream()
                .filter(s -> s.getId() == 99).findFirst()
                .map(Student::getName).orElse("Not found"));

        print("Q30 Id to name         : ", students.stream()
                .collect(Collectors.toMap(Student::getId, Student::getName,
                        (first, second) -> first, TreeMap::new)));

        students.stream()
                .min(Comparator.comparingInt(Student::getAge))
                .ifPresent(s -> print("Q31 Youngest           : ",
                        s.getName() + " (" + s.getAge() + ")"));
    }
}
```

Output:

```text
Q1  Name starts with A : [Aarav, Anjali, Aditi]
Q2  Group by branch    : {CIVIL=[Esha, Harsh], CSE=[Aarav, Bhavna, Gauri, Imran], ECE=[Anjali, Danish, Aditi], MECH=[Chirag, Farhan, Kavya]}
Q3  Total count        : 12
Q4  Max age            : 27
Q5  Branch names       : [CSE, ECE, MECH, CIVIL]
Q6  Count per branch   : {CIVIL=2, CSE=4, ECE=3, MECH=3}
Q7  Age < 21           : [Aarav, Bhavna, Esha, Aditi]
Q8  Age 21 to 23       : [Anjali, Chirag, Danish, Gauri, Imran, Kavya]
Q9  Avg age by gender  : {Female=20.5, Male=23.0}
Q10 Largest branch     : CSE (4 students)
Q11 Delhi, sorted      : [Aarav, Bhavna, Danish, Harsh, Kavya]
Q12 Avg marks/branch   : {CIVIL=68.75, CSE=86.25, ECE=82.5, MECH=65.5}
Q13 Highest marks      : {CIVIL=70.5, CSE=92.0, ECE=90.5, MECH=73.5}
Q14 Marks ascending    : [Farhan (58.5), Chirag (64.5), Harsh (67.0), Esha (70.5), Kavya (73.5), Anjali (76.0), Imran (79.5), Danish (81.0), Gauri (85.0), Aarav (88.5), Aditi (90.5), Bhavna (92.0)]
Q14 Marks descending   : [Bhavna (92.0), Aditi (90.5), Aarav (88.5), Gauri (85.0), Danish (81.0), Imran (79.5), Anjali (76.0), Kavya (73.5), Esha (70.5), Harsh (67.0), Chirag (64.5), Farhan (58.5)]
Q15 Second lowest      : Chirag (64.5)
Q16 Second highest     : Aditi (90.5)
Q17 Marks > 80         : [Aarav, Bhavna, Danish, Gauri, Aditi]
Q18 Pass / Fail        : {false=[Chirag, Farhan, Harsh], true=[Aarav, Anjali, Bhavna, Danish, Esha, Gauri, Aditi, Imran, Kavya]}
Q19 Top 3              : [Bhavna, Aditi, Aarav]
Q20 Joined names       : Aarav, Aditi, Anjali, Bhavna, Chirag, Danish, Esha, Farhan, Gauri, Harsh, Imran, Kavya
Q21 Count per city     : {Delhi=5, Mumbai=3, Patna=2, Pune=2}
Q22 Gender per branch  : {CIVIL={Female=1, Male=1}, CSE={Female=2, Male=2}, ECE={Female=2, Male=1}, MECH={Female=1, Male=2}}
Q23 Topper per branch  : {CIVIL=Esha, CSE=Bhavna, ECE=Aditi, MECH=Kavya}
Q24 Best avg branch    : CSE -> 86.25
Q25 Marks statistics   : DoubleSummaryStatistics{count=12, sum=926.500000, min=58.500000, average=77.208333, max=92.000000}
Q26 Total marks/branch : {CIVIL=137.5, CSE=345.0, ECE=247.5, MECH=196.5}
Q27 Group by grade     : {A=[Bhavna, Aditi], B=[Aarav, Anjali, Danish, Gauri, Imran], C=[Chirag, Esha, Harsh, Kavya], D=[Farhan]}
Q28 any/all/none       : true, true, true
Q29 Find id 5          : Danish
Q29 Find id 99         : Not found
Q30 Id to name         : {1=Aarav, 2=Anjali, 3=Bhavna, 4=Chirag, 5=Danish, 6=Esha, 7=Farhan, 8=Gauri, 9=Harsh, 10=Aditi, 11=Imran, 12=Kavya}
Q31 Youngest           : Bhavna (19)
```

## Key Takeaways

| Task | Stream API tool |
|---|---|
| Filter by condition | `filter()` |
| Transform values | `map()`, `mapToInt()`, `mapToDouble()` |
| Remove duplicates | `distinct()` |
| Sort | `sorted(Comparator...)` |
| Pick the Nth element | `skip(n)` + `findFirst()` |
| Group data | `Collectors.groupingBy()` |
| Split into two groups | `Collectors.partitioningBy()` |
| Count, sum, average | `counting()`, `summingDouble()`, `averagingDouble()` |
| Min / max per group | `Collectors.maxBy()` with `collectingAndThen()` |
| Check conditions | `anyMatch()`, `allMatch()`, `noneMatch()` |
| Safe single result | `findFirst()` with `Optional` |



---


# Online Store Product Problems

This resource solves 31 common Stream API problems on a `Product` dataset from an online store. It covers filtering, grouping, counting, sorting, averages, min/max, partitioning, nested grouping, and `Optional` handling.

## Product Class

```java
package com.example.demo;

public class Product {

    private int id;
    private String name;
    private String category;
    private String brand;
    private double price;
    private int quantity;
    private double rating;

    public Product(int id, String name, String category, String brand,
                   double price, int quantity, double rating) {
        this.id = id;
        this.name = name;
        this.category = category;
        this.brand = brand;
        this.price = price;
        this.quantity = quantity;
        this.rating = rating;
    }

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }

    public String getBrand() { return brand; }
    public void setBrand(String brand) { this.brand = brand; }

    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }

    public int getQuantity() { return quantity; }
    public void setQuantity(int quantity) { this.quantity = quantity; }

    public double getRating() { return rating; }
    public void setRating(double rating) { this.rating = rating; }

    @Override
    public String toString() {
        return "Product{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", category='" + category + '\'' +
                ", brand='" + brand + '\'' +
                ", price=" + price +
                ", quantity=" + quantity +
                ", rating=" + rating +
                '}';
    }
}
```

## Sample Data

| ID | Name | Category | Brand | Price | Quantity | Rating |
|---|---|---|---|---|---|---|
| 1 | Laptop | Electronics | Dell | 55000.0 | 10 | 4.5 |
| 2 | Smartphone | Electronics | Samsung | 30000.0 | 25 | 4.3 |
| 3 | Headphones | Electronics | Sony | 8000.0 | 40 | 4.6 |
| 4 | Backpack | Fashion | Wildcraft | 2500.0 | 60 | 4.1 |
| 5 | Sneakers | Fashion | Nike | 6000.0 | 30 | 4.4 |
| 6 | Jacket | Fashion | Puma | 3500.0 | 0 | 3.9 |
| 7 | Mixer Grinder | Home | Philips | 3500.0 | 15 | 4.2 |
| 8 | Air Fryer | Home | Philips | 7500.0 | 8 | 4.5 |
| 9 | Desk Lamp | Home | Wipro | 1300.0 | 50 | 3.8 |
| 10 | Novel Set | Books | Penguin | 900.0 | 100 | 4.7 |
| 11 | Cookbook | Books | Penguin | 650.0 | 0 | 4.0 |
| 12 | Tablet | Electronics | Apple | 45000.0 | 12 | 4.8 |

A product with `quantity = 0` is out of stock.

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.DoubleSummaryStatistics;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.TreeMap;
import java.util.stream.Collectors;

List<Product> products = Arrays.asList(
        new Product(1,  "Laptop",        "Electronics", "Dell",      55000.0, 10,  4.5),
        new Product(2,  "Smartphone",    "Electronics", "Samsung",   30000.0, 25,  4.3),
        new Product(3,  "Headphones",    "Electronics", "Sony",       8000.0, 40,  4.6),
        new Product(4,  "Backpack",      "Fashion",     "Wildcraft",  2500.0, 60,  4.1),
        new Product(5,  "Sneakers",      "Fashion",     "Nike",       6000.0, 30,  4.4),
        new Product(6,  "Jacket",        "Fashion",     "Puma",       3500.0, 0,   3.9),
        new Product(7,  "Mixer Grinder", "Home",        "Philips",    3500.0, 15,  4.2),
        new Product(8,  "Air Fryer",     "Home",        "Philips",    7500.0, 8,   4.5),
        new Product(9,  "Desk Lamp",     "Home",        "Wipro",      1300.0, 50,  3.8),
        new Product(10, "Novel Set",     "Books",       "Penguin",     900.0, 100, 4.7),
        new Product(11, "Cookbook",      "Books",       "Penguin",     650.0, 0,   4.0),
        new Product(12, "Tablet",        "Electronics", "Apple",     45000.0, 12,  4.8)
);
```

All snippets below use this `products` list and the imports above. Most examples map results to names to keep the output short. Remove `.map(Product::getName)` to get full `Product` objects. Grouping examples use `TreeMap::new` so the output order is predictable.

## Problems and Solutions

### Q1. Find the products whose name starts with the letter S

```java
List<String> result = products.stream()
        .filter(p -> p.getName().startsWith("S"))
        .map(Product::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Smartphone, Sneakers]
```

### Q2. Group the products by category

```java
Map<String, List<String>> byCategory = products.stream()
        .collect(Collectors.groupingBy(
                Product::getCategory,
                TreeMap::new,
                Collectors.mapping(Product::getName, Collectors.toList())));

System.out.println(byCategory);
```

Output:

```text
{Books=[Novel Set, Cookbook], Electronics=[Laptop, Smartphone, Headphones, Tablet], Fashion=[Backpack, Sneakers, Jacket], Home=[Mixer Grinder, Air Fryer, Desk Lamp]}
```

### Q3. Find the total count of products

```java
long total = products.stream().count();

System.out.println(total);
```

Output:

```text
12
```

### Q4. Find the maximum price

```java
double maxPrice = products.stream()
        .mapToDouble(Product::getPrice)
        .max()
        .getAsDouble();

System.out.println(maxPrice);
```

Output:

```text
55000.0
```

### Q5. Find all category names

```java
List<String> categories = products.stream()
        .map(Product::getCategory)
        .distinct()
        .collect(Collectors.toList());

System.out.println(categories);
```

Output:

```text
[Electronics, Fashion, Home, Books]
```

### Q6. Find the count of products in each category

```java
Map<String, Long> countByCategory = products.stream()
        .collect(Collectors.groupingBy(
                Product::getCategory,
                TreeMap::new,
                Collectors.counting()));

System.out.println(countByCategory);
```

Output:

```text
{Books=2, Electronics=4, Fashion=3, Home=3}
```

### Q7. Find the products whose price is less than 5000

```java
List<String> result = products.stream()
        .filter(p -> p.getPrice() < 5000)
        .map(Product::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Backpack, Jacket, Mixer Grinder, Desk Lamp, Novel Set, Cookbook]
```

### Q8. Find the products whose price is between 5000 and 10000 (inclusive)

```java
List<String> result = products.stream()
        .filter(p -> p.getPrice() >= 5000 && p.getPrice() <= 10000)
        .map(Product::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Headphones, Sneakers, Air Fryer]
```

### Q9. Find the average price of in-stock and out-of-stock products

```java
Map<Boolean, Double> avgPriceByStock = products.stream()
        .collect(Collectors.partitioningBy(
                p -> p.getQuantity() > 0,
                Collectors.averagingDouble(Product::getPrice)));

System.out.println(avgPriceByStock);
```

Output:

```text
{false=2075.0, true=15970.0}
```

`false` means out of stock and `true` means in stock.

### Q10. Find the category with the maximum number of products

```java
Optional<Map.Entry<String, Long>> maxCategory = products.stream()
        .collect(Collectors.groupingBy(Product::getCategory, Collectors.counting()))
        .entrySet()
        .stream()
        .max(Map.Entry.comparingByValue());

maxCategory.ifPresent(e ->
        System.out.println(e.getKey() + " (" + e.getValue() + " products)"));
```

Output:

```text
Electronics (4 products)
```

### Q11. Find the products of the brand Philips and sort them by name

```java
List<String> result = products.stream()
        .filter(p -> p.getBrand().equalsIgnoreCase("Philips"))
        .sorted(Comparator.comparing(Product::getName))
        .map(Product::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Air Fryer, Mixer Grinder]
```

### Q12. Find the average price in each category

```java
Map<String, Double> avgPrice = products.stream()
        .collect(Collectors.groupingBy(
                Product::getCategory,
                TreeMap::new,
                Collectors.averagingDouble(Product::getPrice)));

System.out.println(avgPrice);
```

Output:

```text
{Books=775.0, Electronics=34500.0, Fashion=4000.0, Home=4100.0}
```

### Q13. Find the highest price in each category

```java
Map<String, Double> highestPrice = products.stream()
        .collect(Collectors.groupingBy(
                Product::getCategory,
                TreeMap::new,
                Collectors.collectingAndThen(
                        Collectors.maxBy(Comparator.comparingDouble(Product::getPrice)),
                        opt -> opt.get().getPrice())));

System.out.println(highestPrice);
```

Output:

```text
{Books=900.0, Electronics=55000.0, Fashion=6000.0, Home=7500.0}
```

### Q14. Find the list of products sorted by price

```java
List<String> ascending = products.stream()
        .sorted(Comparator.comparingDouble(Product::getPrice))
        .map(p -> p.getName() + " (" + p.getPrice() + ")")
        .collect(Collectors.toList());

List<String> descending = products.stream()
        .sorted(Comparator.comparingDouble(Product::getPrice).reversed())
        .map(p -> p.getName() + " (" + p.getPrice() + ")")
        .collect(Collectors.toList());

System.out.println(ascending);
System.out.println(descending);
```

Output:

```text
[Cookbook (650.0), Novel Set (900.0), Desk Lamp (1300.0), Backpack (2500.0), Jacket (3500.0), Mixer Grinder (3500.0), Sneakers (6000.0), Air Fryer (7500.0), Headphones (8000.0), Smartphone (30000.0), Tablet (45000.0), Laptop (55000.0)]
[Laptop (55000.0), Tablet (45000.0), Smartphone (30000.0), Headphones (8000.0), Air Fryer (7500.0), Sneakers (6000.0), Jacket (3500.0), Mixer Grinder (3500.0), Backpack (2500.0), Desk Lamp (1300.0), Novel Set (900.0), Cookbook (650.0)]
```

Jacket and Mixer Grinder have the same price. `sorted()` is stable, so they keep their original order.

### Q15. Find the product that has the second lowest price

```java
Optional<Product> secondLowest = products.stream()
        .sorted(Comparator.comparingDouble(Product::getPrice))
        .skip(1)
        .findFirst();

secondLowest.ifPresent(p ->
        System.out.println(p.getName() + " (" + p.getPrice() + ")"));

// Safe version when several products can share the same price
double secondLowestPrice = products.stream()
        .map(Product::getPrice)
        .distinct()
        .sorted()
        .skip(1)
        .findFirst()
        .orElse(0.0);

System.out.println(secondLowestPrice);
```

Output:

```text
Novel Set (900.0)
900.0
```

### Q16. Find the product that has the second highest price

```java
Optional<Product> secondHighest = products.stream()
        .sorted(Comparator.comparingDouble(Product::getPrice).reversed())
        .skip(1)
        .findFirst();

secondHighest.ifPresent(p ->
        System.out.println(p.getName() + " (" + p.getPrice() + ")"));
```

Output:

```text
Tablet (45000.0)
```

### Q17. Find the products with a rating above 4.5

```java
List<String> result = products.stream()
        .filter(p -> p.getRating() > 4.5)
        .map(Product::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

Output:

```text
[Headphones, Novel Set, Tablet]
```

### Q18. Split the products into in-stock and out-of-stock groups

```java
Map<Boolean, List<String>> stockStatus = products.stream()
        .collect(Collectors.partitioningBy(
                p -> p.getQuantity() > 0,
                Collectors.mapping(Product::getName, Collectors.toList())));

System.out.println(stockStatus);
```

Output:

```text
{false=[Jacket, Cookbook], true=[Laptop, Smartphone, Headphones, Backpack, Sneakers, Mixer Grinder, Air Fryer, Desk Lamp, Novel Set, Tablet]}
```

### Q19. Find the top 3 products by rating

```java
List<String> top3 = products.stream()
        .sorted(Comparator.comparingDouble(Product::getRating).reversed())
        .limit(3)
        .map(Product::getName)
        .collect(Collectors.toList());

System.out.println(top3);
```

Output:

```text
[Tablet, Novel Set, Headphones]
```

### Q20. Join all product names in alphabetical order

```java
String names = products.stream()
        .map(Product::getName)
        .sorted()
        .collect(Collectors.joining(", "));

System.out.println(names);
```

Output:

```text
Air Fryer, Backpack, Cookbook, Desk Lamp, Headphones, Jacket, Laptop, Mixer Grinder, Novel Set, Smartphone, Sneakers, Tablet
```

### Q21. Find the count of products for each brand

```java
Map<String, Long> countByBrand = products.stream()
        .collect(Collectors.groupingBy(
                Product::getBrand,
                TreeMap::new,
                Collectors.counting()));

System.out.println(countByBrand);
```

Output:

```text
{Apple=1, Dell=1, Nike=1, Penguin=2, Philips=2, Puma=1, Samsung=1, Sony=1, Wildcraft=1, Wipro=1}
```

### Q22. Find the count of in-stock and out-of-stock products in each category (nested grouping)

```java
Map<String, Map<Boolean, Long>> nested = products.stream()
        .collect(Collectors.groupingBy(
                Product::getCategory,
                TreeMap::new,
                Collectors.partitioningBy(
                        p -> p.getQuantity() > 0,
                        Collectors.counting())));

System.out.println(nested);
```

Output:

```text
{Books={false=1, true=1}, Electronics={false=0, true=4}, Fashion={false=1, true=2}, Home={false=0, true=3}}
```

`partitioningBy()` always produces both `false` and `true` keys, even when a group is empty.

### Q23. Find the most expensive product in each category

```java
Map<String, String> priciestByCategory = products.stream()
        .collect(Collectors.groupingBy(
                Product::getCategory,
                TreeMap::new,
                Collectors.collectingAndThen(
                        Collectors.maxBy(Comparator.comparingDouble(Product::getPrice)),
                        opt -> opt.get().getName())));

System.out.println(priciestByCategory);
```

Output:

```text
{Books=Novel Set, Electronics=Laptop, Fashion=Sneakers, Home=Air Fryer}
```

### Q24. Find the category with the highest total stock quantity

```java
products.stream()
        .collect(Collectors.groupingBy(
                Product::getCategory,
                Collectors.summingInt(Product::getQuantity)))
        .entrySet()
        .stream()
        .max(Map.Entry.comparingByValue())
        .ifPresent(e -> System.out.println(e.getKey() + " -> " + e.getValue()));
```

Output:

```text
Books -> 100
```

### Q25. Find summary statistics of prices (count, sum, min, average, max)

```java
DoubleSummaryStatistics stats = products.stream()
        .mapToDouble(Product::getPrice)
        .summaryStatistics();

System.out.println(stats);
```

Output:

```text
DoubleSummaryStatistics{count=12, sum=163850.000000, min=650.000000, average=13654.166667, max=55000.000000}
```

### Q26. Find the total stock value (price x quantity) in each category

```java
Map<String, Double> stockValue = products.stream()
        .collect(Collectors.groupingBy(
                Product::getCategory,
                TreeMap::new,
                Collectors.summingDouble(p -> p.getPrice() * p.getQuantity())));

System.out.println(stockValue);
```

Output:

```text
{Books=90000.0, Electronics=2160000.0, Fashion=330000.0, Home=177500.0}
```

### Q27. Group the products by price band (Budget: below 2000, Standard: 2000 to 9999.99, Premium: 10000 and above)

```java
static String priceBand(double price) {
    if (price >= 10000) return "Premium";
    if (price >= 2000) return "Standard";
    return "Budget";
}

Map<String, List<String>> byBand = products.stream()
        .collect(Collectors.groupingBy(
                p -> priceBand(p.getPrice()),
                TreeMap::new,
                Collectors.mapping(Product::getName, Collectors.toList())));

System.out.println(byBand);
```

Output:

```text
{Budget=[Desk Lamp, Novel Set, Cookbook], Premium=[Laptop, Smartphone, Tablet], Standard=[Headphones, Backpack, Sneakers, Jacket, Mixer Grinder, Air Fryer]}
```

`TreeMap` sorts keys alphabetically, so the order is Budget, Premium, Standard.

### Q28. Check conditions using `anyMatch`, `allMatch`, and `noneMatch`

```java
boolean anyAbove50k = products.stream().anyMatch(p -> p.getPrice() > 50000);
boolean allInStock = products.stream().allMatch(p -> p.getQuantity() > 0);
boolean noneLowRated = products.stream().noneMatch(p -> p.getRating() < 3.5);

System.out.println(anyAbove50k + ", " + allInStock + ", " + noneLowRated);
```

Output:

```text
true, false, true
```

### Q29. Find a product by id using `Optional`

```java
String found = products.stream()
        .filter(p -> p.getId() == 5)
        .findFirst()
        .map(Product::getName)
        .orElse("Not found");

String missing = products.stream()
        .filter(p -> p.getId() == 99)
        .findFirst()
        .map(Product::getName)
        .orElse("Not found");

System.out.println(found);
System.out.println(missing);
```

Output:

```text
Sneakers
Not found
```

### Q30. Convert the list into a map of id to name

```java
Map<Integer, String> idToName = products.stream()
        .collect(Collectors.toMap(
                Product::getId,
                Product::getName,
                (first, second) -> first,
                TreeMap::new));

System.out.println(idToName);
```

Output:

```text
{1=Laptop, 2=Smartphone, 3=Headphones, 4=Backpack, 5=Sneakers, 6=Jacket, 7=Mixer Grinder, 8=Air Fryer, 9=Desk Lamp, 10=Novel Set, 11=Cookbook, 12=Tablet}
```

### Q31. Find the cheapest product

```java
products.stream()
        .min(Comparator.comparingDouble(Product::getPrice))
        .ifPresent(p -> System.out.println(p.getName() + " (" + p.getPrice() + ")"));
```

Output:

```text
Cookbook (650.0)
```

If several products share the minimum price, `min()` returns the first one in encounter order.

## Complete Runnable Program

```java
package com.example.demo;

import java.util.Arrays;
import java.util.Comparator;
import java.util.DoubleSummaryStatistics;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class ProductStreamDemo {

    static void print(String label, Object value) {
        System.out.println(label + value);
    }

    static String priceBand(double price) {
        if (price >= 10000) return "Premium";
        if (price >= 2000) return "Standard";
        return "Budget";
    }

    public static void main(String[] args) {

        List<Product> products = Arrays.asList(
                new Product(1,  "Laptop",        "Electronics", "Dell",      55000.0, 10,  4.5),
                new Product(2,  "Smartphone",    "Electronics", "Samsung",   30000.0, 25,  4.3),
                new Product(3,  "Headphones",    "Electronics", "Sony",       8000.0, 40,  4.6),
                new Product(4,  "Backpack",      "Fashion",     "Wildcraft",  2500.0, 60,  4.1),
                new Product(5,  "Sneakers",      "Fashion",     "Nike",       6000.0, 30,  4.4),
                new Product(6,  "Jacket",        "Fashion",     "Puma",       3500.0, 0,   3.9),
                new Product(7,  "Mixer Grinder", "Home",        "Philips",    3500.0, 15,  4.2),
                new Product(8,  "Air Fryer",     "Home",        "Philips",    7500.0, 8,   4.5),
                new Product(9,  "Desk Lamp",     "Home",        "Wipro",      1300.0, 50,  3.8),
                new Product(10, "Novel Set",     "Books",       "Penguin",     900.0, 100, 4.7),
                new Product(11, "Cookbook",      "Books",       "Penguin",     650.0, 0,   4.0),
                new Product(12, "Tablet",        "Electronics", "Apple",     45000.0, 12,  4.8)
        );

        print("Q1  Name starts with S : ", products.stream()
                .filter(p -> p.getName().startsWith("S"))
                .map(Product::getName)
                .collect(Collectors.toList()));

        print("Q2  Group by category  : ", products.stream()
                .collect(Collectors.groupingBy(Product::getCategory, TreeMap::new,
                        Collectors.mapping(Product::getName, Collectors.toList()))));

        print("Q3  Total count        : ", products.stream().count());

        print("Q4  Max price          : ", products.stream()
                .mapToDouble(Product::getPrice).max().getAsDouble());

        print("Q5  Category names     : ", products.stream()
                .map(Product::getCategory).distinct().collect(Collectors.toList()));

        print("Q6  Count per category : ", products.stream()
                .collect(Collectors.groupingBy(Product::getCategory, TreeMap::new,
                        Collectors.counting())));

        print("Q7  Price < 5000       : ", products.stream()
                .filter(p -> p.getPrice() < 5000)
                .map(Product::getName).collect(Collectors.toList()));

        print("Q8  Price 5000-10000   : ", products.stream()
                .filter(p -> p.getPrice() >= 5000 && p.getPrice() <= 10000)
                .map(Product::getName).collect(Collectors.toList()));

        print("Q9  Avg price by stock : ", products.stream()
                .collect(Collectors.partitioningBy(p -> p.getQuantity() > 0,
                        Collectors.averagingDouble(Product::getPrice))));

        Optional<Map.Entry<String, Long>> maxCategory = products.stream()
                .collect(Collectors.groupingBy(Product::getCategory, Collectors.counting()))
                .entrySet().stream()
                .max(Map.Entry.comparingByValue());
        maxCategory.ifPresent(e ->
                print("Q10 Largest category   : ", e.getKey() + " (" + e.getValue() + " products)"));

        print("Q11 Philips, sorted    : ", products.stream()
                .filter(p -> p.getBrand().equalsIgnoreCase("Philips"))
                .sorted(Comparator.comparing(Product::getName))
                .map(Product::getName).collect(Collectors.toList()));

        print("Q12 Avg price/category : ", products.stream()
                .collect(Collectors.groupingBy(Product::getCategory, TreeMap::new,
                        Collectors.averagingDouble(Product::getPrice))));

        print("Q13 Highest price      : ", products.stream()
                .collect(Collectors.groupingBy(Product::getCategory, TreeMap::new,
                        Collectors.collectingAndThen(
                                Collectors.maxBy(Comparator.comparingDouble(Product::getPrice)),
                                opt -> opt.get().getPrice()))));

        print("Q14 Price ascending    : ", products.stream()
                .sorted(Comparator.comparingDouble(Product::getPrice))
                .map(p -> p.getName() + " (" + p.getPrice() + ")")
                .collect(Collectors.toList()));

        print("Q14 Price descending   : ", products.stream()
                .sorted(Comparator.comparingDouble(Product::getPrice).reversed())
                .map(p -> p.getName() + " (" + p.getPrice() + ")")
                .collect(Collectors.toList()));

        products.stream()
                .sorted(Comparator.comparingDouble(Product::getPrice))
                .skip(1).findFirst()
                .ifPresent(p -> print("Q15 Second lowest      : ",
                        p.getName() + " (" + p.getPrice() + ")"));

        products.stream()
                .sorted(Comparator.comparingDouble(Product::getPrice).reversed())
                .skip(1).findFirst()
                .ifPresent(p -> print("Q16 Second highest     : ",
                        p.getName() + " (" + p.getPrice() + ")"));

        print("Q17 Rating > 4.5       : ", products.stream()
                .filter(p -> p.getRating() > 4.5)
                .map(Product::getName).collect(Collectors.toList()));

        print("Q18 In stock / Out     : ", products.stream()
                .collect(Collectors.partitioningBy(p -> p.getQuantity() > 0,
                        Collectors.mapping(Product::getName, Collectors.toList()))));

        print("Q19 Top 3 by rating    : ", products.stream()
                .sorted(Comparator.comparingDouble(Product::getRating).reversed())
                .limit(3).map(Product::getName).collect(Collectors.toList()));

        print("Q20 Joined names       : ", products.stream()
                .map(Product::getName).sorted().collect(Collectors.joining(", ")));

        print("Q21 Count per brand    : ", products.stream()
                .collect(Collectors.groupingBy(Product::getBrand, TreeMap::new,
                        Collectors.counting())));

        print("Q22 Stock per category : ", products.stream()
                .collect(Collectors.groupingBy(Product::getCategory, TreeMap::new,
                        Collectors.partitioningBy(p -> p.getQuantity() > 0,
                                Collectors.counting()))));

        print("Q23 Priciest/category  : ", products.stream()
                .collect(Collectors.groupingBy(Product::getCategory, TreeMap::new,
                        Collectors.collectingAndThen(
                                Collectors.maxBy(Comparator.comparingDouble(Product::getPrice)),
                                opt -> opt.get().getName()))));

        products.stream()
                .collect(Collectors.groupingBy(Product::getCategory,
                        Collectors.summingInt(Product::getQuantity)))
                .entrySet().stream()
                .max(Map.Entry.comparingByValue())
                .ifPresent(e -> print("Q24 Highest stock cat.  : ", e.getKey() + " -> " + e.getValue()));

        DoubleSummaryStatistics stats = products.stream()
                .mapToDouble(Product::getPrice).summaryStatistics();
        print("Q25 Price statistics   : ", stats);

        print("Q26 Stock value/cat.   : ", products.stream()
                .collect(Collectors.groupingBy(Product::getCategory, TreeMap::new,
                        Collectors.summingDouble(p -> p.getPrice() * p.getQuantity()))));

        print("Q27 Group by band      : ", products.stream()
                .collect(Collectors.groupingBy(p -> priceBand(p.getPrice()), TreeMap::new,
                        Collectors.mapping(Product::getName, Collectors.toList()))));

        print("Q28 any/all/none       : ",
                products.stream().anyMatch(p -> p.getPrice() > 50000) + ", "
                + products.stream().allMatch(p -> p.getQuantity() > 0) + ", "
                + products.stream().noneMatch(p -> p.getRating() < 3.5));

        print("Q29 Find id 5          : ", products.stream()
                .filter(p -> p.getId() == 5).findFirst()
                .map(Product::getName).orElse("Not found"));

        print("Q29 Find id 99         : ", products.stream()
                .filter(p -> p.getId() == 99).findFirst()
                .map(Product::getName).orElse("Not found"));

        print("Q30 Id to name         : ", products.stream()
                .collect(Collectors.toMap(Product::getId, Product::getName,
                        (first, second) -> first, TreeMap::new)));

        products.stream()
                .min(Comparator.comparingDouble(Product::getPrice))
                .ifPresent(p -> print("Q31 Cheapest            : ",
                        p.getName() + " (" + p.getPrice() + ")"));
    }
}
```

Output:

```text
Q1  Name starts with S : [Smartphone, Sneakers]
Q2  Group by category  : {Books=[Novel Set, Cookbook], Electronics=[Laptop, Smartphone, Headphones, Tablet], Fashion=[Backpack, Sneakers, Jacket], Home=[Mixer Grinder, Air Fryer, Desk Lamp]}
Q3  Total count        : 12
Q4  Max price          : 55000.0
Q5  Category names     : [Electronics, Fashion, Home, Books]
Q6  Count per category : {Books=2, Electronics=4, Fashion=3, Home=3}
Q7  Price < 5000       : [Backpack, Jacket, Mixer Grinder, Desk Lamp, Novel Set, Cookbook]
Q8  Price 5000-10000   : [Headphones, Sneakers, Air Fryer]
Q9  Avg price by stock : {false=2075.0, true=15970.0}
Q10 Largest category   : Electronics (4 products)
Q11 Philips, sorted    : [Air Fryer, Mixer Grinder]
Q12 Avg price/category : {Books=775.0, Electronics=34500.0, Fashion=4000.0, Home=4100.0}
Q13 Highest price      : {Books=900.0, Electronics=55000.0, Fashion=6000.0, Home=7500.0}
Q14 Price ascending    : [Cookbook (650.0), Novel Set (900.0), Desk Lamp (1300.0), Backpack (2500.0), Jacket (3500.0), Mixer Grinder (3500.0), Sneakers (6000.0), Air Fryer (7500.0), Headphones (8000.0), Smartphone (30000.0), Tablet (45000.0), Laptop (55000.0)]
Q14 Price descending   : [Laptop (55000.0), Tablet (45000.0), Smartphone (30000.0), Headphones (8000.0), Air Fryer (7500.0), Sneakers (6000.0), Jacket (3500.0), Mixer Grinder (3500.0), Backpack (2500.0), Desk Lamp (1300.0), Novel Set (900.0), Cookbook (650.0)]
Q15 Second lowest      : Novel Set (900.0)
Q16 Second highest     : Tablet (45000.0)
Q17 Rating > 4.5       : [Headphones, Novel Set, Tablet]
Q18 In stock / Out     : {false=[Jacket, Cookbook], true=[Laptop, Smartphone, Headphones, Backpack, Sneakers, Mixer Grinder, Air Fryer, Desk Lamp, Novel Set, Tablet]}
Q19 Top 3 by rating    : [Tablet, Novel Set, Headphones]
Q20 Joined names       : Air Fryer, Backpack, Cookbook, Desk Lamp, Headphones, Jacket, Laptop, Mixer Grinder, Novel Set, Smartphone, Sneakers, Tablet
Q21 Count per brand    : {Apple=1, Dell=1, Nike=1, Penguin=2, Philips=2, Puma=1, Samsung=1, Sony=1, Wildcraft=1, Wipro=1}
Q22 Stock per category : {Books={false=1, true=1}, Electronics={false=0, true=4}, Fashion={false=1, true=2}, Home={false=0, true=3}}
Q23 Priciest/category  : {Books=Novel Set, Electronics=Laptop, Fashion=Sneakers, Home=Air Fryer}
Q24 Highest stock cat.  : Books -> 100
Q25 Price statistics   : DoubleSummaryStatistics{count=12, sum=163850.000000, min=650.000000, average=13654.166667, max=55000.000000}
Q26 Stock value/cat.   : {Books=90000.0, Electronics=2160000.0, Fashion=330000.0, Home=177500.0}
Q27 Group by band      : {Budget=[Desk Lamp, Novel Set, Cookbook], Premium=[Laptop, Smartphone, Tablet], Standard=[Headphones, Backpack, Sneakers, Jacket, Mixer Grinder, Air Fryer]}
Q28 any/all/none       : true, false, true
Q29 Find id 5          : Sneakers
Q29 Find id 99         : Not found
Q30 Id to name         : {1=Laptop, 2=Smartphone, 3=Headphones, 4=Backpack, 5=Sneakers, 6=Jacket, 7=Mixer Grinder, 8=Air Fryer, 9=Desk Lamp, 10=Novel Set, 11=Cookbook, 12=Tablet}
Q31 Cheapest            : Cookbook (650.0)
```

## Key Takeaways

| Task | Stream API tool |
|---|---|
| Filter by condition | `filter()` |
| Transform values | `map()`, `mapToInt()`, `mapToDouble()` |
| Remove duplicates | `distinct()` |
| Sort | `sorted(Comparator...)` |
| Pick the Nth element | `skip(n)` + `findFirst()` |
| Take the first N elements | `limit(n)` |
| Group data | `Collectors.groupingBy()` |
| Split into two groups | `Collectors.partitioningBy()` |
| Count, sum, average | `counting()`, `summingInt()`, `summingDouble()`, `averagingDouble()` |
| Min / max per group | `Collectors.maxBy()` with `collectingAndThen()` |
| Check conditions | `anyMatch()`, `allMatch()`, `noneMatch()` |
| Safe single result | `findFirst()` with `Optional` |
| Summary statistics | `summaryStatistics()` |
| Build a lookup map | `Collectors.toMap()` with a merge function |

## Practice Exercises for Readers

Try these on your own using the same `products` list:

1. Find the products whose brand name starts with `P`.
2. Find the average rating of each brand.
3. Find the product with the highest rating in each category.
4. Find the products that are in stock and cost more than 5000.
5. Find the total number of units in stock across all categories.
6. Find the three most expensive in-stock products.
7. Find the brand that sells the most products.
8. Find the category whose average price is the lowest.