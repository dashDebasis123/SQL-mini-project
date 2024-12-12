# SQL Questions

## These questions are related to the Music datasets and I have put insightful output to the given query 

> All these questions are taken from a YouTube tutorial and most of the answers are answered by me.
>
> I did not include the easy question as it is very easy to solve and not so important to put it in the GitHub community
### Moderate-Level Questions

1. _Write a query to return the email, first name, last name, & Genre of all Rock Music listeners. Return your list ordered alphabetically by email starting with A_

Ans-
 ```
select distinct email, first_name, last_name 
from customer
join invoice on customer.customer_id = invoice.customer_id
join invoice_line on invoice.invoice_id = invoice_line.invoice_id
where track_id in (
    SELECT track_id from track
    join genre on track.genre_id = genre.genre_id
    where genre.name LIKE 'Rock')
order by email; 
   ```
_2. Let's invite the artists who have written the most rock music in our dataset. Write a query that returns the Artist name and total track count of the top 10 rock bands_

Ans-
```
SELECT artist.artist_id, artist.name , count(track.track_id) as total_tracks
from artist
JOIN album ON artist.artist_id = album.artist_id
JOIN track on track.album_id = album.album_id
JOIN genre ON genre.genre_id = track.genre_id
where genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY total_tracks DESC
limit 10
```
_3. Return all the track names that have a song length longer than the average song length. Return the Name and Millisecond for each track. Order by the song length with the longest songs listed first_

Ans-
```
SELECT track_id, name, milliseconds 
FROM track
where milliseconds > (
    SELECT avg(milliseconds) as avg_track_length
    from track
    )
ORDER BY milliseconds DESC
```

### Advance Level Questions
_1. Find how much each customer spent on artists. Write a query to return the customer name, artist name, and total spent._

Ans- 
```
with best_selling_artist as (
    SELECT artist.artist_id as artist_id, artist.name as artist_name, sum (invoice_line.unit_price *invoice_line.quantity) as amount_spent
    FROM invoice_line
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN album ON album.album_id = track.album_id
    JOIN artist ON artist.artist_id = album.artist_id
    GROUP BY 1
    ORDER BY 3 DESC
    LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, sum(il.unit_price*il.quantity) as amount_spent 
from invoice i
JOIN customer c ON i.customer_id = c.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN  album alb ON alb.album_id = t.album_id
join best_selling_artist bsa on bsa.artist_id = alb.artist_id
GROUP by 1,2,3,4
ORDER BY 5 DESC
```

_2. We want to find out each country's most popular music Genre. We determine the most popular genre as the genre with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where the maximum number of purchases is shared return all Genres._

Ans-
```
with best_genre as (SELECT i.billing_country as country, gn.name genre,  count(il.quantity),
    ROW_NUMBER() OVER(PARTITION BY i.billing_country order by count(il.quantity) DESC) AS RowNo
    FROM invoice i
    JOIN invoice_line il  ON il.invoice_id =  i.invoice_id
    JOIN track tr ON tr.track_id = il.track_id
    JOIN  genre gn ON gn.genre_id = tr.genre_id
    GROUP BY gn.genre_id, billing_country, gn.name
    ORDER BY 1 ASC)
SELECT * from best_genre 
where RowNo <= 1

```
_3. Write a query that determines the customer that has spent the most on music for each country. Write a query that returns the country along with the top customers and how much they spent. For countries where the top amount spent is shared, provide all customers who spent this amount._

Ans-
```
with best_customer as (
    SELECT c.customer_id, c.first_name, c.last_name, c.country, SUM(i.total) as total_spent, 
    ROW_NUMBER() OVER(PARTITION BY c.country ORDER BY SUM(i.total) DESC) as RowNo
    FROM customer c
    JOIN invoice i ON i.customer_id = c.customer_id
    GROUP BY 1,2,3,4
    ORDER BY 4 ASC, 5 DESC
)
SELECT * FROM best_customer WHERE RowNo <= 1

```
