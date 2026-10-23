# SQL Practice Solutions

**30 Questions • Organized Q&A Study Notes**  
Basic Queries • Joins • Subqueries • Set Operations • Aggregation  

Formatted for revision, practice, and exam preparation.

---

## How to Use These Notes

Each question includes the database, the main SQL concept, a model answer, and a short explanation. Where useful, an alternative query is included. Table and column names follow the supplied questions; adjust join keys or SQL syntax if your database schema or SQL dialect differs.

### Contents
- Part 1 — Basic SQL Queries (Q1–Q10)
- Part 2 — Joins and Subqueries (Q11–Q20)
- Part 3 — Grouping, Set Operations and Advanced Queries (Q21–Q30)
- Important SQL Notes and Corrections

---

## Part 1: Basic SQL Queries

### Q1. Find authors whose first name contains exactly one character.
**Database:** LIS  
**Concept:** LIKE wildcard

**Answer**
```sql
SELECT author_fname, author_lname
FROM book_authors
WHERE author_fname LIKE '_';
```

**Explanation:** `_` matches exactly one character. This returns authors whose first-name value consists of one character.

---

### Q2. Find books not published in 2015 or 2017.
**Database:** LIS  
**Concept:** EXCEPT / NOT IN

**Answer**
```sql
SELECT title, publisher
FROM book_catalogue
WHERE year NOT IN (2015, 2017);
```

**Alternative solution(s)**
```sql
SELECT title, publisher
FROM book_catalogue
WHERE year <> 2015
AND year <> 2017;
```

```sql
SELECT title, publisher
FROM book_catalogue
EXCEPT
SELECT title, publisher
FROM book_catalogue
WHERE year IN (2015, 2017);
```

**Explanation:** `NOT IN` excludes rows whose year is 2015 or 2017.

**Note:** If year is NULL, these WHERE conditions do not include that row. EXCEPT availability depends on the SQL dialect.

---

### Q3. Find students born in May 2002 or May 2003.
**Database:** LIS  
**Concept:** UNION / OR / date filtering

**Answer**
```sql
SELECT student_fname, student_lname
FROM students
WHERE (dob >= '2002-05-01' AND dob < '2002-06-01')
OR (dob >= '2003-05-01' AND dob < '2003-06-01');
```

**Alternative solution(s)**
```sql
SELECT student_fname, student_lname
FROM students
WHERE dob BETWEEN '2002-05-01' AND '2002-05-31'
UNION
SELECT student_fname, student_lname
FROM students
WHERE dob BETWEEN '2003-05-01' AND '2003-05-31';
```

```sql
SELECT student_fname, student_lname
FROM students
WHERE YEAR(dob) IN (2002, 2003)
AND MONTH(dob) = 5;
```

**Explanation:** The query includes every date in May of either year. The exclusive upper bound also works safely when dob contains a time component.

**Note:** UNION removes duplicate output rows; UNION ALL preserves them. YEAR() and MONTH() are common in MySQL but may vary by dialect.

---

### Q4. Count undergraduate members.
**Database:** LIS  
**Concept:** COUNT and aliases

**Answer**
```sql
SELECT COUNT(*) AS total_member
FROM members
WHERE member_type = 'UG';
```

**Explanation:** COUNT(*) counts all rows that satisfy the undergraduate-member filter.

---

### Q5. Count female students in each department.
**Database:** LIS  
**Concept:** GROUP BY

**Answer**
```sql
SELECT department_code,
COUNT(*) AS no_of_females
FROM students
WHERE gender = 'F'
GROUP BY department_code;
```

**Explanation:** WHERE filters to female students, and GROUP BY creates one result row per department.

---

### Q6. Find managers who joined between 2019 and 2020 inclusive.
**Database:** FLIS  
**Concept:** Date filtering

**Answer**
```sql
SELECT name, dob
FROM managers
WHERE since >= '2019-01-01'
AND since < '2021-01-01';
```

**Alternative solution(s)**
```sql
SELECT name, dob
FROM managers
WHERE since BETWEEN '2019-01-01' AND '2020-12-31';
```

**Explanation:** The date range includes all dates in 2019 and 2020. The half-open range also handles DATETIME values.

**Note:** The BETWEEN version is suitable for a DATE column; if since is DATETIME, it can miss times later on 2020-12-31.

---

### Q7. Find teams whose name contains a surname-like portion beginning with S.
**Database:** FLIS  
**Concept:** LIKE

**Answer**
```sql
SELECT *
FROM teams
WHERE name LIKE '% S%';
```

**Explanation:** The pattern looks for a space followed by S and then any characters.

**Note:** This only matches the intended surname portion if team names follow that naming convention. Case sensitivity depends on the database collation.

---

### Q8. List instructors by department descending and salary ascending.
**Database:** University  
**Concept:** ORDER BY

**Answer**
```sql
SELECT name, dept_name, salary
FROM instructor
ORDER BY dept_name DESC, salary ASC;
```

**Explanation:** Departments are ordered descending; within each department, salary is ordered ascending.

---

### Q9. Find players whose names start with S but do not end with n.
**Database:** FLIS  
**Concept:** EXCEPT / LIKE

**Answer**
```sql
SELECT name
FROM players
WHERE name LIKE 'S%'
AND name NOT LIKE '%n';
```

**Alternative solution(s)**
```sql
SELECT name
FROM players
WHERE name LIKE 'S%'
EXCEPT
SELECT name
FROM players
WHERE name LIKE '%n';
```

**Explanation:** The first pattern requires names to begin with S; NOT LIKE excludes names ending in n.

**Note:** LIKE case sensitivity depends on the database collation.

---

### Q10. Count players belonging to team T0001.
**Database:** FLIS  
**Concept:** COUNT

**Answer**
```sql
SELECT COUNT(*) AS total_players
FROM players
WHERE team_id = 'T0001';
```

**Explanation:** The WHERE clause filters the players by team, and COUNT(*) counts matching rows.

---

## Part 2: Joins and Subqueries

### Q11. Return rows from students and departments using a natural join.
**Database:** LIS  
**Concept:** NATURAL JOIN / INNER JOIN

**Answer**
```sql
SELECT *
FROM students
NATURAL JOIN departments;
```

**Alternative solution(s)**
```sql
SELECT *
FROM students s
JOIN departments d
ON s.department_id = d.department_id;
```

**Explanation:** NATURAL JOIN automatically matches every same-named column in both tables and returns matching rows. Each common join column appears once.

**Note:** Replace department_id with the actual key. NATURAL JOIN can unintentionally join on additional same-named columns.

---

### Q12. Find Accounting instructors earning more than at least one Psychology instructor.
**Database:** University  
**Concept:** ANY / ALL

**Answer**
```sql
SELECT *
FROM instructor
WHERE dept_name = 'Accounting'
AND salary > ANY (
SELECT salary
FROM instructor
WHERE dept_name = 'Psychology'
);
```

**Alternative solution(s)**
```sql
-- More than every Psychology instructor
SELECT *
FROM instructor
WHERE dept_name = 'Accounting'
AND salary > ALL (
SELECT salary
FROM instructor
WHERE dept_name = 'Psychology'
);
```

```sql
-- Alternative to > ALL when the subquery has at least one non-NULL salary
SELECT *
FROM instructor
WHERE dept_name = 'Accounting'
AND salary > (
SELECT MAX(salary)
FROM instructor
WHERE dept_name = 'Psychology'
);
```

**Explanation:** `> ANY` means the salary is greater than at least one salary returned by the subquery.

**Note:** The `> ALL` and MAX versions are equivalent under ordinary non-NULL data assumptions, but can differ if the subquery is empty or contains NULL.

---

### Q13. Find the date of birth of the manager whose team Shlok plays for.
**Database:** FLIS  
**Concept:** Subquery with IN

**Answer**
```sql
SELECT dob
FROM managers
WHERE team_id IN (
SELECT team_id
FROM players
WHERE name = 'Shlok'
);
```

**Explanation:** The inner query finds the team ID associated with Shlok. The outer query returns the DOB of manager(s) for that team.

**Note:** If more than one player is named Shlok, more than one team may be returned.

---

### Q14. Find match numbers where the host team plays at Emirates Stadium.
**Database:** FLIS  
**Concept:** ANY / subquery

**Answer**
```sql
SELECT match_num
FROM matches
WHERE host_team_id = ANY (
SELECT team_id
FROM teams
WHERE playground = 'Emirates Stadium'
);
```

**Alternative solution(s)**
```sql
SELECT m.match_num
FROM matches m
JOIN teams t
ON m.host_team_id = t.team_id
WHERE t.playground = 'Emirates Stadium';
```

**Explanation:** The subquery returns teams associated with Emirates Stadium; the outer query finds matches hosted by those teams.

---

### Q15. List players from the team Rainbow with player ID, DOB, and city.
**Database:** FLIS  
**Concept:** INNER JOIN

**Answer**
```sql
SELECT p.name,
p.player_id,
p.dob,
t.city
FROM players AS p
JOIN teams AS t
ON p.team_id = t.team_id
WHERE t.name = 'Rainbow';
```

**Explanation:** The join links each player to their team. The WHERE condition restricts the result to Rainbow.

---

### Q16. Find teams in the same city as Amigos, excluding Amigos.
**Database:** FLIS  
**Concept:** Self-join / scalar subquery

**Answer**
```sql
SELECT t2.name
FROM teams AS t1
JOIN teams AS t2
ON t1.city = t2.city
WHERE t1.name = 'Amigos'
AND t2.name <> 'Amigos';
```

**Alternative solution(s)**
```sql
SELECT name
FROM teams
WHERE city = (
SELECT city
FROM teams
WHERE name = 'Amigos'
)
AND name <> 'Amigos';
```

**Explanation:** The teams table is joined to itself to find other teams in the city where Amigos is located.

**Note:** The scalar-subquery alternative assumes exactly one team row named Amigos.

---

### Q17. Find the department where Gita Das is studying.
**Database:** LIS  
**Concept:** JOIN

**Answer**
```sql
SELECT d.department_name
FROM departments d
JOIN students s
ON d.department_id = s.department_id
WHERE s.student_fname = 'Gita'
AND s.student_lname = 'Das';
```

**Alternative solution(s)**
```sql
SELECT department_name
FROM departments
NATURAL JOIN students
WHERE student_fname = 'Gita'
AND student_lname = 'Das';
```

**Explanation:** The join matches students to departments, and the WHERE clause identifies Gita Das.

**Note:** The explicit join assumes department_id is the shared foreign key.

---

### Q18. Find roll numbers of male students whose department is in Block 2.
**Database:** LIS  
**Concept:** JOIN and filtering

**Answer**
```sql
SELECT s.roll_no
FROM students s
JOIN departments d
ON s.department_id = d.department_id
WHERE d.department_building = 'Block_2'
AND s.gender = 'M';
```

**Explanation:** The query joins students with departments, then filters by building and gender.

**Note:** Use the actual department key and exact stored building value from your schema.

---

### Q19. Find students whose department is in Block 1.
**Database:** LIS  
**Concept:** JOIN and filtering

**Answer**
```sql
SELECT s.student_fname,
s.student_lname,
s.roll_no
FROM students s
JOIN departments d
ON s.department_id = d.department_id
WHERE d.department_building = 'Block_1';
```

**Explanation:** The join finds each student's department, and the building filter keeps departments in Block 1.

**Note:** Use the actual department key and exact stored building value from your schema.

---

### Q20. Find members who have never issued a book.
**Database:** LIS  
**Concept:** NOT EXISTS

**Answer**
```sql
SELECT m.*
FROM members m
WHERE NOT EXISTS (
SELECT 1
FROM book_issue bi
WHERE bi.member_no = m.member_no
);
```

**Explanation:** For each member, the correlated subquery checks for a matching issue record. NOT EXISTS keeps members with no matching record.

---

## Part 3: Grouping, Set Operations and Advanced Queries

### Q21. Find courses offered in both Fall and Spring.
**Database:** University  
**Concept:** INTERSECT

**Answer**
```sql
SELECT c.title
FROM course c
WHERE c.course_id IN (
SELECT course_id
FROM section
WHERE semester = 'Fall'
INTERSECT
SELECT course_id
FROM section
WHERE semester = 'Spring'
);
```

**Explanation:** INTERSECT returns course IDs that appear in both semester result sets; the outer query returns their titles.

**Note:** INTERSECT is supported by PostgreSQL, SQL Server, Oracle and several other systems, but not all SQL dialects.

---

### Q22. Count distinct students in each building from 2005 through 2008.
**Database:** University  
**Concept:** JOIN and aggregation

**Answer**
```sql
SELECT se.building,
COUNT(DISTINCT t.ID) AS student_count
FROM section AS se
JOIN takes AS t
ON se.course_id = t.course_id
AND se.sec_id = t.sec_id
AND se.semester = t.semester
AND se.year = t.year
WHERE t.year BETWEEN 2005 AND 2008
GROUP BY se.building;
```

**Explanation:** The query links sections with student enrolments and counts each student only once per building.

**Note:** This assumes the section table includes building and that takes.ID identifies the student. Joining the student table is unnecessary if takes.ID is the student identifier.

---

### Q23. Find issue dates for copies of “Learning with Python”.
**Database:** LIS  
**Concept:** JOIN / USING

**Answer**
```sql
SELECT DISTINCT bi.doi
FROM book_catalogue AS bkcat
JOIN book_copies AS bcp
USING (isbn_no)
JOIN book_issue AS bi
USING (accession_no)
WHERE bkcat.title = 'Learning with Python';
```

**Explanation:** The joins connect a book title to its copies and issue records. DISTINCT returns each issue date only once.

**Note:** USING requires the named columns to exist in both joined tables; use ON if your SQL dialect or schema requires it.

---

### Q24. Find the highest-paid instructor(s) in each department whose average salary exceeds the maximum Psychology salary.
**Database:** University  
**Concept:** CTE / GROUP BY / correlated subquery

**Answer**
```sql
WITH qualifying_departments AS (
SELECT dept_name
FROM instructor
GROUP BY dept_name
HAVING AVG(salary) > (
SELECT MAX(salary)
FROM instructor
WHERE dept_name = 'Psychology'
)
)
SELECT i.name, i.salary, i.dept_name
FROM instructor i
JOIN qualifying_departments qd
ON i.dept_name = qd.dept_name
WHERE i.salary = (
SELECT MAX(i2.salary)
FROM instructor i2
WHERE i2.dept_name = i.dept_name
);
```

**Explanation:** The CTE identifies departments whose average salary is greater than the maximum Psychology salary. The outer query returns all instructors tied for the highest salary in each qualifying department.

**Note:** This version expresses the per-department maximum explicitly. CTE support depends on SQL dialect/version.

---

### Q25. Count courses taught in Fall but never in Spring.
**Database:** University  
**Concept:** EXCEPT / COUNT

**Answer**
```sql
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

**Explanation:** EXCEPT removes all course IDs that appear in Spring from the Fall course list. COUNT(*) counts the remaining distinct IDs.

**Note:** The query counts courses that have at least one Fall section and no Spring section in the data being queried, regardless of year.

---

### Q26. Find the instructor(s) who taught the maximum number of classes on Wednesday.
**Database:** University  
**Concept:** CTE / ALL / aggregation

**Answer**
```sql
WITH temp_table AS (
SELECT t.ID AS instructor_id,
COUNT(*) AS class_count
FROM section se
JOIN time_slot ts
ON se.time_slot_id = ts.time_slot_id
JOIN teaches t
ON se.course_id = t.course_id
AND se.sec_id = t.sec_id
AND se.semester = t.semester
AND se.year = t.year
WHERE ts.day = 'W'
GROUP BY t.ID
)
SELECT i.name, tt.instructor_id, tt.class_count
FROM temp_table tt
JOIN instructor i
ON tt.instructor_id = i.ID
WHERE tt.class_count >= ALL (
SELECT class_count
FROM temp_table
);
```

**Explanation:** The CTE counts Wednesday class assignments per instructor. `>= ALL` selects every instructor tied for the largest count.

**Note:** This counts matching section/teaches records, not necessarily distinct courses or distinct time periods. Adjust COUNT if the intended meaning of 'class' differs.

---

### Q27. Find fourth referees and match numbers for matches played on 19 May 2020.
**Database:** FLIS  
**Concept:** Multiple joins

**Answer**
```sql
SELECT r.name,
mr.match_num
FROM referees r
JOIN match_referees mr
ON r.referee_id = mr.fourth_referee
JOIN matches m
ON mr.match_num = m.match_num
WHERE m.match_date = '2020-05-19';
```

**Explanation:** The joins connect referee details, referee assignments and match records; the date filter selects matches on 19 May 2020.

**Note:** This assumes fourth_referee stores referee_id values and match_date is a DATE-compatible column.

---

### Q28. Find research scholars who have not issued any books.
**Database:** LIS  
**Concept:** EXCEPT / subqueries

**Answer**
```sql
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

**Alternative solution(s)**
```sql
SELECT s.student_fname, s.student_lname
FROM students s
JOIN members m
ON s.roll_no = m.roll_no
WHERE m.member_type = 'RS'
AND NOT EXISTS (
SELECT 1
FROM book_issue bi
WHERE bi.member_no = m.member_no
);
```

**Explanation:** The first set contains roll numbers of RS members. The second contains RS members who have issued books. EXCEPT returns RS roll numbers absent from the issued-book set.

**Note:** The EXCEPT version assumes roll_no is populated consistently in members. If one student has multiple member records, the NOT EXISTS version is often more direct.

---

### Q29. Find the oldest player from All Stars.
**Database:** FLIS  
**Concept:** MIN and subquery

**Answer**
```sql
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

**Explanation:** An earlier date of birth indicates an older player. MIN(dob) finds the earliest DOB, and the outer query returns player(s) with that date.

---

### Q30. Find book titles issued to PG members but never issued to UG members.
**Database:** LIS  
**Concept:** EXCEPT / nested subqueries

**Answer**
```sql
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

**Alternative solution(s)**
```sql
SELECT DISTINCT bc.title
FROM book_catalogue bc
WHERE EXISTS (
SELECT 1
FROM book_copies bcp
JOIN book_issue bi ON bi.accession_no = bcp.accession_no
JOIN members m ON m.member_no = bi.member_no
WHERE bcp.isbn_no = bc.isbn_no
AND m.member_type = 'PG'
)
AND NOT EXISTS (
SELECT 1
FROM book_copies bcp
JOIN book_issue bi ON bi.accession_no = bcp.accession_no
JOIN members m ON m.member_no = bi.member_no
WHERE bcp.isbn_no = bc.isbn_no
AND m.member_type = 'UG'
);
```

**Explanation:** The first set contains ISBNs issued to PG members; the second contains ISBNs issued to UG members. EXCEPT leaves ISBNs in the PG set that do not appear in the UG set. DISTINCT avoids repeating titles.

**Note:** This means issued to at least one PG member and to no UG members. It does not mean that every PG issue lacks a corresponding UG issue.

---

## Important SQL Notes and Corrections

- **UNION vs UNION ALL:** UNION removes duplicate result rows; UNION ALL keeps duplicates.
- **EXCEPT:** EXCEPT subtracts the second result set from the first. Both SELECT statements must return the same number of compatible columns. Support varies by SQL dialect.
- **ANY vs ALL:** ANY means a comparison is true for at least one returned value; ALL means it is true for every returned value.
- **NATURAL JOIN:** NATURAL JOIN matches on all same-named columns, not just the intended key. An explicit JOIN ... ON is often safer.
- **NOT IN and NULL:** If a NOT IN subquery returns NULL, the predicate can evaluate to UNKNOWN and unexpectedly filter out rows. NOT EXISTS is often safer.
- **Date/time ranges:** For a full calendar period with possible time values, use a half-open range: `date >= start AND date < day-after-end`.
- **COUNT(*):** COUNT(*) counts rows. COUNT(column) counts only rows where that column is not NULL.
- **Ties:** Queries using MAX or `>= ALL` can return multiple rows when there is a tie for the maximum.

---

## Revision Checklist

- Can I identify the required output columns?
- Can I determine which tables and join keys are needed?
- Do I need WHERE, GROUP BY, HAVING, or ORDER BY?
- Would EXISTS / NOT EXISTS express the condition more clearly?
- Do I understand whether duplicates should be removed?
- Does the date condition include the full intended range?
- Is the query compatible with my SQL dialect?
```