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

# Java Stream API Practice: Student Data Problems

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

## 4. Additional Practice Problems

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

## 5. Complete Runnable Program

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

## 6. Key Takeaways

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