# **Introduction**  

This project is part of my data analytics portfolio, showcasing my ability to work with large, real-world datasets using SQL, Excel, and data visualization tools. I’m passionate about turning raw data into actionable insights and am open to opportunities in data analytics, data science, and even data engineering.

The research subject is the [YELP Open Dataset](https://business.yelp.com/data/resources/open-dataset/).  

The goal of this project was to perform a **comprehensive, end-to-end analysis** of Yelp’s publicly available dataset, with a primary focus on the **restaurant industry**.  
The dataset offers a unique opportunity to explore **real-world, large-scale data** containing millions of reviews, detailed business attributes, geolocation data, and customer feedback spanning multiple years.  

## **The Dataset**  

The dataset contains over **6,000,000 business reviews** from **150'346 businesses** in **27 U.S. states**.  
The main focus of this research is **restaurants** — exploring their ratings, popularity, and customer feedback trends.  
Some tables exceeding **15,000,000** rows.  

![Dataset.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/msedge_EReYpbqCQX.png)

### **Questions I wanted to answer with my SQL Queries were:**  
- How many businesses are there in each state in this dataset?
- Which ten restaurants are the best in each state?  
- What cuisines receive the most reviews and the highest average ratings?  
- Are there patterns between restaurant price range, cuisine, and customer ratings?
- What do the worst reviews say about the best-rated restaurants say?
- How do review counts and ratings change over time for top restaurants?



# **Tools Used**  
+ **SQL:** The main tool used to analyze millions of rows efficiently. Specifically, SQLite was used.  
+ **DBeaver:** The chosen database management software.  
+ **Microsoft Excel:** Used to consolidate data and create charts.  
+ **Git & GitHub:** used to share my SQL analysis.  

# **The Analysis**  

## Hypothesis
1. Mid-priced restaurants will receive higher average ratings than expensive restaurants, as affordability often correlates with broader customer satisfaction;
2. Food type will directly correlate to customer satisfaction. Niche food types will be less liked than general cuisines;
3. User reviews are the driving force behind restaurant success or failure. 


## Data Cleanup
This, honestly, was the hardest part. It required a lot of time to clean up the data and create new tables with recursive queries. 
The data came as JSON files and was imported into DBeaver Database Manager.
The columns didn’t have column names and looked like this:
|Column1|Column2|Column3|Column4|
|-----|--------------|-----|------|
"business_id": "tnhfDv5Il8EaGSXZGiuQGg"|"name": "Garaje"|"address": "475 3rd St"|...
...

Before I could clean the data, I had to create new tables for each given table with the correct Column Names and data types.

### Table Creation
---
An example of how a single table was created:
```sql
CREATE TABLE YELP_RestaurantsCLEANED (
    Business_id TEXT PRIMARY KEY,
    Name TEXT,
    Address TEXT,
    City TEXT,
    State TEXT,
    Postal_Code TEXT,
    Latitude REAL,
    Longitude REAL,
    Rating_stars REAL,
    Review_Count INTEGER,
    Is_Open INTEGER,
    Attributes TEXT,
    Attributes2 TEXT,
    Attributes3 TEXT
);
```
Now, the data cleaning. The following query was used:
```sql
INSERT INTO YELP_RestaurantsCLEANED (
    Business_id, Name, Address, City, State, Postal_Code,
    Latitude, Longitude, Rating_stars, Review_Count, Is_Open,
    Attributes, Attributes2, Attributes3
)
SELECT
    SUBSTRING(Column1, INSTR(Column1, ':') + 2) AS Business_id,
    SUBSTRING(Column2, INSTR(Column2, ':') + 2) AS Name,
    SUBSTRING(Column3, INSTR(Column3, ':') + 2) AS Address,
    SUBSTRING(Column4, INSTR(Column4, ':') + 2) AS City,
    SUBSTRING(Column5, INSTR(Column5, ':') + 2) AS State,
    SUBSTRING(Column6, INSTR(Column6, ':') + 2) AS Postal_Code,
    SUBSTRING(Column7, INSTR(Column7, ':') + 1) AS Latitude,
    SUBSTRING(Column8, INSTR(Column8, ':') + 1) AS Longitude,
    SUBSTRING(Column9, INSTR(Column9, ':') + 1) AS Rating_Stars,
    SUBSTRING(Column10, INSTR(Column10, ':') + 1) AS Review_Count,
    SUBSTRING(Column11, INSTR(Column11, ':') + 1) AS Is_Open,
    SUBSTRING(Column12, INSTR(Column12, ':') + 2) AS Attributes,
    SUBSTRING(Column13, INSTR(Column13, ':') + 2) AS Attributes2,
    SUBSTRING(Column14, INSTR(Column14, ':') + 2) AS Attributes3
FROM
    YELP_Restaurants;
```
The Result:
|Business_id|Name|Address|City|State|Postal_Code|Latitude|Longitude|Rating_stars|Review_Count|Is_Open|Attributes|Attributes2|Attributes3|
|-----------|----|-------|----|-----|-----------|--------|---------|------------|------------|-------|----------|-----------|-----------|
|Pns2l4eNsfO8kk83dixA6A|Abby Rappoport, LAC, CMQ|1616 Chapala St, Ste 2|Santa Barbara|CA|93101|34.4266787|-119.7111968|5.0|7|0|"ByAppointmentOnly":"True"}|Doctors, Traditional Chinese Medicine, Naturopathic/Holistic, Acupuncture, Health & Medical, Nutritionists|ull}|
|mpf3x-BjTdTEA3yCZrAYPw|The UPS Store|87 Grasso Plaza Shopping Center|Affton|MO|63123|38.551126|-90.335695|3.0|15|1|"BusinessAcceptsCreditCards":"True"}|Shipping Centers, Local Services, Notaries, Mailbox Centers, Printing Services|"Monday":"0:0-0:0|
|tUFrWirKiKi_TAnsVWINQQ|Target|5255 E Broadway Blvd|Tucson|AZ|85711|32.223236|-110.880452|3.5|22|0|"BikeParking":"True|True|2|
|MTSW4McQd7CbVtyjqoe9mw|St Honore Pastries|935 Race St|Philadelphia|PA|19107|39.9555052|-75.1555641|4.0|80|1|"RestaurantsDelivery":"False|False|False|
|mWMc6_wTdE0EUBKIGXDVfA|Perkiomen Valley Brewery|101 Walnut St|Green Lane|PA|18054|40.3381827|-75.4716585|4.5|13|1|"BusinessAcceptsCreditCards":"True|True|True|
|CF33F8-E6oudUQ46HnavjQ|Sonic Drive-In|615 S Main St|Ashland City|TN|37015|36.269593|-87.058943|2.0|6|1|"BusinessParking":"None|True|u'casual'|
|...|

Attribute columns I left untouched.
 
Similarly, I did this to the other four tables.
One exception was the second table. It was more complicated. A recursive query was needed to clean up the data. The first column had the business_id; 
however, the second column had all the check-in dates in one row. So, data “exploding” and “imploding” was needed.

This query was much more complicated and ran for about 30 minutes and generated over 15'000'000 rows:
```sql
CREATE TABLE CheckinCLEANED (
    Business_Id TEXT
    Checkin_Date DATETIME 
);
WITH RECURSIVE
prepared_data AS (
    SELECT
        Column1 AS Business_Id,
        Column2 || ', ' AS date_string
    FROM Checkin
),
split_dates(Business_Id, Checkin_Date, remaining_dates) AS (
    SELECT
        p.Business_Id,
        TRIM(SUBSTR(p.date_string, 1, INSTR(p.date_string, ',') - 1)),
        SUBSTR(p.date_string, INSTR(p.date_string, ',') + 1)
    FROM prepared_data p
   
    UNION ALL
 
    SELECT
        s.Business_Id,
        TRIM(SUBSTR(s.remaining_dates, 1, INSTR(s.remaining_dates, ',') - 1)),
        SUBSTR(s.remaining_dates, INSTR(s.remaining_dates, ',') + 1)
    FROM split_dates s
    WHERE INSTR(s.remaining_dates, ',') > 0
)
 
INSERT INTO CheckinCLEANED (Business_Id, Checkin_Date)
SELECT
    SUBSTRING(s.Business_Id, INSTR(s.Business_Id, ':') + 2) AS Business_Id,
    SUBSTRING(s.Checkin_Date, INSTR(s.Checkin_Date, ':') + 2) AS Checkin_Date
FROM split_dates s
WHERE s.Checkin_Date IS NOT NULL AND s.Checkin_Date != '';
```
The result:
![BeforeandAfter](https://github.com/ronalds141/SQL-Portfolio/blob/main/Before%20and%20After.png)


## ER Diagram
ER Diagram was created with an online ER diagram maker - [DBDiagram.io](https://dbdiagram.io/home/).
![ER Diagram.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/ER%20Diagram.png)


## The Queries

Each query in this project was designed to explore specific aspects of restaurant performance, customer preferences, and review trends.

### How many businesses are there in each state in this dataset?
---
This query returns the number of businesses in each state. 
```sql
SELECT DISTINCT
   State, 
    COUNT(*) OVER (PARTITION BY State) AS Business_Count
FROM BusinessCLEANED
ORDER BY Business_Count DESC;
```
The results:
|State|Business_Count|
|-----|--------------|
|PA|34039|
|FL|26330|
|TN|12056|
|IN|11247|
|MO|10913|
|LA|9924|
|AZ|9912|
|NJ|8536|
|NV|7715|
|AB|5573|
|CA|5203|
|ID|4467|
|DE|2265|
|IL|2145|
|TX|4|
|CO|3|
|HI|2|
|MA|2|
|WA|2|
|MI|1|
|MT|1|
|NC|1|
|SD|1|
|UT|1|
|VI|1|
|VT|1|
|XMS|1|

We could count the states with few restaurants without a query, but I wrote this query, in case the data set was large:
```sql
SELECT 
COUNT(State) AS StatesWithFewRestaurants
FROM (
SELECT DISTINCT
   State, 
    COUNT(*) OVER (PARTITION BY State) AS Business_Count
FROM BusinessCLEANED
ORDER BY Business_Count DESC
)
WHERE Business_Count < 5
```
|StatesWithFewRestaurants|
|------------------------|
|13|


We can see that 13 states have only a couple of businesses. Without this knowledge, the results may be skewed. Florida has much higher competition than Texas.


### Which ten restaurants are the best in each state?  
---
Now, let’s find out which businesses are the best ones. I ran a query to find the top 10 best businesses in each state by counting 4.5 and 5.0-star ratings. 
```sql
SELECT 
State,
Name,
Review_Count,
StateRank AS Ranking 
FROM (
    SELECT 
        State, 
        Name,
        Review_Count,
        ROW_NUMBER() OVER (PARTITION BY State ORDER BY Review_Count DESC) as StateRank
    FROM BusinessCLEANED
    WHERE Rating_stars IN (4.5, 5)
) AS RankedRestaurants
WHERE StateRank BETWEEN 1 and 10
ORDER BY State ASC;
```
A sample from the results:
|State|Name|Review_Count|Ranking|
|-----|----|------------|-------|
|AB|Duchess Bake Shop|486|1|
|AB|Padmanadi Vegetarian Restaurant|271|2|
|AB|Chartier|169|3|
|AB|Izakaya Tomo|151|4|
|AB|RGE RD|149|5|
|AB|Cafe Amore Bistro|123|6|
|AB|La Boule|120|7|
|AB|Jack's Burger Shack|117|8|
|AB|Tzin Wine & Tapas|116|9|
|AB|Italian Centre Shop|114|10|
|AZ|Prep & Pastry|2126|1|
|AZ|The Parish|1210|2|
|AZ|Baja Cafe|1074|3|
|AZ|Serial Grillers|986|4|
|AZ|Street- Taco and Beer Co.|805|5|
|...|


### What cuisines receive the most reviews and the highest average ratings?
---
The hypothesis was that middle-priced or cheaper-priced restaurants will be more beloved than expensive ones and that also, the cuisine will be more streamlined to what people like, rather than something exquisite. Before doing the analysis, I removed businesses that weren’t restaurants:
* Travis Darnell Photography (Photography)
* 2 Sisters Photography (Photography)
* City Museum (Attraction/Museum)
* Grounds For Sculpture (Art Park/Museum)
* K Soho Salon (Beauty Salon)
* Southern Luxe Beauty (Beauty Salon)
* Find Your Zen (Wellness)
* All Service Citgo (Gas Station)
* Valvoline Instant Oil Change (Car Service)

AI was used to receive the cuisine type for each restaurant, since it was not provided with the dataset. Sample output:
| State | City          | Business Name                   | Food Type                   |
| ----- | ------------- | ------------------------------- | --------------------------- |
| AB    | Edmonton      | Duchess Bake Shop               | Bakery (French)             |
| AB    | Edmonton      | Padmanadi Vegetarian Restaurant | Asian (Vegan)               |
| AB    | Beaumont      | Chartier                        | French                      |
| AB    | Edmonton      | Izakaya Tomo                    | Asian (Japanese)            |
| AB    | Edmonton      | RGE RD                          | American (Canadian)         |
| AB    | Edmonton      | Cafe Amore Bistro               | Italian                     |
| AB    | Edmonton      | La Boule                        | Bakery (French)             |
| AB    | St Albert     | Jacks Burger Shack              | American (Burgers)          |
| AB    | Edmonton      | Tzin Wine & Tapas               | Mediterranean               |
| AB    | Edmonton      | Italian Centre Shop             | Deli/Italian                |
| AZ    | Tucson        | Prep & Pastry                   | American (Brunch)           |
| AZ    | Tucson        | The Parish                      | Southern/Cajun              |
| AZ    | Tucson        | Baja Cafe                       | American (Brunch)           |
| AZ    | Tucson        | Serial Grillers                 | American (Pizza/Sandwiches) |
| AZ    | Tucson        | Street- Taco and Beer Co.       | Mexican                     |
| AZ    | Tucson        | Wildflower                      | American                    |
| AZ    | Tucson        | Bobos Restaurant                | American (Breakfast)        |
| AZ    | Tucson        | Baja Cafe on Campbell           | American (Brunch)           |
| AZ    | Tucson        | Tumerico                        | Mexican (Vegan)             |
| AZ    | Tucson        | Seis Kitchen                    | Mexican                     |
| CA    | Santa Barbara | Los Agaves                      | Mexican                     |
|...|

Pivot Chart that summarizes this table:
![CuisineType.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/CuisineType.png)
### Are there patterns between restaurant price range, cuisine, and customer ratings?
---
This data set doesn't include the price range of these restaurants. AI again was used to find these restaurants in Google Maps and to retrieve the price category of each restaurant Power Query was used to clean this data, and pivot tables were  used to represent this data:
| State | City          | Business Name                   | Price Category |
| ----- | ------------- | ------------------------------- | -------------- |
| AB    | Edmonton      | Duchess Bake Shop               | Average        |
| AB    | Edmonton      | Padmanadi Vegetarian Restaurant | Average        |
| AB    | Beaumont      | Chartier                        | Expensive      |
| AB    | Edmonton      | Izakaya Tomo                    | Average        |
| AB    | Edmonton      | RGE RD                          | Expensive      |
| AB    | Edmonton      | Cafe Amore Bistro               | Average        |
| AB    | Edmonton      | La Boule                        | Average        |
| AB    | St Albert     | Jacks Burger Shack              | Average        |
| AB    | Edmonton      | Tzin Wine & Tapas               | Expensive      |
| AB    | Edmonton      | Italian Centre Shop             | Cheap          |
| AZ    | Tucson        | Prep & Pastry                   | Average        |
| AZ    | Tucson        | The Parish                      | Average        |
| AZ    | Tucson        | Baja Cafe                       | Average        |
| AZ    | Tucson        | Serial Grillers                 | Average        |
| AZ    | Tucson        | Street- Taco and Beer Co.       | Cheap          |
| AZ    | Tucson        | Wildflower                      | Expensive      |
| AZ    | Tucson        | Bobos Restaurant                | Cheap          |
| AZ    | Tucson        | Baja Cafe on Campbell           | Average        |
| AZ    | Tucson        | Tumerico                        | Average        |
| AZ    | Tucson        | Seis Kitchen                    | Average        |
| CA    | Santa Barbara | Los Agaves                      | Average        |
| CA    | Santa Barbara | Mesa Verde                      | Average        |
| CA    | Santa Barbara | McConnells Fine Ice Creams      | Cheap          |
| CA    | Santa Barbara | The Palace Grill                | Expensive      |
|...|

The summary of the data using pivot tables:

![CuisineType.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/WINWORD_xqZvWXbY7q.png)
![Pricing.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/WINWORD_fZw2ysh1xU.png)

Result - More common cuisines (American, Mexican and Asian) are more beloved than niche (Deli, Mediterranean) ones. 


### What do the worst reviews say about the best-rated restaurants say?
---
Not all reviews are good. The next query explores only 1.0, 1.5, and 2.0 reviews for these same restaurants (if there are any). The query combines two tables to find the businesses by their name, not by their business ID. 
```sql
SELECT Name, Rating_Stars, Rating_Text
FROM (
    SELECT 
        bc.Name, 
        rc.Rating_Stars, 
        rc.Rating_Text,
        ROW_NUMBER() OVER (PARTITION BY bc.Business_id ORDER BY rc.Rating_Stars ASC) as ReviewRank
    FROM BusinessCLEANED bc
    JOIN ReviewCleaned rc ON bc.Business_id = rc.Business_id
    WHERE bc.Name IN (
'Duchess Bake Shop','Padmanadi Vegetarian Restaurant', 
-- list of all restaurants
  'Chartier'
  )
    AND rc.Rating_Stars IN (1.0, 1.5, 2.0)
) as RankedReviews
WHERE ReviewRank <= 5
ORDER BY Name ASC, Rating_Stars ASC;
```
A sample from the result:
|Name|Rating_Stars|Rating_Text|
|----|------------|-----------|
|BEAST Craft BBQ|1.0|BEAST Craft was very disappointing.  After eating there, I am very surprised that Sauce included them in their top 11. nOn Wednesday, December 2, we got there later than we expected to after making a very long wrong turn.  Therefore, when they were out of a number of things, it wasn't an issue.  We patronize many of the metro area BBQ spots and this is the norm.  I would not have expected them to be closed at 7:20, though, and that was a distinct possibility after all the comments from the cashier and later from the rest of the staff.   nnFood:n*Pulled Pork sandwich - COLD n*Kielbasa: COLDn*Half chicken:  Lukewarmn*French fries:  Lukewarm. n*Brussels Sprouts - So hot I couldn't even eat them until lastnIt is very hard to judge food that isn't at the correct temperature and it should be a food safety concern.  There is no way the meat or fries were held above 140 degrees!nnAtmosphere gets a zero!  The dining area was quiet with 4 tables occupied including us and since it was quiet, everything in the kitchen seemed amplified.  All the banter between the workers, all the discussions on what they had left and back and forth on if they were closing, what was already on order for the next day.  On two separate occasions the 'F' word was clearly head by everyone in the dining area.  The second time I had stood up to put on my coat and was facing the kitchen entry and watched the girl say it, she then quickly disappeared to her right out of site. My opinion is, I came to eat not to hear the ins and outs of the operation of the restaurant and most definitely not to hear profanity.  I am just thankful no children were present.nnBottom line - no need for me to spend my gas money traveling that distance when the metro area has super BBQ restaurants that, although very casual, have a much more professional atmosphere.|
|BEAST Craft BBQ|1.0|The food is dry and tasteless. The whole operation appears to be run by teenagers and the food and service show it! Not recommended.|
|Baja Cafe|1.0|We tried to eat at Baja Cafe last night, since all available information said they were open for dinner (their website, yelp, etc.). When we arrived, there were several other annoyed patrons leaving, as the restaurant is clearly not actually open for dinner on Friday and Saturday nights, contrary to what they advertise. This wouldn't be a huge deal, but we drove 35 minutes just to eat here. I emailed the owner that night to let them know, and received an email saying I should have known better somehow (how?!) and they just haven't gotten around to updating their website. She was curt and unapologetic, which makes it clear she doesn't care about lost business. I'll be taking my money and time elsewhere.|
|Baja Cafe|1.0|Ordered the corned beef with eggs. What I was attracted to was the "we crispen our hash browns then add the meat". What I got was raw potatoes or damn near. I guess when the kitchen gets really busy and the dollar signs start rolling the quality dips. Not again.|
|...|

### How do review counts and ratings change over time for top restaurants?
---
This Dataset doesn't include timestamps for the time when the review was given. If it had, this is the query I would use to find the count of reviews from 2001 to 2099 to analyze patterns and determine whether the restaurant is doing better or worse.
```sql
SELECT 
    bc.Name AS Business_Name,
    strftime('%Y', rc.review_date) AS review_year,
    COUNT(*) AS total_reviews
FROM 
    BusinessCLEANED bc
JOIN 
    ReviewCLEANED rc
    ON bc.Business_id = rc.Business_id
WHERE 
    bc.Name = 'Baja Coffee'
    AND strftime('%Y', rc.review_date) LIKE '20%'
    AND rc.Rating_Stars IN (4.5, 5.0)

GROUP BY 
    bc.Name, strftime('%Y', rc.review_date)
ORDER BY 
    review_year;
```
The result would look like this. The count is the number of reviews, that received 4.5 and 5.0 stars:
![RatingsOverYears.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/WINWORD_SwpsB3SlN6.png)

---

# **Conclusions**  
These results confirmed my hypothesis that:
* Averagely priced and cheaper priced restaurants will have more reviews and will be more attended.
* More well-known cuisines will be more beloved than niche cuisines.
* It's important to listen to the customers, since reviews greatly affect the performance of the restaurants. 

# **What I Learned**  
- **Advanced SQL Skills**: Learned to clean data, join multiple tables, use subqueries, and apply CTEs for cleaner, more efficient queries, using ROW_Number functions and lastly recursive commands. 
- **Data Visualization & Reporting**: Used Excel and Power Query to clean, transform, and consolidate results from SQL queries, creating interactive charts and dashboards for clear storytelling.  
- **Insight Generation**: Converted millions of raw data points into actionable insights, such as identifying top cuisines, price to performance ratio and client insights to help business owners make the best restaurant.  

## What else could have  been done
* This dataset contains Latitude and Longitude for each business. With this information, one could explore:
  * *Restaurant Density Heatmaps*: Identify “food clusters” in cities using geospatial clustering (e.g., DBSCAN).
  * *Travel Distance Analysis*: Calculate the  average distance between a user’s reviews to see how far they travel for food.
  * *Proximity Impact*: Measure if restaurants near tourist attractions or transport hubs have higher ratings.
* I barely touched the reviews table. One could extract the main keywords. These results will show what people love and hate the most. This can be done using python wordcloud library.
* Secondly, the reviews don't always paint the full picture. One could also compare the sentiment of the reviews, which range from -1 to +1, to identify what people like most. Python TextBlob or Vader could be used for this.
* And lastly, one could look at the worst restaurants to learn from their mistakes. What do people dislike? Why? 

### Closing thoughts
---
In this project, I explored Yelp's open business data to find insights about restaurants, customer preferences, and business performance. I cleaned and prepared the dataset, then used SQL to answer questions like which restaurants have the highest ratings, which cuisines are most popular, and whether price correlates to customer satisfaction.

This showed me I can complete a data project from start to finish — from raw JSON files to a published GitHub portfolio. I became more proficient in SQL, using filtering, aggregation, joins, and subqueries.

Overall, I improved my SQL skills, strengthened my data-cleaning abilities, and developed a business-owner mindset.
