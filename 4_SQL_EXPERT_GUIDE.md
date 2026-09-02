# SQL Expert Guide - Enterprise-Level Database Mastery

## Table of Contents
1. [Query Execution Plans](#query-execution-plans)
2. [Database Normalization](#database-normalization)
3. [Sharding & Partitioning](#sharding--partitioning)
4. [Replication & High Availability](#replication--high-availability)
5. [Security & Access Control](#security--access-control)
6. [Advanced Indexing Strategies](#advanced-indexing-strategies)
7. [Denormalization & Data Warehouse Design](#denormalization--data-warehouse-design)
8. [Backup & Disaster Recovery](#backup--disaster-recovery)
9. [Monitoring & Performance Tuning](#monitoring--performance-tuning)
10. [Scaling Strategies](#scaling-strategies)

---

## Query Execution Plans

### Understanding EXPLAIN ANALYZE

```sql
-- Detailed execution plan with actual runtime statistics
EXPLAIN ANALYZE
SELECT 
    s.first_name,
    c.course_name,
    e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE s.gpa > 3.5;
```

**Output Interpretation:**

```
Planning Time: 0.125 ms
Execution Time: 2.456 ms

→ Nested Loop (cost=100.00..500.00 rows=50 width=100)
  → Hash Join (cost=50.00..200.00 rows=30)
    → Seq Scan on students s (cost=0.00..50.00 rows=6)
          Filter: (gpa > 3.5)
    → Hash (cost=20.00..20.00 rows=100)
      → Seq Scan on enrollments e (cost=0.00..20.00 rows=100)
  → Index Scan using idx_course_id on courses c
```

**Key Metrics:**

| Metric | Meaning | Good | Bad |
|--------|---------|------|-----|
| cost | Relative expense | Lower | Higher |
| rows | Estimated rows | Accurate | Way off |
| width | Bytes per row | Smaller | Larger |
| Seq Scan | Full table scan | For small tables | For large tables |
| Index Scan | Using index | Efficient | Missing index |

### Index Usage Analysis

```sql
-- Check if query uses indexes
EXPLAIN 
SELECT * FROM students WHERE gpa > 3.5;

-- With index: Index Scan
-- Without index: Seq Scan (slow on large tables)

-- Create index to improve
CREATE INDEX idx_gpa ON students(gpa);

-- Re-run EXPLAIN to verify index usage
EXPLAIN 
SELECT * FROM students WHERE gpa > 3.5;
```

### Query Plan Optimization

```sql
-- ❌ BAD: Seq Scan on all enrollments
EXPLAIN
SELECT s.first_name
FROM students s
WHERE s.student_id IN (SELECT student_id FROM enrollments);

-- ✅ GOOD: Use join instead
EXPLAIN
SELECT DISTINCT s.first_name
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id;

-- ✅ GOOD: Use EXISTS (stops after first match)
EXPLAIN
SELECT s.first_name
FROM students s
WHERE EXISTS (SELECT 1 FROM enrollments WHERE student_id = s.student_id);
```

---

## Database Normalization

### Normalization Levels (1NF to BCNF)

#### First Normal Form (1NF) - Atomic Values
- No repeating groups
- All columns contain single values

```sql
-- ❌ NOT 1NF: Multiple courses in one cell
CREATE TABLE student_courses_bad (
    student_id INT,
    first_name VARCHAR(50),
    courses VARCHAR(500)  -- 'DB101, CS101, CS102'
);

-- ✅ 1NF: One course per row
CREATE TABLE student_courses (
    student_id INT,
    first_name VARCHAR(50),
    course_id INT
);
```

#### Second Normal Form (2NF) - Remove Partial Dependencies
- Must be 1NF
- Non-key columns depend on entire primary key

```sql
-- ❌ NOT 2NF: instructor_name depends only on course_id, not (student_id, course_id)
CREATE TABLE enrollments_bad (
    student_id INT,
    course_id INT,
    instructor_name VARCHAR(50),  -- Depends on course_id only
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id)
);

-- ✅ 2NF: Move instructor to courses table
CREATE TABLE courses_normalized (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100),
    instructor_name VARCHAR(50)
);

CREATE TABLE enrollments_normalized (
    student_id INT,
    course_id INT,
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (course_id) REFERENCES courses_normalized(course_id)
);
```

#### Third Normal Form (3NF) - Remove Transitive Dependencies
- Must be 2NF
- Non-key columns don't depend on other non-key columns

```sql
-- ❌ NOT 3NF: GPA depends on student_id, but department_name depends on department_id
CREATE TABLE students_bad (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    department_id INT,
    department_name VARCHAR(50)  -- Transitive: student→dept→name
);

-- ✅ 3NF: Move department_name to separate table
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

CREATE TABLE students_normalized (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

#### Boyce-Codd Normal Form (BCNF)
- More strict than 3NF
- Every determinant must be a candidate key

```sql
-- BCNF: All determinants are candidate keys
CREATE TABLE professor_courses (
    professor_id INT,
    course_id INT,
    semester VARCHAR(20),
    room_id INT,
    PRIMARY KEY (professor_id, semester),
    UNIQUE (course_id, semester),
    UNIQUE (room_id, semester)
);
-- Each professor teaches one course per semester
-- Each course taught once per semester
-- Each room used once per semester
```

---

## Sharding & Partitioning

### Table Partitioning by Range

```sql
-- Partition students table by enrollment year
CREATE TABLE students (
    student_id INT,
    first_name VARCHAR(50),
    enrollment_date DATE,
    gpa DECIMAL(3, 2)
)
PARTITION BY RANGE(YEAR(enrollment_date)) (
    PARTITION p_2022 VALUES LESS THAN (2023),
    PARTITION p_2023 VALUES LESS THAN (2024),
    PARTITION p_2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

**Benefits:**
- Faster queries on specific year ranges
- Easier maintenance (drop old partitions)
- Parallel processing across partitions

### Partitioning by List

```sql
-- Partition students by department
CREATE TABLE students_by_dept (
    student_id INT,
    first_name VARCHAR(50),
    department VARCHAR(50)
)
PARTITION BY LIST COLUMNS(department) (
    PARTITION p_cs VALUES IN ('Computer Science', 'Engineering'),
    PARTITION p_business VALUES IN ('Business', 'Economics'),
    PARTITION p_arts VALUES IN ('Arts', 'Humanities')
);
```

### Horizontal Sharding (Multi-Server)

```sql
-- Customer database sharded by region
-- Shard 1 (US-East): customers 1-333,333
-- Shard 2 (US-West): customers 333,334-666,666
-- Shard 3 (Europe): customers 666,667-1,000,000

-- Determine shard:
SHARD_ID = customer_id % 3;

-- Applications route to correct shard database
-- Python example:
/*
shard_id = customer_id % 3
if shard_id == 0:
    db = connect('db_us_east')
elif shard_id == 1:
    db = connect('db_us_west')
else:
    db = connect('db_europe')
*/
```

---

## Replication & High Availability

### Master-Slave Replication Setup

```sql
-- On Master (Write Server):
-- Enable binary logging in my.cnf:
/*
[mysqld]
server-id = 1
log_bin = mysql-bin
binlog_do_db = school_db
*/

-- Create replication user
CREATE USER 'replication'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';

-- Get master status
SHOW MASTER STATUS;
-- File: mysql-bin.000003, Position: 154

-- On Slave (Read Server):
CHANGE MASTER TO
    MASTER_HOST = '192.168.1.100',
    MASTER_USER = 'replication',
    MASTER_PASSWORD = 'password',
    MASTER_LOG_FILE = 'mysql-bin.000003',
    MASTER_LOG_POS = 154;

START SLAVE;

-- Verify replication
SHOW SLAVE STATUS;
```

### Multi-Master Replication (Active-Active)

```sql
-- Both servers can accept writes
-- Must prevent conflicts with conflict resolution

-- Server 1:
SET GLOBAL auto_increment_offset = 1;
SET GLOBAL auto_increment_increment = 2;
-- IDs: 1, 3, 5, 7...

-- Server 2:
SET GLOBAL auto_increment_offset = 2;
SET GLOBAL auto_increment_increment = 2;
-- IDs: 2, 4, 6, 8...

-- Prevents ID collisions in multi-master setup
```

---

## Security & Access Control

### User Management

```sql
-- Create users with limited privileges
CREATE USER 'read_only'@'%' IDENTIFIED BY 'password';
CREATE USER 'app_user'@'app_server' IDENTIFIED BY 'password';
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'password';

-- Grant specific privileges
GRANT SELECT ON school_db.* TO 'read_only'@'%';
GRANT SELECT, INSERT, UPDATE ON school_db.students TO 'app_user'@'app_server';
GRANT ALL PRIVILEGES ON school_db.* TO 'admin'@'localhost';

-- Grant privileges on views (safer than table access)
GRANT SELECT ON school_db.student_summary TO 'read_only'@'%';

-- Revoke privileges
REVOKE INSERT ON school_db.students FROM 'app_user'@'app_server';

-- Remove user
DROP USER 'read_only'@'%';
```

### Row-Level Security (Application Level)

```sql
-- View for department coordinators (see only their dept)
CREATE VIEW dept_students AS
SELECT * FROM students
WHERE department = CURRENT_USER_DEPARTMENT();

-- Query as department user
GRANT SELECT ON school_db.dept_students TO 'coord_cs'@'%';

-- User 'coord_cs' can only see CS students
SELECT * FROM dept_students;
```

### SQL Injection Prevention

```sql
-- ❌ VULNERABLE: Direct string concatenation
SELECT * FROM students WHERE student_id = ' + userId + ';

-- ✅ SAFE: Parameterized queries
-- In application code (Python):
cursor.execute("SELECT * FROM students WHERE student_id = %s", (user_id,))

-- In SQL with placeholders:
PREPARE stmt FROM 'SELECT * FROM students WHERE student_id = ?';
SET @student_id = 5;
EXECUTE stmt USING @student_id;
```

### Data Encryption

```sql
-- Encrypt sensitive data at rest
CREATE TABLE students_secure (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    email VARCHAR(100),
    ssn VARBINARY(255),  -- Encrypted
    
    -- Encryption function (application or database)
    encrypted_gpa VARBINARY(255)
);

-- Insert encrypted data
INSERT INTO students_secure VALUES 
(1, 'John', 'john@uni.edu', 
 AES_ENCRYPT('123-45-6789', 'encryption_key'),
 AES_ENCRYPT('3.8', 'encryption_key'));

-- Decrypt when needed
SELECT 
    student_id,
    first_name,
    CAST(AES_DECRYPT(encrypted_gpa, 'encryption_key') AS DECIMAL(3,2)) AS gpa
FROM students_secure;
```

---

## Advanced Indexing Strategies

### Composite Index Optimization

```sql
-- Query: WHERE dept = 'CS' AND gpa > 3.5 ORDER BY enrollment_date
-- Composite index order matters!

-- ❌ Bad order
CREATE INDEX idx_bad ON students(gpa, department, enrollment_date);

-- ✅ Good order (Equality, Range, Sort)
CREATE INDEX idx_good ON students(department, gpa, enrollment_date);
-- department: Equality filter (narrows rows most)
-- gpa: Range filter (second filter)
-- enrollment_date: Sort column (no filtering needed)

-- With this index, query uses:
-- 1. Index to find dept='CS'
-- 2. Continue through index to find gpa>3.5
-- 3. Results already sorted by enrollment_date
```

### Covering Indexes (All Columns in Query)

```sql
-- Query needs: student_id, first_name, gpa (no other columns from table)
SELECT student_id, first_name, gpa
FROM students
WHERE department = 'CS' AND gpa > 3.5;

-- Index that covers entire query (no table lookup needed)
CREATE INDEX idx_covering ON students(department, gpa, student_id, first_name);

-- Index scan is enough - no need to read table
```

### Partial Indexes

```sql
-- Index only active students (smaller, faster)
CREATE INDEX idx_active_students ON students(gpa)
WHERE is_active = TRUE;

-- Query using partial index
SELECT * FROM students
WHERE is_active = TRUE AND gpa > 3.5;
-- Uses partial index (only 90% of rows)

-- Query NOT using partial index
SELECT * FROM students
WHERE gpa > 3.5;  -- Needs all rows, can't use partial index
```

---

## Denormalization & Data Warehouse Design

### Strategic Denormalization

```sql
-- Normalized (multiple joins needed)
SELECT 
    s.first_name,
    COUNT(e.course_id) AS course_count,
    AVG(CASE WHEN e.grade = 'A' THEN 4 ELSE 3 END) AS avg_grade
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id;

-- Denormalized (no joins)
CREATE TABLE student_dashboard (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    course_count INT,  -- Precomputed
    avg_grade DECIMAL(3,2)  -- Precomputed
);

-- Refresh denormalized data nightly
INSERT INTO student_dashboard
SELECT 
    s.student_id,
    s.first_name,
    COUNT(e.course_id),
    AVG(CASE WHEN e.grade = 'A' THEN 4 ELSE 3 END)
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id
ON DUPLICATE KEY UPDATE
    course_count = VALUES(course_count),
    avg_grade = VALUES(avg_grade);
```

### Fact & Dimension Tables (Star Schema)

```sql
-- Dimension tables (slowly changing)
CREATE TABLE dim_students (
    student_key INT PRIMARY KEY,
    student_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department VARCHAR(50)
);

CREATE TABLE dim_courses (
    course_key INT PRIMARY KEY,
    course_id INT,
    course_name VARCHAR(100),
    credits INT
);

CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,
    enrollment_date DATE,
    month VARCHAR(20),
    quarter INT,
    year INT
);

-- Fact table (events)
CREATE TABLE fact_enrollments (
    fact_key INT PRIMARY KEY,
    student_key INT,
    course_key INT,
    date_key INT,
    grade CHAR(1),
    FOREIGN KEY (student_key) REFERENCES dim_students(student_key),
    FOREIGN KEY (course_key) REFERENCES dim_courses(course_key),
    FOREIGN KEY (date_key) REFERENCES dim_date(date_key)
);

-- Star schema enables fast aggregations
SELECT 
    ds.department,
    COUNT(*) AS enrollments,
    AVG(CASE WHEN fe.grade = 'A' THEN 1 ELSE 0 END) AS pct_a
FROM fact_enrollments fe
JOIN dim_students ds ON fe.student_key = ds.student_key
GROUP BY ds.department;
```

---

## Backup & Disaster Recovery

### Full Database Backup

```sql
-- Logical backup (portable, human-readable)
mysqldump -u root -p school_db > backup_2024.sql

-- With all databases
mysqldump -u root -p --all-databases > full_backup_2024.sql

-- Restore from backup
mysql -u root -p school_db < backup_2024.sql
```

### Incremental Backup (Binary Log)

```sql
-- Enable binary logging (my.cnf)
/*
[mysqld]
log_bin = mysql-bin
expire_logs_days = 10
*/

-- Backup only changes since last backup
PURGE BINARY LOGS BEFORE '2024-01-15 00:00:00';

-- Recover to specific point in time
mysqlbinlog --start-datetime='2024-01-14 10:00:00' \
            --stop-datetime='2024-01-14 11:00:00' \
            mysql-bin.000001 | mysql -u root -p

-- Recover specific database only
mysqlbinlog --database=school_db mysql-bin.000001 | mysql -u root -p
```

### Recovery Time Objective (RTO) & Recovery Point Objective (RPO)

```sql
-- Setup for 1-hour RTO, 15-minute RPO

-- 1. Full backup every 12 hours
BACKUP DATABASE school_db TO '/backups/full_2024_01_14.bak';

-- 2. Incremental backups every 30 minutes
BACKUP DATABASE school_db 
       INCREMENTAL TO '/backups/incr_2024_01_14_1400.bak';

-- 3. Binary log replication every 5 minutes
-- (provides 15-min RPO through replication)

-- Recovery process:
-- 1. Restore latest full backup (12 hrs max)
-- 2. Apply incremental backups (30 min each)
-- 3. Apply binary logs (5 min each)
-- Total RTO: ~1 hour
```

---

## Monitoring & Performance Tuning

### Performance Metrics

```sql
-- Key metrics to monitor
SHOW STATUS LIKE 'Threads%';
-- Threads_connected: Active connections
-- Threads_running: Currently executing queries

SHOW STATUS LIKE 'Questions';
-- Total queries executed

SHOW STATUS LIKE 'Slow_queries';
-- Queries taking longer than long_query_time

-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- Log queries > 2 seconds

-- Monitor memory usage
SHOW STATUS LIKE 'Memory%';

-- Connection pool stats
SHOW STATUS LIKE 'Max_used_connections';
```

### Query Performance Profiling

```sql
-- Enable profiling
SET PROFILING = 1;

-- Run query
SELECT * FROM students WHERE gpa > 3.5;

-- View profiling results
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;

-- Detailed breakdown
SHOW PROFILE CPU, BLOCK IO FOR QUERY 1;
```

### Database Health Check

```sql
-- Check for table corruption
ANALYZE TABLE students;
CHECK TABLE students;
REPAIR TABLE students;

-- Optimize tables (reclaim space)
OPTIMIZE TABLE students;
OPTIMIZE TABLE enrollments;
OPTIMIZE TABLE courses;

-- View table statistics
SHOW TABLE STATUS FROM school_db;
```

---

## Scaling Strategies

### Horizontal Scaling - Read Replicas

```sql
-- Master handles writes
-- Replicas handle reads

-- Application routing:
/*
INSERT/UPDATE/DELETE → Master (192.168.1.10:3306)
SELECT → Slave 1 (192.168.1.11:3306)
SELECT → Slave 2 (192.168.1.12:3306)
SELECT → Slave 3 (192.168.1.13:3306)
*/

-- Load balancing reads across replicas
CREATE PROCEDURE get_student(IN student_id INT)
BEGIN
    -- Route to read replica
    SELECT * FROM students WHERE student_id = student_id;
END;

-- Application calls appropriate endpoint based on operation
```

### Vertical Scaling - Hardware Upgrade

```sql
-- Before: 32GB RAM, 8 CPU cores
-- After: 256GB RAM, 32 CPU cores

-- Configuration tuning for larger hardware
SET GLOBAL innodb_buffer_pool_size = 200G;
SET GLOBAL max_connections = 10000;
SET GLOBAL query_cache_size = 32G;

-- Monitor impact
SHOW STATUS LIKE 'Innodb_buffer_pool%';
```

### Federation (Splitting by Domain)

```sql
-- Original: Single database with all data
-- Federated: Separate databases by domain

-- Database 1: Academic data
CREATE DATABASE academic (students, courses, enrollments)

-- Database 2: Administrative data
CREATE DATABASE admin (users, permissions, logs)

-- Database 3: Financial data
CREATE DATABASE finance (payments, billing, scholarships)

-- Application queries appropriate database
-- academic_db.query("SELECT * FROM students")
-- admin_db.query("SELECT * FROM users")
-- finance_db.query("SELECT * FROM payments")
```

### Caching Strategy (Query Result Caching)

```sql
-- Cache frequently used queries

-- Option 1: Application-level caching (Redis)
/*
cache_key = hash("SELECT * FROM courses WHERE semester = ?", semester)
if cache.get(cache_key) exists:
    return cache.get(cache_key)
else:
    result = db.query("SELECT * FROM courses WHERE semester = ?", semester)
    cache.set(cache_key, result, ttl=3600)  -- 1 hour
    return result
*/

-- Option 2: Query cache (MySQL)
SET GLOBAL query_cache_type = 1;
SET GLOBAL query_cache_size = 268435456;  -- 256MB

-- Cache invalidated on writes
UPDATE students SET gpa = 3.9 WHERE student_id = 1;
-- Clears cache entries for students table

-- Monitor cache effectiveness
SHOW STATUS LIKE 'Qcache%';
```

---

## Enterprise Best Practices

### Data Governance

```sql
-- Audit trail table
CREATE TABLE audit_log (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(50),
    operation VARCHAR(10),  -- INSERT, UPDATE, DELETE
    old_values JSON,
    new_values JSON,
    changed_by VARCHAR(50),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_table_time (table_name, changed_at)
);

-- Trigger to log changes
DELIMITER $$
CREATE TRIGGER log_student_changes
AFTER UPDATE ON students
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, operation, old_values, new_values, changed_by)
    VALUES ('students', 'UPDATE', 
            JSON_OBJECT('gpa', OLD.gpa, 'name', OLD.first_name),
            JSON_OBJECT('gpa', NEW.gpa, 'name', NEW.first_name),
            CURRENT_USER());
END$$
DELIMITER ;
```

### Capacity Planning

```sql
-- Estimate storage needs
-- Current: 1M students, 500KB average per record = 500GB
-- Growth: 20% annually for 5 years

-- Year 1: 1.2M students = 600GB
-- Year 2: 1.44M students = 720GB
-- Year 3: 1.728M students = 864GB
-- Year 4: 2.07M students = 1.04TB
-- Year 5: 2.49M students = 1.25TB

-- Provision infrastructure with 30% headroom
-- Purchase: 1.25TB * 1.3 = 1.625TB ≈ 2TB
```

---

## Key Expert Concepts

✅ **Query plans reveal performance bottlenecks**

✅ **Normalization prevents data anomalies; denormalization speeds up reads**

✅ **Partitioning and sharding enable scalability**

✅ **Replication provides redundancy and read scaling**

✅ **Security requires multiple layers: encryption, access control, auditing**

✅ **Indexes must be strategically designed for workload**

✅ **Monitoring and profiling are essential for production systems**

✅ **Scaling is about balancing consistency, availability, and performance**

---

## Production Readiness Checklist

- [ ] Database backed up daily with tested restore procedures
- [ ] Query optimization: all common queries < 1 second
- [ ] Monitoring alerts for CPU, memory, disk, connections
- [ ] User access controls with principle of least privilege
- [ ] Slow query log enabled and reviewed weekly
- [ ] Replication configured for disaster recovery
- [ ] Security patches applied within 48 hours
- [ ] Capacity planning for 12-month growth
- [ ] Documentation of schema, procedures, and runbooks
- [ ] Regular disaster recovery drills (RTO/RPO validation)

---

## Resources for Continued Learning

- [ ] Study database architecture trade-offs
- [ ] Practice with large datasets (multi-GB)
- [ ] Learn about distributed databases (Cassandra, MongoDB)
- [ ] Master query optimization tools and techniques
- [ ] Explore data warehouse design patterns
- [ ] Study consistency models (ACID vs BASE)
- [ ] Practice high availability scenarios

---

**Difficulty Level:** ⭐⭐⭐⭐ Expert  
**Estimated Learning Time:** 20+ hours  
**Prerequisites:** Advanced SQL mastery + production experience recommended  
**Created:** 2026-09-02

---

## From Learning to Mastery

This expert guide covers enterprise-level database concepts. True mastery comes from:

1. **Experience** - Working with production systems under pressure
2. **Failures** - Learning what breaks and why
3. **Optimization** - Continuously improving existing systems
4. **Architecture** - Designing systems for scale and reliability
5. **Mentorship** - Teaching and learning from others

Keep learning, keep building, keep optimizing! 🚀
