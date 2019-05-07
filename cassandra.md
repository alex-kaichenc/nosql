
# Cassandra

## Setup

**All Keyspace**

    DESC KEYSPACES;

**Create table**

    CREATE KEYSPACE IF NOT EXISTS hw2 WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3 };
    USE hw2;
    CREATE TABLE users (
        firstname text,
        PRIMARY KEY (lastname),
    );
    DESC SCHEMA;



## Basic

    INSERT INTO users (firstname, age) VALUES ('John', 46);

    SELECT * FROM users;

    SELECT * FROM users WHERE lastname = 'Doe'; // primary key only, sounds like a super column

    UPDATE users SET city = 'San Jose' WHERE lastname = 'Doe';

    DELETE FROM users WHERE lastname = 'Doe';


## Lab

    CREATE KEYSPACE IF NOT EXISTS lab WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3 };
    USE lab;

    CREATE TABLE users (
        username text PRIMARY KEY,
        password text
    );
    CREATE TABLE tweets (
        tweet_id    uuid PRIMARY KEY,
        username    text,
        body        text
    );
    CREATE TABLE userline (
        username text,
        time timeuuid,
        tweet_id uuid,
        PRIMARY KEY (username, time)
    ) WITH CLUSTERING ORDER BY (time DESC);
    CREATE TABLE timeline (
        username text,
        time timeuuid,
        tweet_id uuid,
        PRIMARY KEY (username, time)
    ) WITH CLUSTERING ORDER BY (time DESC);
    CREATE TABLE friends (
        username    text,
        friend      text,
        since       timestamp,
        PRIMARY KEY (username, friend)
    );
    CREATE TABLE followers (
        username    text,
        follower    text,
        since       timestamp,
        PRIMARY KEY (username, follower)
    );

// 0

    INSERT INTO users (username, password) VALUES ('chenwang', '1');
    INSERT INTO users (username, password) VALUES ('yangyang', '2');
    INSERT INTO friends (username, friend, since) VALUES ('yangyang', 'chenwang', '2018-01-01 12:00:00');
    INSERT INTO followers (username, follower, since) VALUES ('chenwang', 'yangyang', '2018-01-01 12:00:00');

// 1

    INSERT INTO tweets (tweet_id, username, body) VALUES (uuid(), 'chenwang', 'Hi! This is Wang Chen.');

// 2

    SELECT tweet_id FROM tweets WHERE username = 'chenwang';
    INSERT INTO userline (username, time, tweet_id) VALUES ('chenwang', now(), 5ecbaa2a-7672-4c45-b031-942c410e60cd);

// 3

    INSERT INTO timeline (username, time, tweet_id) VALUES ('chenwang', 2ac0b760-6241-11e9-824a-c94d650a52b9, 5ecbaa2a-7672-4c45-b031-942c410e60cd);
    INSERT INTO timeline (username, time, tweet_id) VALUES ('yangyang', 2ac0b760-6241-11e9-824a-c94d650a52b9, 5ecbaa2a-7672-4c45-b031-942c410e60cd);

// 4

    SELECT time, tweet_id FROM userline WHERE username='chenwang' LIMIT 10;

// 5

    SELECT * FROM tweets WHERE tweet_id= 5ecbaa2a-7672-4c45-b031-942c410e60cd;

// 6

    UPDATE users SET password = '123' WHERE username = 'chenwang';

// 7

    SELECT friend FROM friends WHERE username = 'yangyang';
    DELETE FROM friends WHERE username = 'yangyang' AND friend= 'chenwang';
    SELECT follower FROM followers WHERE username = 'chenwang';
    DELETE FROM followers WHERE username = 'chenwang' AND follower = 'yangyang';

// 8

    ALTER TABLE users ADD email set<text>;
    UPDATE users SET email = {'cwang@andrew.cmu.edu, chenwang@cmu.edu'} WHERE username = 'chenwang';


**// Insert missing value to clustering column: forbidden.**

    CREATE TABLE users2 (
        username text,
        password text,
        primary key (username, password)
    ) with clustering order by (password asc);

    INSERT INTO users2 (username, password) VALUES ('c', '1');
    INSERT INTO users2 (username) VALUES ('y');

    SELECT * from users2 WHERE username in ('c', 'y') order by password desc;

## hw2

    CREATE TABLE students (
        StudentID text PRIMARY KEY,
        FirstName text,
        LastName text
    );
    INSERT INTO students (StudentID, FirstName, LastName) VALUES ('s1', 'Dave', 'Davis');
    INSERT INTO students (StudentID, FirstName, LastName) VALUES ('s2', 'John', 'Johnson');
    INSERT INTO students (StudentID, FirstName, LastName) VALUES ('s3', 'Thomas', 'Thompson');

    CREATE TABLE courses (
        CourseID    text PRIMARY KEY,
        CourseName  text
    );
    INSERT INTO courses (CourseID, CourseName) VALUES ('c13', 'Biology');
    INSERT INTO courses (CourseID, CourseName) VALUES ('c11', 'Calculus');
    INSERT INTO courses (CourseID, CourseName) VALUES ('c10', 'AmericanHistory');
    INSERT INTO courses (CourseID, CourseName) VALUES ('c12', 'Physics');

**Denormalization: Set**

Supported Query: returns the student ID of all students who took the American History course.

**Create set and update set**

    CREATE TABLE gradesByCourse (
        CourseName  text PRIMARY KEY,
        students    set<text>
    );
    INSERT INTO gradesByCourse (CourseName) VALUES ('Biology');
    INSERT INTO gradesByCourse (CourseName) VALUES ('Calculus');
    INSERT INTO gradesByCourse (CourseName) VALUES ('AmericanHistory');
    INSERT INTO gradesByCourse (CourseName) VALUES ('Physics');

    UPDATE gradesByCourse SET students = students + {'s1'} WHERE CourseName = 'Biology';
    UPDATE gradesByCourse SET students = students + {'s1'} WHERE CourseName = 'Calculus';
    UPDATE gradesByCourse SET students = students + {'s2'} WHERE CourseName = 'AmericanHistory';
    UPDATE gradesByCourse SET students = students + {'s2'} WHERE CourseName = 'Physics';
    UPDATE gradesByCourse SET students = students + {'s3'} WHERE CourseName = 'AmericanHistory';
    UPDATE gradesByCourse SET students = students + {'s3'} WHERE CourseName = 'Physics';
    UPDATE gradesByCourse SET students = students + {'s3'} WHERE CourseName = 'Biology';

**Denormalization: wide column**

_Wide column family models a list, with each column being one element in that list._

    ALTER TABLE gradesByCourse ADD s1 int;
    UPDATE gradesByCourse SET s1 = 78 WHERE CourseName = 'Biology';
    UPDATE gradesByCourse SET s1 = 76 WHERE CourseName = 'Calculus';

    ALTER TABLE gradesByCourse ADD s2 int;
    UPDATE gradesByCourse SET s2 = 90 WHERE CourseName = 'AmericanHistory';
    UPDATE gradesByCourse SET s2 = 95 WHERE CourseName = 'Physics';
    
    ALTER TABLE gradesByCourse ADD s3 int;
    UPDATE gradesByCourse SET s3 = 88 WHERE CourseName = 'AmericanHistory';
    UPDATE gradesByCourse SET s3 = 79 WHERE CourseName = 'Physics';
    UPDATE gradesByCourse SET s3 = 80 WHERE CourseName = 'Biology';




