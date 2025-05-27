# SQL_Practice



![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741380705978_Screenshot+2025-03-07+at+3.51.41PM.png)



![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1743281563288_Screenshot+2025-03-29+at+4.52.36PM.png)


**Calculate Month-over-Month percent change of total amount spent for each customer:**

[![🚨 CODE WALK THROUGH](https://img.youtube.com/vi/4ncgOyXVCjc/0.jpg)](https://youtu.be/4ncgOyXVCjc)


    WITH CTE1 AS (SELECT CUSTOMER_ID
    , DATE_TRUNC('MONTH', PAYMENT_DATE) AS MNTH
    , SUM(AMOUNT) TOTAL_AMOUNT
    FROM PAYMENT
    GROUP BY 1,2)
    
    SELECT CUSTOMER_ID
    , MNTH
    , TOTAL_AMOUNT
    , LAG(TOTAL_AMOUNT) OVER(
                        PARTITION BY CUSTOMER_ID ORDER BY MNTH) 
                        LAST_MNTH_AMOUNT
    , ROUND((TOTAL_AMOUNT - LAG(TOTAL_AMOUNT) OVER(
                        PARTITION BY CUSTOMER_ID ORDER BY MNTH)  * 1.0)/ LAG(TOTAL_AMOUNT)                     OVER(
                        PARTITION BY CUSTOMER_ID ORDER BY MNTH) * 100,2) 
                        MOM_PERCENT_CHANGE
    FROM CTE1


**Percent of device sign ups of total**

      [![🚨 CODE WALK THROUGH](https://youtu.be/--J_JGyXSJI](https://youtu.be/--J_JGyXSJI?feature=shared)


    WITH CTE1 AS 
    (SELECT COUNT(CASE WHEN SIGN_UP_DEVICE = 'Smart Speaker' 
                                then 1 end) as Smart_Sp_Cnt
      , COUNT(CASE WHEN SIGN_UP_DEVICE = 'Smart TV' 
                                then 1 end) as Smart_TV_Customer_Cnt
      , COUNT(CASE WHEN SIGN_UP_DEVICE = 'Kiosk' 
                                then 1 end) as Kiosk_Customer_Cnt
      , COUNT(CASE WHEN SIGN_UP_DEVICE = 'Tablet' 
                                then 1 end) as Tablet_Customer_Cnt
      , COUNT(CASE WHEN SIGN_UP_DEVICE = 'Game Console' 
                                then 1 end) as Game_Customer_Cnt
      , COUNT(CASE WHEN SIGN_UP_DEVICE = 'Mobile' 
                                then 1 end) as Mobile_Customer_Cnt
      , COUNT(CASE WHEN SIGN_UP_DEVICE = 'Desktop' 
                                then 1 end) as Desktop_Customer_Cnt
    , (SELECT count(*) FROM CUSTOMER 
                                where cancelled_transaction = 'Processed') total_Cnt
    FROM CUSTOMER
    where cancelled_transaction = 'Processed')
    
    
    SELECT ROUND((Smart_Speaker_Customer_Count * 1.0) / TOTAL_COUNT * 100,2)Smart_PERCENT 
    , ROUND((Smart_TV_Customer_Count * 1.0) / TOTAL_COUNT * 100,2)SMART_TV__PERCENT
    , ROUND((Kiosk_Customer_Count * 1.0) / TOTAL_COUNT * 100,2)KIOSK_CUSTOMER_PERCENT
    , ROUND((Tablet_Customer_Count * 1.0) / TOTAL_COUNT * 100,2)TABLET_CUSTOMER_PERCENT
    , ROUND((Game_Console_Customer_Count * 1.0) / TOTAL_COUNT * 100,2)GAME_CONSOLE_PERCENT
    , ROUND((Mobile_Customer_Count * 1.0) / TOTAL_COUNT * 100,2)MOBILE_CUSTOMER_PERCENT
    , ROUND((Desktop_Customer_Count * 1.0) / TOTAL_COUNT * 100,2)Desktop_Customer_PERCENT
    FROM CTE1

**Repeat Purchase Rate**

      [![🚨 CODE WALK THROUGH]


    select COUNT(customer_id) MORE_THAN_ONE
    , (SELECT COUNT (DISTINCT customer_id) TOTAL_CUST_COUNT
        FROM ORDERS)
    , COUNT(customer_id) * 1.0 / (SELECT COUNT (DISTINCT customer_id) TOTAL_CUST_COUNT
    FROM ORDERS)
    from 
    (SELECT customer_id
    , COUNT(*) NUM_ORDERS_EQUAL_TO_ONE
    FROM ORDERS
    group by 1
    HAVING COUNT(*) > 1)temp

**Gross Margin by Product**

[![🚨 CODE WALK THROUGH]

    SELECT CATEGORY_NAME
    , SUM((UNIT_PRICE * QUANTITY) - (UNIT_PRICE * QUANTITY * DISCOUNT)) GROSS_PROFIT
    FROM CATEGORIES
    JOIN PRODUCTS ON PRODUCTS.CATEGORY_ID = CATEGORIES.CATEGORY_ID
    JOIN ORDER_DETAILS ON ORDER_DETAILS.PRODUCT_ID = PRODUCTS.PRODUCT_ID 
    GROUP BY 1

**Determine Sales per Region**  

[![🚨 CODE WALK THROUGH] 

    SELECT REGION_DESCRIPTION
    , ROUND(SUM(QUANTITY * UNIT_PRICE - (QUANTITY * UNIT_PRICE * DISCOUNT)),2)PRICE_AFTER_DISCOUNT
    FROM REGIONS
    JOIN TERRITORIES ON TERRITORIES.REGION_ID = REGIONS.REGION_ID
    JOIN EMPLOYEE_TERRITORIES ON EMPLOYEE_TERRITORIES.TERRITORY_ID = TERRITORIES.TERRITORY_ID
    JOIN EMPLOYEES ON EMPLOYEES.EMPLOYEE_ID = EMPLOYEE_TERRITORIES.EMPLOYEE_ID
    JOIN ORDERS ON ORDERS.EMPLOYEE_ID = EMPLOYEE_TERRITORIES.EMPLOYEE_ID
    JOIN ORDER_DETAILS ON ORDER_DETAILS.ORDER_ID = ORDERS.ORDER_ID
    JOIN PRODUCTS ON PRODUCTS.PRODUCT_ID = ORDER_DETAILS.PRODUCT_ID
    GROUP BY 1


![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741415424296_Screenshot+2025-03-08+at+1.30.18AM.png)



1. **Calculate the running total of sales for each region.**

       🚨  ***CODE WALK THROUGH****:*      **https://youtu.be/QUTaKXXhoog


    WITH sales_by_district AS ( 
      SELECT DISTRICT
      , SUM(AMOUNT) AS TOTAL_SALES 
      FROM ADDRESS 
      JOIN CUSTOMER ON CUSTOMER.ADDRESS_ID = ADDRESS.ADDRESS_ID 
      JOIN PAYMENT ON PAYMENT.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID 
      GROUP BY DISTRICT 
    ) 
    
    
      SELECT DISTRICT
      , TOTAL_SALES
      , SUM(TOTAL_SALES) 
        OVER (ORDER BY TOTAL_SALES) AS RUNNING_TOTAL 
        FROM sales_by_district;
    
                            ***** ALTERNATIVE SOLUTION BELOW ***** 
    
    SELECT DISTRICT
    , SUM(AMOUNT) TOTAL_SALES
    , SUM(
          SUM(AMOUNT)) 
          OVER(
          ORDER BY SUM(AMOUNT))
    FROM ADDRESS
    JOIN CUSTOMER ON CUSTOMER.ADDRESS_ID = ADDRESS.ADDRESS_ID
    JOIN PAYMENT ON PAYMENT.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    GROUP BY 1



2. **Rank products by their sales within each category.**

       [![🚨 CODE WALK THROUGH]    **https://youtu.be/ybvnCmnoMCA

****
    SELECT NAME
    , title
    , sum(amount) total_sales
    , RANK() OVER(
                PARTITION BY NAME 
                ORDER BY SUM(AMOUNT) DESC)
    FROM CATEGORY
    join film_category on film_category.category_id = category.category_id
    join film on film.film_id = film_category.film_id
    join inventory on inventory.film_id = film.film_id
    join rental on rental.inventory_id = inventory.inventory_id
    join payment on payment.rental_id = rental.rental_id
    group by 1,2



3. **Determine a running total of sales per day by transaction date.**

     [![🚨 CODE WALK THROUGH] https://youtu.be/sqfNppocNqw

****
    SELECT CAST(PAYMENT_DATE AS DATE)
    , SUM(AMOUNT)
    , SUM(
          SUM(AMOUNT)) 
          OVER (
                ORDER BY(CAST(PAYMENT_DATE AS DATE) )
                )
    FROM PAYMENT
    GROUP BY 1



4. **Identify the top 5 performing sales representatives by quarter.**

      [![🚨 CODE WALK THROUGH] https://youtu.be/6YTxrdM3jbY
       
****
    SELECT FIRST_NAME
    , LAST_NAME
    , CASE 
            WHEN EXTRACT(MONTH FROM PAYMENT_DATE) IN ('1','2','3') THEN 'Q1' 
            WHEN EXTRACT(MONTH FROM PAYMENT_DATE) IN ('4','5','6') THEN 'Q2' 
            WHEN EXTRACT(MONTH FROM PAYMENT_DATE) IN ('7','8','9') THEN 'Q3' 
            WHEN EXTRACT(MONTH FROM PAYMENT_DATE) IN ('10','11','12') THEN 'Q4' 
            END AS QTR
    , SUM(AMOUNT) TOTAL_SALES
    , RANK() OVER 
              (
              PARTITION BY CASE 
                  WHEN EXTRACT(MONTH FROM PAYMENT_DATE) IN ('1','2','3') THEN 'Q1' 
                  WHEN EXTRACT(MONTH FROM PAYMENT_DATE) IN ('4','5','6') THEN 'Q2' 
                  WHEN EXTRACT(MONTH FROM PAYMENT_DATE) IN ('7','8','9') THEN 'Q3' 
                  WHEN EXTRACT(MONTH FROM PAYMENT_DATE) IN ('10','11','12') THEN 'Q4' 
                  END ORDER BY SUM(AMOUNT) DESC
              ) AS rk
    FROM STAFF
    JOIN PAYMENT ON PAYMENT.STAFF_ID = STAFF.STAFF_ID
    GROUP BY 1,2,3



5. **Compute the month-over-month difference in revenue for each store.**

     [![🚨 CODE WALK THROUGH] https://youtu.be/MWYfW6XtiSg
       
****
    SELECT STAFF.STORE_ID
    , EXTRACT(MONTH FROM PAYMENT_DATE) AS MNTH
    , SUM(AMOUNT) AS TOTAL_REVENUE
    , LAG(SUM(AMOUNT)) 
                    OVER(
                    PARTITION BY STAFF.STORE_ID 
                    ORDER BY EXTRACT(MONTH FROM PAYMENT_DATE ), STAFF.STORE_ID )
    , SUM(AMOUNT) - LAG(SUM(AMOUNT)) 
                    OVER(
                    PARTITION BY STAFF.STORE_ID  
                    ORDER BY EXTRACT(MONTH FROM PAYMENT_DATE ), STAFF.STORE_ID)
    , (SUM(AMOUNT) - LAG(SUM(AMOUNT)) 
                     OVER(
                     PARTITION BY STAFF.STORE_ID 
                     ORDER BY EXTRACT(MONTH FROM PAYMENT_DATE ), STAFF.STORE_ID ))/
                     LAG(SUM(AMOUNT)) 
                     OVER(
                     PARTITION BY STAFF.STORE_ID  
                     ORDER BY EXTRACT(MONTH FROM PAYMENT_DATE ), STAFF.STORE_ID )
                     * 100 AS PERCENT_CHANGE
    FROM STAFF 
    JOIN PAYMENT ON PAYMENT.STAFF_ID = STAFF.STAFF_ID
    GROUP BY 1,2
    ORDER BY MNTH ASC



6. **Find the Longest Renting Customer**

       [![🚨 CODE WALK THROUGH] https://youtu.be/kqvNWfoSNRg
****

    WITH EARLIEST_LATEST_DATES AS (
            SELECT CUSTOMER.CUSTOMER_ID
            , MIN(RENTAL_DATE) EARLIEST_RENTAL_DATE
            , MAX(RETURN_DATE) LATEST_RETURN_DATE
            FROM CUSTOMER
            JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
            GROUP BY 1)
    
    SELECT 
        CUSTOMER_ID,
        EXTRACT(DAY 
                FROM LATEST_RETURN_DATE - EARLIEST_RENTAL_DATE) AS CUSTOMER_RENTAL_LENGTH
    FROM EARLIEST_LATEST_DATES



7. **Return customer_id, first transaction date, total spent during first transaction date, second transaction date, total spent second transaction, and percent change between first and second transaction.**

      [![🚨 CODE WALK THROUGH] 


    WITH CTE1 AS (
            SELECT RENTAL.CUSTOMER_ID
            , CAST(RENTAL_DATE AS DATE) RENTAL_DATE
            , LEAD(CAST(RENTAL_DATE AS DATE)) OVER(PARTITION BY RENTAL.CUSTOMER_ID) SECOND_RENTAL_DATE
            , ROW_NUMBER() OVER(PARTITION BY RENTAL.CUSTOMER_ID 
              ORDER BY CAST(RENTAL_DATE AS DATE)) RK
            , SUM(AMOUNT) FIRST_TRANSACTION_SPEND
            , LEAD(SUM(AMOUNT)) 
            OVER(PARTITION BY RENTAL.CUSTOMER_ID 
            ORDER BY CAST(RENTAL_DATE AS DATE)) SECOND_TRANSACTION_SPEND
            FROM RENTAL
            JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID
            GROUP BY 1,2
            ORDER BY CUSTOMER_ID, RENTAL_DATE
            ) 
    
    
    SELECT CTE1.*
    , (SECOND_TRANSACTION_SPEND / FIRST_TRANSACTION_SPEND - 1)* 100 AS PERCENT_CHANGE
    FROM CTE1
    WHERE RK = 1



![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741415527927_Screenshot+2025-03-08+at+1.32.01AM.png)



1. ***Basic Conditional Aggregation***
**

    1. **Retrieve total number of rentals & the number of rentals for 'Action' movies per customer.**

       [![🚨 CODE WALK THROUGH]     **https://youtu.be/rIAC00-HsC0

    
    SELECT FIRST_NAME
    , LAST_NAME
    , count(*) total_number_of_rentals 
    , COUNT(CASE WHEN NAME = 'Action' then 1 end) total_number_of_action
    , ROUND(CAST(COUNT(CASE WHEN NAME = 'Action' then 1 end) AS DECIMAL(9,2))/ count(*) * 100,2)
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
    JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID
    JOIN FILM_CATEGORY ON FILM_CATEGORY.FILM_ID = FILM.FILM_ID
    JOIN CATEGORY ON CATEGORY.CATEGORY_ID = FILM_CATEGORY.CATEGORY_ID
    group by 1,2


    **b. Find the total revenue from rentals and the revenue from 'Comedy' movies per store.**

       [![🚨 CODE WALK THROUGH]   **https://youtu.be/et2cO3fUakA


    SELECT STORE.STORE_ID
    , SUM(AMOUNT) AS TOTAL_REVENUE_for_Comedy
    , TOTAL_REVENUE
    FROM STORE
    JOIN STAFF ON STAFF.STORE_ID = STORE.STORE_ID
    JOIN PAYMENT ON PAYMENT.STAFF_ID = STAFF.STAFF_ID
    JOIN RENTAL ON RENTAL.RENTAL_ID = PAYMENT.RENTAL_ID
    JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
    JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID
    JOIN FILM_CATEGORY ON FILM_CATEGORY.FILM_ID = FILM.FILM_ID
    JOIN CATEGORY ON CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
    JOIN (    SELECT STORE.STORE_ID
            , SUM(AMOUNT) AS TOTAL_REVENUE
            FROM STORE
            JOIN STAFF ON STAFF.STORE_ID = STORE.STORE_ID
            JOIN PAYMENT ON PAYMENT.STAFF_ID = STAFF.STAFF_ID
            JOIN RENTAL ON RENTAL.RENTAL_ID = PAYMENT.RENTAL_ID
            JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
            JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID
            JOIN FILM_CATEGORY ON FILM_CATEGORY.FILM_ID = FILM.FILM_ID
            JOIN CATEGORY ON CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
            GROUP BY 1) TEMP ON TEMP.STORE_ID = STORE.STORE_ID
    
    WHERE NAME = 'Comedy'
    GROUP BY 1,3



    **c. Count the number of customers who have rented both 'Comedy' and 'Adventure’ movies.**

      [![🚨 CODE WALK THROUGH]    **https://youtu.be/0FiPDl4tyWE

    
    SELECT COUNT(DISTINCT CUSTOMER.CUSTOMER_ID)
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
    JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID
    JOIN FILM_CATEGORY ON FILM_CATEGORY.FILM_ID = FILM.FILM_ID
    JOIN CATEGORY ON CATEGORY.CATEGORY_ID = FILM_CATEGORY.CATEGORY_ID
    JOIN (SELECT CUSTOMER.CUSTOMER_ID
            , COUNT(*) AS NUM_Comedy
            FROM CUSTOMER
            JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
            JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
            JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID
            JOIN FILM_CATEGORY ON FILM_CATEGORY.FILM_ID = FILM.FILM_ID
            JOIN CATEGORY ON CATEGORY.CATEGORY_ID = FILM_CATEGORY.CATEGORY_ID
            where name = 'Comedy'
            GROUP BY 1) TEMP ON TEMP.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    where name = 'Adventure'
    



![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741415576905_Screenshot+2025-03-08+at+1.32.52AM.png)



    1. **Determine the most rented movie category for each customer.**

       [![🚨 CODE WALK THROUGH]    **https://youtu.be/TNEa_P_mAGk


    WITH NUM_CATEGORY_COUNT AS (SELECT FIRST_NAME
    , LAST_NAME
    , NAME
    , COUNT(*) NUM_RENTALS
    , ROW_NUMBER() OVER(PARTITION BY FIRST_NAME,LAST_NAME ORDER BY COUNT(*) DESC) RK
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
    JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID
    JOIN FILM_CATEGORY ON FILM_CATEGORY.FILM_ID = FILM.FILM_ID
    JOIN CATEGORY ON CATEGORY.CATEGORY_ID = FILM_CATEGORY.CATEGORY_ID
    GROUP BY 1,2,3
    ORDER BY FIRST_NAME)
    
    SELECT FIRST_NAME
    , LAST_NAME
    , NUM_RENTALS
    , NAME
    FROM NUM_CATEGORY_COUNT
    WHERE RK = 1



    1. **Compute the average rental duration (in days) for each film, but only consider rentals that lasted more than 3 days.**

       [![🚨 CODE WALK THROUGH]

    
    SELECT TITLE
    , AVG(CAST(RETURN_DATE AS DATE) - CAST(RENTAL_DATE AS DATE)) AS AVG_RENTAL_DURATION
    FROM FILM
    JOIN INVENTORY ON INVENTORY.FILM_ID = FILM.FILM_ID
    JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
    WHERE CAST(RETURN_DATE AS DATE) - CAST(RENTAL_DATE AS DATE) > 3
    GROUP BY 1


    3. **Categorize customers based on their total rental count:**
        - **'Frequent' (More than 30 rentals)**
        - **'Regular' (10-30 rentals)**
        - **'Occasional' (Less than 10 rentals)**

       [![🚨 CODE WALK THROUGH]

        
    SELECT FIRST_NAME ||' '|| LAST_NAME AS FULL_NAME
    , CASE 
            WHEN COUNT(*) > 30 THEN 'Frequent'
            WHEN COUNT(*) BETWEEN 10 AND 30 THEN 'Regular'
            WHEN COUNT(*) < 10 THEN 'Occasional'
            END AS CUSTOMER_RENTAL_TYPE
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    GROUP BY 1


    1. **Find the total revenue per staff member, but separate revenue from rentals before 3/14/2007 and after 3/14/2007.**

       [![🚨 CODE WALK THROUGH]  


    SELECT  FIRST_NAME ||' '||LAST_NAME AS NAME
    , SUM(CASE WHEN PAYMENT_DATE <= '2007-03-14' THEN AMOUNT END) BEFORE_THREE_FOURTEEN
    , SUM(CASE WHEN PAYMENT_DATE > '2007-03-14' THEN AMOUNT END) AFTER_THREE_FOURTEEN
    FROM STAFF
    JOIN PAYMENT ON PAYMENT.STAFF_ID = STAFF.STAFF_ID
    GROUP BY 1
    


    1. **Count how many customers have rented movies from at least 3 different categories.**

      [![🚨 CODE WALK THROUGH]

    SELECT CUSTOMER_ID
    , SUM(NUM_GENRE_MOVIES) NUM_GENRE_MOVIES
    FROM 
    (SELECT CUSTOMER.CUSTOMER_ID
    , NAME
    , COUNT(DISTINCT CASE WHEN NAME IN ('Family','Games','Animation','Documentary'
                                        ,'Classics','Sports','New','Children',
                                        'Music','Travel','Foreign','Horror','Drama',
                                        'Action','Sci-Fi','Comedy')THEN 1 END)
                                        NUM_GENRE_MOVIES
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
    JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID
    JOIN FILM_CATEGORY ON FILM_CATEGORY.FILM_ID = FILM.FILM_ID
    JOIN CATEGORY ON CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
    GROUP BY 1,2) TEMP
    GROUP BY 1
    HAVING SUM(NUM_GENRE_MOVIES) > 3


![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741415643907_Screenshot+2025-03-08+at+1.33.58AM.png)



1. **Determine how many customers have traveled to store 1 and store 2 and return the percentage of total for each store.** 

       [![🚨 CODE WALK THROUGH] 


    WITH STORE_CUST_CNT AS (SELECT COUNT(*) as STORE_NUMBER_1, 
            (SELECT COUNT(*) as STORE_NUMBER_2
            FROM STORE
            JOIN ADDRESS ON ADDRESS.ADDRESS_ID = STORE.ADDRESS_ID
            JOIN CITY ON CITY.CITY_ID = ADDRESS.CITY_ID
            JOIN COUNTRY ON COUNTRY.COUNTRY_ID = CITY.COUNTRY_ID
            JOIN CUSTOMER ON CUSTOMER.STORE_ID = STORE.STORE_ID
            WHERE STORE.STORE_ID = 2),
            (SELECT COUNT(*) TOTAL_CUSTOMER_COUNT
            FROM  STORE
            JOIN ADDRESS ON ADDRESS.ADDRESS_ID = STORE.ADDRESS_ID
            JOIN CITY ON CITY.CITY_ID = ADDRESS.CITY_ID
            JOIN COUNTRY ON COUNTRY.COUNTRY_ID = CITY.COUNTRY_ID
            JOIN CUSTOMER ON CUSTOMER.STORE_ID = STORE.STORE_ID
            WHERE STORE.STORE_ID IN (1,2))
    FROM STORE
    JOIN ADDRESS ON ADDRESS.ADDRESS_ID = STORE.ADDRESS_ID
    JOIN CITY ON CITY.CITY_ID = ADDRESS.CITY_ID
    JOIN COUNTRY ON COUNTRY.COUNTRY_ID = CITY.COUNTRY_ID
    JOIN CUSTOMER ON CUSTOMER.STORE_ID = STORE.STORE_ID
    WHERE STORE.STORE_ID = 1)
    
    SELECT STORE_NUMBER_1
    , TOTAL_CUSTOMER_COUNT
    , ROUND((STORE_NUMBER_1 * 1.0 / TOTAL_CUSTOMER_COUNT) * 100,2) PRCNT_STORE_1
    , STORE_NUMBER_2
    , TOTAL_CUSTOMER_COUNT
    , ROUND((STORE_NUMBER_2 * 1.0 / TOTAL_CUSTOMER_COUNT) * 100,2) PRCNT_STORE_2
    FROM STORE_CUST_CNT



2. **Return Rental Date, number of new customers, rolling sum of new customer count, number of customers churned, rolling sum of churning customers per day, percentage of customers who have churned**

       [![🚨 CODE WALK THROUGH] 


    WITH CTE1 AS (
            SELECT RENTAL.CUSTOMER_ID
            , MIN(CAST(RENTAL_DATE AS DATE)) AS FIRST_RENTAL_DATE
            , MAX(CAST(RENTAL_DATE AS DATE)) AS LAST_RENTAL_DATE
            FROM RENTAL
            GROUP BY 1)
    
    ,CTE2 AS (
            SELECT FIRST_RENTAL_DATE 
      , COUNT(DISTINCT cte1.CUSTOMER_ID) AS NEW_CUSTOMERS
            FROM CTE1
            JOIN RENTAL ON RENTAL.CUSTOMER_ID = CTE1.CUSTOMER_ID
            GROUP BY 1
            ORDER BY FIRST_RENTAL_DATE)
    
    ,CTE3 AS (
            SELECT LAST_RENTAL_DATE 
            , COUNT(DISTINCT cte1.CUSTOMER_ID) AS EXITING_CUSTOMERS
            FROM CTE1
            JOIN RENTAL ON RENTAL.CUSTOMER_ID = CTE1.CUSTOMER_ID
            GROUP BY 1
            ORDER BY LAST_RENTAL_DATE)
    
    ,CTE4 AS (SELECT  distinct CAST(RENTAL_DATE AS DATE) RENTAL_DATES
    FROM RENTAL 
    )
    
    SELECT RENTAL_DATES
    , COALESCE(NEW_CUSTOMERS,0) NEW_CUSTOMER_CNT
    , SUM(NEW_CUSTOMERS) 
              OVER(ORDER BY(RENTAL_DATES)) SUM_NEW_CUSTOMER_COUNT_PER_DAY
    , COALESCE(EXITING_CUSTOMERS, 0) CHURNING_CUSTOMER_CNT
    , COALESCE(SUM(EXITING_CUSTOMERS) 
               OVER(ORDER BY(RENTAL_DATES)), 0) SUM_EXITING_CUSTOMER_COUNT_PER_DAY
    , COALESCE(SUM(NEW_CUSTOMERS) 
              OVER(ORDER BY(RENTAL_DATES)),0) - COALESCE(SUM(EXITING_CUSTOMERS) 
              OVER(ORDER BY(RENTAL_DATES)),0) Total_number_of_customers_per_day 
            , round(COALESCE(EXITING_CUSTOMERS, 0) / (COALESCE(EXITING_CUSTOMERS, 0) +
    COALESCE(SUM(NEW_CUSTOMERS) 
              OVER(ORDER BY(RENTAL_DATES)),0) - COALESCE(SUM(EXITING_CUSTOMERS) 
              OVER(ORDER BY(RENTAL_DATES)),0)),2)
    FROM CTE4
    LEFT JOIN CTE2 ON CTE2.FIRST_RENTAL_DATE = CTE4.RENTAL_DATES
    LEFT JOIN CTE3 ON CTE3.LAST_RENTAL_DATE = CTE4.RENTAL_DATES
    ORDER BY RENTAL_DATES


2. **Get the top 10 best selling items for each product group and how much each item contributed to the total sales of the product group.** 
    **Expected output columns: TITLE | NAME |  NUM_RENTALS | %_of_product_group_sales**

       [![🚨 CODE WALK THROUGH] 


    
    WITH CTE1 AS (
          SELECT TITLE
          , NAME
          , COUNT(*) NUM_RENTAL
          , ROW_NUMBER() OVER(PARTITION BY NAME ORDER BY COUNT(*) DESC) RK
          , SUM(AMOUNT) SALE
          FROM CATEGORY
          JOIN FILM_CATEGORY ON FILM_CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
          JOIN FILM ON FILM.FILM_ID = FILM_CATEGORY.FILM_ID
          JOIN INVENTORY ON INVENTORY.FILM_ID = FILM.FILM_ID
          JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
          JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID
          GROUP BY 1,2
            )
    
    
    ,CTE2 AS (
          SELECT NAME
          ,SUM(AMOUNT) TOTAL_SALES_OF_PRODUCT
          FROM CATEGORY
          JOIN FILM_CATEGORY ON FILM_CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
          JOIN FILM ON FILM.FILM_ID = FILM_CATEGORY.FILM_ID
          JOIN INVENTORY ON INVENTORY.FILM_ID = FILM.FILM_ID
          JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
          JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID
          GROUP BY 1
          )
    
    SELECT TITLE
    , CTE1.NAME
    , NUM_RENTAL
    , ROUND((SALE / TOTAL_SALES_OF_PRODUCT) * 100,2)PERCENT_OF_PRODUCT_GROUP_SALES
    FROM CTE1 
    JOIN CTE2 ON CTE2.NAME = CTE1.NAME
    WHERE RK IN (1,2,3,4,5,6,7,8,9,10)
    
    


2. **Return Store Number, Month, Total_revenue for Store 1, percent change for store 1, store 2, total revenue for store 2, percent change for store 2.**

      [![🚨 CODE WALK THROUGH] 


    with cte1 as (
              SELECT STORE.STORE_ID as STORE_1
            , EXTRACT(MONTH FROM PAYMENT_DATE) AS MNTH
            , SUM(AMOUNT) AS TOTAL_REVENUE_STORE_1
            , LAG(SUM(AMOUNT)) OVER(
                        ORDER BY EXTRACT(MONTH FROM PAYMENT_DATE) ) PREV_MONTH_REVENUE
            , ROUND((SUM(AMOUNT) - LAG(SUM(AMOUNT)) OVER(
                        ORDER BY EXTRACT(MONTH FROM PAYMENT_DATE) ) )
                        / SUM(AMOUNT) * 100,2) PERCENT_CHANGE_STORE_1
    FROM STORE
    JOIN STAFF ON STAFF.STORE_ID = STORE.STORE_ID
    JOIN PAYMENT ON PAYMENT.STAFF_ID = STAFF.STAFF_ID
    WHERE STORE.STORE_ID = 1
    GROUP BY 1,2
    ORDER BY MNTH)
    
    , cte2 as (SELECT STORE.STORE_ID as STORE_2
    , EXTRACT(MONTH FROM PAYMENT_DATE) AS MNTH
    , SUM(AMOUNT) AS TOTAL_REVENUE_STORE_2
    , LAG(SUM(AMOUNT)) OVER(
                       ORDER BY EXTRACT(MONTH FROM PAYMENT_DATE) ) PREV_MONTH_REVENUE
    , ROUND((SUM(AMOUNT) - LAG(SUM(AMOUNT)) OVER(
                       ORDER BY EXTRACT(MONTH FROM PAYMENT_DATE) ) )
                      / SUM(AMOUNT) * 100,2) PERCENT_CHANGE_STORE_2
    FROM STORE
    JOIN STAFF ON STAFF.STORE_ID = STORE.STORE_ID
    JOIN PAYMENT ON PAYMENT.STAFF_ID = STAFF.STAFF_ID
    WHERE STORE.STORE_ID = 2
    GROUP BY 1,2
    ORDER BY MNTH)
    
    
    
    SELECT STORE_1, CTE1.MNTH, TOTAL_REVENUE_STORE_1, PERCENT_CHANGE_STORE_1
    , STORE_2,TOTAL_REVENUE_STORE_2, PERCENT_CHANGE_STORE_2
    FROM CTE1
    JOIN CTE2 ON CTE2.MNTH = CTE1.MNTH
    



3. **Return the number of new customers and repeat customers per day.**

       [![🚨 CODE WALK THROUGH] 


    WITH FIRST_VISIT AS (
        SELECT CUSTOMER_ID
               , MIN(CAST(RENTAL_DATE AS DATE)) AS FIRST_RENTAL_DATE
        FROM RENTAL
        GROUP BY CUSTOMER_ID
    ),
    
    DAILY_VISITORS_FLAG AS (
      SELECT CAST(RENTAL.RENTAL_DATE AS DATE) AS RENTAL_DATE
      , RENTAL.CUSTOMER_ID
      , CASE WHEN 
        CAST(RENTAL.RENTAL_DATE AS DATE) = FIRST_VISIT.FIRST_RENTAL_DATE THEN 1 
              ELSE 0 END AS FIRST_TIME_VISITOR
      , CASE WHEN 
        CAST(RENTAL.RENTAL_DATE AS DATE) != FIRST_VISIT.FIRST_RENTAL_DATE THEN 1 
              ELSE 0 END AS REPEAT_VISITOR
      FROM RENTAL 
      JOIN FIRST_VISIT ON RENTAL.CUSTOMER_ID = FIRST_VISIT.CUSTOMER_ID
    )
    
    SELECT RENTAL_DATE
           , SUM(FIRST_TIME_VISITOR) AS NUM_NEW_CUSTOMERS
           , SUM(REPEAT_VISITOR) AS NUM_REPEAT_CUSTOMERS
    FROM DAILY_VISITS
    GROUP BY RENTAL_DATE
    ORDER BY RENTAL_DATE;
    


2. **Using a CTE find the Top 3 Most Rented Movies by Month.**

       [![🚨 CODE WALK THROUGH]    **https://youtu.be/UQMyxmXSuzg


    WITH MOVIE_RANK AS (
        SELECT TITLE
      , EXTRACT(MONTH FROM RENTAL_DATE) AS MNTH 
      , COUNT(*) NUM_RENTALS
      , DENSE_RANK() OVER(
                    PARTITION BY EXTRACT(MONTH FROM RENTAL_DATE) 
                    ORDER BY COUNT(*)) RK
    FROM FILM
    JOIN INVENTORY ON INVENTORY.FILM_ID = FILM.FILM_ID
    JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
    GROUP BY 1,2
    )
    
    SELECT TITLE
    ,NUM_RENTALS
    ,MNTH
    ,RK
    FROM MOVIE_RANK
    WHERE RK IN (1,2,3)
    ORDER BY MNTH, NUM_RENTALS DESC


3. **Calculate the total number of rentals and total payment amount for each customer.**
    - **Filter only active customers (active = 1).**
    - **Rank customers by their total spending in descending order.**

       [![🚨 CODE WALK THROUGH]    **https://youtu.be/8nxcLxXtgfo

    
    WITH NUM_RENTALS_PLUS_TOTAL_SPEND AS 
    (
            SELECT FIRST_NAME
            , LAST_NAME        
            , COUNT(*) NUM_RENTALS
            , SUM(PAYMENT.AMOUNT) TOTAL_SPENT
            FROM customer
            JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
            JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID
            WHERE CUSTOMER.ACTIVE = '1'
            GROUP BY 1,2
    )
    
    SELECT FIRST_NAME
    , LAST_NAME
    , TOTAL_SPENT
    , RANK() OVER(ORDER BY TOTAL_SPENT ASC)
    FROM NUM_RENTALS_PLUS_TOTAL_SPEND
    


4.  **Determine the total number of rentals for each movie.**
    - **Calculate the total revenue generated by each movie (rental_rate * number of rentals).**
    - **Rank movies by highest total revenue in descending order.**

[![🚨 CODE WALK THROUGH]       

    WITH FILM_REV AS 
    (
            SELECT FILM.FILM_ID
            , RENTAL_RATE
            , COUNT(*) NUM_RENTALS
            , COUNT(*) * RENTAL_RATE AS TOTAL_REV
            FROM FILM
            JOIN INVENTORY ON INVENTORY.FILM_ID = FILM.FILM_ID
            JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
            GROUP BY 1,2
    )
    
    SELECT FILM_ID
    , TOTAL_REV
    , RANK() OVER(ORDER BY TOTAL_REV DESC) AS RK 
    FROM FILM_REV


5. **Find the most recent rental date for each movie.**
    - **Find the next rental date for each movie.**
    - **Calculate days between rentals.**
    

[![🚨 CODE WALK THROUGH]
    
    WITH RECENT_RENTALS AS 
    (
            SELECT TITLE
            , RENTAL_DATE
            , MAX(RENTAL_DATE) MOST_RECENT_RENTAL_DATE
            , LEAD(RENTAL_DATE) 
              OVER(
                    PARTITION BY TITLE 
                    ORDER BY RENTAL_DATE) NEXT_RENTAL_DATE
            FROM FILM 
            JOIN INVENTORY ON INVENTORY.FILM_ID = FILM.FILM_ID
            JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
            GROUP BY 1,2
    )
    
    SELECT TITLE
    , MOST_RECENT_RENTAL_DATE
    , NEXT_RENTAL_DATE
    ,CAST(NEXT_RENTAL_DATE AS DATE) - CAST(MOST_RECENT_RENTAL_DATE AS DATE) 
          AS NUM_DAYS_BTW_RENTALS
    FROM RECENT_RENTALS

****
6. **Determine each customer’s previous rental date.**
    - **Calculate the number of days between their current rental and their previous rental.**
    - **Filter out customers who have only rented once (i.e., exclude NULL gaps).**
    - **Sort the results by customer and rental date.**

[![🚨 CODE WALK THROUGH]
    
    WITH CUST_RENTAL AS 
    (
            SELECT RENTAL.CUSTOMER_ID
            , CAST(RENTAL_DATE AS DATE) RENTAL_DATE
            , LAG(CAST(RENTAL_DATE AS DATE)) 
            OVER(
            PARTITION BY CUSTOMER_ID 
            ORDER BY CAST(RENTAL_DATE AS DATE)) PREVIOUS_RENTAL_DATE
            FROM RENTAL 
            GROUP BY 1,2
    )
    
    , MOST_RECENT_RENTAL_DATE AS 
    (
          SELECT CUSTOMER_ID
        , COUNT(*) NUM_RENTALS
            , MAX(CAST(RENTAL_DATE AS DATE)) AS MOST_RECENT_RENTAL_DATE
            FROM RENTAL 
            GROUP BY 1
            HAVING COUNT(*) > 1
    )
    
    SELECT FIRST_NAME
            , LAST_NAME
            , CUST_RENTAL.CUSTOMER_ID
            , MOST_RECENT_RENTAL_DATE.MOST_RECENT_RENTAL_DATE
            , CUST_RENTAL.PREVIOUS_RENTAL_DATE
            , MOST_RECENT_RENTAL_DATE - PREVIOUS_RENTAL_DATE AS DAYS_DIFFERERNCE
            FROM CUST_RENTAL 
            JOIN MOST_RECENT_RENTAL_DATE 
            ON MOST_RECENT_RENTAL_DATE.CUSTOMER_ID = CUST_RENTAL.CUSTOMER_ID 
            AND CUST_RENTAL.RENTAL_DATE = MOST_RECENT_RENTAL_DATE.MOST_RECENT_RENTAL_DATE
            JOIN CUSTOMER ON CUSTOMER.CUSTOMER_ID = CUST_RENTAL.CUSTOMER_ID
    


7. **For each month of 2007, calculate what percentage of movie types that have reached at least 100$ or more in monthly sales.**

[![🚨 CODE WALK THROUGH]

    WITH OVER_100 AS 
    (
            SELECT NAME
            ,EXTRACT(MONTH FROM PAYMENT_DATE) AS MNTH 
            , SUM(AMOUNT) > 100 AS AMNT_OVER_100
            FROM CATEGORY
            JOIN FILM_CATEGORY ON FILM_ID = FILM_CATEGORY.FILM_ID
            JOIN INVENTORY ON INVENTORY.FILM_ID = FILM_CATEGORY.FILM_ID
            JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
            JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID 
            WHERE EXTRACT(YEAR FROM PAYMENT_DATE) = 2007
            GROUP BY 1,2
    )
    
    SELECT MNTH,
    100.0 * avg(CASE WHEN AMNT_OVER_100 = True THEN 1 ELSE 0 END) AS
    perc_over_100
    FROM OVER_100
    group by 1
    


8. **Return the Sales date, daily totals, running total and moving 7 day average using CTE’s**

[![🚨 CODE WALK THROUGH]       

    WITH DAILY_SALES AS (
            SELECT EXTRACT(DAY FROM PAYMENT_DATE) AS SALES_DATE
                    , SUM(AMOUNT) AS DAILY_TOTAL
            FROM PAYMENT
            GROUP BY 1        
    ),
    
    RUNNING_TOTALS AS (
            SELECT SALES_DATE
            , DAILY_TOTAL
            , SUM(DAILY_TOTAL) OVER(ORDER BY SALES_DATE) AS RUNNING_TOTAL
            FROM DAILY_SALES
    ),
    
    MOVING_AVERAGES AS (
            SELECT SALES_DATE
            , DAILY_TOTAL
            , RUNNING_TOTAL
            , AVG(DAILY_TOTAL) OVER(ORDER BY SALES_DATE ROWS 
                            BETWEEN 6 PRECEDING AND CURRENT ROW) AS MOVING_AVERAGE_7DAY
            FROM RUNNING_TOTALS
    )
    
    SELECT * FROM MOVING_AVERAGES
    


9. **Write a CTE to find the longest consecutive streak of sales for an employee.**


![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741415808534_Screenshot+2025-03-08+at+1.36.42AM.png)



1.  **Examine cases where the calculated extra earnings and extra days differ. Aggregate counts   of such occurrences.**

[![🚨 CODE WALK THROUGH]

    SELECT RENTAL_STATUS
    , TOTAL_RENTAL_COUNT
    , ROUND(COUNT(*) * 1.0 / TOTAL_RENTAL_COUNT * 100,2)
    FROM 
    (SELECT CUSTOMER.CUSTOMER_ID
    , FILM.FILM_ID
    , TITLE
    , AMOUNT
    , RENTAL_RATE
    , PAYMENT_DATE
    , RETURN_DATE
    , RENTAL_DURATION
    , (SELECT COUNT(*) FROM RENTAL) TOTAL_RENTAL_COUNT
    , CAST(RETURN_DATE AS DATE) - CAST(RENTAL_DATE AS DATE)  ACTUAL_RENTAL_DURATION 
    , (CAST(RETURN_DATE AS DATE) - CAST(RENTAL_DATE AS DATE))- RENTAL_DURATION EXTRA_DAYS
    , (CAST(RETURN_DATE AS DATE) - CAST(RENTAL_DATE AS DATE))- RENTAL_DURATION * 1.00 as late_charge_fee
    , CASE 
            WHEN (CAST(RETURN_DATE AS DATE) - CAST(RENTAL_DATE AS DATE))- RENTAL_DURATION * 1.00 < 0 THEN 'EARLY RETURN'
            WHEN (CAST(RETURN_DATE AS DATE) - CAST(RENTAL_DATE AS DATE))- RENTAL_DURATION * 1.00 = 0 THEN 'ON TIME RETURN'
            WHEN (CAST(RETURN_DATE AS DATE) - CAST(RENTAL_DATE AS DATE))- RENTAL_DURATION * 1.00 > 0 THEN 'LATE RETURN'
            WHEN RETURN_DATE IS NULL THEN 'DID NOT RETURN RENTAL' END AS RENTAL_STATUS
    FROM CUSTOMER 
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    JOIN inventory ON inventory.inventory_id = RENTAL.inventory_id
    LEFT JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID
    JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID) TEMP 
    GROUP BY 1,2



2. **Calculate the sum of replacement costs for DVDs that were not returned.** 

[![🚨 CODE WALK THROUGH]

    SELECT SUM(REPLACEMENT_COST) - SUM(AMOUNT) TOTAL_AMNT_REPLACEMENT_FOR_MOVIES_NOT_RETURNED
    FROM 
      (SELECT customer.CUSTOMER_ID
      , FILM.FILM_ID
      , TITLE
      , REPLACEMENT_COST
      , AMOUNT 
      , RENTAL_RATE
      , PAYMENT_DATE
      , RENTAL_DATE 
      , RETURN_DATE 
      , RENTAL_DURATION
      FROM customer
      JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
      JOIN inventory ON inventory.inventory_id = RENTAL.inventory_id
      LEFT JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID
      JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID 
      WHERE RETURN_DATE IS NULL)TEMP


3. **Return percentage of rentals for each day of the week in 2005.**

       🚨  ***CODE WALK THROUGH****:*  


    SELECT DAY_OF_WEEK
    , ROUND((NUMBER_RENTALS_PER_WEEK * 1.0) / TOTAL_RENTALS,3) * 100
    FROM
    (SELECT CASE 
            WHEN EXTRACT(DOW FROM RENTAL_DATE) = 6 THEN 'SATURDAY'
            WHEN EXTRACT(DOW FROM RENTAL_DATE) = 0 THEN 'SUNDAY'
            WHEN EXTRACT(DOW FROM RENTAL_DATE) = 1 THEN 'MONDAY'
            WHEN EXTRACT(DOW FROM RENTAL_DATE) = 2 THEN 'TUESDAY'
            WHEN EXTRACT(DOW FROM RENTAL_DATE) = 3 THEN 'WEDNESDAY'
            WHEN EXTRACT(DOW FROM RENTAL_DATE) = 4 THEN 'THURSDAY'
            WHEN EXTRACT(DOW FROM RENTAL_DATE) = 5 THEN 'FRIDAY'
            END AS DAY_OF_WEEK
    , COUNT(*) AS NUMBER_RENTALS_PER_WEEK
    , (SELECT COUNT(*) FROM RENTAL) TOTAL_RENTALS
    FROM RENTAL
    WHERE EXTRACT(YEAR FROM RENTAL_DATE) = 2005
    GROUP BY 1
    ORDER BY DAY_OF_WEEK) TEMP


4. **Find the number of customers below and above average total amount customer spend** 

[![🚨 CODE WALK THROUGH]

    select CASE 
              WHEN AVG_TOTAL_SPEND < TOTAL_SPENT 
              THEN 'ABOVE AVG' ELSE 'BELOW AVG' 
              END AS BELOW_OR_ABOVE
    , COUNT(*)
    FROM 
            (SELECT PAYMENT.customer_id
            , sum(amount) total_spent
            , AVG(SUM(AMOUNT)) OVER () AVG_TOTAL_SPEND
            FROM PAYMENT
            GROUP BY 1) temp
    GROUP BY 1



5. **Find products priced above the average price in their category.**

[![🚨 CODE WALK THROUGH]

    SELECT CATEGORY.NAME
    , TITLE
    , RENTAL_RATE
    , AVG_RENTAL_RATE
    FROM CATEGORY
    JOIN FILM_CATEGORY ON FILM_CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
    JOIN FILM ON FILM.FILM_ID = FILM_CATEGORY.FILM_ID
    JOIN  (
          SELECT AVG(RENTAL_RATE) AVG_RENTAL_RATE
                    , NAME
            FROM CATEGORY
            JOIN FILM_CATEGORY ON FILM_CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
            JOIN FILM ON FILM.FILM_ID = FILM_CATEGORY.FILM_ID
            GROUP BY 2
           ) TEMP ON TEMP.NAME = CATEGORY.NAME
    WHERE RENTAL_RATE > AVG_RENTAL_RATE

****
6. **List customers who placed orders in every month of the year.**

[![🚨 CODE WALK THROUGH]

    SELECT CUSTOMER_ID
    ,COUNT(*)
    FROM 
    (
            SELECT CUSTOMER_ID
          , EXTRACT(MONTH FROM RENTAL_DATE) AS MNTH
          , EXTRACT(YEAR FROM RENTAL_DATE) AS YR
          , COUNT(*) NUM_RENTALS
            FROM RENTAL
            WHERE CAST(RENTAL_DATE AS DATE) BETWEEN '2005-06-01' AND '2006-01-31'
            GROUP BY 1,2,3
    )
    GROUP BY 1
    HAVING COUNT(*) > 7
    ORDER BY CUSTOMER_ID DESC


7. **Find customer with lowest average time between rentals.**

       🚨  ***CODE WALK THROUGH****:*  


    SELECT CUSTOMER_ID 
    , AVG(TIME_BTW_RENTALS) AVG_TIME_BTW_RENTALS
    FROM
    (
            SELECT CUSTOMER.CUSTOMER_ID
            , RENTAL_DATE
            , LEAD(RENTAL_DATE) OVER(
                                PARTITION BY CUSTOMER.CUSTOMER_ID 
                                ORDER BY RENTAL_DATE) 
            , LEAD(RENTAL_DATE) OVER(
                                PARTITION BY CUSTOMER.CUSTOMER_ID ORDER BY RENTAL_DATE) - 
                                RENTAL_DATE  TIME_BTW_RENTALS
            FROM CUSTOMER
            JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    ) TEMP 
    GROUP BY 1
    ORDER BY AVG_TIME_BTW_RENTALS ASC


8. **Calculate the percentage of total sales contributed by each product.**

[![🚨 CODE WALK THROUGH]

    SELECT NAME
    , SUM(percentage_of_total)
    FROM 
    (SELECT NAME, 
           (AMOUNT / (SELECT SUM(AMOUNT) FROM PAYMENT)) * 100 AS percentage_of_total
    FROM CATEGORY
    JOIN FILM_CATEGORY ON FILM_CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
    JOIN FILM ON FILM.FILM_ID = FILM_CATEGORY.FILM_ID
    JOIN INVENTORY ON INVENTORY.FILM_ID = FILM.FILM_ID
    JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
    JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID)TEMP
    GROUP BY 1


9. **Return each customer and the store in which they rented from the most.**

       🚨  ***CODE WALK THROUGH****:*  


    SELECT CUSTOMER_ID
    , STORE_ID
    FROM 
    (SELECT CUSTOMER.CUSTOMER_ID
    , STAFF.STORE_ID
    , COUNT(*) AS NUMBER_OF_RENTALS
    , RANK() OVER(PARTITION BY CUSTOMER.CUSTOMER_ID ORDER BY COUNT(*) DESC) RK
    FROM RENTAL
    JOIN CUSTOMER ON CUSTOMER.CUSTOMER_ID = RENTAL.CUSTOMER_ID
    JOIN STAFF ON STAFF.STAFF_ID = RENTAL.STAFF_ID
    GROUP BY 1,2) TEMP
    WHERE RK = 1


10. **Get the top 3 DVDS in revenue by category for top 3 markets in the US in 2005.** 

[![🚨 CODE WALK THROUGH]

    
    SELECT NAME
    , TITLE 
    , TOTAL_REVENUE
    FROM 
    (.      SELECT NAME
            , TITLE
            , SUM(PAYMENT.AMOUNT) AS TOTAL_REVENUE
            , RANK() OVER(PARTITION BY NAME ORDER BY SUM(PAYMENT.AMOUNT) DESC) AS RK 
            FROM CATEGORY
            JOIN FILM_CATEGORY ON FILM_CATEGORY.CATEGORY_ID = CATEGORY.CATEGORY_ID
            JOIN FILM ON FILM.FILM_ID = FILM_CATEGORY.FILM_ID
            JOIN INVENTORY ON INVENTORY.FILM_ID = FILM.FILM_ID
            JOIN RENTAL ON RENTAL.INVENTORY_ID = INVENTORY.INVENTORY_ID
            JOIN PAYMENT ON PAYMENT.RENTAL_ID = RENTAL.RENTAL_ID
            JOIN CUSTOMER ON CUSTOMER.CUSTOMER_ID = RENTAL.CUSTOMER_ID
            JOIN ADDRESS ON ADDRESS.ADDRESS_ID = CUSTOMER.ADDRESS_ID
            JOIN CITY ON CITY.CITY_ID = ADDRESS.CITY_ID 
            JOIN COUNTRY ON COUNTRY.COUNTRY_ID = CITY.COUNTRY_ID
    JOIN (
            SELECT COUNTRY
            ,SUM(AMOUNT) REV_BY_COUNTRY
            FROM COUNTRY
            JOIN CITY ON CITY.COUNTRY_ID = COUNTRY.COUNTRY_ID
            JOIN ADDRESS ON ADDRESS.CITY_ID = CITY.CITY_ID 
            JOIN CUSTOMER ON CUSTOMER.ADDRESS_ID = ADDRESS.ADDRESS_ID
            JOIN PAYMENT ON PAYMENT.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
            GROUP BY 1
            ORDER BY REV_BY_COUNTRY DESC 
            LIMIT 3) TEMP ON TEMP.COUNTRY = COUNTRY.COUNTRY
    WHERE EXTRACT(YEAR FROM RENTAL_DATE) = 2005
    GROUP BY 1,2)
    WHERE RK IN (1,2,3)
    


![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741415948533_Screenshot+2025-03-08+at+1.39.01AM.png)


**1. For the genres Family and Animation, what is the average number of actors per film and** 
   **the total number of rentals for each of the genres.**
   
[![🚨 CODE WALK THROUGH]   

    select round(avg(numActor),1) avgNumActorPerFilm
    , numActorInFilm.name
    ,numRentals
    from
            (select category.name
            ,film.title
            ,COUNT(film_actor.actor_id) numActor
            from category
            join film_category on film_category.category_id = category.category_id
            join film on film.film_id = film_category.film_id
            join film_actor on film_actor.film_id = film.film_id
            where name in ('Animation', 'Family')
            group by 1,2) as numActorInFilm
    join
            (select category.name
            ,count(rental_id) numRentals
            from category
            join film_category on film_category.category_id = category.category_id
            join film on film.film_id = film_category.film_id
            join inventory on inventory.film_id = film.film_id
            join rental on rental.inventory_id = inventory.inventory_id
            where name in ('Animation', 'Family')
            group by 1) countPerGenre
            on countperGenre.name = numactorinFilm.name
    group by 2,3

   


2. **Find Customers who have rented same DVD more than once.** 

[![🚨 CODE WALK THROUGH]

    SELECT CUSTOMER.CUSTOMER_ID
    , FILM_ID
    , COUNT(*) NUM_RENTALS
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
    GROUP BY 1,2
    HAVING COUNT(*) > 1


****
1. **Find the average number of DVDs rented per customer.**

[![🚨 CODE WALK THROUGH]

    SELECT AVG(TOTAL_NUM_RENTALS)
    FROM 
    (        SELECT CUSTOMER.CUSTOMER_ID
            , COUNT(*) TOTAL_NUM_RENTALS
            FROM CUSTOMER
            JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
            GROUP BY 1
    )TEMP



![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741416044157_Screenshot+2025-03-08+at+1.40.37AM.png)



2. **Retrieve the first letter of the first name and last name of each customer, concatenated together as their initials.  Ensure that the initials are displayed in uppercase.**

       🚨  ***CODE WALK THROUGH****:*  


    SELECT  CONCAT(LEFT(FIRST_NAME,1)
    , LEFT(LAST_NAME,1)) AS CUSTOMER_INITIAL
    , COUNT(*) as num_rentals
    , COUNT(DISTINCT CUSTOMER.CUSTOMER_ID)
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    GROUP BY 1
    


1. **Write a query that returns the number of customers who have rented a movie for fifteen consecutive days.**

[![🚨 CODE WALK THROUGH]

    WITH daily_rental_counts AS (
        SELECT customer_id, COUNT(DISTINCT DATE(rental_date)) AS rental_days
        FROM rental
        WHERE rental_date BETWEEN '2006-03-01' AND '2006-03-15'
        GROUP BY customer_id
    )
    SELECT COUNT(customer_id) AS total_active_customers
    FROM daily_rental_counts
    WHERE rental_days = 15;  
    


1. **Find the second-highest paying customer in a table without using LIMIT or TOP.**

[![🚨 CODE WALK THROUGH]
****
    SELECT CUSTOMER_ID
    , AMOUNT_PAID 
    FROM 
    (
          SELECT CUSTOMER.CUSTOMER_ID
        , SUM(AMOUNT) AS AMOUNT_PAID
        , ROW_NUMBER() OVER(ORDER BY SUM(AMOUNT) DESC ) AS RK
          FROM CUSTOMER
          JOIN PAYMENT ON PAYMENT.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
          GROUP BY 1
          ORDER BY AMOUNT_PAID DESC
    ) TEMP
    WHERE RK = 2


****
2. **Find the duplicate rows in a table without using GROUP BY.**

[![🚨 CODE WALK THROUGH]

    SELECT CUSTOMER_ID
    , RENTAL_ID
    , RK
    FROM
    (
            SELECT CUSTOMER_ID
            , RENTAL_ID
            , RANK() OVER(PARTITION BY CUSTOMER_ID ORDER BY RENTAL_ID DESC) RK 
              FROM RENTAL
    ) TEMP
    WHERE RK > 1 


****
4. **Write a SQL query to find the top 10% spenders in a table.**

[![🚨 CODE WALK THROUGH]

    SELECT CUSTOMER_ID
    , TOTAL_SPENT 
    , RK 
    FROM 
    (
        SELECT CUSTOMER.CUSTOMER_ID
      , SUM(AMOUNT) AS TOTAL_SPENT
      , PERCENT_RANK() OVER(ORDER BY SUM(AMOUNT) DESC ) RK 
        FROM CUSTOMER
        JOIN PAYMENT ON PAYMENT.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
        GROUP BY 1
        ORDER BY TOTAL_SPENT DESC
    ) TEMP
    WHERE RK <= 0.10


5. **Find customers who have rented more than 1 DVD.**

[![🚨 CODE WALK THROUGH]

    SELECT CUSTOMER.CUSTOMER_ID
    , FILM_ID
    , COUNT(*) NUM_RENTALS
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
    GROUP BY 1,2
    HAVING COUNT(*) > 1



6. **Show the percent of movies that are 'drama’  as their movie type. Round the answer to the nearest hundredth number and in percent form.**

[![🚨 CODE WALK THROUGH]

    SELECT ROUND((1.0 * COUNT(CASE WHEN NAME = 'Drama' THEN 1 END) 
                  / COUNT(*)) * 100,2) 
                  || '%'AS percent_of_drama
     FROM CATEGORY

****
7. **Write a SQL query to find all employees who have taken more than 2 weeks between rentals.**

       🚨  ***CODE WALK THROUGH****:*  

****
    SELECT CUSTOMER_ID
    , TIME_DIFF_BTW_RENTALS
    FROM 
    (        SELECT CUSTOMER.CUSTOMER_ID
            , RENTAL_DATE
            , LAG(RENTAL_DATE) OVER(ORDER BY RENTAL_DATE)
            , RENTAL_DATE -  LAG(RENTAL_DATE) OVER(
                                              ORDER BY RENTAL_DATE) 
                                              TIME_DIFF_BTW_RENTALS
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    GROUP BY 1,2
    ORDER BY CUSTOMER.CUSTOMER_ID) TEMP
    WHERE EXTRACT(DAY FROM TIME_DIFF_BTW_RENTALS) >= 14


8. **Return the number of movies that were rented per category and the percentage of number of movie rentals per category out of the total number of movies rented.** 

[![🚨 CODE WALK THROUGH]

    select category.name
    ,count(*) as numRentals
    ,( count(*)*1.0) / (select count(*) from rental) * 100 proportionofRentalsPerCat
      from category
    join film_category on film_category.category_id = category.category_id
    join film on film.film_id = film_category.film_id
    join inventory on inventory.film_id = film.film_id
    join rental on rental.inventory_id = inventory.inventory_id
    group by 1




![](https://paper-attachments.dropboxusercontent.com/s_CE3EBDF199A73624B3941B85D90E3041173FCC28028C76BA957E79B3B92ABFF3_1741416076528_Screenshot+2025-03-08+at+1.41.10AM.png)

1. **Calculate the genres  that are frequently rented together.**

       🚨  ***CODE WALK THROUGH****:*  


    WITH CTE1 AS (SELECT CUSTOMER_ID
        , CAST(RENTAL_DATE AS DATE) DT
        , CONCAT(MAX(CASE WHEN NAME = 'Family' THEN 'Family' END)
        , MAX(CASE WHEN NAME = 'Games' THEN 'Games' END) 
        , MAX(CASE WHEN NAME = 'Animation' THEN 'Animation' END)
        , MAX(CASE WHEN NAME = 'Documentary' THEN 'Documentary' END)
        , MAX(CASE WHEN NAME = 'Classics' THEN 'Classics' END) 
        , MAX(CASE WHEN NAME = 'Sports' THEN 'Sports' END) 
        , MAX(CASE WHEN NAME = 'New' THEN 'New' END) 
        , MAX(CASE WHEN NAME = 'Children' THEN 'Children' END) 
        , MAX(CASE WHEN NAME = 'Music' THEN 'Music' END) 
        , MAX(CASE WHEN NAME = 'Travel' THEN 'Travel' END)
        , MAX(CASE WHEN NAME = 'Foreign' THEN 'Foreign' END) 
        , MAX(CASE WHEN NAME = 'Horror' THEN 'Horror' END) 
        , MAX(CASE WHEN NAME = 'Drama' THEN 'Drama' END) 
        , MAX(CASE WHEN NAME = 'Action' THEN 'Action' END) 
        , MAX(CASE WHEN NAME = 'Sci-Fi' THEN 'Sci-Fi' END) 
        , MAX(CASE WHEN NAME = 'Comedy' THEN 'Comedy' END)) MULTIPLE_GENRE
    FROM RENTAL
    JOIN INVENTORY ON INVENTORY.INVENTORY_ID = RENTAL.INVENTORY_ID
    JOIN FILM ON FILM.FILM_ID = INVENTORY.FILM_ID
    JOIN FILM_CATEGORY ON FILM_CATEGORY.FILM_ID = FILM.FILM_ID
    JOIN CATEGORY ON CATEGORY.CATEGORY_ID = FILM_CATEGORY.CATEGORY_ID
    GROUP BY 1,2)
    
    SELECT MULTIPLE_GENRE
    , COUNT(*)
    FROM CTE1
    GROUP BY 1
    ORDER BY COUNT(*) DESC


****
2. **Calculate number of Comedy & Documentary movies each customer has rented.**

[![🚨 CODE WALK THROUGH]     https://youtu.be/3EwOJm6K1Ys


    SELECT 
        c.FIRST_NAME,
        c.LAST_NAME,
        COUNT(CASE 
              WHEN category.NAME = 'Comedy' 
              THEN 1 END) AS COMEDY_NUM_RENTALS,
        COUNT(CASE 
                WHEN category.NAME = 'Documentary' 
                THEN 1 END) AS DOCUMENTARY_NUM_RENTALS
    FROM CUSTOMER c
    JOIN rental r ON r.customer_id = c.customer_id
    JOIN inventory i ON i.inventory_id = r.inventory_id
    JOIN film f ON f.film_id = i.film_id
    JOIN film_category fc ON fc.film_id = f.film_id
    JOIN category ON category.category_id = fc.category_id
    WHERE category.NAME IN ('Comedy', 'Documentary')
    GROUP BY c.FIRST_NAME, c.LAST_NAME;


2. **Return month, number of active users per month number of non_active users per month and the percent of active users in 2005.**

[![🚨 CODE WALK THROUGH]   **https://youtu.be/6lAIKzQ2NIc


    SELECT EXTRACT(MONTH FROM RENTAL_DATE) MNTH
    , COUNT(CASE WHEN ACTIVE = 1 THEN 1 END) NUMBER_ACTIVE
    , COUNT(CASE WHEN ACTIVE = 0 THEN 1 END) NUMBER_NON_ACTIVE
    , ROUND(COUNT(CASE 
                    WHEN ACTIVE = 1 THEN 1 END) * 1.0 / 
        (COUNT(CASE 
                  WHEN ACTIVE = 1 THEN 1 END) + COUNT(CASE 
                  WHEN ACTIVE = 0 THEN 1 END)),2)
                  PERCENT_ACTIVE 
    
    FROM CUSTOMER
    JOIN RENTAL ON RENTAL.CUSTOMER_ID = CUSTOMER.CUSTOMER_ID
    WHERE EXTRACT(YEAR FROM RENTAL_DATE) = 2005
    GROUP BY 1






