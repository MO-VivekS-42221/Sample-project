# SQL Intermediate Guide - Building Professional Skills

## Table of Contents
1. [Data Types & Constraints](#data-types--constraints)
2. [Complex WHERE Conditions](#complex-where-conditions)
3. [GROUP BY & HAVING](#group-by--having)
4. [Advanced Aggregation](#advanced-aggregation)
5. [All JOIN Types](#all-join-types)
6. [Subqueries](#subqueries)
7. [UNION & Set Operations](#union--set-operations)
8. [String & Date Functions](#string--date-functions)
9. [Views](#views)
10. [Practice Exercises](#practice-exercises)

---

## Data Types & Constraints

### Extended Data Types

```sql
CREATE TABLE students_extended (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    
    -- Text types
    first_name VARCHAR(50) NOT NULL,           -- Fixed max length
    bio TEXT,                                  -- Unlimited text
    
    -- Number types
    gpa DECIMAL(3, 2),                         -- 3 total digits, 2 decimal places
    age TINYINT,                               -- 0-255
    student_rank SMALLINT,                     -- -32768 to 32767
    account_balance DECIMAL(10, 2),            -- Money
    
    -- Date/Time types
    enrollment_date DATE,                      -- YYYY-MM-DD
    last_login DATETIME,                       -- YYYY-MM-DD HH:MM:SS
    birth_time TIME,                           -- HH:MM:SS
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Boolean
    is_active BOOLEAN DEFAULT TRUE,
    
    -- JSON (MySQL 5.7+)
    preferences JSON,
    
    -- Constraints
    email VARCHAR(100) UNIQUE NOT NULL,        -- No duplicates
    phone VARCHAR(20) UNIQUE,
    
    CONSTRAINT check_gpa CHECK (gpa >= 0 AND gpa <= 4.0),
    CONSTRAINT check_age CHECK (age >= 18)
);
```

### Column Constraints Explained

```sql
-- NOT NULL: Value is required
CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,  -- Must provide
    
    -- UNIQUE: No duplicate values allowed (NULL allowed)
    course_code VARCHAR(10) UNIQUE,
    
    -- PRIMARY KEY: Unique identifier (NOT NULL + UNIQUE)
    -- DEFAULT: Automatic value if not provided
    created_date DATE DEFAULT CURDATE(),
    
    -- FOREIGN KEY: Link to another table
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

### Data Type Selection Guide

| Need | Type | Example |
|------|------|---------|
| Whole numbers | INT | 42, 1000 |
| Decimals/Money | DECIMAL(10,2) | 99.99, 1000.50 |
| Text (fixed) | VARCHAR(n) | Names, emails |
| Text (unlimited) | TEXT | Descriptions |
| Dates | DATE | 2024-01-15 |
| Date + Time | DATETIME | 2024-01-15 10:30:00 |
| True/False | BOOLEAN | TRUE, FALSE |
| Large numbers | BIGINT | Timestamps, IDs |

---

## Complex WHERE Conditions

### NOT Operator

```sql
-- Students NOT in Computer Science
SELECT * FROM students WHERE NOT department = 'Computer Science';
-- Equivalent: WHERE department != 'Computer Science'

-- Students with GPA NOT between 3.0 and 3.5
SELECT * FROM students WHERE NOT (gpa BETWEEN 3.0 AND 3.5);

-- Courses NOT taught by Dr. Taylor
SELECT * FROM courses WHERE course_name NOT LIKE '%Design%';
```

### Complex AND/OR with Parentheses

```sql
-- Students with high GPA in CS or any GPA in Engineering
SELECT * FROM students
WHERE (department = 'Computer Science' AND gpa > 3.7)
   OR (department = 'Engineering');

-- Students who are NOT CS majors with good GPA
SELECT * FROM students
WHERE NOT (department = 'Computer Science' AND gpa < 3.5);
```

### CASE Statement in WHERE

```sql
-- Categorize students, then filter
SELECT 
    first_name,
    CASE 
        WHEN gpa >= 3.8 THEN 'Excellent'
        WHEN gpa >= 3.5 THEN 'Good'
        WHEN gpa >= 3.0 THEN 'Average'
        ELSE 'Below Average'
    END AS category
FROM students
WHERE CASE 
    WHEN gpa >= 3.8 THEN 'Excellent'
    WHEN gpa >= 3.5 THEN 'Good'
    WHEN gpa >= 3.0 THEN 'Average'
    ELSE 'Below Average'
END IN ('Excellent', 'Good');
```

---

## GROUP BY & HAVING

### Basic GROUP BY

```sql
-- Count students by department
SELECT 
    department,
    COUNT(*) AS student_count
FROM students
GROUP BY department;
```

**Result:**
| department | student_count |
|---|---|
| Computer Science | 3 |
| Engineering | 2 |
| Business | 1 |

### GROUP BY Multiple Columns

```sql
-- Students per department, sorted by department and GPA
SELECT 
    department,
    ROUND(gpa, 1) AS gpa_bracket,
    COUNT(*) AS count,
    AVG(gpa) AS avg_gpa
FROM students
GROUP BY department, ROUND(gpa, 1)
ORDER BY department, gpa_bracket DESC;
```

### HAVING - Filter After Grouping

⚠️ **KEY DIFFERENCE:**
- `WHERE` - Filters BEFORE grouping
- `HAVING` - Filters AFTER grouping/aggregation

```sql
-- Departments with MORE THAN 2 students
SELECT 
    department,
    COUNT(*) AS student_count,
    AVG(gpa) AS avg_gpa
FROM students
GROUP BY department
HAVING COUNT(*) > 2;

-- Instructors with average course credits > 3.5
SELECT 
    instructor,
    COUNT(*) AS course_count,
    AVG(credits) AS avg_credits
FROM courses
GROUP BY instructor
HAVING AVG(credits) > 3.5;

-- Departments where average GPA > 3.5 AND has 2+ students
SELECT 
    department,
    COUNT(*) AS count,
    AVG(gpa) AS avg_gpa
FROM students
WHERE gpa IS NOT NULL
GROUP BY department
HAVING COUNT(*) >= 2 AND AVG(gpa) > 3.5;
```

### GROUP BY with ORDER BY

```sql
-- Courses by enrollment count (descending)
SELECT 
    c.course_name,
    COUNT(e.enrollment_id) AS enrollment_count,
    c.max_capacity,
    ROUND(COUNT(e.enrollment_id) * 100.0 / c.max_capacity, 1) AS capacity_percent
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name, c.max_capacity
ORDER BY enrollment_count DESC;
```

---

## Advanced Aggregation

### Multiple Aggregations

```sql
-- Comprehensive course statistics
SELECT 
    c.course_name,
    COUNT(DISTINCT e.student_id) AS total_students,
    COUNT(CASE WHEN e.grade = 'A' THEN 1 END) AS grade_a_count,
    COUNT(CASE WHEN e.grade = 'B' THEN 1 END) AS grade_b_count,
    COUNT(CASE WHEN e.grade IS NULL THEN 1 END) AS incomplete_count,
    AVG(CASE WHEN e.grade = 'A' THEN 4
             WHEN e.grade = 'B' THEN 3
             WHEN e.grade = 'C' THEN 2
             WHEN e.grade = 'D' THEN 1
             ELSE 0 END) AS avg_gpa_equivalent,
    MIN(e.enrollment_date) AS first_enrollment,
    MAX(e.enrollment_date) AS last_enrollment
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name;
```

### DISTINCT within Aggregates

```sql
-- Count unique departments that have students with GPA > 3.7
SELECT COUNT(DISTINCT department) AS high_performing_departments
FROM students
WHERE gpa > 3.7;

-- String aggregation of students in each course
SELECT 
    c.course_name,
    GROUP_CONCAT(s.first_name, ', ') AS student_names
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
LEFT JOIN students s ON e.student_id = s.student_id
GROUP BY c.course_id, c.course_name;
```

### Conditional Aggregation

```sql
-- Count grades by type
SELECT 
    course_id,
    COUNT(*) AS total_enrollments,
    SUM(CASE WHEN grade = 'A' THEN 1 ELSE 0 END) AS count_a,
    SUM(CASE WHEN grade = 'B' THEN 1 ELSE 0 END) AS count_b,
    SUM(CASE WHEN grade = 'C' THEN 1 ELSE 0 END) AS count_c,
    SUM(CASE WHEN grade IS NULL THEN 1 ELSE 0 END) AS incomplete
FROM enrollments
GROUP BY course_id;
```

---

## All JOIN Types

### Complete JOIN Reference

```sql
-- Setup sample data
INSERT INTO students (student_id, first_name) VALUES
(1, 'John'),
(2, 'Sarah'),
(3, 'Michael');

INSERT INTO enrollments (student_id, course_id) VALUES
(1, 101),
(2, 101),
(2, 102),
(4, 103);  -- student_id 4 doesn't exist in students
```

### INNER JOIN - Only Matching Rows

```sql
SELECT s.first_name, e.course_id
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id;
```

**Result:** Only students with enrollments
```
John - 101
Sarah - 101
Sarah - 102
```

### LEFT JOIN - All from Left Table

```sql
SELECT s.first_name, e.course_id
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id;
```

**Result:** All students, even Michael with NULL
```
John - 101
Sarah - 101
Sarah - 102
Michael - NULL
```

### RIGHT JOIN - All from Right Table

```sql
SELECT s.first_name, e.course_id
FROM students s
RIGHT JOIN enrollments e ON s.student_id = e.student_id;
```

**Result:** All enrollments, including orphaned (no student)
```
John - 101
Sarah - 101
Sarah - 102
NULL - 103
```

### FULL OUTER JOIN (UNION of LEFT and RIGHT)

```sql
SELECT s.first_name, e.course_id
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
UNION
SELECT s.first_name, e.course_id
FROM students s
RIGHT JOIN enrollments e ON s.student_id = e.student_id;
```

**Result:** All records from both sides
```
John - 101
Sarah - 101
Sarah - 102
Michael - NULL
NULL - 103
```

### CROSS JOIN - Cartesian Product

```sql
-- Every student paired with every course (all combinations)
SELECT s.first_name, c.course_name
FROM students s
CROSS JOIN courses c;
```

**Result:** If 3 students and 4 courses = 12 rows (3 × 4)

### Self-JOIN - Table Joined to Itself

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

-- Find employees and their managers
SELECT 
    e.name AS employee_name,
    m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

---

## Subqueries

### Subquery in SELECT Clause

```sql
-- Show each student with average GPA of their department
SELECT 
    s.first_name,
    s.gpa,
    (SELECT AVG(gpa) 
     FROM students 
     WHERE department = s.department) AS dept_avg_gpa
FROM students s;
```

### Subquery in WHERE Clause - IN

```sql
-- Students enrolled in courses taught by Dr. Taylor
SELECT s.first_name, s.last_name
FROM students s
WHERE s.student_id IN (
    SELECT e.student_id
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE c.instructor = 'Dr. Taylor'
);
```

### Subquery in WHERE Clause - Comparison

```sql
-- Students with above-average GPA
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

### Subquery in WHERE Clause - EXISTS

```sql
-- Courses that have at least one enrollment
SELECT c.course_name
FROM courses c
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.course_id = c.course_id
);

-- Students with NO enrollments
SELECT first_name
FROM students s
WHERE NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
);
```

### Subquery in FROM Clause (Derived Table)

```sql
-- Get top departments by student count
SELECT dept_stats.department, dept_stats.student_count
FROM (
    SELECT 
        department,
        COUNT(*) AS student_count
    FROM students
    GROUP BY department
) AS dept_stats
WHERE dept_stats.student_count > 1
ORDER BY dept_stats.student_count DESC;
```

### Correlated Subquery

```sql
-- Show each student and their rank within their department
SELECT 
    s.first_name,
    s.department,
    s.gpa,
    (SELECT COUNT(*) + 1
     FROM students s2
     WHERE s2.department = s.department
       AND s2.gpa > s.gpa) AS rank_in_dept
FROM students s
ORDER BY s.department, rank_in_dept;
```

---

## UNION & Set Operations

### UNION - Combine Results (Remove Duplicates)

```sql
-- All unique names (both students and instructors)
SELECT first_name AS name FROM students
UNION
SELECT SUBSTRING_INDEX(instructor, ' ', 1) AS name FROM courses
WHERE instructor IS NOT NULL;
```

### UNION ALL - Keep Duplicates

```sql
-- All enrollments from 2023 and 2024 combined
SELECT student_id, course_id FROM enrollments WHERE YEAR(enrollment_date) = 2023
UNION ALL
SELECT student_id, course_id FROM enrollments WHERE YEAR(enrollment_date) = 2024;
```

### INTERSECT - Common Records (Some databases)

```sql
-- Students who took both DB101 and CS101
SELECT student_id FROM enrollments WHERE course_id = 1
INTERSECT
SELECT student_id FROM enrollments WHERE course_id = 3;

-- Manual workaround for MySQL:
SELECT DISTINCT e1.student_id
FROM enrollments e1
WHERE e1.course_id = 1
  AND EXISTS (SELECT 1 FROM enrollments e2 
              WHERE e2.student_id = e1.student_id 
              AND e2.course_id = 3);
```

---

## String & Date Functions

### String Functions

```sql
-- Concatenation
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM students;
SELECT CONCAT_WS(' - ', first_name, last_name, gpa) FROM students;

-- Length
SELECT first_name, LENGTH(first_name) AS name_length FROM students;

-- Case conversion
SELECT UPPER(first_name), LOWER(last_name) FROM students;

-- Substring
SELECT SUBSTRING(email, 1, POSITION('@' IN email) - 1) AS username FROM students;

-- Replace
SELECT REPLACE(email, '@university.edu', '@school.edu') FROM students;

-- Trim whitespace
SELECT TRIM(first_name), LTRIM(first_name), RTRIM(first_name) FROM students;

-- Find position
SELECT email, POSITION('@' IN email) AS at_position FROM students;

-- Repeat
SELECT REPEAT('*', 5);  -- *****
```

### Date Functions

```sql
-- Current date/time
SELECT NOW(), CURDATE(), CURTIME();

-- Date arithmetic
SELECT DATE_ADD(CURDATE(), INTERVAL 7 DAY);
SELECT DATE_SUB(CURDATE(), INTERVAL 1 MONTH);
SELECT DATE_ADD(CURDATE(), INTERVAL 1 YEAR);

-- Date differences
SELECT DATEDIFF(CURDATE(), enrollment_date) AS days_enrolled FROM students;
SELECT TIMESTAMPDIFF(MONTH, enrollment_date, CURDATE()) FROM students;

-- Extract parts
SELECT 
    YEAR(enrollment_date),
    QUARTER(enrollment_date),
    MONTH(enrollment_date),
    WEEK(enrollment_date),
    DAY(enrollment_date),
    DAYNAME(enrollment_date),
    MONTHNAME(enrollment_date)
FROM students;

-- Date formatting
SELECT DATE_FORMAT(enrollment_date, '%d/%m/%Y') FROM students;
SELECT DATE_FORMAT(enrollment_date, '%M %d, %Y') FROM students;

-- Last day of month
SELECT LAST_DAY(enrollment_date) FROM students;

-- Week calculations
SELECT WEEK(CURDATE()) AS current_week;
SELECT WEEKOFYEAR(enrollment_date) FROM students;
```

---

## Views

### Creating Views

```sql
-- Simple view: Top performers
CREATE VIEW top_students AS
SELECT first_name, last_name, gpa, department
FROM students
WHERE gpa > 3.7
ORDER BY gpa DESC;

-- Query the view
SELECT * FROM top_students;
```

### Complex Views with JOINs

```sql
-- Student enrollment summary
CREATE VIEW student_summary AS
SELECT 
    s.student_id,
    s.first_name,
    s.last_name,
    COUNT(DISTINCT e.course_id) AS courses_taken,
    COUNT(DISTINCT CASE WHEN e.grade IS NOT NULL THEN e.course_id END) AS courses_completed,
    AVG(CASE WHEN e.grade = 'A' THEN 4
             WHEN e.grade = 'B' THEN 3
             WHEN e.grade = 'C' THEN 2
             WHEN e.grade = 'D' THEN 1
             ELSE 0 END) AS avg_grade
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

### Modifying Views

```sql
-- Update view definition
CREATE OR REPLACE VIEW top_students AS
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > 3.8;  -- Changed from 3.7

-- Drop view
DROP VIEW top_students;
DROP VIEW IF EXISTS top_students;
```

---

## Practice Exercises

### GROUP BY & HAVING

1. **Count courses by semester, only show semesters with 3+ courses**
   ```sql
   SELECT semester, COUNT(*) AS course_count
   FROM courses
   GROUP BY semester
   HAVING COUNT(*) >= 3;
   ```

2. **Average grade per course**
   ```sql
   SELECT 
       c.course_name,
       ROUND(AVG(CASE WHEN e.grade = 'A' THEN 4 WHEN e.grade = 'B' THEN 3 ELSE 2 END), 2) AS avg_grade
   FROM courses c
   LEFT JOIN enrollments e ON c.course_id = e.course_id
   GROUP BY c.course_id, c.course_name;
   ```

### Complex JOINs

3. **Students and ALL their courses (including not enrolled)**
   ```sql
   SELECT s.first_name, c.course_name, e.grade
   FROM students s
   CROSS JOIN courses c
   LEFT JOIN enrollments e ON s.student_id = e.student_id AND c.course_id = e.course_id;
   ```

### Subqueries

4. **Departments with above-average number of students**
   ```sql
   SELECT department, COUNT(*) AS count
   FROM students
   GROUP BY department
   HAVING COUNT(*) > (SELECT AVG(dept_count) FROM (
       SELECT COUNT(*) AS dept_count FROM students GROUP BY department
   ) AS dept_stats);
   ```

### String & Date Functions

5. **Extract semester year and format**
   ```sql
   SELECT 
       course_name,
       CONCAT(SUBSTRING(semester, 1, 4), ' ', SUBSTRING(semester, 6)) AS formatted_semester
   FROM courses;
   ```

---

## Key Intermediate Concepts

✅ **GROUP BY groups rows, HAVING filters groups**

✅ **Subqueries can go in SELECT, WHERE, FROM clauses**

✅ **JOINs connect tables; LEFT/RIGHT/FULL control which rows show**

✅ **UNION combines result sets vertically**

✅ **Views simplify complex queries**

✅ **Functions work with dates and strings**

---

## Next Steps
Graduate to **Advanced Guide** for:
- Window functions (RANK, ROW_NUMBER, DENSE_RANK)
- CTEs (Common Table Expressions)
- Transactions & ACID
- Query optimization
- Advanced indexing

---

**Difficulty Level:** ⭐⭐ Intermediate  
**Estimated Learning Time:** 10-15 hours  
**Created:** 2026-09-02
