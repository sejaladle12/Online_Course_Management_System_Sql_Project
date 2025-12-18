# Online Course Management System using SQL 

![Online Course Management System logo](https://github.com/sejaladle12/Online_Course_Management_System_Sql_Project/blob/main/jpg.jpg)

## Overview
The Online Learning Management System (OLMS) is a PostgreSQL-based database project that manages students, courses, instructors, and enrollments. It demonstrates real-world SQL concepts such as advanced queries, functions, triggers, views, indexing, and rDesign a relational database for an online learning system

## Objective
- Design a relational database for an online learning system
- Manage student enrollments and course data efficiently
- Implement advanced SQL queries and joins
- Automate tasks using functions, triggers, and procedures
- Track course price changes using audit logs
- Improve performance using indexes
- Apply role-based database security

## Dtabase Schema
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
```

