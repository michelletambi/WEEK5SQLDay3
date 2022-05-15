# Task - Employee Records

The major corporation BusinessCorp&#8482; wants to do some analysis of varioius metrics around its employees and have contracted the job out to you. They have provided you with two SQL files, you will need to write the queries which will extract the data they are looking for.

## Setup

- Create a database to run the files against
- Use the `psql -d database -f file` command to run the SQL files
- Before starting to write the queries take some time to look at the data and figure out the relationships between the tables - maybe even draw an ERD to help.

## Tasks

### CTEs

1) Calculate the average salary of all employees

```sql
SELECT AVG(salary) FROM employees;
```


2) Calculate the average salary of the employees in each team (hint: you'll need to `JOIN` and `GROUP BY` here)

```sql
SELECT AVG(salary), departments.name FROM employees
INNER JOIN departments ON employees.department_id = departments.id
GROUP BY departments.name
```

3) Using a CTE find the ratio of each employees salary to their team average, eg. an employee earning £55000 in a team where the average is £50000 has a ratio of 1.1
Didn't actually have to use an alias here because the CTE is basically a whole new table, so it would work without an alias.
```sql
WITH department_averages (average_salary, department, dep_id) AS (
SELECT AVG(salary), departments.name, departments.id FROM employees
INNER JOIN departments ON employees.department_id = departments.id
GROUP BY departments.id)

SELECT d.average_salary, d.department, 
e.first_name, e.salary, e.salary / d.average_salary AS ratio
FROM department_averages AS d
INNER JOIN employees AS e ON e.department_id = d.dep_id;
```

4) Find the employee with the highest ratio in Argentina

```sql
WITH department_averages (average_salary, department, dep_id) AS (
SELECT AVG(salary), departments.name, departments.id FROM employees
INNER JOIN departments ON employees.department_id = departments.id
GROUP BY departments.id)

SELECT d.average_salary, d.department, 
e.first_name, e.salary, e.salary / d.average_salary AS ratio
FROM department_averages AS d
INNER JOIN employees AS e ON e.department_id = d.dep_id
WHERE e.country = 'Argentina' ORDER BY ratio DESC LIMIT 1;
```

5) **Extension:** Add a second CTE calculating the average salary for each country and add a column showing the difference between each employee's salary and their country average

```sql
<!--Copy solution here-->
```

---

### Window Functions

1) Find the running total of salary costs as the business has grown and hired more people

```sql
SELECT start_date, SUM(salary) OVER (ORDER BY start_date) FROM employees;
```

2) Determine if any employees started on the same day (hint: some sort of ranking may be useful here)

```sql
<!--Colin's solution (followed along during recap-->
```
SELECT 
DENSE_RANK() OVER (ORDER BY start_date)AS ranking
FROM employees
ORDER BY ranking DESC;

<!--DENSE_RANK is used here because since there are 1000 employees, you'd expect there to be 1000 ranks of start date if everyoe started on a different date. This query returns 961 as the highest rank meaning atleast two people had to start on the same date.-->

///////////
<!--My solution: partitions the table by start date and counts how many started in each partitioned group. As we scroll, we see dates where only 1 person started with a count of 1. Other dates have a count of 2, meaning two people started on these dates.-->
SELECT DISTINCT start_date, COUNT (*) OVER (PARTITION BY start_date) FROM employees;

3) Find how many employees there are from each country

```sql
<!--Looked at solution-->:
SELECT DISTINCT country, COUNT (*) OVER (PARTITION BY country) FROM employees;
```

4) Show how the average salary cost for each department has changed as the number of employees has increased

```sql
<!--Done in recap-->:
SELECT employees.first_name, 
employees.last_name, 
departments.name, 
employees.start_date, 
employees.salary, 
AVG(employees.salary) OVER (PARTITION BY employees.department_id ORDER BY employees.start_date) AS dep_avg_salary
FROM employees 
INNER JOIN departments ON employees.department_id = departments.id
```

5) **Extension:** Research the `EXTRACT` function and use it in conjunction with `PARTITION` and `COUNT` to show how many employees started working for BusinessCorp&#8482; each year. If you're feeling adventurous you could further partition by month...

```sql
<!--Copy solution here-->
```

---

### Combining the two

1) Find the maximum and minimum salaries

```sql
SELECT 
MAX(salary),
MIN (salary)
FROM 
employees 
```

2) Find the difference between the maximum and minimum salaries and each employee's own salary

```sql
<!
SELECT
first_name,
last_name,
salary,
MAX(salary) OVER () AS max_salary,
(MAX(salary) OVER () - salary) AS max_difference,
MIN(salary) OVER () AS min_salary,
salary - MIN(salary) OVER () AS min_difference
FROM 
employees;
```

3) Order the employees by start date. Research how to calculate the **median** salary value and the **standard deviation** in salary values and show how these change as more employees join the company

```sql
<!--Copy solution here-->
```

4) Limit this query to only Research & Development team members and show a rolling value for only the 5 most recent employees.

```sql
<!--Copy solution here-->
```

