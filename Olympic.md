# 🏅  Olympic Games (1896 - 2016) - SQL and Power BI

**POWER BI IMAGE**

## 📚 Table of Contents
- [Business Task](#-business-task)
- [Data Exploration - SQL](#-data-exploration---sql)
- [Task Solution - SQL](#-task-solution)
- [Questions](#-questions)
- [Power BI](#-power-bi)

##  📌 Business Task

"As a data analyst working at a news company you are asked to visualize data that will help readers understand how countries have performed historically in the summer Olympic Games.
You also know that there is an interest in details about the competitors, so if you find anything interesting then don’t hesitate to bring that in also. 
The main task is still to show historical performance for different countries, with the possibility to select your own country."

"In addition, we need to answer 16 specific questions. Good job!"

## 📌 DATA EXPLORATION - SQL

```sql -- Add 3 backticks followed by sql
describe olympic.olympictable;
``` 

![image](https://user-images.githubusercontent.com/87883824/202052022-4638ea48-a711-41e7-8a8c-c8802583273e.png)
```sql
select * from olympic.olympictable limit 10
```
![image](https://user-images.githubusercontent.com/87883824/202052274-019e4efe-a2e2-4b43-be9c-d3407c683991.png)

With these two initial queries we see that we have 15 columns of data:

| Column  |  |
| ------------- | ------------- |
| ID  | Unique number for each athlete |
| Name  |  Athlete's name  |
| Sex  | M or F  |
| Age | Integer |
| Height  | In centimeters  |
| Weight   | In kilograms |
| Team | Team name |
| NOC  | National Olympic Committee 3-letter code  |
| Games  | Year and season |
| Year| Integer  |
| Season| Summer or Winter |
| City   | Host city |
| Sport | Sport |
| Event | Event |
| Medal |  Gold, Silver, Bronze, or NA   |

Another important issue is the analysis that we have **repetition of athletes with the same id and different sports**

```sql
select count(*) as Qty_Lines from olympic.olympictable 
```
![image](https://user-images.githubusercontent.com/87883824/202057525-74639ff0-a5e3-4d01-9e65-f1142e73db8c.png)

The file contains 271116 rows and 15 columns.

Now let's find out how many Olympic committees we have and how many athletes, remembering that we analyzed by ID, because we have the same athlete in different sports:

```sql
select count(distinct noc)
from olympic.olympictable
```
![image](https://user-images.githubusercontent.com/87883824/202058039-72809a83-c186-4cd0-b7b0-0957e4936af7.png)

```sql
select count(distinct id)
from olympic.olympictable
```
![image](https://user-images.githubusercontent.com/87883824/202058402-5595eb2f-ba04-4227-a4a4-13da64cde43c.png)

We have 230 Olympic Committees with 135571 athletes.

```sql
select sex, count(distinct id)
from olympic.olympictable
group by sex
```

![image](https://user-images.githubusercontent.com/87883824/202058801-1a663bbb-b5bd-433a-bb5a-0c085318e0c4.png)

Within that 135571 athletes, we have 101590 men and 33981 women.

```sql
select avg(age), max (age), min (age)
from olympic.olympictable
```
![image](https://user-images.githubusercontent.com/87883824/202059229-60bdb741-d98b-458a-9837-0e5b01c2c613.png)

The oldest athlete to ever participate in an edition of the games was 97 years old, while the youngest was 10 years old. The average age of competitors is 25.5 years.

But is the average age very different depending on the athlete's gender?

```sql
select sex ,avg(age), max (age), min (age)
from olympic.olympictable
group by sex 
```
![image](https://user-images.githubusercontent.com/87883824/202059530-797c366d-27a9-4831-b5b1-0d67b19ae915.png)

A not very marked difference, of 2.54 years. The biggest difference is in the maximum age of the athletes, 23 years between the oldest male (97y) and the oldest female (74y).

Let's find out who are the athletes with maximum and minimum age of each gender:

```sql
select age , sex ,  o.name, o.team, o.noc, o.sport , o.medal 
from olympic.olympictable o
order by age desc 
limit 1
```
![image](https://user-images.githubusercontent.com/87883824/202060186-69469d5f-e26e-40d4-81d5-39e9de67e209.png)

```sql
select age , sex ,  o.name, o.team, o.noc, o.sport , o.medal 
from olympic.olympictable o
where age != 'NULL'
order by age asc 
limit 1
```
![image](https://user-images.githubusercontent.com/87883824/202060242-0260d2e1-5027-4d3a-80a2-cbc123e1f327.png)

```sql
select age , sex ,  o.name, o.team, o.noc, o.sport , o.medal 
from olympic.olympictable o
where sex = 'F'
order by age desc 
limit 1
```
![image](https://user-images.githubusercontent.com/87883824/202060689-29901fd5-cad7-4b72-9d53-c3d3827e70c7.png)

```sql
select age , sex ,  o.name, o.team, o.noc, o.sport , o.medal 
from olympic.olympictable o
where sex = 'F'
and age !='NULL'
order by age asc 
limit 1
```
![image](https://user-images.githubusercontent.com/87883824/202060607-8565ed0f-caa2-420e-aea2-de384cbdf312.png)

The oldest athletes of each gender, John Quincy from the USA and Ernestine Lonie from France, participated in the games in 'Art Competitions' and ended up not winning a medal. Among the younger athletes. Dimitrios Loundras of Greece won a bronze medal in Gymnastics. The youngest female athlete, Magdalena Cecilia, from Great Britain, participated in figure skating and did not take home a medal

## 📌 TASK SOLUTION

To solve the proposed task, we need to make a query that shows us the performance of the countries in the summer games and details about the competitors. For this, we are going to remove some unnecessary columns and create others, such as, for example, the participant's age range, for future analysis in Power BI.

```sql
select 
id,
name as Athlete_Name, 
case
when sex = 'M' then 'Man'
when sex = 'F' then 'Woman'
end as Sex, 
age, 
case
when age <18 then 'Under 18'
when age between 18 and 25 then '18-25'
when age between 25 and 30 then '25-30'
when age >30 then 'Over 30'
end as 'Age Grouping',
height,
weight,
noc as Nation_Code, 
SUBSTRING_INDEX(games,' ',1) as 'Year',
sport, 
event, 
COALESCE( medal , 'Not Registered' ) as Medal
from olympic.olympictable o 
where SUBSTRING_INDEX(games,' ',-1) = 'Summer'
```

### 📌 QUESTIONS

#### 1. How many olympics games have been held? 

```sql
select count(distinct games)
from olympic.olympictable o 
```
![image](https://user-images.githubusercontent.com/87883824/202064205-55ae5d3d-c32e-4200-a572-1ea2419696cb.png)

#### 2. List down all Olympics games held so far.

```sql
select distinct year, season , city 
from olympic.olympictable o 
order by `year` asc 
```
![image](https://user-images.githubusercontent.com/87883824/202064890-27cd7b2f-f5f1-4ee6-93be-2ffc9abea61c.png)

#### 3.Mention the total no of nations who participated in each olympics game? 

Steps: 
- **JOIN** with the other table that relates the NOC with the nations

```sql
	select games, count(distinct nr.region )
	from olympic.olympictable o
	join olympic.olympic_regions nr on nr.noc = o.noc 
	group by games
```
![image](https://user-images.githubusercontent.com/87883824/202065537-408bf6fe-bcef-4baa-94ed-8e0563587541.png)

#### 4. Which nation has participated in all of the olympic games

```sql
  select nr.region , count(distinct games) as total
	from olympic.olympictable o
	join olympic.olympic_regions nr on nr.noc = o.noc 
	group by nr.region 
	order by total desc  
 ```
 
![image](https://user-images.githubusercontent.com/87883824/202066213-ff25c071-2afe-4739-bada-12e08df58e5b.png)

#### 5. Identify the sport which was played in all summer olympics.

```sql
      with t1 as
          	(select count(distinct games) as total_games
          		from olympic.olympictable o  where season = 'Summer'),
          t2 as
          	(select distinct games, sport
          	from olympic.olympictable o  where season = 'Summer'),
          t3 as
          	(select sport, count(1) as no_of_games
          	from t2
          	group by sport)
      select *
      from t3
      join t1 on t1.total_games = t3.no_of_games;
```

![image](https://user-images.githubusercontent.com/87883824/202066541-f2e606c8-779d-4ee8-9513-fbc0f798fa77.png)

#### 6.Which Sports were just played only once in the olympics?
```sql
     select sport, count(distinct games) as 1_game, games  
     from olympic.olympictable o
     group by sport    
     having 1_game = 1
```

![image](https://user-images.githubusercontent.com/87883824/202067471-a14bbed5-0a06-484d-8870-88bfa4d0c767.png)

#### 7.Fetch the top 5 athletes who have won the most gold medals. 

Steps: 
- **COUNT** the number of the medals.
- **DENSE_RANK** to sort by number of gold medals.

```sql
     select name , team , count(medal) gold_medals, dense_rank () over (order by count(medal) desc) densi
     from olympic.olympictable o 
     where medal = 'Gold'
     group by name 
     having count(medal) >= 7 
     order by count(medal) desc 
```
![image](https://user-images.githubusercontent.com/87883824/202068100-120c8c93-affc-4f6d-b6ad-e45b0e2a7eb0.png)

#### 8.Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won.

```sql
     select region, count(medal) total_medals, dense_rank () over (order by count(medal)desc) densi 
     from olympic.olympictable o 
     join olympic.olympic_regions or2 on or2.noc = o.noc 
     group by or2.region  
     order by count(medal) desc 
     limit  5
```
![image](https://user-images.githubusercontent.com/87883824/202068290-7e1366c9-78fc-48f6-be65-febd5a467239.png)

#### 9.List down total gold, silver and broze medals won by each country.
```sql   
     with gold as
     			( select region ,o.noc ,count(medal) as golde
     			from olympic.olympictable o
     			join olympic.olympic_regions or2 on or2.noc = o.noc 
     			where medal = 'Gold'
     			group by region
     			order by count(medal) desc ),
     	  silver as
     	 		 ( select  noc ,count(medal) as silvere
     			from olympic.olympictable o
     			where medal = 'Silver'
     			group by noc
     			order by count(medal) desc),
     	bronze as 
     	( select noc, count(medal) as bronzee
     			from  olympic.olympictable o
     			where medal = 'Bronze'
     			group by noc
     			order by count(medal) desc)
     select  region , golde as gold, silvere as silver, bronzee as bronze
     from gold
     join silver on silver.noc = gold.noc
     join bronze on bronze.noc = silver.noc
     group by region 
     order by golde desc
```
|region|gold|silver|bronze|
|------|----|------|------|
|USA|2638|1641|1358|
|Russia|1599|732|689|
|Germany|1301|674|746|
|UK|678|739|651|
|Italy|575|531|531|
|France|501|610|666|
|Sweden|479|522|535|
|Canada|463|438|451|
|Hungary|432|332|371|
|Norway|378|361|294|
|Australia|368|455|517|
|China|351|347|292|
|Netherlands|287|340|413|
|Japan|247|309|357|
|South Korea|221|232|185|
|Finland|198|270|432|
|Denmark|179|241|177|
|Switzerland|175|248|268|
|Cuba|164|129|116|
|Romania|161|200|292|
|Serbia|157|29|41|
|India|138|19|40|
|Czech Republic|123|36|66|
|Poland|117|195|253|
|Spain|110|243|136|
|Brazil|109|175|191|
|Austria|108|186|156|
|Belgium|98|197|173|
|Argentina|91|92|91|
|New Zealand|90|56|82|
|Greece|62|109|84|
|Croatia|58|54|37|
|Bulgaria|54|144|144|
|Ukraine|47|52|100|
|Pakistan|42|45|34|
|Turkey|40|27|28|
|Jamaica|38|75|44|
|Kenya|34|41|31|
|South Africa|32|47|52|
|Uruguay|31|2|30|
|Mexico|30|26|54|
|Belarus|24|44|71|
|Nigeria|23|30|46|
|Ethiopia|22|9|22|
|Kazakhstan|20|25|32|
|Cameroon|20|1|1|
|Iran|18|21|29|
|Zimbabwe|17|4|1|
|North Korea|16|16|35|
|Slovakia|15|19|13|
|Bahamas|14|11|15|
|Estonia|13|12|25|
|Indonesia|11|17|13|
|Uzbekistan|10|7|17|
|Thailand|9|8|13|
|Ireland|9|13|13|
|Slovenia|8|13|27|
|Georgia|8|6|18|
|Azerbaijan|7|12|25|
|Trinidad|7|8|17|
|Egypt|7|8|12|
|Lithuania|6|7|48|
|Morocco|6|5|12|
|Colombia|5|9|14|
|Algeria|5|4|8|
|Portugal|4|11|26|
|Latvia|3|19|13|
|Chile|3|9|20|
|Taiwan|3|28|18|
|Tunisia|3|3|7|
|Dominican Republic|3|2|2|
|Armenia|2|5|9|
|Venezuela|2|3|10|
|Liechtenstein|2|2|5|
|Mongolia|2|10|14|
|Uganda|2|3|2|
|Puerto Rico|1|2|6|
|Ivory Coast|1|1|1|
|Haiti|1|1|5|
|Costa Rica|1|1|2|
|Bahrain|1|1|1|
|Individual Olympic Athletes|1|1|3|
|Tajikistan|1|1|2|
|Israel|1|1|7|
|Syria|1|1|1|

### 📌 POWER BI

With the [Task Solution - SQL](#-task-solution) data I produced a simple dashboard for solving the task: 






