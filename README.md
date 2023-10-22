# Music_SQL_Project

Used JOINS, CTE (Common Table Expressions), Subqueries to analyse the sales data.

select * from artist;
select * from customer;
select * from album;
select * from employee;
select * from track;
select * from invoice;
select * from invoice_line;
select * from playlist;
select * from playlist_track;

--1. Who is the senior-most employee based on Job Title?

select first_name, last_name, levels from employee
order by levels desc
limit 1;

--2. Which countries have the most invoices?

select count(invoice_id) as Total_invoices, billing_country from invoice 
group by billing_country
order by count(invoice_id) desc;

--3. What are the top 3 values of total invoices?

select total from invoice
order by total DESC
limit 3;

--4. Which city has the best customers? We would like to throw a promotional music festival in the city we made the most money. Write a query that returns one city that has the higherst sum of invoice totals. Return both the city name and sum of all invoice totals. 
select round(sum(total)) as Sum_Invoices, billing_city 
from invoice
group by 2
order by Sum_Invoices desc
limit 1;

--5. Who is the best customer? The customer who has spent the most money will be declared the best customer. Write the query that returns the person who spent the most money.

select c.first_name, c.last_name, sum(i.total) as Sum_Invoice from customer c
join invoice i on i.customer_id = c.customer_id
group by 1,2
order by Sum_Invoice desc
limit 1;

-- 1. Write a query to return the email, first name, last name and genre of all the Rock music listeners. Returns yout list ordered alphabetically by email starting with A

select count(c.email), c.email, c.first_name, c.last_name, gr.name from customer c
join invoice i on c.customer_id = i.customer_id
join invoice_line il on i.invoice_id = il.invoice_id
join track tr on il.track_id = tr.track_id
join genre gr on gr.genre_id = tr.genre_id
where gr.name = 'Rock'
group by c.email, c.first_name, c.last_name, gr.name
order by c.email asc;

select distinct email, first_name, last_name
from customer
join invoice on customer.customer_id = invoice.customer_id
join invoice_line on invoice_line.invoice_id = invoice.invoice_id
where track_id in
	(select track_id from track
	join genre on track.genre_id = genre.genre_id
	where genre.name like 'Rock')
order by email;

select distinct c.email, c.first_name, c.last_name, gr.name from customer c
join invoice i on c.customer_id = i.customer_id
join invoice_line il on i.invoice_id = il.invoice_id
join track tr on il.track_id = tr.track_id
join genre gr on gr.genre_id = tr.genre_id
where gr.name = 'Rock'
order by c.email asc;

-- 2. Let's invite the artists who have written the most rock music in our dataset. Write a query that returns the Artist name and total track count of the top 10 rock bands.

select ar.name, count(tr.track_id) as total from artist ar
join album al on ar.artist_id = al.artist_id
join track tr on al.album_id = tr.album_id
join genre gr on gr.genre_id = tr.genre_id
where gr.name = 'Rock'
group by ar.name
order by total desc
limit 10;

-- Returns all the track names that have a song length longer than the average song length. Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first.

select "name", milliseconds from track
where milliseconds > (select avg(milliseconds) from track)
order by milliseconds desc;

-- 1. Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent

select ar.name as artist_name, c.first_name, c.last_name, sum(il.unit_price*il.quantity) as Total_price from artist ar
join album al on ar.artist_id = al.artist_id
join track tr on al.album_id = tr.album_id
join invoice_line il on il.track_id = tr.track_id
join invoice i on i.invoice_id = il.invoice_id
join customer c on c.customer_id = i.customer_id
group by ar.name, c.customer_id
order by Total_price desc;

-- 2. We want to find out the most popular music genre for each country. We determine the most popular genre as the genre with the highest amount of purchases. Write a query that returns each country along with the top genre. For countries where the maximum number of purchases is shared return all genres

with table1 as (
				select c.country, gr.name, count(il.quantity) as total_sales,
				ROW_NUMBER() over (partition by c.country order by count(il.quantity) desc) as Row_num 
				from customer c
				join invoice i on c.customer_id = i.customer_id
				join invoice_line il on i.invoice_id = il.invoice_id
				join track tr on il.track_id = tr.track_id
				join genre gr on gr.genre_id = tr.genre_id
				group by c.country, gr.name
				order by total_sales DESC)
select * from table1 where Row_num< '2';


-- 3. Write a query that determines the customer that has spent the most on music for each country. Write a query that returns the country along with the top customer and how much they spent. For countries where the top amount spend is shared provide all customer who spent this amount.

with table2 as (
				select c.country, c.first_name, c.last_name, sum(il.unit_price*il.quantity) as Total_price,
				ROW_NUMBER() over (partition by c.country order by sum(il.unit_price*il.quantity)) as Row_num 
				from customer c
				join invoice i on c.customer_id = i.customer_id
				join invoice_line il on i.invoice_id = il.invoice_id
				join track tr on il.track_id = tr.track_id
				join genre gr on gr.genre_id = tr.genre_id
				group by 1,2,3
				order by total_price DESC
				)
select * from table2 where Row_num< '2';

