# Digital-Music-Store-Analysis

##Project Objective 
 Analyze the operations of a digital music store to uncover business insights related to customer behavior, sales trends, and product popularity.


--Data Used--
 - <a href=" https://github.com/sukritideb28/Digital-Music-Store-Analysis/commit/010bed0644e8e49d5d1934966148e924db6b76ed">Dataset</a>


 -- Problem Statements--
 
--Q1-- Who is the senior most employee based on job title 
 SELECT * FROM EMPLOYEE$;

 SELECT  TOP 1 * FROM EMPLOYEE$
 ORDER BY LEVELS DESC ;
 
 --Q2-- Which countries has the most invoices ?

 SELECT * FROM INVOICE$;

 select COUNT(*) AS C, BILLING_COUNTRY
 FROM INVOICE$
 GROUP BY billing_country
 order by C DESC;

 --Q3--What are top 3 values of total invoice ?

 SELECT * FROM INVOICE$;

 SELECT TOP 3 TOTAL FROM INVOICE$
 ORDER BY TOTAL DESC;

 --Q4-- Which city has the best customers ? We would like throw a promotional Music Festival in the city,
      -- we made the most money.Write a query that returns one city that has the hihest sum of invoice total.
	  --Return both the city name and sum of all invoice totals.

	  SELECT * FROM INVOICE$;

	  SELECT SUM(TOTAL) AS 'INVOICE_TOTALS' ,BILLING_CITY 
	  FROM INVOICE$
	  GROUP BY BILLING_CITY
	  ORDER BY INVOICE_TOTALS DESC;

  --Q5-- Who is the best customer ? The customer who has spent the most money will be declared as the best customer?
     --Write a query that returns the person who has spent the most money.

	 SELECT * FROM CUSTOMER$;

	 SELECT TOP 1 CUSTOMER$.CUSTOMER_ID ,CUSTOMER$.FIRST_NAME , CUSTOMER$.LAST_NAME ,SUM(INVOICE$.TOTAL) AS 'TOTAL'
	 FROM CUSTOMER$
	 JOIN INVOICE$ ON CUSTOMER$.CUSTOMER_ID = INVOICE$.CUSTOMER_ID
	 GROUP By CUSTOMER$.CUSTOMER_ID,CUSTOMER$.FIRST_NAME,CUSTOMER$.LAST_NAME
	 ORDER BY TOTAL DESC;

	       --MODERATE--

  --Q1-- Write a query to return the email,first name , last name and genre of all Rock Music Listenrs.Return your
       -- list ordered alphabetically by email starting with A 
 
  SELECT DISTINCT EMAIL,FIRST_NAME,LAST_NAME
 FROM CUSTOMER$
 JOIN INVOICE$ ON CUSTOMER$.CUSTOMER_ID = INVOICE$.CUSTOMER_ID
 JOIN INVOICE_LINE$ ON INVOICE$.INVOICE_ID = INVOICE_LINE$.INVOICE_ID
 WHERE TRACK_ID IN 
        (SELECT TRACK_ID FROM TRACK$
		 JOIN GENRE$ ON TRACK$.GENRE_ID =GENRE$.GENRE_ID
		 WHERE GENRE$.NAME LIKE 'ROCK'
		 )
 ORDER BY EMAIL;
     
 --Q2-- Let's ivite the artists who have written the most rock music in our dataset.Write a query that
        -- returns a Artist name and total track count of the top 10 rock bands 
   
      SELECT TOP 10 ARTIST$.ARTIST_ID ,ARTIST$.NAME ,COUNT(ARTIST$.ARTIST_ID) AS 'NUMBER_OF_SONGS'
	  FROM TRACK$
	  JOIN ALBUM$ ON ALBUM$.ALBUM_ID =TRACK$.ALBUM_ID
	  JOIN ARTIST$ ON ARTIST$.ARTIST_ID = ALBUM$.ARTIST_ID
	  JOIN GENRE$ ON GENRE$.GENRE_ID =TRACK$.GENRE_ID
	  WHERE GENRE$.NAME LIKE 'ROCK'
--Q3-- Writen all the track names that have a song lenght longer than the average song length.Return the
      -- name and Miliseconds for each track.Order by the song lenghth with the longest songs 
	  -- listed first

	   SELECT NAME,MILLISECONDS
	   FROM TRACK$
	   WHERE MILLISECONDS >
	    (SELECT AVG(MILLISECONDS) AS 'AVG_TRACK_LENGTH'
		 FROM TRACK$)
       ORDER BY MILLISECONDS DESC;


	-- ADVANCED --
 
 --Q1-- Find how much amount spent by each customer on artists? Write a query to retun customer name,
     -- artist name and total spent
	 
	    WITH BEST_SELLING_ARTIST AS (
		SELECT TOP 1 ARTIST$.ARTIST_ID AS ARTIST_ID,ARTIST$.NAME AS ARTIST_NAME,
		SUM (INVOICE_LINE$.UNIT_PRICE * INVOICE_LINE$.QUANTITY) AS TOTAL_SALES
		FROM INVOICE_LINE$
		JOIN TRACK$ ON TRACK$.TRACK_ID = INVOICE_LINE$.TRACK_ID
		JOIN ALBUM$ ON ALBUM$.ALBUM_ID =TRACK$.ALBUM_ID
		JOIN ARTIST$ ON ARTIST$.ARTIST_ID =ALBUM$.ARTIST_ID
		GROUP BY ARTIST$.ARTIST_ID,ARTIST$.NAME
		ORDER BY TOTAL_SALES DESC 
	)
	
	SELECT * FROM BEST_SELLING_ARTIST;	

	SELECT 
    C.CUSTOMER_ID,
    C.FIRST_NAME, 
    C.LAST_NAME,
    BSA.ARTIST_NAME,
    SUM(INVOICE_LINE$.UNIT_PRICE * INVOICE_LINE$.QUANTITY) AS AMOUNT_SPENT
FROM 
    INVOICE$ I
JOIN 
    CUSTOMER$ C ON C.CUSTOMER_ID = I.CUSTOMER_ID
JOIN 
    INVOICE_LINE$ IL ON IL.INVOICE_ID = I.INVOICE_ID
JOIN 
    TRACK$ T ON T.TRACK_ID = IL.TRACK_ID
JOIN 
    ALBUM$ ALB ON ALB.ALBUM_ID = T.ALBUM_ID
JOIN 
    BEST_SELLING_ARTIST BSA ON BSA.ARTIST_ID = ALB.ARTIST_ID
GROUP BY 
    C.CUSTOMER_ID, C.FIRST_NAME, C.LAST_NAME, BSA.ARTIST_NAME
ORDER BY 
    AMOUNT_SPENT DESC;

--Q2-- We want to find the most popular music genre for each country. We determine the most
    -- popular genre as the genre with the highest amount of purchases.Write a query that
	-- returns each country along with the top genre.For the countrie where the maximum
	-- number purchases is shared return all the genres.

	  WITH POPULAR_GENERE AS (
	  SELECT COUNT(INVOICE_LINE$.QUANTITY) AS PURCHASES ,CUSTOMER$.COUNTRY ,GENRE$.NAME,GENRE$.GENRE_ID,
	  ROW_NUMBER() OVER (PARTITION BY CUSTOMER$.COUNTRY ORDER BY COUNT(INVOICE_LINE$.QUANTITY) DESC) AS ROWNUm
	  FROM INVOICE_LINE$
	  JOIN INVOICE$ ON INVOICE$.INVOICE_ID =INVOICE_LINE$.INVOICE_ID
	  JOIN CUSTOMER$ ON CUSTOMER$.CUSTOMER_ID =INVOICE$.CUSTOMER_ID
	  JOIN TRACK$ ON TRACK$.TRACK_ID = INVOICE_LINE$.TRACK_ID
	  JOIN GENRE$ ON GENRE$.GENRE_ID =TRACK$.GENRE_ID
	  GROUP BY CUSTOMER$.COUNTRY, GENRE$.NAME, GENRE$.GENRE_ID
	 
)
        
 

     SELECT * FROM POPULAR_GENRE WHERE ROWNUM <=1;

	 
        




	            
