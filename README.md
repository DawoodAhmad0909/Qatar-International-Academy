# Qatar International Academy
## Overview 
## Objectives 
To design and implement a comprehensive School ERP database system using MySQL that efficiently manages student records, staff information, class schedules, attendance tracking, and exam results for streamlined academic administration.
## Database Creation
``` sql
CREATE DATABASE QIA_db;
USE QIA_db;
```
## Table Creation
### Table:academic_years
`` sql
CREATE TABLE academic_years(
    year_id     INT PRIMARY KEY AUTO_INCREMENT,
    year_name   VARCHAR(25),
    start_date  DATE,
    end_date    DATE,
    is_current  BOOLEAN
);

SELECT * FROM academic_years;
```
### Table:departments
``` sql
CREATE TABLE departments(
    dept_id      INT PRIMARY KEY AUTO_INCREMENT,
    dept_name    VARCHAR(50), 
    head_of_dept VARCHAR(50),
    location     TEXT
);

SELECT * FROM departments;
```
### Table:staff
``` sql
CREATE TABLE staff(
    staff_id    INT PRIMARY KEY AUTO_INCREMENT,
    first_name  VARCHAR(25),
    last_name   VARCHAR(25),
    gender      VARCHAR(25),
    dob         DATE,
    email       TEXT,
    phone       VARCHAR(15),
    address     TEXT,
    hire_date   DATE,
    dept_id     INT,
    position    VARCHAR(50),
    salary      DECIMAL(10,2),
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

SELECT * FROM staff;
```
### Table:classes
``` sql
CREATE TABLE classes(
    class_id          INT PRIMARY KEY AUTO_INCREMENT,
    class_name        VARCHAR(25),
    class_teacher_id  INT,
    academic_year_id  INT,
    room_number       VARCHAR(25),
    FOREIGN KEY (academic_year_id) REFERENCES academic_years(year_id)
);

SELECT * FROM classes;
```
### Table:students
``` sql
CREATE TABLE students(
    student_id     INT PRIMARY KEY AUTO_INCREMENT,
    first_name     VARCHAR(25),
    last_name      VARCHAR(25),
    gender         VARCHAR(25),
    dob            DATE,
    email          TEXT,
    phone          VARCHAR(15),
    address        TEXT,
    admission_date DATE,
    class_id       INT,
    parent_name    VARCHAR(50),
    parent_contact VARCHAR(15),
    FOREIGN KEY (class_id) REFERENCES classes(class_id)
);

SELECT * FROM students;
```
### Table:subjects
``` sql
CREATE TABLE subjects(
    subject_id   INT PRIMARY KEY AUTO_INCREMENT,
    subject_name VARCHAR(50),
    subject_code VARCHAR(25),
    dept_id      INT,
    description  TEXT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

SELECT * FROM subjects;
```
### Table:class_subjects
``` sql
CREATE TABLE class_subjects(
    class_subject_id INT PRIMARY KEY AUTO_INCREMENT,
    class_id         INT,
    subject_id       INT,
    teacher_id       INT,
    schedule_day     VARCHAR(15),
    schedule_time    TIME,
    FOREIGN KEY (class_id) REFERENCES classes(class_id),
    FOREIGN KEY (subject_id) REFERENCES subjects(subject_id)
); 

SELECT * FROM class_subjects;
```
### Table:attendance
``` sql
CREATE TABLE attendance(
    attendance_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id    INT,
    date          DATE,
    status        VARCHAR(25),
    remarks       TEXT,
    FOREIGN KEY (student_id) REFERENCES students(student_id)
);

SELECT * FROM attendance;
```
### Table:exams
``` sql
CREATE TABLE exams(
    exam_id           INT PRIMARY KEY AUTO_INCREMENT,
    exam_name         TEXT,
    academic_year_id  INT,
    start_date        DATE,
    end_date          DATE,
    FOREIGN KEY (academic_year_id) REFERENCES academic_years(year_id)
);

SELECT * FROM exams;
```
### Table:exam_results
``` sql
CREATE TABLE exam_results(
    result_id      INT PRIMARY KEY AUTO_INCREMENT,
    exam_id        INT,
    student_id     INT,
    subject_id     INT,
    marks_obtained DECIMAL(8,2),
    max_marks      INT DEFAULT 100,
    grade          VARCHAR(15),
    remarks        TEXT,
    FOREIGN KEY (exam_id) REFERENCES exams(exam_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (subject_id) REFERENCES subjects(subject_id)
);

SELECT * FROM exam_results;
```
## Key Queries 

#### 1. List all students with their class names and class teachers.
``` sql
SELECT 
        CONCAT(s.first_name,' ',s.last_name) AS Student_name,c.class_name,CONCAT(st.first_name,' ',st.last_name) AS Teacher_name
FROM students s 
LEFT JOIN classes c ON s.class_id=c.class_id
LEFT JOIN staff st ON st.staff_id=c.class_teacher_id;
```
#### 2.Find all teachers who teach more than one subject.
``` sql
SELECT 
        CONCAT(st.first_name,' ',st.last_name) AS Teacher_name,COUNT(DISTINCT cs.subject_id) AS Total_subjects
FROM staff st 
JOIN class_subjects cs ON st.staff_id=cs.teacher_id
GROUP BY Teacher_name
HAVING Total_subjects>1;
```
#### 3.Show the daily schedule for a specific class (include subject name, teacher, and time).
``` sql
SELECT 
        c.class_name,cs.schedule_day,cs.schedule_time,s.subject_name,CONCAT(st.first_name,' ',st.last_name) AS Teacher_name
FROM classes c 
JOIN class_subjects cs ON c.class_id=cs.class_id
JOIN subjects s ON s.subject_id=cs.subject_id
JOIN staff st ON st.staff_id=cs.teacher_id
WHERE c.class_id=1;
```
#### 4.Calculate the total number of absent students per day in a given week.
``` sql
SELECT
        date,COUNT(attendance_id) AS Total_absents
FROM attendance
WHERE 
        LOWER(status)='absent'
    AND date BETWEEN '2023-10-01' AND '2023-10-07'
GROUP BY date
ORDER BY date;
```
#### 5.List all students who scored above 90% in any subject during the last exam.
``` sql
SELECT 
        CONCAT(s.first_name,' ',s.last_name) AS Student_name,su.subject_name,
    er.marks_obtained,ROUND((er.marks_obtained*100.0)/er.max_marks,2) AS Percentage_marks
FROM exam_results er 
JOIN students s ON er.student_id=s.student_id
JOIN subjects su ON er.subject_id=su.subject_id
WHERE (er.marks_obtained*100.0)/er.max_marks>90;
```
#### 6.Find the average marks for each subject in a particular exam.
``` sql
SELECT 
        e.exam_name,su.subject_name,ROUND(AVG(er.marks_obtained),2) AS Average_marks
FROM exam_results er 
JOIN subjects su ON er.subject_id=su.subject_id
JOIN exams e ON er.exam_id=e.exam_id
GROUP BY e.exam_name,su.subject_name;
```
#### 7.Show all classes with their student count and class teacher's name.
``` sql
SELECT 
        c.class_name,CONCAT(st.first_name,' ',st.last_name) AS Teacher_name,COUNT(s.student_id) AS Total_students
FROM classes c 
LEFT JOIN students s ON c.class_id=s.class_id
LEFT JOIN staff st ON st.staff_id=c.class_teacher_id
GROUP BY c.class_name,Teacher_name;
```
#### 8.List students who were absent more than 3 times in a month.
``` sql
SELECT 
        CONCAT(s.first_name,' ',s.last_name) AS Student_name,COUNT(a.attendance_id) AS Total_absents,
    MONTHNAME(a.date) AS Month,YEAR(a.date) AS Year
FROM attendance a 
JOIN students s ON s.student_id=a.student_id
WHERE LOWER(a.status)='absent'
GROUP BY Student_name,Month,Year
HAVING Total_absents>3;
```
#### 9.Find teachers who don't have any assigned classes.
``` sql
SELECT 
        CONCAT(first_name,' ',last_name) AS Teacher_name,position
FROM staff 
WHERE 
        LOWER(position) LIKE '%teacher%'
    AND staff_id NOT IN (
                SELECT DISTINCT teacher_id FROM class_subjects
        WHERE teacher_id IS NOT NULL
        );
```
#### 10.Display the top 5 performing students across all subjects in the latest exam.
``` sql
WITH latest_exam AS (
        SELECT exam_id
    FROM exams 
    ORDER BY start_date DESC 
    LIMIT 2
    ),
student_stats AS (
        SELECT er.student_id,SUM(er.marks_obtained) AS Total_marks
    FROM exam_results er 
    JOIN latest_exam le ON er.exam_id=le.exam_id
    GROUP BY er.student_id
    )
SELECT 
        CONCAT(s.first_name,' ',s.last_name) AS Student_name,ss.Total_marks
FROM student_stats ss 
JOIN students s ON ss.student_id=s.student_id
ORDER BY ss.Total_marks DESC 
LIMIT 5;
```
#### 11.Calculate the percentage of present vs. absent students for each class.
``` sql
SELECT 
    c.class_name,
    ROUND((SUM(CASE WHEN a.status = 'Present' THEN 1 ELSE 0 END) /
    COUNT(a.student_id)) * 100, 2) AS present_percentage,
    ROUND((SUM(CASE WHEN a.status = 'Absent' OR a.status = 'Late' THEN 1 ELSE 0 END) / 
    COUNT(a.student_id)) * 100, 2) AS absent_late_percentage
FROM 
    classes c
JOIN 
    students s ON c.class_id = s.class_id
LEFT JOIN 
    attendance a ON s.student_id = a.student_id
GROUP BY 
    c.class_id, c.class_name
ORDER BY 
    c.class_name;
```
#### 12.List all subjects that don't have any assigned teachers.
``` sql
SELECT 
    su.subject_name,
    su.subject_code,
    d.dept_name
FROM subjects su
JOIN departments d ON su.dept_id = d.dept_id
WHERE su.subject_id NOT IN (
    SELECT DISTINCT subject_id FROM class_subjects
);
```
#### 13.Find students who are performing below average in more than two subjects.
``` sql
WITH subject_avg AS (
    SELECT 
        subject_id,
        AVG(marks_obtained) AS avg_marks
    FROM exam_results
    GROUP BY subject_id
)        ,
below_avg AS (
    SELECT 
        er.student_id,
        er.subject_id,
        er.marks_obtained,
        sa.avg_marks
    FROM exam_results er
    JOIN subject_avg sa ON er.subject_id = sa.subject_id
    WHERE er.marks_obtained < sa.avg_marks
        ),
below_avg_count AS (
    SELECT 
        student_id,
        COUNT(*) AS below_avg_subjects
    FROM below_avg
    GROUP BY student_id
    HAVING COUNT(*) > 2
        )
SELECT 
    CONCAT(s.first_name,' ',s.last_name) AS Student_name,
    b.below_avg_subjects
FROM below_avg_count b
JOIN students s ON b.student_id = s.student_id
ORDER BY b.below_avg_subjects DESC;
```
#### 14.Show the complete timetable for a specific teacher.
``` sql
SELECT 
        CONCAT(st.first_name,' ',st.last_name) AS Teacher_name,
    cs.schedule_day,cs.schedule_time,c.class_name,su.subject_name
FROM class_subjects cs
JOIN staff st ON cs.teacher_id=st.staff_id
JOIN classes c ON cs.class_id=c.class_id
JOIN subjects su ON cs.subject_id=su.subject_id
WHERE st.staff_id=3
ORDER BY Teacher_name;
```
#### 15.List all exams scheduled for the current academic year with their date ranges.
``` sql
SELECT         e.exam_name,ay.year_name,e.start_date,e.start_date,ay.is_current
FROM exams e 
JOIN academic_years ay ON ay.year_id=e.academic_year_id
WHERE ay.is_current=TRUE
ORDER BY e.start_date;
```

