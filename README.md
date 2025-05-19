# Library Management System

## Project Overview


This project presents the development of a Library Management System using SQL, focusing on relational database schema design, normalization, and integrity constraints. It includes the creation and management of normalized tables, implementation of foreign key relationships, and formulation of complex queries involving joins, subqueries, and aggregation. The objective is to demonstrate advanced SQL proficiency for effective data organization and retrieval in library operations.

### Problem 1: Create a New Book Record
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```
### Problem 2: Update an Existing Member's Address as '125  Oak st' and member id as 'C103'

```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

### Problem 3: Delete a Record from the Issued Status Table where issued id = 'IS121'


```sql
DELETE FROM issued_status
WHERE   issued_id =   'IS121';
```

### Problem 4: Retrieve All Books Issued by a Specific Employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```


### Problem 5: List Members Who Have Issued More Than One Book.


```sql
SELECT issued_emp_id, COUNT(*)
FROM issued_status
GROUP BY 1
HAVING COUNT(*) > 1
```



### Problem 6: Create Summary Tables each book and total book_issued_cnt.

```sql
CREATE TABLE book_issued_cnt AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issue_count
FROM issued_status as ist JOIN books as b
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
```


### Problem 7: Retrieve All Books in a Specific Category.

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

### Problem 8: Find Total Rental Income by Category.

```sql
SELECT b.category, SUM(b.rental_price), COUNT(*)
FROM issued_status as ist JOIN books as b
ON b.isbn = ist.issued_book_isbn
GROUP BY 1
```

### Problem 9: List Members Who Registered in the Last 180 Days.
```sql
SELECT * FROM members
WHERE reg_date >= CURDATE() - INTERVAL 180 DAY;
```

### Problem 10: List Employees with Their Branch Manager's Name and their branch details.

```sql
SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.*,
    e2.emp_name AS manager
FROM employees AS e1 JOIN branch AS b
    ON e1.branch_id = b.branch_id    
JOIN employees AS e2
    ON e2.emp_id = b.manager_id;
```
### Problem 11: Create a Table of Books with Rental Price Above a Certain Threshold.
```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
```
### Problem 12: Retrieve the List of Books Not Yet Returned.
```sql
SELECT * FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```



### Problem 13: Identify Members with Overdue Books (assume a 30-day return period). Display the member's name, book title, issue date, and days overdue.

```sql
SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    DATEDIFF(CURDATE(), ist.issued_date) AS over_due_days
FROM issued_status AS ist
JOIN members AS m
    ON m.member_id = ist.issued_member_id
JOIN books AS bk
    ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status AS rs
    ON rs.issued_id = ist.issued_id
WHERE 
    rs.return_date IS NULL
    AND DATEDIFF(CURDATE(), ist.issued_date) > 30
ORDER BY ist.issued_member_id;
```


### Problem 14: Write a query to update the status of books in the books table to "available" when they are returned (based on entries in the return_status table).


```sql
DELIMITER //

CREATE PROCEDURE add_return_records(
    IN p_return_id VARCHAR(10), 
    IN p_issued_id VARCHAR(10), 
    IN p_book_quality VARCHAR(10)
)
BEGIN
    DECLARE v_isbn VARCHAR(50);
    DECLARE v_book_name VARCHAR(80);

    -- Inserting into return_status
    INSERT INTO return_status (return_id, issued_id, return_date, book_quality)
    VALUES (p_return_id, p_issued_id, CURDATE(), p_book_quality);

    -- Book info from issued_status
    SELECT issued_book_isbn, issued_book_name
    INTO v_isbn, v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    -- Update book status to available
    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    SELECT CONCAT('Thank you for returning the book: ', v_book_name) AS message;
END //

DELIMITER ;

```




### Problem 15: Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE branch_reports AS
SELECT 
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) AS number_book_issued,
    COUNT(rs.return_id) AS number_of_book_return,
    SUM(bk.rental_price) AS total_revenue
FROM issued_status AS ist
JOIN employees AS e 
    ON ist.issued_emp_id = e.emp_id
JOIN branch AS b 
    ON e.branch_id = b.branch_id
LEFT JOIN return_status AS rs 
    ON rs.issued_id = ist.issued_id
JOIN books AS bk 
    ON ist.issued_book_isbn = bk.isbn
GROUP BY 
    b.branch_id, 
    b.manager_id;

SELECT * FROM branch_reports;

```

### Problem 16: Create a new table active_members containing members who have issued at least one book in the last 6 months.

```sql

CREATE TABLE active_members AS
SELECT * 
FROM members
WHERE member_id IN (
    SELECT DISTINCT issued_member_id
    FROM issued_status
    WHERE issued_date >= CURDATE() - INTERVAL 2 MONTH
);
SELECT * FROM active_members;
```


### Problem 17: Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
SELECT 
    e.emp_name,
    b.branch_id,
    b.branch_name,
    b.location,
    COUNT(ist.issued_id) AS no_book_issued
FROM issued_status AS ist
JOIN employees AS e ON e.emp_id = ist.issued_emp_id
JOIN branch AS b ON e.branch_id = b.branch_id
GROUP BY 
    e.emp_name,
    b.branch_id,
    b.branch_name,
    b.location;

```

### Problem 18: Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books. 

```sql
SELECT 
    m.member_name,
    b.book_title,
    COUNT(*) AS damaged_issue_count
FROM issued_status ist
JOIN members m ON ist.issued_member_id = m.member_id
JOIN books b ON ist.issued_book_id = b.book_id
WHERE ist.status = 'damaged'
GROUP BY m.member_name, b.book_title
HAVING COUNT(*) > 2;
```

## Conclusion

This project demonstrates key SQL skills in building and managing a library management system. It covers database creation, data manipulation, and advanced queries to effectively track members and book statuses. The project highlights how SQL enables efficient data organization and insightful analysis, providing a solid foundation for managing real-world data systems.

