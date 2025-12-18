# 🎓 Online Course Management System using SQL 

![Online Course Management System logo](https://github.com/sejaladle12/Online_Course_Management_System_Sql_Project/blob/main/jpg.jpg)

## 📌 Overview
The Online Course Management System (OCMS) is a PostgreSQL-based database project designed to manage students, courses, instructors, and enrollments for an online learning platform.
The project focuses on implementing real-world database operations such as automation, auditing, analytics, performance optimization, and role-based security using SQL.

## 🎯 Objective
- Design a relational database for an online learning system
- Manage student enrollments and course data efficiently
- Implement advanced SQL queries and joins
- Automate tasks using functions, triggers, and procedures
- Track course price changes using audit logs
- Improve performance using indexes
- Apply role-based database security

## 🛠️ Technologies Used
- Database: PostgreSQL / MySQL
- Language: SQL (PL/pgSQL)
- Tools: pgAdmin / psql

## 🏗️ Dtabase Schema
```sql
create table courses(
course_id int primary key,
title varchar(50),
category varchar(50),
instructor_id int,
price int
)

create table enrollments(
enrollment_id int primary key,
student_id int,
course_id int,
enrollment_date date
)

create table instructors(
instructor_id int primary key,
name varchar(50),
email varchar(100),
expertise varchar(50) 
)

create table students(
student_id int primary key,
name varchar(50),
email varchar(100),
city varchar(50)
)

create table course_price_audit (
audit_id serial primary key,
course_id int,
old_price int,
new_price int,
changed_on timestamp default current_timestamp
)
```

## 🔍 SQL Solutions
## (Part 1: Basic Queries)
### 1.Retrieve the title and price of all courses in the 'Data Science' category that cost less than 2000.Order the results from the cheapest to the most expensive.
 ```sql
select title,price from courses where category = 'Data Science' and price < 2000 order by price asc;
```

### 2.Determine the number of courses offered in each distinct category.
```sql
select category, count(*) as total_courses from courses group by category;
```

### 3.Identify the categories where the total number of enrollments is greater than 150.Display the category name and the total enrollment count.
```sql
select c.category, count(e.enrollment_id) as total_enrollments from courses as c
inner join enrollments as e on c.course_id = e.course_id group by c.category having count (e.enrollment_id) > 150;
```

### 4.Find the top 5 most expensive courses. Display the course title and its price.
```sql
select title,price from courses order by price desc limit 5;
```

### 5.List the 6th through 10th students (by ascending student_id) who live in 'London'.
```sql
select student_id,name,city from students where city = 'London' order by student_id asc offset 5 limit 5;
```

## (Part 2: Advanced Queries)
### 6.List the title of every course along with the name of the instructor who teaches it.
```sql
select c.title,i.name as instructor_name from courses as c inner join instructors as i on c.instructor_id = i.instructor_id;
```

### 7.Display the name of every instructor. If an instructor is teaching any courses, also list the title of their courses. If an instructor is not teaching any courses, their name should still appear with blank or NULL course title.
```sql
select i.name, c.title from instructors i left join courses c on i.instructor_id = c.instructor_id;
```

### 8.Find the student name, course title, and the instructor name for all enrollments made by students living in the city 'New York'.
```sql
select s.name as student_name,c.title as course_title,i.name as instructor_name from students as s 
inner join enrollments as e on s.student_id = e.student_id
inner join courses as c on e.course_id = c.course_id
inner join instructors as i on c.instructor_id = i.instructor_id;
```

### 9.Find the name and city of all students who are enrolled in any course with the category of 'Cloud'.
```sql
select s.name,s.city from students as s 
inner join enrollments as e on s.student_id = e.student_id
inner join courses as c on e.course_id = c.course_id where category = 'Cloud';
```

### 10.Retrieve the title and price of all courses that are more expensive than the average price of all courses in the entire catalog.
```sql
select title, price from courses where price > (select avg(price) from courses);
```

### 11.Identify the name and expertise of the instructor who teaches the highest number of courses.
```sql
select i.name, i.expertise from instructors as i
inner join courses as c on i.instructor_id = c.instructor_id
GROUP BY i.instructor_id ORDER BY COUNT(c.course_id) DESC LIMIT 1;
```

### 12.Find all unique names that appear in either the students table or the instructors table.
```sql
select name from students
UNION
select name from instructors;
```

### 13.Find the course_id values that have at least one enrollment recorded in both the year 2024 and the year 2025.
```sql
select course_id from enrollments
where extract(year from enrollment_date) in (2024, 2025)
group by  course_id having count(distinct extract(year from enrollment_date)) = 2;
```

### 14.Find the course_id values that have enrollments in the year 2025 but not in 2024.
```sql
select distinct course_id from enrollments
where extract(year from enrollment_date) = 2025 and course_id NOT IN(
select course_id from enrollments
where extract(year from enrollment_date) = 2024 );
```

### 15.For each course category, assign a rank to each course based on its price (highest price gets rank 1). The ranking should reset for each category. Display the course title, category, price, and the calculated rank.
```sql
select title, category, price,
rank() over (partition by category order by price desc) as price_rank from courses;
```

### 16.Calculate the difference between each course's price and the average price of all courses within its own category. Display the course title, price, category average price, and the calculated difference.
```sql
select title, price, category,
avg(price) over(partition by category) as category_avg_price,
price - avg(price) over (partition by category) as price_difference from courses;
```

## (Part 3: Database Management and Security) 
### 17.Write a PostgreSQL function called get_course_enrollment_count that accepts a course_id as an integer input and returns the total number of enrollments for that specific course as an integer output.
```sql
create or replace function get_course_enrollment_count(p_course_id INT)
returns INT as $$
declare
total_enrollments INT;
begin
select count(*) into total_enrollments from enrollments where course_id = p_course_id;
returns total_enrollments;
end;
$$ language plpgsql;
```

### 18.Write the DDL to create an audit table named course_price_audit.
```sql
create table course_price_audit (
audit_id serial primary key,
course_id int,
old_price int,
new_price int,
changed_on timestamp default current_timestamp
)
```

### 19.Create a trigger that automatically logs the course_id, old_price,and new_price into the course_price_audit table whenever the price of a course is updated in the courses table.
```sql
create or replace function logs_course_price_update()
returns trigger as $$
begin
insert into course_price_audit(course_id, old_price, new_price)
values(OLD.course_id, OLD.price, NEW.price);
return new;
end;
$$ language plpgsql;

create trigger trg_course_price_audit
after update on courses
for each row 
execute function logs_course_price_update();
```

### 20.Create a stored procedure (or a function with LANGUAGE plpgsql) called add_new_enrollment that inserts a new record intothe enrollments table using three parameters: p_student_id (INT), p_course_id (INT), and an optional p_enrollment_date (DATE).If no date is provided, the procedure must use the current date.
```sql
create or replace function add_new_enrollment(
p_student_id int,
p_course_id int,
p_enrollment_date date DEFAULT CURRENT_DATE)
returns VOID as $$
begin
insert into enrollments(student_id, course_id, enrollment_date)
values(p_student_id, p_course_id, p_enrollment_date);
end;
$$ language plpgsql;
```

### 21.Calculate the number of months that have passed between the earliest enrollment date recorded in the system and the current date.
```sql
select (DATE_PART('year', AGE(CURRENT_DATE, MIN(enrollment_date)))) * 12 +
(DATE_PART('month', AGE(CURRENT_DATE, MIN(enrollment_date)))) as total_months from enrollments;
```

### 22.Find the name of all students whose name contains 'Jackson' (case-insensitive). Display their name in uppercase and their email address.
```sql
select upper(name) as name, email from students where name ILIKE '%jackson%';
```

### 23.Create a new PostgreSQL user role named auditor. Grant this role SELECT access on the enrollments table. Revoke the SELECT privilege on the city column of the students table from the auditor role.
```sql
create role auditor;

grant select on enrollments to auditor;

revoke select (city) on students from auditor;
```

### 24.Create a view named v_instructor_performance that shows the instructor name, the title of the courses they teach, and the total number of enrollments for each of those courses.
```sql
create view v_instructor_performance as select 
    i.name AS instructor_name,
    c.title AS course_title,
    count(e.enrollment_id) as total_enrollments
from instructors i
inner join courses as c on i.instructor_id = c.instructor_id
left join enrollments as e on c.course_id = e.course_id
group by i.name, c.title;
```

### 25.Create a non-unique index on the category column of the courses table.Briefly explain the performance benefit of creating this index.
```sql
create index idx_courses_category on courses(category);
```

## 📂 Project Structure
 Online Course Management System
 │
 ├── schema/
 │   ├── tables.sql
 │   ├── functions.sql
 │   ├── triggers.sql
 │   ├── views.sql
 │   └── indexes.sql
 │
 ├── queries/
 │   ├── basic_queries.sql
 │   ├── advanced_queries.sql
 │   └── security.sql
 │
 ├── README.md

## 🚀 How to Run
- Create a PostgreSQL database
- Execute SQL files from the schema/ folder
- Run query files from the queries/ folder to test functionality

## 📚 Learning Outcomes
- Strong understanding of relational database design
- Hands-on experience with PostgreSQL automation features
- Improved SQL performance tuning skills
- Practical knowledge of database security

## ✅ Conclusion
The Online Course Management System project demonstrates practical use of PostgreSQL for managing courses, students, and enrollments using advanced SQL features. It strengthens understanding of database automation, performance optimization, and security in real-world scenarios.

## 👩‍💻 Author
Sejal Adle
🔗GitHub: https:https://github.com/sejaladle12








