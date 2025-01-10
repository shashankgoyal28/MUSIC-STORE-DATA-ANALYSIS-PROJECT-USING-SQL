/* Q1: Who is the senior most employee based on job title? */
select * from employee 
order by levels desc
limit 1;

/* Q2: Which countries have the most Invoices? */
select * from invoice
select count(*) as c, billing_country from invoice 
group by billing_country
order by c desc
limit 1;

/* Q3: What are top 3 values of total invoice? */ 
select * from invoice
select total from invoice 
order by total desc
limit 3;

/* Q4: Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. 
Write a query that returns one city that has the highest sum of invoice totals. 
Return both the city name & sum of all invoice totals */
select * from invoice
select billing_city, sum(total) as invoice_totals from invoice 
group by billing_city
order by invoice_totals desc
limit 1;

/* Q5: Who is the best customer? The customer who has spent the most money will be declared the best customer. 
Write a query that returns the person who has spent the most money.*/
select c.first_name, c.last_name , i.total from customer as c join invoice as i 
on c.customer_id = i.customer_id
order by total desc
limit 1;
-- the above one was my approach 
-- the below is his approach 
select c.customer_id, c.first_name, c.last_name , sum(i.total) as total 
from customer as c join invoice as i 
on c.customer_id = i.customer_id
GROUP BY c.customer_id
order by total desc
limit 1;

/* Question Set 2 - Moderate */

/* Q1: Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
Return your list ordered alphabetically by email starting with A. */

select distinct email, first_name, last_name from customer as c 
join invoice as i on c.customer_id = i.customer_id
join invoice_line as il on i.invoice_id = il.invoice_id
where track_id in(
	select track_id from track as t 
	join genre as g on t.genre_id = g.genre_id
	where g.name like 'Rock'
)
order by email;

 /* Method 2 */

SELECT DISTINCT email,first_name, last_name, g.name 
FROM customer as c 
JOIN invoice as i ON c.customer_id = i.customer_id
JOIN invoice_line as il ON i.invoice_id = il.invoice_id
JOIN track as t ON t.track_id = il.track_id
JOIN genre as g ON t.genre_id = g.genre_id
WHERE g.name LIKE 'Rock'
ORDER BY email;

/* Q2: Let's invite the artists who have written the most rock music in our dataset. 
Write a query that returns the Artist name and total track count of the top 10 rock bands. */

-- artist,album,track,genre 
select ar.artist_id , ar.name as Artist_name, count(ar.artist_id) as total_track_count
from artist as ar join album as al 
on ar.artist_id = al.artist_id 
join track as t on al.album_id = t.album_id 
join genre as g on t.genre_id = g.genre_id 
where g.name like 'Rock'
group by ar.artist_id
order by total_track_count
limit 10;



/* Q3: Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first. */
select name, milliseconds from track
where milliseconds > (select avg(milliseconds) as avg_track_length from track )
order by milliseconds desc;


/* Question Set 3 - Advance*/
-- q1.find how much amount spent by each customers on artist? write a query to return customer name,
-- artist name and total spend 

-- this a temporary query through which a temporary column best_selling_artist will be created
-- 1 = artist_id 
-- 3 = SUM(il.unit_price*il.quantity) as total_sales
with best_selling_artist as (
select a.artist_id as artist_id, a.name as artist_name,
SUM(il.unit_price*il.quantity) as total_sales
from invoice_line as il
join track as t on t.track_id = il.track_id
join album as al on al.album_id = t.album_id 
join artist as a on a.artist_id = al.artist_id
group by 1
order by 3 desc 
limit 1
)
-- customer detail 
select c.customer_id, c.first_name, c.last_name, bsa.artist_name, 
sum(il.unit_price*il.quantity) as amount_spent
from invoice as i 
join customer as c on c.customer_id = i.customer_id
join invoice_line as il on il.invoice_id = i.invoice_id
join track as t on t.track_id = il.track_id
join album as al on al.album_id = t.album_id
join best_selling_artist bsa on bsa.artist_id = al.artist_id
group by 1,2,3,4
order by 5 desc;


-- Q2: We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre 
-- with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where 
-- the maximum number of purchases is shared return all Genres.

with popular_genre as (
select count(il.quantity) as purchases, customer.country, g.name, g.genre_id,
row_number() over(partition by customer.country order by count(il.quantity)desc) as row_no
from invoice_line as il 
join invoice as i on i.invoice_id = il.invoice_id
join customer on customer.customer_id  = i.customer_id
join track as t on t.track_id = il.track_id
join genre as g on g.genre_id = t.genre_id
group by 2,3,4
order by 2 asc, 1 desc 
)
select * from popular_genre where row_no <=1 

-- Q3: Write a query that determines the customer that has spent the most on music for each country. 
-- Write a query that returns the country along with the top customer and how much they spent. 
-- For countries where the top amount spent is shared, provide all customers who spent this amount.
WITH Customter_with_country AS (
		SELECT c.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice as i
		JOIN customer as c ON c.customer_id = i.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customter_with_country WHERE RowNo <= 1
