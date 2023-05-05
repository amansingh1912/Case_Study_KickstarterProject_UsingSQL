# Case Study on Kickstarter Project Using SQL

I did a case study on the Kickstarter Project using SQL. I took this dataset from one of the data challenges of LinkedIn which I took part.

I asked several questions with respect to the dataset, and then answered them using SQL queries.

### Tool Used: PostGreSQL

#### Q1) Find % successful, % cancelled and % failed overall.

Ans:

### Code:

with t1 as
(
SELECT COUNT(*) AS Total_count
from kickstarter_projects
),

t2 as
(
SELECT count(*) as Successful_count
from kickstarter_projects
where state = 'Successful'
),

t3 as
(
SELECT count(*) as Failed_count
from kickstarter_projects
where state = 'Failed'
),
t4 as 
(
SELECT count(*) as Canceled_count
from kickstarter_projects
where state = 'Canceled'
)

SELECT ROUND(((Successful_count)*100.0/(Total_count)),2) as percent_successful,
ROUND(((Failed_count)*100.0/(Total_count)),2) as percent_failed,
ROUND(((Canceled_count)*100.0/(Total_count)),2) as percent_canceled
FROM t1,t2,t3,t4;


### Output:

![image](https://user-images.githubusercontent.com/72240938/236443855-1f772da0-cbbc-4492-91a7-5e93eaeb4b15.png)


#### Q2) Find the country which has got the highest no. of total backers overall and vice versa.

Ans:

#### Code:

with t1 as 
(
SELECT Country as c1, sum(backers) as Total_backers
from kickstarter_projects
group by Country
limit 1
),

t2 as
(
SELECT Country as c2, sum(backers) as Total_backers
from kickstarter_projects
group by Country
order by total_backers asc
limit 1
)
SELECT c1 as MostBackedCountry, c2 as LeastBackedCountry
from t1, t2;


#### Output:

![image](https://user-images.githubusercontent.com/72240938/236444282-947ec39f-8d53-4f83-9cd1-57dc4089e731.png)


#### Q3) Find the year by number of launches per year.

Ans:

#### Code:

SELECT to_char(launched, 'YYYY') as year_num, count(*) as Total_projects_launched
 from kickstarter_projects
 group by year_num
 order by year_num asc;
 
 #### Output:
 
 ![image](https://user-images.githubusercontent.com/72240938/236444535-ed67eecf-98cc-4c4e-bbb9-abcf832b6e6c.png)


#### Q4) Find for each category which country has the highest average goal.

Ans:

#### Code:

SELECT category, country
from
(
SELECT category, country, avg(goal) as avg_goal,
rank() over(partition by category order by avg(goal) desc) as rnk
from kickstarter_projects
group by category, country
) a where a.rnk<2;


#### Output:

![image](https://user-images.githubusercontent.com/72240938/236444699-94cae61f-67e6-4528-b1c8-10de077a6edd.png)




#### Q5) Find category wise highest percent successful for each of the countries.
Ans:

#### Code:

with cte_1 as
(select 
  *,
  case when state = 'Successful' then 1 else 0 end as state_for_analysis
 from kickstarter_projects),
cte_2 as( 
 select
  category,
  country,
  ROUND(sum(state_for_analysis)*100.0/count(state_for_analysis),2) as percentage_successful
from cte_1
group by category,country
),

cte_3 as (
select
 *,
 rank() over(partition by category order by percentage_successful desc) as rn
from cte_2  
)
select *
from cte_3
where rn =1;


#### Output:

![image](https://user-images.githubusercontent.com/72240938/236444998-60ffeb25-dbe6-4dbd-ba73-36c321506a75.png)


#### Q6) Find the record which has the earliest project year of launch and was the successful having maximum goal.

Ans:

#### Code:

SELECT * from kickstarter_projects 
where state = 'Successful'
order by to_char(launched, 'YYYY') asc, goal desc 
limit 1;

#### Output:

![image](https://user-images.githubusercontent.com/72240938/236445233-debdcf27-2183-4058-8ba0-95cac75ead1f.png)


#### Q7) What project with a goal over $1,000 USD, had the biggest Goal Completion % (Pledged / Goal) and the lowest? 

Ans:

#### Code:

with t1 as
(
SELECT * FROM kickstarter_projects
where goal>1000 and state = 'Successful'
),
t2 as
(
SELECT *, ROUND(((pledged)*100.0/(goal)),2) as percent_pledge_by_goal
from t1
),
t3 as
(
SELECT category, ROUND(avg(percent_pledge_by_goal),2) as avg_percent_pledgebygoal
from t2
group by category
order by avg_percent_pledgebygoal DESC
)
SELECT * FROM t3;

#### Output:


![image](https://user-images.githubusercontent.com/72240938/236445714-8096bedc-b794-49cd-867b-58ae8ac75ae3.png)


#### Q8) Find year wise most successful category i.e. (find year with the project most successful in terms of percent_success).

Ans:

#### Code:

with t1 as
(
SELECT *, to_char(launched, 'YYYY') as launch_year 
from kickstarter_projects
),
t2 as
(
SELECT *, 
CASE when state = 'Successful' then 1 else 0 end as state_for_analysis
from t1
),
t3 as
(
SELECT launch_year, category,
ROUND(sum(state_for_analysis)*100.0/count(state_for_analysis),2) as percent_successful
from t2
group by launch_year, category
order by percent_successful desc
),
t4 as
(
SELECT *, rank() over(partition by launch_year order by percent_successful desc) as rn
from t3
)
SELECT launch_year, category FROM t4
where rn<2;


#### Output:

![image](https://user-images.githubusercontent.com/72240938/236445942-832fe860-26ce-4337-b3fe-6b78586e7984.png)


 Over the years i.e. between 2009-2018, Dance category is the most successful one among all the categories.
 
 
 ### Key Insights:
 
 * Dance, Journalism and Publishing has the most success percentage among all the categories.
 * Biggest goal completion %(Pledged/Goal) is of the Design category among all the categories.
 * Over the years, Dance category has been the most successful.



As an investor, I would like to invest in the category which has been the most successful over the years and the most successful percentage overall among all categories.
So, Dance category would be the one I would invest on.






































