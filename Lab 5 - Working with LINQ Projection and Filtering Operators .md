# LINQ: Projection and Filtering Operators

## 1. Introduction to LINQ

### What is LINQ?

- **LINQ** stands for **Language Integrated Query**.
- It is a powerful feature in C# that allows you to write queries directly inside your C# code to work with data from different sources like **collections, databases, or XML files**.

### Why Was LINQ Introduced?

Before LINQ, developers had to learn different query languages for different data sources:

- SQL for databases
- XPath for XML
- Custom loops and conditions for collections

LINQ was introduced to solve this problem by providing **one unified way** to query all types of data sources.

### Advantages of LINQ

- **Consistent Syntax**: Same query style for arrays, lists, databases, XML, etc.
- **Type Safety**: Errors are caught at compile-time, not runtime
- **Readable Code**: Queries look like natural language
- **IntelliSense Support**: Auto-completion helps you write queries faster
- **Less Code**: No need for complex loops and conditions

### How LINQ Works

LINQ follows a simple flow:

```
Data Source → Filter → Transform → Execute → Results
```

**Visual Flow:**

```
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]  ← Data Source
        ↓
   [Filter: n > 5]
        ↓
[6, 7, 8, 9, 10]  ← Filtered Data
        ↓
 [Transform: n * 2]
        ↓
[12, 14, 16, 18, 20]  ← Final Result
```

### LINQ Architecture (Three Layers)

```
┌─────────────────────────────────────┐
│  Language Extensions Layer          │
│  (Query Syntax & Lambda Expressions)│
└────────────┬────────────────────────┘
             ↓
┌─────────────────────────────────────┐
│  LINQ Providers Layer               │
│  (LINQ to Objects, SQL, XML, etc.)  │
└────────────┬────────────────────────┘
             ↓
┌─────────────────────────────────────┐
│  Data Sources Layer                 │
│  (Arrays, Lists, Databases, XML)    │
└─────────────────────────────────────┘
```

**Example of Simple LINQ Query:**

```csharp
int[] numbers = { 2, 5, 7, 8, 10 };

var result = numbers.Where(n => n > 5).Select(n => n * 2);
```

**What happens here:**

1. **Data Source**: Array of numbers
2. **Filter**: `Where(n => n > 5)` keeps only numbers greater than 5
3. **Transform**: `Select(n => n * 2)` doubles each number
4. **Result**: Returns [14, 16, 20]

---

## 2. LINQ Operators: Projection and Filtering

### Projection Operators:

- Select
- SelectMany

### Filtering Operators:

- Where
- OfType
- Take
- TakeWhile
- Skip
- SkipWhile
- Distinct
- First / FirstOrDefault
- Single / SingleOrDefault
- Any
- All
- Contains

### Operator Quick Reference

### **Sample Data**

```csharp
var students = new List<Student>
{
    new Student { Rno = 1, Name = "Amit", Branch = "CE", Sem = 3, CPI = 8.2, Age = 20, Courses = new List<string>{ "Math", "C#" } },
    new Student { Rno = 2, Name = "Neha", Branch = "IT", Sem = 5, CPI = 9.1, Age = 21, Courses = new List<string>{ "DBMS", "C#" } },
    new Student { Rno = 3, Name = "Raj", Branch = "CE", Sem = 3, CPI = 7.5, Age = 19, Courses = new List<string>{ "Math", "Physics" } },
    new Student { Rno = 4, Name = "Simran", Branch = "IT", Sem = 5, CPI = 9.1, Age = 22, Courses = new List<string>{ "DBMS", "AI" } }
};
```

### **LINQ Methods**

| **LINQ Method**     | **What It Does (Easy Meaning)** | **Example Code**                                     | **Output Meaning**                                |
| ------------------- | ------------------------------- | ---------------------------------------------------- | ------------------------------------------------- |
| **Select**          | Pick/transform specific fields  | `students.Select(s => s.Name)`                       | Returns all student names                         |
| **SelectMany**      | Flatten nested lists            | `students.SelectMany(s => s.Courses)`                | Returns all courses from all students in one list |
| **Where**           | Filter items by condition       | `students.Where(s => s.Branch == "CE")`              | Only CE branch students                           |
| **OfType**          | Keep only specific type         | `mixedList.OfType<Student>()`                        | Only Student objects from mixed collection        |
| **Take**            | Take first N items              | `students.Take(2)`                                   | First 2 students                                  |
| **TakeWhile**       | Take until condition fails      | `students.TakeWhile(s => s.CPI > 8)`                 | Students with CPI > 8 until first failure         |
| **Skip**            | Skip first N items              | `students.Skip(2)`                                   | Skip first 2 students                             |
| **SkipWhile**       | Skip while condition is true    | `students.SkipWhile(s => s.Sem == 3)`                | Skip all Sem–3 students until Sem changes         |
| **Distinct**        | Remove duplicate values         | `students.Select(s => s.Branch).Distinct()`          | Unique branches (CE, IT)                          |
| **First**           | First matching item             | `students.First(s => s.CPI > 9)`                     | First student with CPI > 9                        |
| **FirstOrDefault**  | First match OR null             | `students.FirstOrDefault(s => s.Age > 30)`           | Returns null (no student above 30)                |
| **Single**          | Exactly one item must match     | `students.Single(s => s.Rno == 1)`                   | Returns student with roll no. 1                   |
| **SingleOrDefault** | Safe version of Single          | `students.SingleOrDefault(s => s.Rno == 10)`         | Returns null (no match)                           |
| **Any**             | Check if ANY record matches     | `students.Any(s => s.CPI > 9)`                       | True (Neha & Simran have CPI > 9)                 |
| **All**             | Check if ALL records match      | `students.All(s => s.Sem >= 3)`                      | True (all students are Sem ≥ 3)                   |
| **Contains**        | Check if value exists           | `students.SelectMany(s => s.Courses).Contains("AI")` | True (AI course exists)                           |

---

## 3. Query Syntax vs Method Syntax

LINQ can be written in two different styles:

### Method Syntax (Lambda-Based)

### **What is Lambda Syntax?**

A **lambda expression** is a short function you write directly inside the code.

It has this shape:

```
input => output
```

You can read it as:

"**input goes to output**"

---

### **Lambda Expression = Short Anonymous Function**

Example:

```csharp
s => s.Name
```

means:

- take each **s** (student)
- return **s.Name** (student name)

It’s a short version of:

```csharp
string GetName(Student s)
{
    return s.Name;
}
```

---

### **Basic Syntax Breakdown**

### ✔ Single parameter

```csharp
x => x * 2
```

### ✔ Multiple parameters

(enclosed in parentheses)

```csharp
(x, y) => x + y
```

### ✔ Full block with return

(when logic is bigger)

```csharp
s =>
{
    var result = s.CPI * 10;
    return result;
}
```

---

### **Why Do We Use Lambdas in LINQ?**

Because LINQ methods expect **functions**.

Example:

```csharp
students.Where(s => s.CPI > 8);
```

Here:

- `s` = each student
- `s.CPI > 8` = condition

LINQ internally uses your lambda to filter data.

---

### **Common Lambda Examples with Students**

| Operation                     | Lambda                  |
| ----------------------------- | ----------------------- |
| Select names                  | `s => s.Name`           |
| Filter by branch              | `s => s.Branch == "CE"` |
| Filter by CPI                 | `s => s.CPI > 9`        |
| Check if any student in Sem 5 | `s => s.Sem == 5`       |
| Select many courses           | `s => s.Courses`        |

**Lambda syntax is a short, arrow-based way of writing small functions directly inside LINQ.**

**LINQ method syntax** uses functions inside methods.
And those small functions are written using lambda expressions (=>).

```csharp
var result = students.Where(s => s.Age > 18).Select(s => s.Name);
```

### Query Syntax (SQL-Like)

Looks similar to SQL queries with keywords like `from`, `where`, `select`.

```csharp
var result = from s in students
             where s.Age > 18
             select s.Name;
```

### Key Differences

| Feature     | Method Syntax                | Query Syntax                           |
| ----------- | ---------------------------- | -------------------------------------- |
| Style       | Functional, uses lambdas     | Declarative, SQL-like                  |
| Readability | Compact, chained methods     | More readable for SQL users            |
| Features    | All LINQ operators available | Limited operators available            |
| Compilation | Direct                       | Converted to method syntax by compiler |

### Example Showing Both Syntaxes

**Scenario**: Get names of students who scored more than 70.

```csharp
List<Student> students = new List<Student>
{
    new Student { Name = "Ali", Score = 85 },
    new Student { Name = "Sara", Score = 65 },
    new Student { Name = "John", Score = 90 }
};
```

**Method Syntax:**

```csharp
var topStudents = students.Where(s => s.Score > 70).Select(s => s.Name);
```

**Query Syntax:**

```csharp
var topStudents = from s in students
                  where s.Score > 70
                  select s.Name;
```

Both produce the same result: `["Ali", "John"]`

---

## 4. Detailed Explanation of Each Operator

### **Projection Operator: Select**

#### What it Does

The `Select` operator transforms each element in a collection into a new form. Think of it as projecting or mapping data from one shape to another.

#### When to Use

- When you need specific properties from objects
- When you want to transform data into a different format
- When creating new objects from existing data

#### Example

Imagine you have a list of products in a store, and you want to display only the product names with a 10% discount applied to their prices. Let's solve this:

```csharp
List<Product> products = new List<Product>
{
    new Product { Name = "Laptop", Price = 1000 },
    new Product { Name = "Mouse", Price = 20 },
    new Product { Name = "Keyboard", Price = 50 }
};
```

**Method Syntax:**

```csharp
var discountedProducts = products.Select(p => new { p.Name, DiscountPrice = p.Price * 0.9 });
```

**Query Syntax:**

```csharp
var discountedProducts = from p in products
                         select new { p.Name, DiscountPrice = p.Price * 0.9 };
```

**Result:**

- `{ Name = "Laptop", DiscountPrice = 900 }`
- `{ Name = "Mouse", DiscountPrice = 18 }`
- `{ Name = "Keyboard", DiscountPrice = 45 }`

---

### **Projection Operator: SelectMany**

#### What it Does

`SelectMany` flattens nested collections into a single sequence. It's like unwrapping multiple boxes and putting all items in one pile.

#### When to Use

- When working with collections inside collections
- When you need to flatten hierarchical data
- When dealing with one-to-many relationships

#### Example

Suppose you have students, and each student has enrolled in multiple courses. You want to get a complete list of all course names across all students:

```csharp
List<Student> students = new List<Student>
{
    new Student { Name = "Ali", Courses = new List<string> { "Math", "Science" } },
    new Student { Name = "Sara", Courses = new List<string> { "English", "History" } },
    new Student { Name = "John", Courses = new List<string> { "Math", "Art" } }
};
```

**Method Syntax:**

```csharp
var allCourses = students.SelectMany(s => s.Courses);
```

**Query Syntax:**

```csharp
var allCourses = from s in students
                 from c in s.Courses
                 select c;
```

**Result:** `["Math", "Science", "English", "History", "Math", "Art"]`

---

### **Filtering Operator: Where**

#### What it Does

The `Where` operator filters elements based on a condition. Only elements that satisfy the condition pass through.

#### When to Use

- When you need to filter data based on criteria
- When implementing search functionality
- When applying business rules to data

#### Example

You manage a warehouse and need to find all products that are running low on stock (quantity less than 10) to reorder them:

```csharp
List<Product> inventory = new List<Product>
{
    new Product { Name = "Laptop", Quantity = 5 },
    new Product { Name = "Mouse", Quantity = 25 },
    new Product { Name = "Keyboard", Quantity = 8 }
};
```

**Method Syntax:**

```csharp
var lowStockProducts = inventory.Where(p => p.Quantity < 10);
```

**Query Syntax:**

```csharp
var lowStockProducts = from p in inventory
                       where p.Quantity < 10
                       select p;
```

**Result:** Returns Laptop (quantity 5) and Keyboard (quantity 8)

---

### **Filtering Operator: OfType**

#### What it Does

`OfType` filters elements by their type. It only keeps elements that are of a specific type.

#### When to Use

- When working with mixed-type collections (like ArrayList)
- When you need type-safe filtering
- When extracting specific types from object collections

#### Example

You have a mixed collection of items (some integers, some strings) and you need to extract only the numbers to calculate their sum:

```csharp
ArrayList mixedData = new ArrayList { 1, "hello", 2, "world", 3, 4.5, 5 };
```

**Method Syntax:**

```csharp
var numbers = mixedData.OfType<int>();
```

**Query Syntax:**

```csharp
var numbers = from int n in mixedData select n;
```

**Result:** `[1, 2, 3, 5]` (only integers extracted)

---

### **Filtering Operator: Take**

#### What it Does

`Take` returns the first N elements from a sequence. It's like taking the top items from a list.

#### When to Use

- When implementing pagination
- When showing "top N" results
- When limiting result sets

#### Example

You have a leaderboard and want to display only the top 3 scorers:

```csharp
List<Player> players = new List<Player>
{
    new Player { Name = "Ali", Score = 95 },
    new Player { Name = "Sara", Score = 88 },
    new Player { Name = "John", Score = 92 },
    new Player { Name = "Emma", Score = 79 }
};
```

**Method Syntax:**

```csharp
var topThree = players.OrderByDescending(p => p.Score).Take(3);
```

**Query Syntax:**

```csharp
var topThree = (from p in players
                orderby p.Score descending
                select p).Take(3);
```

**Result:** Ali (95), John (92), Sara (88)

---

### **Filtering Operator: TakeWhile**

#### What it Does

`TakeWhile` takes elements as long as a condition is true, then stops immediately when the condition becomes false.

#### When to Use

- When processing data until a certain condition

#### Example

You're reviewing daily sales, and you want to know all the days at the start of the month where sales exceeded $1000, but stop as soon as a day has sales below $1000:

```csharp
List<DailySales> sales = new List<DailySales>
{
    new DailySales { Day = 1, Amount = 1200 },
    new DailySales { Day = 2, Amount = 1500 },
    new DailySales { Day = 3, Amount = 1100 },
    new DailySales { Day = 4, Amount = 800 },
    new DailySales { Day = 5, Amount = 1300 }
};
```

**Method Syntax:**

```csharp
var consecutiveHighSales = sales.TakeWhile(s => s.Amount > 1000);
```

**Query Syntax:**
Not directly supported in query syntax.

**Result:** Days 1, 2, and 3 only (stops at day 4 even though day 5 meets the condition)

> **Tip:** `TakeWhile` reads the sequence in order—once it hits a value that fails, it stops checking the rest.

---

### **Filtering Operator: Skip**

#### What it Does

`Skip` bypasses the first N elements and returns the rest. It's the opposite of `Take`.

#### When to Use

- When implementing pagination (skip already viewed pages)
- When ignoring header rows
- When processing data in batches

#### Example

You're displaying a list of 20 products per page. To show page 2, you need to skip the first 20 products:

```csharp
List<Product> allProducts = GetProducts(); // Assume 50 products
```

**Method Syntax:**

```csharp
var page2Products = allProducts.Skip(20).Take(20);
```

**Query Syntax:**

```csharp
var page2Products = (from p in allProducts select p).Skip(20).Take(20);
```

**Result:** Products 21-40

---

### **Filtering Operator: SkipWhile**

#### What it Does

`SkipWhile` skips elements as long as a condition is true, then returns all remaining elements once the condition becomes false.

#### When to Use

- When ignoring initial elements until a condition
- When processing data after a marker
- When skipping headers or metadata

#### Example

You have a log file where the first few lines are metadata, and you want to start reading from the first actual log entry (marked by starting with "LOG:"):

```csharp
List<string> logLines = new List<string>
{
    "# System Log",
    "# Date: 2024-01-01",
    "LOG: System started",
    "LOG: User logged in",
    "# Comment"
};
```

**Method Syntax:**

```csharp
var actualLogs = logLines.SkipWhile(line => !line.StartsWith("LOG:"));
```

**Query Syntax:**
Not directly supported in query syntax.

**Result:** All lines starting from "LOG: System started" onward

> **Tip:** `SkipWhile` keeps skipping until the first item that fails the condition, then returns everything after that point.

---

### **Filtering Operator: Distinct**

#### What it Does

`Distinct` removes duplicate elements from a collection, returning only unique values.

#### When to Use

- When removing duplicates
- When getting unique values
- When consolidating repeated data

#### Example

You have a list of customer cities from orders, and you want to know which unique cities you're shipping to:

```csharp
List<Order> orders = new List<Order>
{
    new Order { Customer = "Ali", City = "Karachi" },
    new Order { Customer = "Sara", City = "Lahore" },
    new Order { Customer = "John", City = "Karachi" },
    new Order { Customer = "Emma", City = "Islamabad" },
    new Order { Customer = "Mike", City = "Lahore" }
};
```

**Method Syntax:**

```csharp
var uniqueCities = orders.Select(o => o.City).Distinct();
```

**Query Syntax:**

```csharp
var uniqueCities = (from o in orders select o.City).Distinct();
```

**Result:** `["Karachi", "Lahore", "Islamabad"]`

---

### **Filtering Operator: First / FirstOrDefault**

#### What it Does

- `First`: Returns the first element. Throws exception if collection is empty.
- `FirstOrDefault`: Returns the first element, or default value (null/0) if empty.

#### When to Use

- When you need just the first matching element
- When implementing "find first" logic
- `FirstOrDefault` when data might not exist

#### Example

You need to find the first student who scored above 90 in your class:

```csharp
List<Student> students = new List<Student>
{
    new Student { Name = "Ali", Score = 75 },
    new Student { Name = "Sara", Score = 92 },
    new Student { Name = "John", Score = 88 }
};
```

**Method Syntax:**

```csharp
var topStudent = students.FirstOrDefault(s => s.Score > 90);
```

**Query Syntax:**

```csharp
var topStudent = (from s in students
                  where s.Score > 90
                  select s).FirstOrDefault();
```

**Result:** Sara (92). If no student scored above 90, returns null instead of throwing an error.

> **Tip:** Use `FirstOrDefault` when a match might not exist—`First` will throw error if the sequence is empty or no value matches.

---

### **Filtering Operator: Single / SingleOrDefault**

#### What it Does

- `Single`: Returns the only element. Throws exception if 0 or more than 1 element exists.
- `SingleOrDefault`: Returns the only element, or default if empty. Throws if more than 1.

#### When to Use

- When you expect exactly one match
- When retrieving by unique identifier
- When enforcing data integrity

#### Example

You're looking up a student by their unique student ID:

```csharp
List<Student> students = new List<Student>
{
    new Student { ID = 101, Name = "Ali" },
    new Student { ID = 102, Name = "Sara" },
    new Student { ID = 103, Name = "John" }
};
```

**Method Syntax:**

```csharp
var student = students.SingleOrDefault(s => s.ID == 102);
```

**Query Syntax:**

```csharp
var student = (from s in students
               where s.ID == 102
               select s).SingleOrDefault();
```

**Result:** Sara. If ID 102 doesn't exist, returns null. If multiple students have ID 102, throws exception (data integrity violation).

> **Tip:** Reach for `Single/SingleOrDefault` only when you expect exactly one record (like a primary key lookup). Otherwise prefer `First` variants.

---

### **Filtering Operator: Any**

#### What it Does

`Any` checks if at least one element satisfies a condition. Returns true or false.

#### When to Use

- When checking for existence
- When validating data presence
- When implementing "contains any" logic

#### Example

You want to check if any student failed (score below 50) so you can arrange remedial classes:

```csharp
List<Student> students = new List<Student>
{
    new Student { Name = "Ali", Score = 75 },
    new Student { Name = "Sara", Score = 45 },
    new Student { Name = "John", Score = 88 }
};
```

**Method Syntax:**

```csharp
bool hasFailures = students.Any(s => s.Score < 50);
```

**Query Syntax:**

```csharp
bool hasFailures = (from s in students where s.Score < 50 select s).Any();
```

**Result:** `true` (Sara scored 45)

> **Tip:** `Any` stops as soon as it finds a match—perfect for quick existence checks.

---

### **Filtering Operator: All**

#### What it Does

`All` checks if all elements satisfy a condition. Returns true only if every element passes.

#### When to Use

- When validating all elements
- When checking compliance
- When ensuring data quality

#### Example

Before processing an order, you need to verify that all items are in stock:

```csharp
List<OrderItem> orderItems = new List<OrderItem>
{
    new OrderItem { Product = "Laptop", InStock = true },
    new OrderItem { Product = "Mouse", InStock = true },
    new OrderItem { Product = "Keyboard", InStock = false }
};
```

**Method Syntax:**

```csharp
bool canFulfillOrder = orderItems.All(item => item.InStock);
```

**Query Syntax:**

```csharp
bool canFulfillOrder = (from item in orderItems select item).All(item => item.InStock);
```

**Result:** `false` (Keyboard is not in stock)

---

### **Filtering Operator: Contains**

#### What it Does

`Contains` checks if a collection contains a specific element. Returns true or false.

#### When to Use

- When checking for specific values
- When implementing search
- When validating input

#### Example

You want to check if "Islamabad" is in your list of delivery cities before offering same-day delivery:

```csharp
List<string> deliveryCities = new List<string>
{
    "Karachi", "Lahore", "Islamabad", "Peshawar"
};
```

**Method Syntax:**

```csharp
bool offerSameDayDelivery = deliveryCities.Contains("Islamabad");
```

**Query Syntax:**
Not directly supported in query syntax.

**Result:** `true`

---

## 5. Complex Realistic LINQ Tasks

---

### **Task 1: E-Commerce Product Search**

**Scenario:** You manage an online store with thousands of products. A customer is searching for "gaming laptops under $1500" and wants to see the top 5 results sorted by price. Use LINQ to find products that have "laptop" in their name, cost less than $1500, and return only the product name and price for the first 5 items.

```csharp
List<Product> products = new List<Product>
{
    new Product { Name = "Gaming Laptop Pro", Category = "Electronics", Price = 1200 },
    new Product { Name = "Office Laptop", Category = "Electronics", Price = 800 },
    new Product { Name = "Gaming Laptop Elite", Category = "Electronics", Price = 1800 },
    new Product { Name = "Budget Laptop", Category = "Electronics", Price = 400 },
    new Product { Name = "Gaming Laptop X", Category = "Electronics", Price = 1400 },
    new Product { Name = "Gaming Mouse", Category = "Accessories", Price = 50 },
    new Product { Name = "Ultra Gaming Laptop", Category = "Electronics", Price = 1100 }
};
```

**Method Syntax:**

```csharp
var searchResults = products
    .Where(p => p.Name.Contains("Laptop") && p.Price < 1500)
    .OrderBy(p => p.Price)
    .Take(5)
    .Select(p => new { p.Name, p.Price });
```

**Query Syntax:**

```csharp
var searchResults = (from p in products
                     where p.Name.Contains("Laptop") && p.Price < 1500
                     orderby p.Price
                     select new { p.Name, p.Price })
                     .Take(5);
```

**Logic Explanation:**

1. `Where` filters products with "Laptop" in name AND price under $1500
2. `OrderBy` sorts results by price (cheapest first)
3. `Take(5)` limits results to first 5 items
4. `Select` projects only Name and Price fields

**Result:**

- Budget Laptop ($400)
- Office Laptop ($800)
- Ultra Gaming Laptop ($1100)
- Gaming Laptop Pro ($1200)
- Gaming Laptop X ($1400)

---

### **Task 2: Student Attendance Analysis**

**Scenario:** A teacher wants to identify students who need attention. Find all students who have attended fewer than 75% of classes AND have a score below 60. Show their names and attendance percentage. Also, check if there are any students in the "Mathematics" course who meet this criteria.

```csharp
List<Student> students = new List<Student>
{
    new Student { Name = "Ali", Course = "Mathematics", AttendancePercent = 70, Score = 55 },
    new Student { Name = "Sara", Course = "Physics", AttendancePercent = 85, Score = 75 },
    new Student { Name = "John", Course = "Mathematics", AttendancePercent = 60, Score = 45 },
    new Student { Name = "Emma", Course = "Chemistry", AttendancePercent = 80, Score = 80 },
    new Student { Name = "Mike", Course = "Mathematics", AttendancePercent = 90, Score = 70 },
    new Student { Name = "Zara", Course = "Physics", AttendancePercent = 65, Score = 58 }
};
```

**Method Syntax:**

```csharp
var atRiskStudents = students
    .Where(s => s.AttendancePercent < 75 && s.Score < 60)
    .Select(s => new { s.Name, s.AttendancePercent });

var hasMathStudentsAtRisk = students
    .Any(s => s.Course == "Mathematics" && s.AttendancePercent < 75 && s.Score < 60);
```

**Query Syntax:**

```csharp
var atRiskStudents = from s in students
                     where s.AttendancePercent < 75 && s.Score < 60
                     select new { s.Name, s.AttendancePercent };

var hasMathStudentsAtRisk = (from s in students
                             where s.Course == "Mathematics"
                                   && s.AttendancePercent < 75
                                   && s.Score < 60
                             select s).Any();
```

**Logic Explanation:**

1. First query: `Where` filters students with attendance < 75% AND score < 60
2. `Select` projects Name and AttendancePercent
3. Second query: `Any` checks if Mathematics students exist in filtered results
4. Returns boolean indicating if Math students need attention

**Result:**

- At-risk students: Ali (70%), John (60%), Zara (65%)
- Math students at risk: `true` (Ali and John)

---

## Summary

You've now learned the fundamental projection and filtering operators in LINQ:

**Projection Operators** transform data:

- `Select` - transforms each element
- `SelectMany` - flattens nested collections

**Filtering Operators** select specific data:

- `Where` - filters by condition
- `OfType` - filters by type
- `Take/TakeWhile` - takes elements from start
- `Skip/SkipWhile` - skips elements from start
- `Distinct` - removes duplicates
- `First/FirstOrDefault` - gets first element
- `Single/SingleOrDefault` - gets single element
- `Any` - checks if any element matches
- `All` - checks if all elements match
- `Contains` - checks if collection contains element

Remember:

- Use **Method Syntax** for full LINQ power and chaining
- Use **Query Syntax** when it's more readable (especially for complex queries)
- Combine operators to solve real-world problems
- Start simple, then build complex queries step by step
