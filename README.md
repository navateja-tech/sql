# 🎵 Music Database SQL Analysis

A PostgreSQL SQL project based on a music store database. The project contains
practical SQL queries ranging from basic data retrieval and aggregation to
joins, subqueries, CTEs, and window functions.

## 📌 Project Overview

The database contains information about:

- Customers
- Employees
- Artists
- Albums
- Tracks
- Genres
- Invoices
- Invoice Line Items
- Playlists
- Media Types

The goal of this project is to answer business-oriented questions using SQL
and demonstrate practical PostgreSQL querying skills.

---

## 🗂️ Database Tables

The main tables used in this project are:

| Table | Purpose |
|---|---|
| `employee` | Employee information and job titles |
| `customer` | Customer details |
| `invoice` | Customer invoices and billing information |
| `invoice_line` | Individual purchased tracks |
| `track` | Track details such as name, duration and genre |
| `album` | Album information |
| `artist` | Artist information |
| `genre` | Music genre information |
| `playlist` | Playlist information |

---

## 🧠 SQL Concepts Demonstrated

### Basic SQL
- `SELECT`
- `WHERE`
- `ORDER BY`
- `LIMIT`
- `GROUP BY`
- `DISTINCT`

### Aggregation
- `COUNT()`
- `SUM()`
- `AVG()`

### Joins
- `JOIN`
- Multi-table joins

### Advanced SQL
- Subqueries
- Common Table Expressions (CTEs)
- Window Functions
- `ROW_NUMBER()`
- `PARTITION BY`

---

# 📊 SQL Questions & Solutions

## Question Set 1 — Easy

### Q1. Who is the senior-most employee based on job title?

```sql
SELECT title, last_name, first_name 
FROM employee
ORDER BY levels DESC
LIMIT 1;
```

### Q2. Which countries have the most invoices?

```sql
SELECT COUNT(*) AS c, billing_country 
FROM invoice
GROUP BY billing_country
ORDER BY c DESC;
```

### Q3. What are the top values of total invoice?

```sql
SELECT total 
FROM invoice
ORDER BY total DESC;
```

### Q4. Which city has the best customers?

Find the city that generated the highest total invoice amount.

```sql
SELECT billing_city,
       SUM(total) AS InvoiceTotal
FROM invoice
GROUP BY billing_city
ORDER BY InvoiceTotal DESC
LIMIT 1;
```

### Q5. Who is the best customer?

Find the customer who has spent the most money.

```sql
SELECT customer.customer_id,
       first_name,
       last_name,
       SUM(total) AS total_spending
FROM customer
JOIN invoice
    ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spending DESC
LIMIT 1;
```

---

# Question Set 2 — Moderate

### Q1. Find all Rock Music listeners

Return the email, first name, last name, and genre of customers who purchased
Rock music. Results are ordered alphabetically by email.

```sql
SELECT DISTINCT
       email AS Email,
       first_name AS FirstName,
       last_name AS LastName,
       genre.name AS Name
FROM customer
JOIN invoice
    ON invoice.customer_id = customer.customer_id
JOIN invoice_line
    ON invoice_line.invoice_id = invoice.invoice_id
JOIN track
    ON track.track_id = invoice_line.track_id
JOIN genre
    ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
ORDER BY email;
```

### Q2. Find the top 10 Rock artists

Return the artist name and total number of Rock tracks.

```sql
SELECT artist.artist_id,
       artist.name,
       COUNT(artist.artist_id) AS number_of_songs
FROM track
JOIN album
    ON album.album_id = track.album_id
JOIN artist
    ON artist.artist_id = album.artist_id
JOIN genre
    ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;
```

### Q3. Find tracks longer than the average song length

Return the track name and duration, with the longest tracks first.

```sql
SELECT name,
       milliseconds
FROM track
WHERE milliseconds > (
    SELECT AVG(milliseconds)
    FROM track
)
ORDER BY milliseconds DESC;
```

---

# Question Set 3 — Advanced

## Q1. Find customer spending on the best-selling artist

Calculate how much each customer spent on the best-selling artist.

```sql
WITH best_selling_artist AS (
    SELECT
        artist.artist_id AS artist_id,
        artist.name AS artist_name,
        SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
    FROM invoice_line
    JOIN track
        ON track.track_id = invoice_line.track_id
    JOIN album
        ON album.album_id = track.album_id
    JOIN artist
        ON artist.artist_id = album.artist_id
    GROUP BY 1
    ORDER BY 3 DESC
    LIMIT 1
)
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    bsa.artist_name,
    SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c
    ON c.customer_id = i.customer_id
JOIN invoice_line il
    ON il.invoice_id = i.invoice_id
JOIN track t
    ON t.track_id = il.track_id
JOIN album alb
    ON alb.album_id = t.album_id
JOIN best_selling_artist bsa
    ON bsa.artist_id = alb.artist_id
GROUP BY 1, 2, 3, 4
ORDER BY 5 DESC;
```

## Q2. Find the top customer in each country

Determine the customer who has spent the most on music in each country.

The query uses `ROW_NUMBER()` with `PARTITION BY` to identify the top
customer within each country.

```sql
WITH Customter_with_country AS (
    SELECT
        customer.customer_id,
        first_name,
        last_name,
        billing_country,
        SUM(total) AS total_spending,
        ROW_NUMBER() OVER(
            PARTITION BY billing_country
            ORDER BY SUM(total) DESC
        ) AS RowNo
    FROM invoice
    JOIN customer
        ON customer.customer_id = invoice.customer_id
    GROUP BY 1, 2, 3, 4
    ORDER BY 4 ASC, 5 DESC
)
SELECT *
FROM Customter_with_country
WHERE RowNo <= 1;
```

---

# 🔍 Query Summary

| Level | Questions | Main Concepts |
|---|---:|---|
| Easy | 5 | Filtering, grouping, sorting, aggregation |
| Moderate | 3 | Joins, `DISTINCT`, subqueries, aggregation |
| Advanced | 2 | CTEs, window functions, `ROW_NUMBER()`, complex joins |

**Total: 10 SQL questions**

---

# 🛠️ Technologies Used

- PostgreSQL
- SQL
- pgAdmin 4

---

# 🚀 How to Run the Project

## 1. Install PostgreSQL

Install PostgreSQL and pgAdmin 4.

## 2. Create the database

Create a database named:

```text
music_database
```

## 3. Import the database

Import the provided PostgreSQL database dump into `music_database`.

## 4. Open the queries

Open:

```text
queries/music_database_queries.sql
```

in pgAdmin Query Tool.

## 5. Execute the queries

Run the queries individually and inspect the results.

---

# 📁 Recommended Repository Structure

```text
music-database-sql-project/
│
├── README.md
│
├── database/
│   └── music_database.sql
│
└── queries/
    └── music_database_queries.sql
```

---

# 🎯 Learning Outcomes

This project demonstrates practical experience with:

- Relational databases
- PostgreSQL
- SQL querying
- Data aggregation
- Multi-table joins
- Subqueries
- Common Table Expressions
- Window functions
- Business-oriented data analysis

---

## 👨‍💻 Author

**Navateja**

B.Tech Computer Science Student
