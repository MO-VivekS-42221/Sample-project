# SQL In-Depth Guide with Courses & Students Examples

## Table of Contents
1. [Database Setup](#database-setup)
2. [Basic SELECT Queries](#basic-select-queries)
3. [Filtering with WHERE](#filtering-with-where)
4. [JOINs](#joins)
5. [Aggregation Functions](#aggregation-functions)
6. [Grouping & Sorting](#grouping--sorting)
7. [Subqueries](#subqueries)
8. [Advanced Concepts](#advanced-concepts)

---

## Database Setup

### Create Tables Schema

```sql
-- Create Students Table
CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    enrollment_date DATE,
    gpa DECIMAL(3, 2),
    department VARCHAR(50)
);

-- Create Courses Table
CREATE TABLE courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    course_name VARCHAR(100) NOT NULL,
    course_code VARCHAR(10) UNIQUE,
    instructor VARCHAR(50),
    credits INT,
    semester VARCHAR(20),
    max_capacity INT
);

-- Create Enrollments Table (Junction Table)
CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    grade CHAR(1),
    enrollment_date DATE,
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

### Sample Data

```sql
-- Insert Students
INSERT INTO students (first_name, last_name, email, enrollment_date, gpa, department) VALUES
('John', 'Smith', 'john.smith@university.edu', '2023-09-01', 3.8, 'Computer Science'),
('Sarah', 'Johnson', 'sarah.johnson@university.edu', '2023-09-01', 3.6, 'Computer Science'),
('Michael', 'Brown', 'michael.brown@university.edu', '2023-09-01', 3.4, 'Engineering'),
('Emily', 'Davis', 'emily.davis@university.edu', '2023-09-15', 3.9, 'Computer Science'),
('David', 'Wilson', 'david.wilson@university.edu', '2023-09-01', 3.2, 'Business'),
('Lisa', 'Anderson', 'lisa.anderson@university.edu', '2023-10-01', 3.7, 'Engineering');

-- Insert Courses
INSERT INTO courses (course_name, course_code, instructor, credits, semester, max_capacity) VALUES
('Introduction to SQL', 'DB101', 'Dr. Robert Taylor', 3, 'Fall 2023', 40),
('Database Design', 'DB201', 'Dr. Robert Taylor', 4, 'Fall 2023', 35),
('Web Development', 'CS101', 'Prof. Alice Green', 3, 'Fall 2023', 50),
('Data Structures', 'CS102', 'Prof. James White', 4, 'Fall 2023', 45),
('Machine Learning', 'CS301', 'Dr. Maria Garcia', 4, 'Spring 2024', 30),
('Advanced SQL', 'DB301', 'Dr. Robert Taylor', 4, 'Spring 2024', 25);

-- Insert Enrollments
INSERT INTO enrollments (student_id, course_id, grade, enrollment_date) VALUES
(1, 1, 'A', '2023-09-01'),
(1, 2, 'A', '2023-09-01'),
(1, 5, NULL, '2024-01-15'),
(2, 1, 'B', '2023-09-01'),
(2, 3, 'A', '2023-09-01'),
(3, 2, 'B', '2023-09-01'),
(3, 4, 'C', '2023-09-01'),
(4, 1, 'A', '2023-09-15'),
(4, 3, 'A', '2023-09-15'),
(4, 5, NULL, '2024-01-15'),
(5, 3, 'C', '2023-09-01'),
(5, 4, 'D', '2023-09-01'),
(6, 2, 'B', '2023-10-01'),
(6, 4, 'B', '2023-10-01');
```

---

## Basic SELECT Queries

### 1. SELECT All Columns
```sql
SELECT * FROM students;
```
**Result:** Returns all columns for all students
| student_id | first_name | last_name | email | enrollment_date | gpa | department |
|---|---|---|---|---|---|---|
| 1 | John | Smith | john.smith@university.edu | 2023-09-01 | 3.8 | Computer Science |
| 2 | Sarah | Johnson | sarah.johnson@university.edu | 2023-09-01 | 3.6 | Computer Science |
| ... | ... | ... | ... | ... | ... | ... |

### 2. SELECT Specific Columns
```sql
SELECT student_id, first_name, last_name, gpa FROM students;
```
**Result:** Returns only the specified columns

### 3. SELECT with Column Alias
```sql
SELECT 
    student_id AS ID,
    CONCAT(first_name, ' ', last_name) AS full_name,
    gpa AS grade_point_average
FROM students;
```
**Explanation:** Using `AS` creates aliases for better readability

### 4. SELECT DISTINCT Values
```sql
SELECT DISTINCT department FROM students;
```
**Result:** Returns unique departments without duplicates
```
Computer Science
Engineering
Business
```

---

## Filtering with WHERE

### 1. Simple Equality Filter
```sql
SELECT first_name, last_name, gpa FROM students WHERE gpa > 3.5;
```
**Result:** Students with GPA greater than 3.5
```
John Smith - 3.8
Sarah Johnson - 3.6
Emily Davis - 3.9
Lisa Anderson - 3.7
```

### 2. Multiple Conditions (AND/OR)
```sql
SELECT first_name, last_name, department, gpa 
FROM students 
WHERE department = 'Computer Science' AND gpa >= 3.6;
```
**Result:** CS students with GPA 3.6 or higher

```sql
SELECT first_name, last_name 
FROM students 
WHERE department = 'Computer Science' OR department = 'Engineering';
```
**Result:** Students in CS or Engineering

### 3. IN Operator
```sql
SELECT course_name, credits 
FROM courses 
WHERE credits IN (3, 4);
```
**Result:** All 3 or 4 credit courses

### 4. BETWEEN Operator
```sql
SELECT first_name, last_name, gpa 
FROM students 
WHERE gpa BETWEEN 3.4 AND 3.8;
```
**Result:** Students with GPA between 3.4 and 3.8

### 5. LIKE Pattern Matching
```sql
-- Names starting with 'J'
SELECT first_name, last_name FROM students WHERE first_name LIKE 'J%';

-- Names containing 'son'
SELECT first_name, last_name FROM students WHERE last_name LIKE '%son';

-- Names with exactly 5 characters
SELECT first_name FROM students WHERE first_name LIKE '_____';
```

### 6. IS NULL / IS NOT NULL
```sql
-- Find enrollments without grades (current semester)
SELECT student_id, course_id FROM enrollments WHERE grade IS NULL;

-- Find students with GPA recorded
SELECT first_name, gpa FROM students WHERE gpa IS NOT NULL;
```

---

## JOINs

### 1. INNER JOIN
```sql
SELECT 
    s.first_name,
    s.last_name,
    c.course_name,
    e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id;
```
**Explanation:** Returns only matching records from all tables
**Result:** Shows each student with their courses and grades

| first_name | last_name | course_name | grade |
|---|---|---|---|
| John | Smith | Introduction to SQL | A |
| John | Smith | Database Design | A |
| Sarah | Johnson | Introduction to SQL | B |
| Sarah | Johnson | Web Development | A |

### 2. LEFT JOIN
```sql
SELECT 
    s.first_name,
    s.last_name,
    COUNT(e.course_id) AS courses_enrolled
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```
**Explanation:** Returns all students, even if they have no enrollments
**Result:** Shows course count for each student (0 if none)

### 3. RIGHT JOIN
```sql
SELECT 
    c.course_name,
    s.first_name,
    s.last_name
FROM students s
RIGHT JOIN enrollments e ON s.student_id = e.student_id
RIGHT JOIN courses c ON e.course_id = c.course_id;
```
**Explanation:** Returns all courses, even if no students enrolled

### 4. FULL OUTER JOIN (Using UNION)
```sql
SELECT s.first_name, c.course_name
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id
UNION
SELECT s.first_name, c.course_name
FROM students s
RIGHT JOIN enrollments e ON s.student_id = e.student_id
RIGHT JOIN courses c ON e.course_id = c.course_id;
```
**Explanation:** Returns all records from both tables

---

## Aggregation Functions

### 1. COUNT()
```sql
-- Total number of students
SELECT COUNT(*) AS total_students FROM students;
-- Result: 6

-- Students per department
SELECT department, COUNT(*) AS student_count
FROM students
GROUP BY department;
```

### 2. SUM()
```sql
-- Total credits enrolled by John Smith
SELECT SUM(c.credits) AS total_credits
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE s.first_name = 'John' AND s.last_name = 'Smith';
-- Result: 7 (3+4)
```

### 3. AVG()
```sql
-- Average GPA by department
SELECT department, AVG(gpa) AS avg_gpa
FROM students
GROUP BY department;
```

### 4. MIN() and MAX()
```sql
-- Highest and lowest GPA
SELECT 
    MAX(gpa) AS highest_gpa,
    MIN(gpa) AS lowest_gpa
FROM students;
-- Result: highest_gpa = 3.9, lowest_gpa = 3.2

-- Courses with most and least credits
SELECT 
    MAX(credits) AS max_credits,
    MIN(credits) AS min_credits
FROM courses;
```

---

## Grouping & Sorting

### 1. GROUP BY
```sql
-- Courses with enrollment count
SELECT 
    c.course_name,
    COUNT(e.enrollment_id) AS enrolled_students
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name;
```

**Result:**
| course_name | enrolled_students |
|---|---|
| Introduction to SQL | 3 |
| Database Design | 2 |
| Web Development | 2 |
| Data Structures | 3 |
| Machine Learning | 2 |
| Advanced SQL | 0 |

### 2. GROUP BY with HAVING
```sql
-- Find departments with average GPA above 3.5
SELECT 
    department,
    AVG(gpa) AS avg_gpa,
    COUNT(*) AS student_count
FROM students
GROUP BY department
HAVING AVG(gpa) > 3.5;
```

**Result:** Only Computer Science (avg: 3.77) and Engineering (avg: 3.55)

### 3. ORDER BY (Sorting)
```sql
-- Students sorted by GPA (descending)
SELECT first_name, last_name, gpa
FROM students
ORDER BY gpa DESC;

-- Sort by multiple columns
SELECT first_name, last_name, gpa
FROM students
ORDER BY department ASC, gpa DESC;
```

### 4. LIMIT and OFFSET
```sql
-- Top 3 students by GPA
SELECT first_name, last_name, gpa
FROM students
ORDER BY gpa DESC
LIMIT 3;

-- Pagination (skip first 2, get next 3)
SELECT first_name, last_name, gpa
FROM students
ORDER BY enrollment_date
LIMIT 3 OFFSET 2;
```

---

## Subqueries

### 1. Scalar Subquery (Returns single value)
```sql
-- Find students with GPA above average
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

### 2. Subquery in FROM Clause
```sql
-- Get average GPA per department
SELECT department, avg_gpa
FROM (
    SELECT department, AVG(gpa) AS avg_gpa
    FROM students
    GROUP BY department
) AS dept_stats
WHERE avg_gpa > 3.5;
```

### 3. IN Subquery
```sql
-- Find courses taught by Dr. Robert Taylor
SELECT first_name, last_name
FROM students
WHERE student_id IN (
    SELECT e.student_id
    FROM enrollments e
    INNER JOIN courses c ON e.course_id = c.course_id
    WHERE c.instructor = 'Dr. Robert Taylor'
);
```

### 4. EXISTS Subquery
```sql
-- Students who have enrolled in at least one course
SELECT first_name, last_name
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
);
```

---

## Advanced Concepts

### 1. Window Functions
```sql
-- Rank students by GPA within each department
SELECT 
    first_name,
    last_name,
    department,
    gpa,
    RANK() OVER (PARTITION BY department ORDER BY gpa DESC) AS rank_in_dept
FROM students;
```

**Result:**
| first_name | last_name | department | gpa | rank_in_dept |
|---|---|---|---|---|
| Emily | Davis | Computer Science | 3.9 | 1 |
| John | Smith | Computer Science | 3.8 | 2 |
| Sarah | Johnson | Computer Science | 3.6 | 3 |

### 2. CASE Statements
```sql
-- Categorize students by GPA
SELECT 
    first_name,
    last_name,
    gpa,
    CASE 
        WHEN gpa >= 3.8 THEN 'Excellent'
        WHEN gpa >= 3.5 THEN 'Good'
        WHEN gpa >= 3.0 THEN 'Average'
        ELSE 'Below Average'
    END AS gpa_category
FROM students;
```

### 3. Complex Joins with Multiple Conditions
```sql
-- Students and their current semester courses with grades
SELECT 
    s.first_name,
    s.last_name,
    c.course_name,
    c.semester,
    e.grade,
    c.credits
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE c.semester = 'Fall 2023'
    AND e.grade IS NOT NULL
ORDER BY s.last_name, c.course_name;
```

### 4. INSERT INTO ... SELECT
```sql
-- Create a summary table of student performance
CREATE TABLE student_summary AS
SELECT 
    s.student_id,
    s.first_name,
    s.last_name,
    COUNT(e.course_id) AS total_courses,
    AVG(CASE WHEN e.grade = 'A' THEN 4 WHEN e.grade = 'B' THEN 3 WHEN e.grade = 'C' THEN 2 WHEN e.grade = 'D' THEN 1 ELSE 0 END) AS gpa_calculated
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

### 5. UPDATE with Subquery
```sql
-- Update grades based on enrollment status
UPDATE enrollments
SET grade = 'F'
WHERE grade IS NULL AND enrollment_date < DATE_SUB(NOW(), INTERVAL 6 MONTH);
```

### 6. DELETE with JOIN
```sql
-- Remove students who haven't enrolled in any courses for 1 year
DELETE s
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
WHERE e.enrollment_id IS NULL
    AND s.enrollment_date < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

---

## Query Performance Tips

1. **Use indexes** on frequently searched columns (student_id, course_id)
2. **Avoid SELECT *** - only select needed columns
3. **Use EXPLAIN** to analyze query performance
4. **Use JOINs instead of subqueries** when possible
5. **Partition large tables** by date ranges
6. **Aggregate at the database level**, not in application

---

## Practice Exercises

1. Find all students enrolled in "Database Design" course
2. Calculate the average grade for each course (A=4, B=3, C=2, D=1)
3. Find students with the highest GPA in each department
4. List courses with fewer than 2 enrolled students
5. Find students who haven't taken any courses
6. Calculate total credits completed by each student
7. Find duplicate course enrollments (student taking same course twice)
8. Create a report of student performance by semester

---

**Created:** 2026-09-02  
**Version:** 1.0
