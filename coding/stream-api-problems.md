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



## Student Data Processing Interview Questions

### 1. Student Model

```java
public class Student {
    private String name;
    private int age;
    private String city;
    private int marks;
    private String department;

    public Student(String name, int age, String city, int marks) {
        this.name = name;
        this.age = age;
        this.city = city;
        this.marks = marks;
    }

    public Student(String name, int age, String city,
                   int marks, String department) {
        this(name, age, city, marks);
        this.department = department;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public String getCity() {
        return city;
    }

    public int getMarks() {
        return marks;
    }

    public String getDepartment() {
        return department;
    }

    @Override
    public String toString() {
        return "Student{name='" + name + "', age=" + age
                + ", city='" + city + "', marks=" + marks
                + ", department='" + department + "'}";
    }
}
```

### 2. Sample Student Data

```java
import java.util.*;
import java.util.stream.*;
import java.util.function.Function;
import java.util.stream.Collectors;

public class StudentStreamExamples {

    public static void main(String[] args) {

        List<Student> students = Arrays.asList(
            new Student("Danish", 25, "Hyderabad", 95, "IT"),
            new Student("Rahul", 22, "Delhi", 85, "CSE"),
            new Student("Danish", 19, "Hyderabad", 75, "IT"),
            new Student("Amit", 24, "Mumbai", 95, "ECE"),
            new Student("Sara", 23, "Delhi", 90, "CSE"),
            new Student("Zoya", 21, "Hyderabad", 80, "IT"),
            new Student("John", 26, "Mumbai", 85, "ECE"),
            new Student("Priya", 22, "Delhi", 70, "CSE")
        );

        // Examples below use this list.
    }
}
```

### 3. Filter Students Whose Name Is "Danish"

**Problem:** Retrieve every student whose name equals `"Danish"`.

```java
List<Student> result = students.stream()
    .filter(s -> s.getName().equals("Danish"))
    .collect(Collectors.toList());

result.forEach(System.out::println);
```

**Explanation**

- `stream()` creates a stream from the list.
- `filter()` retains elements that satisfy the condition.
- `collect(Collectors.toList())` collects the matching students into a new list.

**Output**

```text
Student{name='Danish', age=25, city='Hyderabad', marks=95, department='IT'}
Student{name='Danish', age=19, city='Hyderabad', marks=75, department='IT'}
```

Both records are returned because both students have the name Danish.

For a null-safe comparison, use:

```java
.filter(s -> "Danish".equals(s.getName()))
```

### 4. Filter Using Multiple Dynamic Conditions

**Problem:** Find students whose name is Danish, age is greater than 20, and city is Hyderabad.

```java
List<Student> result = students.stream()
    .filter(s -> "Danish".equals(s.getName()))
    .filter(s -> s.getAge() > 20)
    .filter(s -> "Hyderabad".equals(s.getCity()))
    .collect(Collectors.toList());

result.forEach(System.out::println);
```

You can also combine all conditions into one predicate:

```java
List<Student> result = students.stream()
    .filter(s -> "Danish".equals(s.getName())
        && s.getAge() > 20
        && "Hyderabad".equals(s.getCity()))
    .collect(Collectors.toList());
```

**Output**

```text
Student{name='Danish', age=25, city='Hyderabad', marks=95, department='IT'}
```

#### Make the conditions dynamic

Instead of hardcoding values, accept them as variables:

```java
String name = "Danish";
int minAge = 20;
String city = "Hyderabad";

List<Student> result = students.stream()
    .filter(s -> name.equals(s.getName()))
    .filter(s -> s.getAge() > minAge)
    .filter(s -> city.equals(s.getCity()))
    .collect(Collectors.toList());
```

This is useful when filter values come from API request parameters or user input.

### 5. Find the Student With the Second-Highest Marks

**Problem:** Return the student with the second-highest marks.

```java
Optional<Student> secondHighestStudent = students.stream()
    .sorted(Comparator.comparingInt(Student::getMarks).reversed())
    .skip(1)
    .findFirst();

secondHighestStudent.ifPresent(System.out::println);
```

**Important:** This version finds the second student after sorting, not necessarily the second-highest *distinct* marks. If several students have the highest score, one of those students might be returned.

For second-highest distinct marks, use the next solution.

### 6. Find the Second-Highest Distinct Marks

**Problem:** Find the second-highest marks value, counting duplicate scores only once.

```java
Optional<Integer> secondHighestMarks = students.stream()
    .map(Student::getMarks)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst();

System.out.println(secondHighestMarks.orElse(-1));
```

**Output**

```text
90
```

**Explanation**

The distinct marks are:

```text
95, 90, 85, 80, 75, 70
```

After sorting in descending order and skipping the highest value (`95`), the next value is `90`.

#### Find the student who has the second-highest distinct marks

```java
Optional<Student> secondHighestStudent = students.stream()
    .map(Student::getMarks)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .flatMap(secondMarks ->
        students.stream()
            .filter(s -> s.getMarks() == secondMarks)
            .findFirst()
    );

secondHighestStudent.ifPresent(System.out::println);
```

This returns the first student in the original list who has the second-highest distinct score.

**Complexity:** Sorting takes \(O(n \log n)\) time and generally \(O(n)\) additional space for this pipeline.

### 7. Group Students by City

**Problem:** Group students into a map where each city maps to a list of students living there.

```java
Map<String, List<Student>> studentsByCity = students.stream()
    .collect(Collectors.groupingBy(Student::getCity));

studentsByCity.forEach((city, list) -> {
    System.out.println(city + " -> " + list);
});
```

**Explanation**

`Collectors.groupingBy(Student::getCity)` uses the city as the key and groups all matching students into a list.

**Conceptual output**

```text
Hyderabad -> [Danish, Danish, Zoya]
Delhi -> [Rahul, Sara, Priya]
Mumbai -> [Amit, John]
```

The order in which city keys are printed is not guaranteed when using the default `groupingBy()` collector.

### 8. Count Students in Each City

**Problem:** Count how many students belong to each city.

```java
Map<String, Long> studentCountByCity = students.stream()
    .collect(Collectors.groupingBy(
        Student::getCity,
        Collectors.counting()
    ));

studentCountByCity.forEach((city, count) ->
    System.out.println(city + " -> " + count)
);
```

**Output**

```text
Hyderabad -> 3
Delhi -> 3
Mumbai -> 2
```

**Explanation**

- `groupingBy(Student::getCity)` groups records by city.
- `counting()` counts the records in each group.
- The result is a `Map<String, Long>` because `Collectors.counting()` returns a `Long`.

### 9. Group Students by Department

**Problem:** Group all students by department.

```java
Map<String, List<Student>> studentsByDepartment = students.stream()
    .collect(Collectors.groupingBy(Student::getDepartment));

studentsByDepartment.forEach((department, list) -> {
    System.out.println(department + " -> " + list);
});
```

**Conceptual output**

```text
IT -> [Danish, Danish, Zoya]
CSE -> [Rahul, Sara, Priya]
ECE -> [Amit, John]
```

The result is a map in which the department is the key and the corresponding students form the value list.

### 10. Count Students in Each Department

A common follow-up interview question is to count the number of students in each department.

```java
Map<String, Long> countByDepartment = students.stream()
    .collect(Collectors.groupingBy(
        Student::getDepartment,
        Collectors.counting()
    ));

System.out.println(countByDepartment);
```

**Output**

```text
{IT=3, CSE=3, ECE=2}
```

### 11. Bonus: Average Marks by City

```java
Map<String, Double> averageMarksByCity = students.stream()
    .collect(Collectors.groupingBy(
        Student::getCity,
        Collectors.averagingInt(Student::getMarks)
    ));

System.out.println(averageMarksByCity);
```

**Output**

```text
{Hyderabad=83.33333333333333, Delhi=81.66666666666667, Mumbai=90.0}
```

### Quick Interview Revision

| Requirement | Stream operation |
|---|---|
| Filter by name | `filter()` |
| Filter by multiple conditions | `filter()` with `&&` |
| Sort by marks | `sorted()` |
| Get second element | `skip(1)` |
| Remove duplicate scores | `distinct()` |
| Get first matching student | `findFirst()` |
| Group students by city | `groupingBy()` |
| Count students per city | `groupingBy()` + `counting()` |
| Group by department | `groupingBy()` |
| Calculate average marks | `averagingInt()` |

**Interview tip:** When a question says *second-highest distinct marks*, remember the sequence: `map()` → `distinct()` → `sorted(Comparator.reverseOrder())` → `skip(1)` → `findFirst()`. This avoids the common mistake of counting tied scores more than once.


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
