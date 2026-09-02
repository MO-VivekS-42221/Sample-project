# SQL Beginner Guide - Complete Learning Path

## Table of Contents
1. [Understanding Databases](#understanding-databases)
2. [Basic SQL Syntax](#basic-sql-syntax)
3. [CRUD Operations](#crud-operations)
4. [Filtering Data](#filtering-data)
5. [Sorting & Limiting Results](#sorting--limiting-results)
6. [Introduction to JOINs](#introduction-to-joins)
7. [Common Functions](#common-functions)
8. [Practice Exercises](#practice-exercises)

---

## Understanding Databases

### What is a Database?
A database is an organized collection of structured data stored in tables. Think of it like an Excel spreadsheet with multiple sheets that can talk to each other.

### What is SQL?
SQL (Structured Query Language) is the language used to communicate with databases. It allows you to:
- **Create** new data
- **Read** existing data
- **Update** data
- **Delete** data

### Database vs Table vs Record
```
DATABASE (School System)
  └── TABLE (Students)
       ├── student_id | first_name | last_name | gpa
       ├── Record 1: 1 | John | Smith | 3.8
       ├── Record 2: 2 | Sarah | Johnson | 3.6
       └── Record 3: 3 | Michael | Brown | 3.4
```

---

## Basic SQL Syntax

### SQL Keywords (Case-Insensitive, but convention: UPPERCASE)
```sql
SELECT * FROM students;
```

**Breaking it down:**
- `SELECT` - What columns do you want?
- `*` - All columns
- `FROM` - Which table?
- `students` - The table name

### Basic Structure
```
SELECT [columns]
FROM [table_name]
WHERE [conditions]
ORDER BY [column]
LIMIT [number];
```

---

## CRUD Operations

### CREATE TABLE - Making a Table

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    enrollment_date DATE,
    gpa DECIMAL(3, 2)
);
```

**Breakdown:**
- `student_id INT` - Whole number column
- `PRIMARY KEY` - Unique identifier for each row
- `AUTO_INCREMENT` - Automatically number rows (1, 2, 3...)
- `VARCHAR(50)` - Text up to 50 characters
- `NOT NULL` - This field is required
- `DATE` - Calendar date (YYYY-MM-DD)
- `DECIMAL(3, 2)` - Number with 3 total digits, 2 after decimal (e.g., 3.85)

### CREATE TABLE - Courses

```sql
CREATE TABLE courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    course_name VARCHAR(100) NOT NULL,
    course_code VARCHAR(10),
    instructor VARCHAR(50),
    credits INT,
    semester VARCHAR(20)
);
```

### Common Data Types

| Type | Example | Use Case |
|------|---------|----------|
| INT | 42, 1000 | Whole numbers, IDs, counts |
| VARCHAR(n) | 'John', 'CS101' | Text with max length |
| TEXT | Long paragraphs | Unlimited text |
| DATE | 2024-01-15 | Calendar dates |
| DECIMAL(m,n) | 3.85, 99.99 | Money, GPAs, precise numbers |
| BOOLEAN | TRUE, FALSE | Yes/No values |

### INSERT - Adding Data

```sql
-- Single record
INSERT INTO students (first_name, last_name, email, gpa)
VALUES ('John', 'Smith', 'john@uni.edu', 3.8);

-- Multiple records at once
INSERT INTO students (first_name, last_name, email, gpa) VALUES
('Sarah', 'Johnson', 'sarah@uni.edu', 3.6),
('Michael', 'Brown', 'michael@uni.edu', 3.4),
('Emily', 'Davis', 'emily@uni.edu', 3.9);
```

**Key Points:**
- Column names in parentheses must match table structure
- String values need single quotes: `'John'`
- Numbers don't need quotes: `3.8`
- NULL is used for empty values (if column allows)

### READ - Retrieving Data

```sql
-- All data from students table
SELECT * FROM students;

-- Specific columns
SELECT first_name, last_name, gpa FROM students;

-- With readable headers (aliases)
SELECT first_name AS Name, gpa AS GPA FROM students;
```

### UPDATE - Changing Data

```sql
-- Update one student's GPA
UPDATE students
SET gpa = 3.9
WHERE student_id = 1;

-- Update multiple columns
UPDATE students
SET gpa = 3.5, email = 'newemail@uni.edu'
WHERE first_name = 'John';

-- Update multiple rows
UPDATE students
SET gpa = 4.0
WHERE gpa > 3.8;
```

⚠️ **IMPORTANT:** Always use WHERE clause! Without it, ALL rows will be updated!

### DELETE - Removing Data

```sql
-- Delete one student
DELETE FROM students
WHERE student_id = 1;

-- Delete multiple rows
DELETE FROM students
WHERE gpa < 2.0;
```

⚠️ **CRITICAL:** Always use WHERE clause! Without it, ALL rows will be deleted!

---

## Filtering Data

### WHERE Clause - Basic Operators

```sql
-- Equals
SELECT * FROM students WHERE gpa = 3.8;

-- Not equals
SELECT * FROM students WHERE gpa != 3.8;
-- or: WHERE gpa <> 3.8;

-- Greater than
SELECT * FROM students WHERE gpa > 3.5;

-- Greater than or equal
SELECT * FROM students WHERE gpa >= 3.5;

-- Less than
SELECT * FROM students WHERE gpa < 3.0;

-- Less than or equal
SELECT * FROM students WHERE gpa <= 3.0;
```

### Combining Conditions - AND / OR

```sql
-- Both conditions must be true (AND)
SELECT * FROM students
WHERE gpa > 3.5 AND first_name = 'John';

-- At least one condition must be true (OR)
SELECT * FROM students
WHERE gpa > 3.8 OR gpa < 2.0;

-- Combining AND & OR (AND has priority)
SELECT * FROM students
WHERE gpa > 3.5 AND (first_name = 'John' OR first_name = 'Sarah');
```

### IN - Multiple Options

```sql
-- Instead of: WHERE first_name = 'John' OR first_name = 'Sarah' OR first_name = 'Michael'
SELECT * FROM students
WHERE first_name IN ('John', 'Sarah', 'Michael');

-- Numbers
SELECT * FROM courses
WHERE credits IN (3, 4);
```

### BETWEEN - Range

```sql
-- GPA between 3.0 and 3.8
SELECT * FROM students
WHERE gpa BETWEEN 3.0 AND 3.8;

-- Equivalent to: WHERE gpa >= 3.0 AND gpa <= 3.8
```

### LIKE - Pattern Matching

```sql
-- Names starting with 'J'
SELECT * FROM students WHERE first_name LIKE 'J%';

-- Names ending with 'son'
SELECT * FROM students WHERE last_name LIKE '%son';

-- Names containing 'oh' anywhere
SELECT * FROM students WHERE first_name LIKE '%oh%';

-- Exact 4-letter names
SELECT * FROM students WHERE first_name LIKE '____';
```

**Wildcard Symbols:**
- `%` = Any number of characters (0 or more)
- `_` = Exactly one character

### IS NULL / IS NOT NULL

```sql
-- Students with no email recorded
SELECT * FROM students WHERE email IS NULL;

-- Students WITH email recorded
SELECT * FROM students WHERE email IS NOT NULL;
```

---

## Sorting & Limiting Results

### ORDER BY - Sorting

```sql
-- Sort by GPA (lowest to highest)
SELECT first_name, last_name, gpa
FROM students
ORDER BY gpa ASC;

-- Sort by GPA (highest to lowest) - DEFAULT
SELECT first_name, last_name, gpa
FROM students
ORDER BY gpa DESC;

-- Sort by multiple columns (Department A-Z, then GPA high to low)
SELECT first_name, last_name, gpa
FROM students
ORDER BY gpa DESC, last_name ASC;
```

### LIMIT - Get Only Top Rows

```sql
-- Get first 5 students
SELECT * FROM students LIMIT 5;

-- Get top 3 by GPA
SELECT first_name, last_name, gpa
FROM students
ORDER BY gpa DESC
LIMIT 3;

-- Skip first 2, then get 3 (Pagination)
SELECT first_name, last_name, gpa
FROM students
ORDER BY gpa DESC
LIMIT 3 OFFSET 2;
```

---

## Introduction to JOINs

### Why JOINs Matter

Imagine you have two tables:
- **students**: student_id, name, gpa
- **courses**: course_id, name, instructor
- **enrollments**: student_id, course_id, grade

To get "Which students took which courses?", you need to JOIN tables.

### INNER JOIN - Only Matching Records

```sql
CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT,
    course_id INT,
    grade CHAR(1)
);

INSERT INTO enrollments (student_id, course_id, grade) VALUES
(1, 101, 'A'),
(1, 102, 'B'),
(2, 101, 'A'),
(3, 103, 'C');
```

```sql
-- Show student names with their courses
SELECT 
    s.first_name,
    s.last_name,
    c.course_name,
    e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id;
```

**How it works:**
1. `s` = short name for students table
2. `INNER JOIN` = only show rows where BOTH tables have matching data
3. `ON s.student_id = e.student_id` = match rule

**Result:**
| first_name | last_name | course_name | grade |
|---|---|---|---|
| John | Smith | Intro to SQL | A |
| John | Smith | Database Design | B |
| Sarah | Johnson | Intro to SQL | A |

### LEFT JOIN - All from First Table

```sql
-- Show all students, even if they're not enrolled in any courses
SELECT 
    s.first_name,
    s.last_name,
    c.course_name
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id;
```

**Key Difference:** If a student has no enrollment, they still show up with NULL for course_name.

---

## Common Functions

### String Functions

```sql
-- Combine columns
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM students;
-- Result: 'John Smith'

-- Length of text
SELECT first_name, LENGTH(first_name) AS name_length FROM students;
-- Result: John = 4, Sarah = 5

-- Uppercase/Lowercase
SELECT UPPER(first_name), LOWER(last_name) FROM students;
-- Result: JOHN smith

-- Extract part of text
SELECT SUBSTRING(email, 1, 5) FROM students;
-- Result: 'john.' from 'john.smith@uni.edu'
```

### Number Functions

```sql
-- Round decimal
SELECT gpa, ROUND(gpa, 1) FROM students;
-- 3.85 rounds to 3.9

-- Absolute value (always positive)
SELECT ABS(-5); -- Returns 5

-- Round up/down
SELECT CEIL(3.2), FLOOR(3.8); -- Returns 4, 3
```

### Date Functions

```sql
-- Today's date
SELECT NOW(); -- 2024-01-15 14:30:45

-- Just the date
SELECT CURDATE(); -- 2024-01-15

-- Add days
SELECT DATE_ADD(CURDATE(), INTERVAL 7 DAY); -- 2024-01-22

-- Days between two dates
SELECT DATEDIFF('2024-01-15', '2024-01-10'); -- 5

-- Extract part of date
SELECT EXTRACT(YEAR FROM enrollment_date) FROM students;
```

### Counting & Aggregates

```sql
-- Count students
SELECT COUNT(*) FROM students; -- 4

-- Count non-null emails
SELECT COUNT(email) FROM students;

-- Average GPA
SELECT AVG(gpa) FROM students; -- 3.675

-- Highest/Lowest
SELECT MAX(gpa), MIN(gpa) FROM students;
-- Result: 3.9, 3.4

-- Sum all credits
SELECT SUM(credits) FROM courses;
```

---

## Practice Exercises

### Level 1 - Basic SELECT

1. **Show all students**
   ```sql
   SELECT * FROM students;
   ```

2. **Show only names and GPA**
   ```sql
   SELECT first_name, last_name, gpa FROM students;
   ```

3. **Show students with GPA > 3.5**
   ```sql
   SELECT first_name, last_name, gpa FROM students WHERE gpa > 3.5;
   ```

### Level 2 - WHERE & Operators

4. **Find students named 'John' or 'Sarah'**
   ```sql
   SELECT * FROM students WHERE first_name IN ('John', 'Sarah');
   ```

5. **Find students with GPA between 3.2 and 3.7**
   ```sql
   SELECT * FROM students WHERE gpa BETWEEN 3.2 AND 3.7;
   ```

6. **Find students with email addresses**
   ```sql
   SELECT * FROM students WHERE email IS NOT NULL;
   ```

### Level 3 - Sorting & Limiting

7. **Top 3 students by GPA**
   ```sql
   SELECT first_name, last_name, gpa FROM students ORDER BY gpa DESC LIMIT 3;
   ```

8. **All students sorted alphabetically**
   ```sql
   SELECT first_name, last_name FROM students ORDER BY last_name ASC;
   ```

### Level 4 - JOINs

9. **Show which students took which courses**
   ```sql
   SELECT s.first_name, s.last_name, c.course_name
   FROM students s
   INNER JOIN enrollments e ON s.student_id = e.student_id
   INNER JOIN courses c ON e.course_id = c.course_id;
   ```

### Level 5 - Aggregates

10. **Count total students**
    ```sql
    SELECT COUNT(*) AS total_students FROM students;
    ```

11. **Average GPA of all students**
    ```sql
    SELECT AVG(gpa) AS average_gpa FROM students;
    ```

12. **How many courses each student took**
    ```sql
    SELECT s.first_name, s.last_name, COUNT(e.course_id) AS courses_taken
    FROM students s
    LEFT JOIN enrollments e ON s.student_id = e.student_id
    GROUP BY s.student_id;
    ```

---

## Key Takeaways for Beginners

✅ **SQL is about asking questions from data**
- SELECT = What do I want?
- FROM = Where is it?
- WHERE = Any filters?
- ORDER BY = How should I sort?
- LIMIT = How many do I want?

✅ **Always use WHERE clause when deleting/updating**

✅ **String values need single quotes, numbers don't**

✅ **NULL means "no value" - use IS NULL, not = NULL**

✅ **JOINs connect multiple tables using matching values**

✅ **Practice makes perfect - run these queries in order!**

---

## Next Steps
Once you master these basics, move on to the **Intermediate Guide** for:
- GROUP BY and aggregate functions
- Subqueries
- More complex JOINs
- Window functions

---

**Difficulty Level:** ⭐ Beginner  
**Estimated Learning Time:** 5-8 hours  
**Created:** 2026-09-02