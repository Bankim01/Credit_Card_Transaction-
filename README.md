# Credit Card Transaction Analysis :-


## Project Overview : 
This Data Analysis Project aims to provide insights of customer's spending behaviour over the past years by using different types of Credit Cards.
By analyzing various aspects of the data, we seek to identify trends , make data-driven decisions and gain a deeper understanding of the company's
preformance.


## Data Source :
Credit_Card_Transcations : This is the primary dataset 'credit_card_transcations.csv' file , containing detailed information about each transaction 
made by the Credit Card holders.



![image](https://github.com/user-attachments/assets/f308efe6-9483-40b2-b844-d764fb7e0608)


## Tool used : 
- Excel
- MS SQL Server (SSMS)

## Data Analysis : 

#### Basic Data Exploration :

Included some intersting codes worked with


```sql
SELECT * FROM credit_card;
```

- Total number of records
```sql
SELECT count(*) AS no_of_records 
FROM credit_card;
```
![image](https://github.com/user-attachments/assets/ca102320-c823-42ba-8b80-57f9826da9ca)


- Distinct cards type
```sql
SELECT DISTINCT card_type AS distinct_card
FROM credit_card ;
```

![image](https://github.com/user-attachments/assets/5afbdd21-fdf4-4aab-86ed-976f72089599)


- Number of transactions by cards_type
```sql
SELECT card_type , 
	COUNT(transaction_id) AS no_of_transactions
FROM credit_card 
GROUP BY card_type;
```
![image](https://github.com/user-attachments/assets/e08cc946-f416-4332-a4c8-8bd725db2d85)

- TOP 10 Transactions by city
```sql
SELECT TOP 10 city ,
	COUNT( transaction_id ) AS no_of_transactions
FROM credit_card
GROUP BY city 
ORDER BY 2 DESC;
```
![image](https://github.com/user-attachments/assets/3e386c50-7410-48b1-813a-4f05aff3de37)


- Transactions by gender
```sql
SELECT gender , 
	COUNT(1) AS counts
FROM credit_card
GROUP BY gender 
ORDER BY  2 DESC;
```
- F-> Female and M-> Male
![image](https://github.com/user-attachments/assets/8a41c61a-2f13-4a5c-8ffe-695bbf95b36c)



## Questions from Stakeholder :

1- write a query to print top 5 cities with highest spends and their percentage contribution of 
total credit card spends?

```sql
WITH amount_spend_by_city AS (
	SELECT city , SUM(amount) AS amount_spend
	FROM credit_card
	GROUP BY city
	),
total_amount_spend AS (
	SELECT SUM( CAST(amount AS BIGINT) ) AS total_amount
	FROM credit_card
	)
SELECT TOP 5 t1.* , ((t1.amount_spend*1.0)*100/t2.total_amount) AS percent_spend
FROM amount_spend_by_city t1
INNER JOIN total_amount_spend t2
ON 1=1
ORDER BY t1.amount_spend DESC ;
```
![image](https://github.com/user-attachments/assets/ad93f631-5892-4e85-90c5-ca0e9b0c0edf)

2- write a query to print highest spend month and amount spent in that month for each card type ?

```sql
WITH cte1 AS (
	SELECT DATEPART(MONTH , transaction_date) AS month_no ,
		 DATEPART(YEAR , transaction_date) AS year_of_transaction ,
		 card_type, amount
	FROM credit_card
	),
	cte2 AS (
	SELECT card_type , month_no , year_of_transaction , SUM(amount) AS amount_spend
	FROM cte1
	GROUP BY card_type ,year_of_transaction , month_no
	),
rn AS (
	SELECT * , DENSE_RANK() OVER(PARTITION BY card_type ORDER BY amount_spend DESC) AS drnk
	FROM cte2
	)
SELECT card_type ,year_of_transaction , month_no , amount_spend
FROM rn
WHERE drnk=1
ORDER BY amount_spend DESC ;
```
![image](https://github.com/user-attachments/assets/79d01f60-d13b-4b70-acd5-438fabd83650)

3- write a query to print the transaction details(all columns from the table) for each card type when
it reaches a cumulative of 1000000 total spends(We should have 4 rows in the o/p one for each card type) 

```sql
WITH cte AS(
	SELECT * , 
	SUM(CAST(amount AS BIGINT)) OVER(PARTITION BY card_type ORDER BY transaction_date,transaction_id) AS cumm_amt
	FROM credit_card
	),
cte2 AS (
	SELECT * ,
	DENSE_RANK() OVER(PARTITION BY card_type ORDER BY cumm_amt) AS rk
	FROM cte
	WHERE cumm_amt > 1000000
	)
SELECT *
FROM cte2
WHERE rk=1 ;
```
![image](https://github.com/user-attachments/assets/2ab6438b-6617-400d-a602-5bb632425009)


4- write a query to find city which had lowest percentage spend for gold card type ?

```sql
WITH cte AS (
	SELECT city , card_type , SUM(CAST(amount AS BIGINT)) as total_amount,
		SUM(CASE WHEN card_type='Gold' THEN amount ELSE 0 END) AS gold_amt
	FROM credit_card
	GROUP BY city , card_type
	)
SELECT TOP 1 city , ((SUM(gold_amt)*1.0)/SUM(total_amount))*100 AS gold_pct
FROM cte
GROUP BY city
HAVING SUM(gold_amt)>0 
ORDER BY gold_pct ASC;
```
![image](https://github.com/user-attachments/assets/fa027120-192d-482e-ae3c-ce439fb82704)

5- write a query to print 3 columns:  city, highest_expense_type , lowest_expense_type 
(example format : Delhi , bills, Fuel)?

```sql
WITH total_exp AS (
	SELECT city ,exp_type , SUM(amount) AS total_expense
	FROM credit_card
	GROUP BY city ,exp_type
	) ,
rnk AS (
	 SELECT city , exp_type , total_expense,
		DENSE_RANK() OVER(PARTITION BY city ORDER BY total_expense ASC) AS asc_rnk ,
		DENSE_RANK() OVER(PARTITION BY city ORDER BY total_expense DESC) AS des_rnk
	 FROM total_exp 
	 )
SELECT city ,
	MAX(CASE WHEN asc_rnk = 1 THEN exp_type END) AS lowest_expense_type ,
	MAX(CASE WHEN des_rnk = 1 THEN exp_type END) AS highest_expense_type
FROM rnk
GROUP BY city ;
```
![image](https://github.com/user-attachments/assets/91738fa8-3e85-4e11-b745-af99e5823149)


6- write a query to find percentage contribution of spends by females for each expense type ?
```sql
SELECT exp_type , 
	(SUM(CASE WHEN gender = 'F' THEN amount ELSE 0 END)*1.0 / SUM(amount) )*100 AS pct_females_contribution
FROM credit_card 
GROUP BY exp_type
ORDER BY pct_females_contribution DESC ;
```
![image](https://github.com/user-attachments/assets/52d65600-e5f8-43af-a287-afb9b3718a40)

7- which card and expense type combination saw highest month over month growth in Jan-2014 ?

```sql
WITH cte AS (
	SELECT card_type , exp_type , amount ,
		DATEPART(YEAR , transaction_date) AS yr ,
		DATEPART(MONTH , transaction_date) AS mnth 
	FROM credit_card
	),
amount_spend AS (
	SELECT card_type , exp_type , yr , mnth ,
		SUM(amount) AS total_exp
	FROM cte
	GROUP BY card_type , exp_type , yr , mnth 
	),
previous_expenses AS (
	SELECT card_type , exp_type , yr , mnth , total_exp,
		LAG(total_exp , 1 , 0) OVER(PARTITION BY card_type , exp_type ORDER BY yr , mnth) AS prev_month_exp
	FROM amount_spend
	)
SELECT TOP 1 * --card_type , exp_type , total_exp , prev_month_exp 
	,(total_exp - prev_month_exp) AS MoM_Growth
FROM previous_expenses
WHERE yr = '2014' AND mnth = '1'
ORDER BY MoM_Growth DESC ;
```
![image](https://github.com/user-attachments/assets/dc5043a2-5051-44db-a51a-3d9638902e47)


8- during weekends which city has highest total spend to total no of transcations ratio ?

```sql
SELECT * FROM credit_card;

SELECT TOP 1 city , SUM(amount) AS total_spend , SUM(amount)*1.0/COUNT(1) AS ratio
FROM 
	(SELECT * ,
		DATEPART(WEEKDAY , transaction_date) AS week_day --,DATENAME(WEEKDAY , transaction_date) => SUN=1 , SAT=7
	FROM credit_card) a
WHERE week_day IN (1,7)
GROUP BY city
ORDER BY ratio DESC;
```
![image](https://github.com/user-attachments/assets/637ef44a-9784-40e3-9244-c2f909c7b1c3)


9- which city took least number of days to reach its 500th transaction after the first transaction in that city ?

```sql
WITH cte AS (
	SELECT * ,
		ROW_NUMBER() OVER(PARTITION BY city ORDER BY transaction_date , transaction_id) AS rn
	FROM credit_card
	)
SELECT TOP 1 city ,  
	DATEDIFF(DAY , MIN(transaction_date) , MAX(transaction_date) ) AS no_of_days
FROM cte
WHERE rn=1 OR rn=500
GROUP BY city
HAVING COUNT(1) = 2
ORDER BY no_of_days ;
```
![image](https://github.com/user-attachments/assets/9f8d2b96-e59f-4009-a1f2-4538ad48d321)





