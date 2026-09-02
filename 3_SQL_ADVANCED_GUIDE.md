# SQL Advanced Guide - Mastering Complex Queries

## Table of Contents
1. [Window Functions](#window-functions)
2. [Common Table Expressions (CTEs)](#common-table-expressions-ctes)
3. [Recursive Queries](#recursive-queries)
4. [Transactions & ACID](#transactions--acid)
5. [Indexes & Performance](#indexes--performance)
6. [Query Optimization](#query-optimization)
7. [Stored Procedures & Functions](#stored-procedures--functions)
8. [Triggers](#triggers)
9. [Advanced Data Manipulation](#advanced-data-manipulation)
10. [Complex Scenarios](#complex-scenarios)

---

## Window Functions

### What are Window Functions?

Window functions perform calculations across a set of rows related to the current row, without collapsing the result set.

**Key Formula:** `FUNCTION() OVER (PARTITION BY ... ORDER BY ...)`

### ROW_NUMBER - Unique Sequential Number

```sql
-- Rank all students by GPA within each department
SELECT 
    first_name,
    last_name,
    department,
    gpa,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY gpa DESC) AS rank
FROM students;
```

**Result:**
| first_name | last_name | department | gpa | rank |
|---|---|---|---|---|
| Emily | Davis | Computer Science | 3.9 | 1 |
| John | Smith | Computer Science | 3.8 | 2 |
| Sarah | Johnson | Computer Science | 3.6 | 3 |
| Lisa | Anderson | Engineering | 3.7 | 1 |
| Michael | Brown | Engineering | 3.4 | 2 |

### RANK & DENSE_RANK - Ranking with Ties

```sql
-- RANK skips numbers after ties
SELECT 
    first_name,
    gpa,
    RANK() OVER (ORDER BY gpa DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY gpa DESC) AS dense_rank
FROM students;
```

**Example Result with ties:**
| first_name | gpa | rank | dense_rank |
|---|---|---|---|
| Emily | 3.9 | 1 | 1 |
| John | 3.8 | 2 | 2 |
| Sarah | 3.6 | 3 | 3 |
| Lisa | 3.7 | 4 | 4 | ← Same as next, but different ranks
| Michael | 3.7 | 4 | 4 | ← Tied at 3.7

**RANK:** 1, 2, 3, 4, 4, 6 (skips 5)  
**DENSE_RANK:** 1, 2, 3, 4, 4, 5 (no skips)

### LAG & LEAD - Access Previous/Next Row

```sql
-- Compare each student's GPA with previous student (sorted by enrollment date)
SELECT 
    first_name,
    enrollment_date,
    gpa,
    LAG(gpa) OVER (ORDER BY enrollment_date) AS previous_gpa,
    LEAD(gpa) OVER (ORDER BY enrollment_date) AS next_gpa
FROM students;
```

**Result:**
| first_name | enrollment_date | gpa | previous_gpa | next_gpa |
|---|---|---|---|---|
| John | 2023-09-01 | 3.8 | NULL | 3.6 |
| Sarah | 2023-09-01 | 3.6 | 3.8 | 3.4 |
| Michael | 2023-09-01 | 3.4 | 3.6 | 3.2 |

### FIRST_VALUE & LAST_VALUE - Get First/Last in Window

```sql
-- Show each student with highest and lowest GPA in their department
SELECT 
    first_name,
    department,
    gpa,
    FIRST_VALUE(gpa) OVER (PARTITION BY department ORDER BY gpa DESC) AS highest_in_dept,
    LAST_VALUE(gpa) OVER (PARTITION BY department ORDER BY gpa DESC 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS lowest_in_dept
FROM students;
```

### Running Totals & Cumulative Sums

```sql
-- Calculate running total of credits
SELECT 
    first_name,
    enrollment_date,
    credits,
    SUM(credits) OVER (ORDER BY enrollment_date) AS cumulative_credits
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id;
```

### NTILE - Divide into Buckets

```sql
-- Divide students into 4 quartiles by GPA
SELECT 
    first_name,
    gpa,
    NTILE(4) OVER (ORDER BY gpa DESC) AS quartile
FROM students;
```

**Result:** Quartile 1 (top 25%), 2, 3, 4 (bottom 25%)

---

## Common Table Expressions (CTEs)

### Basic CTE (WITH Clause)

```sql
-- Step 1: Create temporary named result set
-- Step 2: Use it in main query
WITH top_students AS (
    SELECT student_id, first_name, gpa
    FROM students
    WHERE gpa > 3.7
)
SELECT * FROM top_students;
```

### Multiple CTEs

```sql
WITH high_achievers AS (
    SELECT student_id, first_name, gpa
    FROM students
    WHERE gpa >= 3.8
),
course_loads AS (
    SELECT 
        student_id,
        COUNT(course_id) AS course_count
    FROM enrollments
    GROUP BY student_id
)
SELECT 
    h.first_name,
    h.gpa,
    c.course_count
FROM high_achievers h
LEFT JOIN course_loads c ON h.student_id = c.student_id;
```

### Complex CTE with Aggregation

```sql
-- Department statistics
WITH dept_stats AS (
    SELECT 
        department,
        COUNT(*) AS student_count,
        AVG(gpa) AS avg_gpa,
        MAX(gpa) AS max_gpa,
        MIN(gpa) AS min_gpa
    FROM students
    GROUP BY department
)
SELECT 
    department,
    student_count,
    ROUND(avg_gpa, 2) AS avg_gpa,
    max_gpa,
    min_gpa,
    ROUND(max_gpa - min_gpa, 2) AS gpa_range
FROM dept_stats
WHERE student_count > 1
ORDER BY avg_gpa DESC;
```

---

## Recursive Queries

### Recursive CTE - Hierarchical Data

```sql
-- Find all employees and their management hierarchy
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Start with employees (no manager)
    SELECT 
        employee_id,
        name,
        manager_id,
        1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: Get subordinates
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        eh.level + 1
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy
ORDER BY level, name;
```

**Result:**
| employee_id | name | manager_id | level |
|---|---|---|---|
| 1 | CEO | NULL | 1 |
| 2 | CTO | 1 | 2 |
| 3 | Dev Lead | 2 | 3 |
| 4 | Developer | 3 | 4 |

### Recursive CTE - Generate Series

```sql
-- Generate all dates in a range
WITH RECURSIVE date_series AS (
    SELECT '2024-01-01' AS date
    
    UNION ALL
    
    SELECT DATE_ADD(date, INTERVAL 1 DAY)
    FROM date_series
    WHERE date < '2024-01-31'
)
SELECT * FROM date_series;
```

---

## Transactions & ACID

### Basic Transaction

```sql
-- ACID: Atomicity, Consistency, Isolation, Durability
START TRANSACTION;

-- Multiple related changes
UPDATE students SET gpa = 3.5 WHERE student_id = 1;
UPDATE enrollments SET grade = 'A' WHERE student_id = 1;
INSERT INTO audit_log VALUES (1, 'Updated student', NOW());

-- Commit all changes or rollback if error
COMMIT;  -- Save all changes
-- or ROLLBACK;  -- Undo all changes
```

### Savepoints

```sql
START TRANSACTION;

INSERT INTO students (first_name, last_name) VALUES ('Alice', 'Wonder');
SAVEPOINT before_course;

INSERT INTO enrollments (student_id, course_id) VALUES (1, 101);
-- Error occurs here
ROLLBACK TO before_course;  -- Undo only enrollment insertion
COMMIT;  -- Student still inserted
```

### Transaction Isolation Levels

```sql
-- Set isolation level
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- Levels: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE

START TRANSACTION;
SELECT * FROM students WHERE gpa > 3.5;
COMMIT;
```

---

## Indexes & Performance

### Creating Indexes

```sql
-- Single column index (speeds up searches)
CREATE INDEX idx_gpa ON students(gpa);

-- Unique index
CREATE UNIQUE INDEX idx_email ON students(email);

-- Composite (multi-column) index
CREATE INDEX idx_dept_gpa ON students(department, gpa);

-- Full-text index for text searches
CREATE FULLTEXT INDEX idx_bio ON students(bio);
```

### Index Best Practices

```sql
-- ✅ Good: Columns frequently used in WHERE, JOIN, ORDER BY
CREATE INDEX idx_student_id ON enrollments(student_id);
CREATE INDEX idx_course_dept ON courses(department_id);

-- ❌ Bad: Too many indexes (slows inserts), unused columns
-- ❌ Bad: Indexing columns with low selectivity (many duplicates)
```

### Viewing & Dropping Indexes

```sql
-- See all indexes on a table
SHOW INDEX FROM students;

-- Drop index
DROP INDEX idx_gpa ON students;
```

---

## Query Optimization

### EXPLAIN - Analyze Query Performance

```sql
-- Before optimization
EXPLAIN SELECT 
    s.first_name, 
    c.course_name
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE s.gpa > 3.5;
```

**Output explains:**
- `type`: Query type (ALL=slow, index=good, const=best)
- `rows`: Estimated rows examined
- `key`: Which index used
- `Extra`: Additional info

### Query Optimization Techniques

```sql
-- ❌ SLOW: SELECT * with JOIN
SELECT * FROM students s
JOIN enrollments e ON s.student_id = e.student_id;

-- ✅ FAST: Select only needed columns
SELECT s.first_name, e.grade FROM students s
JOIN enrollments e ON s.student_id = e.student_id;

-- ❌ SLOW: Subquery in SELECT (runs for each row)
SELECT s.first_name, (SELECT COUNT(*) FROM enrollments WHERE student_id = s.student_id)
FROM students s;

-- ✅ FAST: Use JOIN instead
SELECT s.first_name, COUNT(e.course_id) AS course_count
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id;

-- ❌ SLOW: LIKE with leading wildcard
SELECT * FROM students WHERE first_name LIKE '%john%';

-- ✅ FAST: LIKE with trailing wildcard (uses index)
SELECT * FROM students WHERE first_name LIKE 'john%';
```

---

## Stored Procedures & Functions

### Stored Procedure - Reusable SQL Block

```sql
-- Create procedure
DELIMITER $$

CREATE PROCEDURE get_student_info(IN student_id_param INT)
BEGIN
    SELECT 
        s.first_name,
        s.last_name,
        s.gpa,
        COUNT(e.course_id) AS courses_taken
    FROM students s
    LEFT JOIN enrollments e ON s.student_id = e.student_id
    WHERE s.student_id = student_id_param
    GROUP BY s.student_id;
END$$

DELIMITER ;

-- Call procedure
CALL get_student_info(1);
```

### Stored Procedure with Output Parameters

```sql
DELIMITER $$

CREATE PROCEDURE update_student_gpa(
    IN student_id_param INT,
    IN new_gpa DECIMAL(3,2),
    OUT update_status VARCHAR(50)
)
BEGIN
    IF new_gpa < 0 OR new_gpa > 4.0 THEN
        SET update_status = 'ERROR: GPA out of range';
    ELSE
        UPDATE students SET gpa = new_gpa WHERE student_id = student_id_param;
        SET update_status = 'SUCCESS: Updated';
    END IF;
END$$

DELIMITER ;

-- Call with output
CALL update_student_gpa(1, 3.9, @status);
SELECT @status;
```

### User-Defined Function

```sql
DELIMITER $$

CREATE FUNCTION calculate_grade_points(grade CHAR)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN CASE 
        WHEN grade = 'A' THEN 4
        WHEN grade = 'B' THEN 3
        WHEN grade = 'C' THEN 2
        WHEN grade = 'D' THEN 1
        ELSE 0
    END;
END$$

DELIMITER ;

-- Use function in queries
SELECT 
    s.first_name,
    e.grade,
    calculate_grade_points(e.grade) AS points
FROM students s
JOIN enrollments e ON s.student_id = e.student_id;
```

---

## Triggers

### Trigger - Automatic Action on INSERT

```sql
DELIMITER $$

CREATE TRIGGER after_student_insert
AFTER INSERT ON students
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (action, description, timestamp)
    VALUES ('INSERT', CONCAT('New student: ', NEW.first_name), NOW());
END$$

DELIMITER ;
```

### Trigger - Automatic Action on UPDATE

```sql
DELIMITER $$

CREATE TRIGGER before_gpa_update
BEFORE UPDATE ON students
FOR EACH ROW
BEGIN
    IF NEW.gpa < OLD.gpa THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'GPA cannot decrease!';
    END IF;
END$$

DELIMITER ;
```

### Trigger - Prevent Deletion

```sql
DELIMITER $$

CREATE TRIGGER prevent_course_delete
BEFORE DELETE ON courses
FOR EACH ROW
BEGIN
    IF EXISTS (SELECT 1 FROM enrollments WHERE course_id = OLD.course_id) THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Cannot delete course with enrollments!';
    END IF;
END$$

DELIMITER ;
```

---

## Advanced Data Manipulation

### Bulk Insert with INSERT...SELECT

```sql
-- Copy data from one table to another
INSERT INTO student_archive (first_name, last_name, gpa)
SELECT first_name, last_name, gpa
FROM students
WHERE gpa < 2.0;
```

### UPDATE with JOIN

```sql
-- Update all students in CS department to have minimum GPA
UPDATE students s
INNER JOIN departments d ON s.department = d.department_name
SET s.gpa = 3.0
WHERE d.department_name = 'Computer Science' AND s.gpa < 3.0;
```

### DELETE with Subquery

```sql
-- Remove enrollments older than 2 years
DELETE FROM enrollments
WHERE enrollment_date < DATE_SUB(CURDATE(), INTERVAL 2 YEAR);

-- Remove students with no enrollments
DELETE FROM students
WHERE student_id NOT IN (
    SELECT DISTINCT student_id FROM enrollments
);
```

### Batch Operations

```sql
-- More efficient than individual inserts
INSERT INTO enrollments (student_id, course_id) VALUES
(1, 101),
(1, 102),
(2, 101),
(2, 103),
(3, 102);
```

---

## Complex Scenarios

### Student Performance Dashboard

```sql
WITH student_stats AS (
    SELECT 
        s.student_id,
        s.first_name,
        s.last_name,
        s.department,
        s.gpa,
        COUNT(DISTINCT e.course_id) AS total_courses,
        COUNT(DISTINCT CASE WHEN e.grade IS NOT NULL THEN e.course_id END) AS completed_courses,
        AVG(CASE WHEN e.grade = 'A' THEN 4 
                 WHEN e.grade = 'B' THEN 3 
                 WHEN e.grade = 'C' THEN 2 
                 WHEN e.grade = 'D' THEN 1 ELSE 0 END) AS avg_points,
        ROW_NUMBER() OVER (PARTITION BY s.department ORDER BY s.gpa DESC) AS rank_in_dept
    FROM students s
    LEFT JOIN enrollments e ON s.student_id = e.student_id
    GROUP BY s.student_id, s.first_name, s.last_name, s.department, s.gpa
)
SELECT 
    first_name,
    last_name,
    department,
    gpa,
    total_courses,
    completed_courses,
    ROUND(avg_points, 2) AS avg_grade_points,
    CONCAT('Rank #', rank_in_dept, ' in ', department) AS ranking
FROM student_stats
WHERE total_courses > 0
ORDER BY department, rank_in_dept;
```

### Grade Distribution by Course

```sql
WITH grade_counts AS (
    SELECT 
        c.course_id,
        c.course_name,
        COUNT(*) AS total_students,
        COUNT(CASE WHEN e.grade = 'A' THEN 1 END) AS count_a,
        COUNT(CASE WHEN e.grade = 'B' THEN 1 END) AS count_b,
        COUNT(CASE WHEN e.grade = 'C' THEN 1 END) AS count_c,
        COUNT(CASE WHEN e.grade = 'D' THEN 1 END) AS count_d,
        COUNT(CASE WHEN e.grade IS NULL THEN 1 END) AS incomplete
    FROM courses c
    LEFT JOIN enrollments e ON c.course_id = e.course_id
    GROUP BY c.course_id, c.course_name
)
SELECT 
    course_name,
    total_students,
    count_a,
    ROUND(count_a * 100.0 / total_students, 1) AS pct_a,
    count_b,
    ROUND(count_b * 100.0 / total_students, 1) AS pct_b,
    count_c,
    ROUND(count_c * 100.0 / total_students, 1) AS pct_c,
    count_d,
    ROUND(count_d * 100.0 / total_students, 1) AS pct_d,
    incomplete,
    ROUND(incomplete * 100.0 / total_students, 1) AS pct_incomplete
FROM grade_counts
ORDER BY course_name;
```

### Course Enrollment Trends

```sql
WITH monthly_enrollment AS (
    SELECT 
        YEAR(enrollment_date) AS enrollment_year,
        MONTH(enrollment_date) AS enrollment_month,
        c.course_id,
        c.course_name,
        COUNT(*) AS new_enrollments,
        SUM(COUNT(*)) OVER (
            PARTITION BY c.course_id 
            ORDER BY YEAR(enrollment_date), MONTH(enrollment_date)
        ) AS cumulative_enrollments
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    GROUP BY enrollment_year, enrollment_month, c.course_id, c.course_name
)
SELECT 
    enrollment_year,
    enrollment_month,
    course_name,
    new_enrollments,
    cumulative_enrollments
FROM monthly_enrollment
ORDER BY enrollment_year, enrollment_month, course_name;
```

---

## Key Advanced Concepts

✅ **Window functions compute values without reducing rows**

✅ **CTEs make complex queries readable and reusable**

✅ **Recursive CTEs handle hierarchical data**

✅ **Transactions ensure data consistency**

✅ **Indexes speed up queries but slow down inserts**

✅ **Stored procedures encapsulate business logic**

✅ **Triggers enforce rules automatically**

✅ **Query optimization is essential for scalability**

---

## Next Steps
Graduate to **Expert Guide** for:
- Query execution plans deep dive
- Sharding & partitioning strategies
- Database replication
- Normalization & denormalization
- Advanced security & encryption
- Performance tuning at scale

---

**Difficulty Level:** ⭐⭐⭐ Advanced  
**Estimated Learning Time:** 15-20 hours  
**Prerequisites:** Intermediate SQL mastery  
**Created:** 2026-09-02
