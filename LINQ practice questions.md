# LINQ practice questions

---

# **Sample Data (use this for all questions)**

```csharp
public class Student
{
    public int Rno { get; set; }
    public string Name { get; set; }
    public string Branch { get; set; }
    public int Sem { get; set; }
    public double CPI { get; set; }
    public int Age { get; set; }
}

public class Course
{
    public int Rno { get; set; }
    public string CourseName { get; set; }
    public int Credits { get; set; }
}

var students = new List<Student>
{
    new Student { Rno = 1, Name = "Amit", Branch = "CE", Sem = 3, CPI = 8.2, Age = 19 },
    new Student { Rno = 2, Name = "Priya", Branch = "IT", Sem = 5, CPI = 9.1, Age = 21 },
    new Student { Rno = 3, Name = "Rahul", Branch = "CE", Sem = 1, CPI = 7.5, Age = 18 },
    new Student { Rno = 4, Name = "Sneha", Branch = "ME", Sem = 7, CPI = 8.8, Age = 22 },
    new Student { Rno = 5, Name = "Karan", Branch = "IT", Sem = 3, CPI = 6.9, Age = 20 }
};

var courses = new List<Course>
{
    new Course { Rno = 1, CourseName = "DBMS", Credits = 4 },
    new Course { Rno = 1, CourseName = "C#", Credits = 3 },
    new Course { Rno = 2, CourseName = "Java", Credits = 4 },
    new Course { Rno = 3, CourseName = "Python", Credits = 3 },
    new Course { Rno = 5, CourseName = "AI", Credits = 5 }
};
```

---

# **SECTION 1 — FILTERING (Where) — 15 Questions**

1. Get all CE branch students.
2. Students having CPI > 8.
3. Students older than 20.
4. Students in Semester 3.
5. CPI between 7 and 9.
6. Name starting with 'A'.
7. Branch = IT AND Sem = 3.
8. Age < 20 OR CPI > 8.
9. Names containing 'a'.
10. Students NOT in CE.
11. Sem in {1,3,5}.
12. Students whose CPI is a whole number.
13. Students with even Roll No.
14. Students whose age is between 18 and 21.
15. Students having name length > 4.

---

# **SECTION 2 — SELECT (Projection) — 10 Questions**

16. Select only names.
17. Select Name + CPI.
18. Select Roll No + Branch.
19. Select anonymous type: Name, Sem, Age.
20. Create 'FullInfo' string (e.g., "Name (Branch)").
21. Project all to CPI only.
22. Select Name in lowercase.
23. Select Name + Status based on CPI (Good/Average).
24. Extract only distinct branches.
25. Convert student to “DTO” format (Rno, Name).

---

# **SECTION 3 — SORTING (OrderBy) — 10 Questions**

26. Sort names alphabetically.
27. Sort by CPI descending.
28. Sort by Sem, then Name.
29. Sort by Age, then CPI desc.
30. Sort by Branch.
31. Sort by Name length.
32. Sort by Sem DESC.
33. Sort by CPI then Age.
34. Sort by Rno descending.
35. Sort by Branch then Sem.

---

# **SECTION 4 — AGGREGATION — 10 Questions**

36. Count total students.
37. Count CE students.
38. Max CPI.
39. Min CPI.
40. Average CPI.
41. Total credits for Rno = 1.
42. Oldest student's age.
43. Youngest student's age.
44. Highest Sem.
45. Sum of all credits.

---

# **SECTION 5 — ELEMENT OPERATIONS — 10 Questions**

46. Get first student.
47. First student with CPI > 9.
48. Last student.
49. Get student at index 2.
50. Single student with Rno = 3.
51. Safe single (e.g., Rno = 10).
52. First IT student.
53. Last CE student.
54. First student older than 18.
55. Element at index 0.

---

# **SECTION 6 — ANY / ALL — 10 Questions**

56. Any CE students?
57. All students older than 17?
58. Any CPI > 9?
59. All semesters are > 0?
60. Any student with name length > 6?
61. All belong to CE?
62. Any course with credits > 4?
63. All credits > 2?
64. Any course named "Java"?
65. Any student younger than 18?

---

# **SECTION 7 — GROUPING — 10 Questions**

66. Group students by branch.
67. Group by Semester.
68. Group by Age.
69. Group by CPI category (High/Low).
70. Group courses by Rno.
71. Group students by first letter of name.
72. Group by Branch then Sem.
73. Group by age range (Teen/Adult).
74. Group courses by Credits.
75. Group students by CPI rounded.

---

# **SECTION 8 — JOIN — 10 Questions**

76. Inner Join students + courses.
77. Join to get total credits per student.
78. Students with courses (Name, CourseName, Credits).
79. Left join to include students without courses.
80. List of all distinct courses taken.
81. Students having more than 1 course.
82. Join and order by credits.
83. Students of IT with courses.
84. Students who have no course.
85. Students + number of courses.

---

# **SECTION 9 — SET OPERATIONS — 5 Questions**

86. Distinct branches.
87. Students in CE or IT (Union).
88. Students in CE but not IT (Except).
89. Common semesters between CE and IT students (Intersect).
90. Get all courses that have credits other than 3.

---

# **SECTION 10 — CONVERSION (ToList, Dictionary) — 5 Questions**

91. Convert to list.
92. Convert to dictionary (Rno → Name).
93. Convert projected type to array (Names → string[]).
94. Create lookup (Rno → Courses).
95. Convert branch list to HashSet.

---

# **SECTION 11 — BASIC / MIXED — 5 Questions**

96. Top 2 highest CPI students.
97. Skip first 2, take next 2.
98. Find student with max CPI (full object).
99. Get students with course count + sort by count.
100.  Show students grouped by Branch and sorted by CPI.

---

# **ADVANCED LINQ QUERIES**

## **Deferred Execution — 5 Questions**

101. Show deferred execution: create a query, modify the source, then enumerate to show the change.
102. Convert deferred to immediate execution (force materialization).
103. Deferred + modifying list: show how changing an element affects query results.
104. Illustrate the multiple-enumeration problem and how to avoid it.
105. Deferred with filtering later: create a deferred query then apply further filters before enumeration.

---

## **Let Keyword — 3 Questions**

106. Use `let` to calculate squared CPI and select Name + squaredCPI.
107. Use `let` with filtering: compute ageCategory (Teen/Adult) and select names where category is "Adult".
108. Use `let` with multiple calculations (e.g., bonusCPI and isHigh) and project Name, bonusCPI, isHigh.

---

## **Into / Grouping Continuation — 3 Questions**

109. Group by branch into groups and select Branch + Count for groups having more than one student.
110. Group by Sem and select Sem with student names in that Sem.
111. Nested grouping: group by Branch, then within each branch group by Sem, and select Branch, Sem, and student names.

---

## **Complex Multi-Join — 3 Questions**

112. 3-table join example (Student + Course + Departments) — select Student Name, CourseName, DeptHead.
113. For each student compute total credits and include the branch head by joining with Departments.
114. Left join students with Departments, selecting Student Name, Branch, and DeptHead (or "No Dept" if missing).
