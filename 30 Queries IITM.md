# 30 Queries IITM

I’ve organized your 30 SQL practice questions into a clean, readable study-note format. The original question numbering, database names, and SQL concepts are preserved, with corrected formatting and explanations where useful.

A few queries in the supplied notes may need correction depending on the database schema or SQL dialect. I’ve highlighted the most important issues rather than silently changing the intended answers.

# SQL Practice Solutions — Organized Notes

## Part 1: Basic SQL Queries

### 1. Authors whose first name contains one character

Database: LIS · `LIKE`

```
SELECT author_fname, author_lname
FROM book_authors
WHERE author_fname LIKE '_';
```

Explanation: `_` matches exactly one character.

### 2. Books not published in 2015 or 2017

Database: LIS · `EXCEPT`

```
SELECT title, publisher
FROM book_catalogue

EXCEPT

SELECT title, publisher
FROM book_catalogue
WHERE year IN (2015, 2017);

----------------------------

SELECT title, publisher
FROM book_catalogue
WHERE year NOT IN (2015, 2017);

-----------------------
SELECT title, publisher
FROM book_catalogue
WHERE year <> 2015
  AND year <> 2017;
```

Explanation: `EXCEPT` removes rows returned by the second query from the first query. Both queries must return compatible columns.

### 3. Students born in May 2002 or May 2003

Database: LIS · `UNION`

```
SELECT student_fname, student_lname
FROM students
WHERE dob BETWEEN '2002-05-01' AND '2002-05-31'

UNION

SELECT student_fname, student_lname
FROM students
WHERE dob BETWEEN '2003-05-01' AND '2003-05-31';

-----------------

SELECT student_fname, student_lname
FROM students
WHERE dob BETWEEN '2002-05-01' AND '2002-05-31'

UNION ALL

SELECT student_fname, student_lname
FROM students
WHERE dob BETWEEN '2003-05-01' AND '2003-05-31';

---------------------------------

SELECT student_fname, student_lname
FROM students
WHERE MONTH(dob) = 5
  AND DAY(dob) BETWEEN 1 AND 31
  AND YEAR(dob) IN (2002, 2003);
 
 ---------------------
 
 SELECT student_fname, student_lname
FROM students
WHERE (dob BETWEEN '2002-05-01' AND '2002-05-31')
   OR (dob BETWEEN '2003-05-01' AND '2003-05-31');
```

Explanation: `UNION` combines the results and removes duplicate rows. Using May 31 includes every date in May for these years.

### 4. Count undergraduate members

Database: LIS · `COUNT` and aliases

```
SELECT COUNT(*) AS total_member
FROM members
WHERE member_type = 'UG';
```

Explanation: `COUNT(*)` counts all rows belonging to undergraduate members. The alias `total_member` is the output column name.

### 5. Count female students in each department

Database: LIS · `GROUP BY`

```
SELECT department_code,
       COUNT(*) AS no_of_females
FROM students
WHERE gender = 'F'
GROUP BY department_code;
```

Explanation: The `WHERE` condition selects female students, and `GROUP BY` groups them by department.

### 6. Managers who joined in 2019 or 2020

Database: FLIS · Date filtering

```
SELECT name, dob
FROM managers
WHERE since BETWEEN '2019-01-01' AND '2020-12-31';

-------------------

SELECT name, dob
FROM managers
WHERE since >= '2019-01-01'
  AND since <= '2020-12-31';
```

Explanation: `BETWEEN` includes both endpoints. This assumes `since` is a date column and the dates represent the joining period.

### 7. Teams whose name contains a last-name portion beginning with S

Database: FLIS · `LIKE`

```
SELECT *
FROM teams
WHERE name LIKE '% S%';
```

Explanation: The pattern searches for a space followed by `S` and then any characters. It assumes the team name stores a surname-like portion after a space, as in the original question.

### 8. Instructors sorted by department and salary

Database: University · `ORDER BY`

```
SELECT name, dept_name, salary
FROM instructor
ORDER BY dept_name DESC, salary ASC;
```

Explanation: Departments are sorted in descending order. Within each department, instructors are sorted by salary in ascending order.

### 9. Players whose names start with S but do not end with n

Database: FLIS · `EXCEPT`

```
SELECT name
FROM players
WHERE name LIKE 'S%'

EXCEPT

SELECT name
FROM players
WHERE name LIKE '%n';

-----------------------

SELECT name
FROM players
WHERE name LIKE 'S%'
  AND name NOT LIKE '%n';
```

Explanation: The first query selects names starting with `S`. The second selects names ending with `n`. `EXCEPT` removes the latter from the former.

### 10. Count players belonging to team T0001

Database: FLIS · `COUNT`

```
SELECT COUNT(player_id) AS total_players
FROM players
WHERE team_id = 'T0001';
```

Explanation: The `WHERE` clause filters players belonging to the specified team, and `COUNT` returns their number.

## Part 2: Joins and Subqueries

### 11. Natural join of students and departments

Database: LIS · `NATURAL JOIN`

```
SELECT *
FROM students
NATURAL JOIN departments;

------------

SELECT *
FROM students
JOIN departments
ON students.department_id = departments.department_id;
```

Explanation: A natural join matches columns with the same name in both tables and returns only rows satisfying the matching condition. It displays each common join column once.

---

Yes. **NATURAL JOIN is a type of INNER JOIN** in SQL.

The difference is:

- **INNER JOIN:** You explicitly specify the columns to match using `ON`.
- **NATURAL JOIN:** SQL automatically matches columns that have the **same names** (and compatible data types) in both tables.

Example:

```sql
SELECT *
FROM Employee
NATURAL JOIN Department;
```

If both tables have a column called `dept_id`, the natural join automatically joins on `dept_id`.

So, **Natural Join ⊂ Inner Join** in terms of the matching behavior.

---

### 12. Accounting instructors earning more than at least one Psychology instructor

Database: University · `ANY`

```
SELECT *
FROM instructor
WHERE dept_name = 'Accounting'
  AND salary > ANY (
      SELECT salary
      FROM instructor
      WHERE dept_name = 'Psychology'
  );
  
  ------------------------
  
**Find Accounting instructors who earn more than every Psychology instructor.  
Method 1**

SELECT *
FROM instructor
WHERE dept_name = 'Accounting'
  AND salary > ALL (
      SELECT salary
      FROM instructor
      WHERE dept_name = 'Psychology'
  );
 

**Method 2**

SELECT *
FROM instructor
WHERE dept_name = 'Accounting'
  AND salary > (
      SELECT MAX(salary)
      FROM instructor
      WHERE dept_name = 'Psychology'
  );
```

Explanation: `ANY` means the Accounting instructor's salary must be greater than at least one salary returned by the Psychology subquery.

### 13. Date of birth of the manager whose team Shlok plays for

Database: FLIS · Subquery with `IN`

```
SELECT dob
FROM managers
WHERE team_id IN (
    SELECT team_id
    FROM players
    WHERE name = 'Shlok'
);
```

Explanation: The inner query finds Shlok's team ID. The outer query returns the date of birth of the manager of that team.

### 14. Match numbers played at Emirates Stadium

Database: FLIS · Subquery with `ANY`

```
SELECT match_num
FROM matches
WHERE host_team_id = ANY (
    SELECT team_id
    FROM teams
    WHERE playground = 'Emirates Stadium'
);
```

Explanation: The query finds matches where the host team uses Emirates Stadium as its playground.

Note: This follows the supplied solution's assumption that the stadium is identified through the host team's `playground` column.

### 15. Players from the team Rainbow

Database: FLIS · Table join

```
SELECT p.name,
       p.player_id,
       p.dob,
       t.city
FROM players AS p
JOIN teams AS t
    ON p.team_id = t.team_id
WHERE t.name = 'Rainbow';
```

Explanation: The tables are joined using `team_id`, and the `WHERE` condition selects players from Rainbow.

### 16. Teams located in the same city as Amigos

Database: FLIS · Self-join

```
SELECT t2.name
FROM teams AS t1
JOIN teams AS t2
    ON t1.city = t2.city
WHERE t1.name = 'Amigos'
  AND t2.name <> 'Amigos';
  
  
  ---------------------------------------------------
SELECT name
FROM teams
WHERE city = (SELECT city FROM teams WHERE name = 'Amigos')
  AND name <> 'Amigos';
```

Explanation: The `teams` table is joined with itself. The query finds other teams in Amigos' city and excludes Amigos itself.

### 17. Department in which Gita Das is studying

Database: LIS · Natural join

```
SELECT department_name
FROM departments
NATURAL JOIN students
WHERE student_fname = 'Gita'
  AND student_lname = 'Das';
  
 --------------------
SELECT d.department_name
FROM departments d
JOIN students s ON d.department_id = s.department_id
WHERE s.student_fname = 'Gita' AND s.student_lname = 'Das';
```

Explanation: The natural join connects students with their departments through matching column names. The filters identify Gita Das.

### 18. Roll numbers of male students whose department is in Block 2

Database: LIS · Join and filtering

```
SELECT roll_no
FROM departments
NATURAL JOIN students
WHERE department_building = 'Block_2'
  AND gender = 'M';
```

Explanation: The query returns roll numbers for male students whose department building matches the specified value.

### 19. Students whose department is in Block 1

Database: LIS · Natural join

```
SELECT student_fname,
       student_lname,
       roll_no
FROM departments
NATURAL JOIN students
WHERE department_building = 'Block_1';
```

Explanation: The result contains each matching student's first name, last name and roll number.

### 20. Members who have not issued any books

Database: LIS · `NOT EXISTS`

```
SELECT *
FROM members m
WHERE NOT EXISTS (
    SELECT *
    FROM book_issue bi
    WHERE bi.member_no = m.member_no
);
```

Explanation: The correlated subquery checks whether each member has a matching entry in `book_issue`. `NOT EXISTS` returns members for whom no such entry exists.

## Part 3: Grouping, Set Operations and Advanced Queries

### 21. Courses taught in both Fall and Spring

Database: University · `INTERSECT`

```
SELECT title
FROM course
WHERE course_id IN (
    SELECT course_id
    FROM section
    WHERE semester = 'Fall'

    INTERSECT

    SELECT course_id
    FROM section
    WHERE semester = 'Spring'
);
```

Explanation: `INTERSECT` returns course IDs present in both result sets. The outer query retrieves the corresponding course titles.

### 22. Number of distinct students in each building from 2005 through 2008

Database: University · Joins and aggregation

```
SELECT se.building,
       COUNT(DISTINCT st.ID) AS student_count
FROM section AS se
JOIN takes AS t
    ON se.course_id = t.course_id
   AND se.sec_id = t.sec_id
   AND se.semester = t.semester
   AND se.year = t.year
JOIN student AS st
    ON st.ID = t.ID
WHERE t.year BETWEEN 2005 AND 2008
GROUP BY se.building;
```

Explanation: The query connects sections, student enrolments and student records. `COUNT(DISTINCT st.ID)` counts each student once per building, even if that student took multiple courses there.

### 23. Dates when copies of “Learning with Python” were issued

Database: LIS · Multiple joins

```
SELECT DISTINCT bi.doi
FROM book_catalogue AS bkcat
JOIN book_copies AS bcp
    USING (isbn_no)
JOIN book_issue AS bi
    USING (accession_no)
WHERE bkcat.title = 'Learning with Python';
```

Explanation: The query connects the catalogue, individual book copies and issue records. `DISTINCT` prevents repeated issue dates from appearing more than once.

### 24. Highest-paid instructors in departments meeting the salary condition

Database: University · `WITH` and aggregation

```
WITH temp_table AS (
    SELECT i.name AS iname,
           i.salary AS isalary,
           i.dept_name AS idept_name
    FROM instructor i
    WHERE i.dept_name IN (
        SELECT dept_name
        FROM instructor
        GROUP BY dept_name
        HAVING AVG(salary) > (
            SELECT MAX(salary)
            FROM instructor
            WHERE dept_name = 'Psychology'
        )
    )
)
SELECT iname, isalary
FROM temp_table t
WHERE isalary = (
    SELECT MAX(t2.isalary)
    FROM temp_table t2
    WHERE t2.idept_name = t.idept_name
);
```

Explanation: The common table expression selects instructors belonging to departments whose average salary exceeds the maximum Psychology salary. The outer query finds the maximum salary in each qualifying department and returns every instructor who earns that amount.

### 25. Number of courses taught in Fall but never in Spring

Database: University · `EXCEPT`

```
SELECT COUNT(*) AS total_courses
FROM (
    SELECT course_id
    FROM section
    WHERE semester = 'Fall'

    EXCEPT

    SELECT course_id
    FROM section
    WHERE semester = 'Spring'
) AS temp_table;
```

Explanation: `EXCEPT` returns courses offered in Fall but not in Spring. `COUNT(*)` counts the resulting course IDs.

### 26. Instructor(s) who taught the maximum number of classes on Wednesday

Database: University · CTE and `ALL`

```
WITH temp_table AS (
    SELECT i.ID AS iid,
           COUNT(*) AS icount
    FROM section se
    JOIN time_slot ts
        ON se.time_slot_id = ts.time_slot_id
    JOIN teaches t
        ON se.course_id = t.course_id
       AND se.sec_id = t.sec_id
       AND se.semester = t.semester
       AND se.year = t.year
    JOIN instructor i
        ON t.ID = i.ID
    WHERE ts.day = 'W'
    GROUP BY i.ID
)
SELECT i.name,
       tt.iid,
       tt.icount
FROM temp_table tt
JOIN instructor i
    ON tt.iid = i.ID
WHERE tt.icount >= ALL (
    SELECT icount
    FROM temp_table
);
```

Explanation: The CTE counts the classes taught by each instructor on Wednesday. The `>= ALL` condition returns the instructor or instructors whose count is equal to the highest count.

### 27. Fourth referees and match numbers for matches on 19 May 2020

Database: FLIS · Multiple joins

```
SELECT r.name,
       mr.match_num
FROM referees r
JOIN match_referees mr
    ON r.referee_id = mr.fourth_referee
JOIN matches m
    ON mr.match_num = m.match_num
WHERE m.match_date = '2020-05-19';
```

Explanation: The query matches referees to their fourth-referee assignments and then connects those assignments to the matches played on the specified date.

### 28. Research scholars who have not issued any books

Database: LIS · Nested subqueries and `EXCEPT`

```
SELECT s.student_fname,
       s.student_lname
FROM students s
WHERE s.roll_no IN (
    SELECT m.roll_no
    FROM members m
    WHERE m.member_type = 'RS'

    EXCEPT

    SELECT m.roll_no
    FROM members m
    JOIN book_issue bi
        ON m.member_no = bi.member_no
    WHERE m.member_type = 'RS'
);
```

Explanation: The first part identifies research scholars who are members. The second part identifies research scholars who have issued books. `EXCEPT` returns research scholars who have not issued any books.

This version assumes `members.roll_no` identifies the corresponding student and that each research scholar is represented by a member record.

### 29. Oldest player from All Stars

Database: FLIS · Aggregate subquery

```
SELECT p.name
FROM players p
JOIN teams t
    ON p.team_id = t.team_id
WHERE t.name = 'All Stars'
  AND p.dob = (
      SELECT MIN(p2.dob)
      FROM players p2
      JOIN teams t2
          ON p2.team_id = t2.team_id
      WHERE t2.name = 'All Stars'
  );
```

Explanation: For date of birth, the smallest date represents the oldest player. The subquery finds the earliest date of birth among All Stars players, and the outer query returns the matching name or names.

### 30. Book titles issued to PG students but not UG students

Database: LIS · `EXCEPT` and nested subqueries

```
SELECT DISTINCT bc.title
FROM book_catalogue bc
WHERE bc.isbn_no IN (
    SELECT bcp.isbn_no
    FROM book_issue bi
    JOIN book_copies bcp
        ON bi.accession_no = bcp.accession_no
    JOIN members m
        ON bi.member_no = m.member_no
    WHERE m.member_type = 'PG'

    EXCEPT

    SELECT bcp.isbn_no
    FROM book_issue bi
    JOIN book_copies bcp
        ON bi.accession_no = bcp.accession_no
    JOIN members m
        ON bi.member_no = m.member_no
    WHERE m.member_type = 'UG'
);
```

Explanation: The first subquery returns ISBN numbers issued to postgraduate members. The second returns ISBN numbers issued to undergraduate members. `EXCEPT` removes books appearing in the undergraduate result, leaving books issued to postgraduate members but not undergraduate members.

`DISTINCT` ensures each qualifying book title appears only once.

## Important corrections to remember

- Question 3: Use May 31 rather than May 30 to include the entire month.
- Question 5: `COUNT(*)` is clearer than `COUNT(gender)` after filtering for female students.
- Question 7: `LIKE '% S%'` searches for a space followed by `S`; it does not prove that the final surname starts with `S` unless the naming convention supports that assumption.
- Question 10: Use `WHERE team_id = 'T0001'` to filter the team directly rather than grouping and filtering with `HAVING`.
- Question 24: The maximum salary condition should be correlated with the department. Otherwise, the original query could return an instructor with a salary that is the maximum in a different department.
- Question 28: Use `roll_no` consistently when comparing students and members, and `member_no` when matching book-issue records.
- Question 30: Join through `book_copies` and `members` so that the PG and UG categories are applied to the correct issue records.

Study tip: Practice each query by identifying the required columns, the tables involved, the row filters (`WHERE`), and whether grouping, joins or subqueries are needed. This makes it easier to recreate the solution independently during an exam.
