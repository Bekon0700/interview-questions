# SQL — Questions

You list PostgreSQL/MySQL, so a SQL round is possible. Practice writing queries by hand.

---

## Beginner

1. What is the difference between SQL and NoSQL? (also in databases file)
2. What are the main SQL clauses and their order (SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY, LIMIT)?
3. What is the difference between `WHERE` and `HAVING`?
4. What are the types of JOINs (INNER, LEFT, RIGHT, FULL, CROSS)? (commonly asked)
5. What is a primary key vs a unique key vs a foreign key?
6. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?
7. What is the difference between `COUNT(*)`, `COUNT(column)`, and `COUNT(DISTINCT column)`?

## Intermediate

8. Write a query to find the second highest salary from an employees table. (classic)
9. How do GROUP BY and aggregate functions (SUM, AVG, COUNT, MIN, MAX) work?
10. What is an index and how does it work under the hood (B-tree)? What are the downsides?
11. What is the difference between a clustered and non-clustered index?
12. What is normalization (1NF, 2NF, 3NF) and when would you denormalize?
13. What are subqueries and CTEs (WITH clause)? When use each?
14. What is the N+1 query problem and how do you fix it in SQL? (also in databases file)
15. How do you find duplicate rows in a table?

## Advanced

16. What are window functions (ROW_NUMBER, RANK, DENSE_RANK, LAG/LEAD)? Give a use case.
17. Write a query to get the top N records per group (e.g., top 3 orders per customer).
18. What are transactions and isolation levels? (also in databases file)
19. How do you optimize a slow SQL query? Walk through your approach (EXPLAIN/EXPLAIN ANALYZE).
20. What is a deadlock and how do you prevent it?
21. What is the difference between OLTP and OLAP?
22. How do you paginate efficiently in SQL (OFFSET vs keyset pagination)?
23. When would you choose PostgreSQL over MySQL and vice versa?
