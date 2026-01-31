# Lab 6: Working with LINQ – Aggregate, GroupBy, Join and Sorting Operators

## 1. Aggregate Operators

Aggregate operators combine a sequence of values into a single result.

Common operators:
`Count`
`Sum`
`Min`
`Max`
`Average`

```csharp
var numbers = new[] { 3, 5, 7, 9 };
// Method syntax
int count = numbers.Count();
int sum = numbers.Sum();
int min = numbers.Min();
int max = numbers.Max();
double average = numbers.Average();

// Query syntax
var qCount = (from n in numbers select n).Count();
var qSum   = (from n in numbers select n).Sum();
var qMin   = (from n in numbers select n).Min();
var qMax   = (from n in numbers select n).Max();
var qAvg   = (from n in numbers select n).Average();
```

**Output**

```
Count = 4
Sum = 24
Min = 3
Max = 9
Average = 6
```

---

## 2. GroupBy Operator

GroupBy creates groups of elements that share a common key.

```csharp
var students = new[]
{
    new { Name = "Asha", Grade = "A" },
    new { Name = "Ben",  Grade = "B" },
    new { Name = "Cory", Grade = "A" },
    new { Name = "Dia",  Grade = "C" }
};

// Method syntax
var grouped1 = students.GroupBy(s => s.Grade);

// Query syntax
var grouped2 = from s in students
               group s by s.Grade;
```

**Display**

```csharp
foreach (var g in grouped1)
{
    Console.WriteLine($"Grade {g.Key}:");
    foreach (var s in g)
        Console.WriteLine("  " + s.Name);
}
```

**Output**

```
Grade A:
  Asha
  Cory
Grade B:
  Ben
Grade C:
  Dia
```

---

## 3. Join Operators

```csharp
public class Student
{
    public int Rno { get; set; }
    public string Name { get; set; }
}

public class Course
{
    public int Rno { get; set; }
    public string CourseName { get; set; }
}

var studentsList = new List<Student>
{
    new Student { Rno = 1, Name = "Asha" },
    new Student { Rno = 2, Name = "Ben" },
    new Student { Rno = 3, Name = "Cory" }
};

var coursesList = new List<Course>
{
    new Course { Rno = 1, CourseName = "Math" },
    new Course { Rno = 2, CourseName = "Science" },
    new Course { Rno = 2, CourseName = "Physics" }
};
```

### 3.1 Inner Join

Returns only matching records from both sequences.

Syntax difference:
`outer.Join(inner, outerKey, innerKey, resultSelector)`

```csharp
// Method syntax
var innerJoin = studentsList.Join(coursesList,
                    s => s.Rno, c => c.Rno,
                    (s, c) => new { s.Name, c.CourseName });

// Query syntax
var innerJoin2 = from s in studentsList
                 join c in coursesList on s.Rno equals c.Rno
                 select new { s.Name, c.CourseName };

foreach (var x in innerJoin)
    Console.WriteLine($"{x.Name} - {x.CourseName}");
```

**Output**

```
Asha - Math
Ben - Science
Ben - Physics
```

### 3.2 Group Join

Returns each outer element with a collection of its matching inner elements.

GroupJoin returns each student with a group of matched courses.

Syntax difference:
`outer.GroupJoin(inner, outerKey, innerKey, resultSelector)`

Result selector receives a group (IEnumerable)

```csharp
// Method syntax
var groupJoin = studentsList.GroupJoin(coursesList,
                    s => s.Rno, c => c.Rno,
                    (s, courses) => new { s.Name, Courses = courses });

// Query syntax
var groupJoin2 = from s in studentsList
                 join c in coursesList on s.Rno equals c.Rno into g
                 select new { s.Name, Courses = g };
```

**Display & Output**

```csharp
foreach (var item in groupJoin)
{
    Console.WriteLine(item.Name);
    if (item.Courses.Any())
        foreach (var c in item.Courses)
            Console.WriteLine("  " + c.CourseName);
    else
        Console.WriteLine("  (No courses)");
}
```

```
Asha
  Math
Ben
  Science
  Physics
Cory
  (No courses)
```

### 3.3 Left Outer Join

Returns all students even if they have no course.

Syntax difference:
Use `GroupJoin + SelectMany + DefaultIfEmpty()`

```csharp
// Method syntax
var leftJoin = studentsList
    .GroupJoin(coursesList,
        s => s.Rno, c => c.Rno,
        (s, cg) => new { s, cg })
    .SelectMany(x => x.cg.DefaultIfEmpty(),
        (x, c) => new
        {
            Name = x.s.Name,
            Course = c?.CourseName ?? "No Course Assigned"
        });

// Query syntax
var leftJoin2 = from s in studentsList
                join c in coursesList on s.Rno equals c.Rno into g
                from cg in g.DefaultIfEmpty()
                select new
                {
                    Name = s.Name,
                    Course = cg?.CourseName ?? "No Course Assigned"
                };

foreach (var x in leftJoin)
    Console.WriteLine($"{x.Name} - {x.Course}");
```

**Output**

```
Asha - Math
Ben - Science
Ben - Physics
Cory - No Course Assigned
```

---

## Summary

| **Join Type**       | **Method Syntax**                               | **Query Syntax**                        | **Simple Explanation**                                                                                                              |
| ------------------- | ----------------------------------------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Inner Join**      | `Join()`                                        | `join … on … equals`                    | Returns **only matching records** from both collections. If Rno matches in both lists → include it.                                 |
| **Group Join**      | `GroupJoin()`                                   | `join … into`                           | Groups the matching items together. For each student, create a **collection of all matched courses**.                               |
| **Left Outer Join** | `GroupJoin() + SelectMany() + DefaultIfEmpty()` | `join … into … from … DefaultIfEmpty()` | Returns **all items from the left list**, even if no match exists on the right. Missing matches are filled with **null / default**. |

---

## 4. Sorting Operators

Sorting arranges data in a specific order.
Operators:
`OrderBy`
`OrderByDescending`
`ThenBy`
`ThenByDescending`
`Reverse`

```csharp
var students = new[]
{
    new { Name = "Ben",  Grade = 75 },
    new { Name = "Asha", Grade = 90 },
    new { Name = "Cory", Grade = 90 }
};

// OrderByDescending + ThenBy (Name ascending)
var sorted = students
    .OrderByDescending(s => s.Grade)
    .ThenBy(s => s.Name);

// Query syntax
var sorted2 = from s in students
              orderby s.Grade descending, s.Name
              // Name ascending
              select s;

foreach (var s in sorted)
    Console.WriteLine($"{s.Name} - {s.Grade}");
```

**Output**

```
Asha - 90
Cory - 90
Ben - 75
```

---

# Lab 6 – Practical Example Scenarios (1–7)

## Scenario 1: Top Two Scorers

```csharp
var examResults = new[] { new { Name="Asha",Score=92 }, new { Name="Ben",Score=81 }, new { Name="Cory",Score=88 } };

var topTwo = examResults.OrderByDescending(r => r.Score).Take(2);

foreach (var r in topTwo) Console.WriteLine($"{r.Name} - {r.Score}");
```

**Output**

```
Asha - 92
Cory - 88
```

## Scenario 2: Project Count Per Team

```csharp
var projects = new[] { new { Team="Alpha", Project="Mobile App" }, new { Team="Alpha", Project="Dashboard" }, new { Team="Beta", Project="Website" } };

var counts = projects.GroupBy(p => p.Team)
                     .Select(g => new { Team = g.Key, Count = g.Count() });

foreach (var c in counts) Console.WriteLine($"{c.Team}: {c.Count}");
```

**Output**

```
Alpha: 2
Beta: 1
```

## Scenario 3: Orders with Customer Names (Inner Join)

```csharp
var customers = new[] { new { Id=1, Name="Luna" }, new { Id=2, Name="Milo" } };
var orders = new[] { new { CustomerId=1, Item="Notebook" }, new { CustomerId=2, Item="Pen" } };

var result = customers.Join(orders, c=>c.Id, o=>o.CustomerId, (c,o)=> new { c.Name, o.Item });

foreach (var x in result) Console.WriteLine($"{x.Name} ordered {x.Item}");
```

**Output**

```
Luna ordered Notebook
Milo ordered Pen
```

## Scenario 4: Average Study Time Per Subject

```csharp
var study = new[] { new { Subject="Math", Hours=2.5 }, new { Subject="Math", Hours=3.0 }, new { Subject="Science", Hours=1.5 } };

var avg = study.GroupBy(s=>s.Subject)
               .Select(g => new { Sub = g.Key, Avg = g.Average(x=>x.Hours) });

foreach (var a in avg) Console.WriteLine($"{a.Sub}: {a.Avg:F1} hrs");
```

**Output**

```
Math: 2.8 hrs
Science: 1.5 hrs
```

## Scenario 5: Highest Revenue Product per Category

```csharp
var sales = new[]
{
    new { Category="Books", Product="Novel", Revenue=5200 },
    new { Category="Books", Product="Planner", Revenue=6800 },
    new { Category="Stationery", Product="Notebook", Revenue=4300 },
    new { Category="Stationery", Product="Pen Set", Revenue=4700 },
    new { Category="Gadgets", Product="Tablet", Revenue=9100 }
};

var top = sales.GroupBy(s=>s.Category)
               .Select(g => g.OrderByDescending(x=>x.Revenue).First())
               .OrderBy(x=>x.Category);

foreach (var t in top)
    Console.WriteLine($"{t.Category}: {t.Product} ({t.Revenue})");
```

**Output**

```
Books: Planner (6800)
Gadgets: Tablet (9100)
Stationery: Pen Set (4700)
```

## Scenario 6: Total Credits per Student (Group Join + Sum)

```csharp
var enrollments = new[] { new { Rno=1, Credits=4 }, new { Rno=1, Credits=3 }, new { Rno=2, Credits=3 } };

var credits = studentsList
    .GroupJoin(enrollments, s=>s.Rno, e=>e.Rno, (s,g)=> new { s.Name, Total = g.Sum(e=>e.Credits) });

foreach (var c in credits)
    Console.WriteLine($"{c.Name}: {c.Total} credits");
```

**Output**

```
Asha: 7 credits
Ben: 3 credits
Cory: 0 credits
```

## Scenario 7: Names in Reverse Alphabetical Order

```csharp
string[] names = { "Zoe", "Alice", "Mike", "Julia", "Ben" };

var rev = names.OrderBy(n=>n).Reverse();

foreach (var n in rev) Console.WriteLine(n);
```

**Output**

```
Zoe
Mike
Julia
Ben
Alice
```
