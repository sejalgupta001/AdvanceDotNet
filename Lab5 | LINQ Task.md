# LINQ Practice Exercises – Projection & Filtering Operators  
**(Only Select, SelectMany, Where, OfType, Take/TakeWhile, Skip/SkipWhile, Distinct, First/FirstOrDefault, Single/SingleOrDefault, Any, All, Contains)**

### Data Setup (Use this in all questions)

```csharp
class Department
{
    public int DeptId { get; set; }
    public string DeptName { get; set; }
}

class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public decimal Salary { get; set; }
    public int DeptId { get; set; }
    public List<string> Skills { get; set; }
}

var departments = new List<Department>
{
    new Department { DeptId = 1, DeptName = "HR" },
    new Department { DeptId = 2, DeptName = "IT" },
    new Department { DeptId = 3, DeptName = "Finance" },
    new Department { DeptId = 4, DeptName = "Marketing" }
};

var employees = new List<Employee>
{
    new Employee { Id = 101, Name = "Amit",   Age = 28, Salary = 75000, DeptId = 2, Skills = new List<string>{ "C#", "SQL", "Angular" } },
    new Employee { Id = 102, Name = "Neha",   Age = 34, Salary = 95000, DeptId = 2, Skills = new List<string>{ "Java", "C#", "React" } },
    new Employee { Id = 103, Name = "Raj",    Age = 45, Salary = 60000, DeptId = 1, Skills = new List<string>{ "Excel", "Communication" } },
    new Employee { Id = 104, Name = "Priya",  Age = 29, Salary = 82000, DeptId = 3, Skills = new List<string>{ "Accounting", "SQL" } },
    new Employee { Id = 105, Name = "Karan",  Age = 31, Salary = 88000, DeptId = 2, Skills = new List<string>{ "C#", "Azure", "Docker" } },
    new Employee { Id = 106, Name = "Simran", Age = 26, Salary = 72000, DeptId = 4, Skills = new List<string>{ "Design", "Photoshop" } }
};
```

### Practice Questions 

1. Get a list containing only the names of all employees.  
2. Create a list of anonymous objects with each employee’s Name and Annual Salary (Salary × 12).  
3. Retrieve Name and Salary of all employees older than 30 years.  
4. Show complete details of all employees who belong to the IT department.  
5. Produce a single flat list of every skill known by any employee.  
6. Get a list of all unique skills present in the company (no duplicates).  
7. List all skills known by employees earning more than 80,000.  
8. Given this mixed list:  
   `List<object> mixed = new List<object> { employees[0], "hello", employees[1], 999, employees[2] };`  
   Extract only the actual Employee objects.  
9. From the mixed list above, extract only the names of the Employee objects.  
10. Return the first three employees in the current order of the list.  
11. Display the names and salaries of the four highest-paid employees.  
12. Show employees in positions 3 to 5 (pagination – third, fourth, and fifth employees).  
13. Assuming employees are sorted by increasing Age, return employees until you reach the first one aged 32 or older (do not include that person).  
14. Skip all employees earning 80,000 or more, then return the remaining employees.  
15. Get the distinct department names that employees belong to.  
16. Find the first employee whose name starts with the letter 'P'.  
17. Find the first employee in the Finance department; return null if none exists.  
18. Retrieve the employee with Id = 103 (Id is unique).  
19. Try to find an employee with Id = 999; return null if not found.  
20. Check whether there is at least one employee younger than 28 years.  
21. Check whether any employee in the IT department knows the skill "React".  
22. Verify whether every employee in the IT department earns more than 70,000.  
23. Verify whether every employee has at least one skill in their Skills list.  
24. Determine whether the skill "Docker" is known by at least one employee's skill list.  
25. Get the names of the three youngest employees who know "C#", sorted from youngest to oldest.

# LINQ printing guid

| **Result Type**                        | **How to Print**                                                     | **Example**                                                                                                                                          | **Expected Output**                                       |
| -------------------------------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| **Collection / List**                  | Add `.ToList()` → use `foreach` loop                                 | `var names = students.Select(s => s.Name).ToList();`<br>`foreach (var n in names) Console.WriteLine(n);`                                             | Amit<br>Neha<br>Raj<br>Simran                             |
| **Filtered List (`Where`)**            | `.Where(...).ToList()` → `foreach` loop                              | `var ceStudents = students.Where(s => s.Branch=="CE").ToList();`<br>`foreach(var s in ceStudents) Console.WriteLine($"{s.Rno}, {s.Name}, {s.Age}");` | 1, Amit, 20<br>3, Raj, 19                                 |
| **Flattened List (`SelectMany`)**      | `.SelectMany(...).ToList()` → `foreach` loop                         | `var allCourses = students.SelectMany(s=>s.Courses).ToList();`<br>`foreach(var c in allCourses) Console.WriteLine(c);`                               | Math<br>C#<br>DBMS<br>C#<br>Math<br>Physics<br>DBMS<br>AI |
| **Distinct Items**                     | `.Distinct().ToList()` → `foreach` loop                              | `var uniqueBranches = students.Select(s=>s.Branch).Distinct().ToList();`<br>`foreach(var b in uniqueBranches) Console.WriteLine(b);`                 | CE<br>IT                                                  |
| **Single Item (`First`)**              | Direct → `Console.WriteLine(item.Property)`                          | `var topStudent = students.First(s=>s.CPI>9);`<br>`Console.WriteLine(topStudent.Name);`                                                              | Neha                                                      |
| **Single Item (`FirstOrDefault`)**     | Check null → `Console.WriteLine(item==null ? "None": item.Property)` | `var oldStudent = students.FirstOrDefault(s=>s.Age>30);`<br>`Console.WriteLine(oldStudent==null?"None":oldStudent.Name);`                            | None                                                      |
| **Single Item (`Single`)**             | Direct → `Console.WriteLine(item.Property)`                          | `var s = students.Single(s=>s.Rno==1);`<br>`Console.WriteLine(s.Name);`                                                                              | Amit                                                      |
| **Boolean (`Any`, `All`, `Contains`)** | Direct → `Console.WriteLine(bool)`                                   | `bool hasAI = students.SelectMany(s=>s.Courses).Contains("AI");`<br>`Console.WriteLine(hasAI);`                                                      | True                                                      |
