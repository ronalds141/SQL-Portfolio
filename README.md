# **Introduction**  
---

This project is part of my data analytics portfolio, showcasing my ability to work with large, real-world datasets using SQL, Excel, and data visualization tools. I’m passionate about turning raw data into actionable insights and am open to opportunities in data analytics, data science, and even data engineering.

The research subject is the [YELP Open Dataset](https://business.yelp.com/data/resources/open-dataset/).  

The goal of this project was to perform a **comprehensive, end-to-end analysis** of Yelp’s publicly available dataset, with a primary focus on the **restaurant industry**.  
The dataset offers a unique opportunity to explore **real-world, large-scale data** containing millions of reviews, detailed business attributes, geolocation data, and customer feedback spanning multiple years.  

## **The Dataset**  
---

The dataset contains over **6,000,000 business reviews** from **150'346 businesses** in **27 U.S. states**.  
The main focus of this research is **restaurants** — exploring their ratings, popularity, and customer feedback trends.  
Some tables exceeding **15,000,000** rows.  

![msedge_EReYpbqCQX.png](jb-image:img_1769687011601_3367465a7b35c)  

### **Questions I wanted to answer with my SQL Queries were:**  
- How many businesses are there in each state in this dataset?
- Which ten restaurants are the best in each state?  
- What cuisines receive the most reviews and the highest average ratings?  
- Are there patterns between restaurant price range, cuisine, and customer ratings?
- What do the worst reviews say about the best-rated restaurants say?
- How do review counts and ratings change over time for top restaurants?


---

# **Tools Used**  
---
+ **SQL:** The main tool used to analyze millions of rows efficiently. Specifically, SQLite was used.  
+ **DBeaver:** The chosen database management software.  
+ **Microsoft Excel:** Used to consolidate data and create charts.  
+ **Git & GitHub:** used to share my SQL analysis.  

---

# **The Analysis**  

## Hypothesis
---



---
## Data Cleanup
---
This, honestly, was the hardest part. It required lots of time to clean up the data and create new tables with recursive queries. Data and the tables grew to convert given **JSON files** to cleaned tables.

## ER Diagram
---



## The Queries
---

Each query in this project was designed to explore specific aspects of restaurant performance, customer preferences, and review trends.

### How many businesses are there in each state in this dataset?
---
This query returns the number of businesses in each state. 
```sql
SELECT
   State, 
    COUNT(*) OVER (PARTITION BY State) AS Business_Count
FROM BusinessCLEANED
ORDER BY Business_Count DESC;
```
The results:
![WINWORD_XnhrHc2Bgc.png](jb-image:img_1769693597916_5b619ad191e3d8)


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
![WINWORD_gEfdb8v1YE.png](jb-image:img_1769693894749_e9733d7b0ded9)

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


### Are there patterns between restaurant price range, cuisine, and customer ratings?
---
This data set doesn't include the price and cuisine type of the restaurant. AI was used to find these restaurants in Google Maps and to retrieve the price category and food type served. Power Query was used to clean this data, and pivot tables were  used to represent this data:
![WINWORD_xqZvWXbY7q.png](jb-image:img_1769694131026_2e93273d028c9)

Cheaper and averagely priced restaurants are more beloved than expensive or very expensive restaurants. But what about the cuisine type?

![WINWORD_rf7rUEZHzB.png](jb-image:img_1769694240164_598dc4695c0998)
![WINWORD_fZw2ysh1xU.png](jb-image:img_1769694257364_cb13133fd16158)

More common cuisines (American, Mexian and Asian) are more beloved than niche (Deli, Mediterranean) ones. 


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
![WINWORD_07I2ZLsOeJ.png](jb-image:img_1769694636542_9cd06a2d9dbd2)

### How do review counts and ratings change over time for top restaurants?
---
This Dataset doesn't include timestamps for the time when the review was given. If it had, then this is the query I would use to find the count of the reviews ranging from 2001 to 2099, to analyze the patterns, if the restaurant is doing better or worse.
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
The result would look like this:
![WINWORD_SwpsB3SlN6.png](jb-image:img_1769694950493_2bdf45cc7c0de8)

---

# **What I Learned**  
---
- 📜 **Advanced SQL Skills**: Learned to clean data, join multiple tables, use subqueries, and apply CTEs for cleaner, more efficient queries, using ROW_Number functions and lastly recursive commands. 
- 📈 **Data Visualization & Reporting**: Used Excel and Power Query to clean, transform, and consolidate results from SQL queries, creating interactive charts and dashboards for clear storytelling.  
- 🧠 **Insight Generation**: Converted millions of raw data points into actionable insights, such as identifying top cuisines, price to performance ratio and client insights to help business owners make the best restaurant.  

---

# **Conclusions**  
---
These results confirmed my hypothesis that:
* Averagely priced and cheaper priced restaurants will have more reviews and will be more attended.
* More well-known cuisines will be more beloved than niche cuisines.
* It's important to listen to the customers, since reviews greatly affect the performance of the restaurants. 

## What else could have  been done
---
* 🌎 This dataset contains Latitude and Longitude for each business. With this information, one could explore:
  * *Restaurant Density Heatmaps*: Identify “food clusters” in cities using geospatial clustering (e.g., DBSCAN).
  * *Travel Distance Analysis*: Calculate the  average distance between a user’s reviews to see how far they travel for food.
  * *Proximity Impact*: Measure if restaurants near tourist attractions or transport hubs have higher ratings.
* 📚 I barely touched the reviews table. One could extract the main keywords. These results will show what people love and hate the most. This can be done using python wordcloud library.
* 🥳 Secondly, the reviews don't always paint the full picture. One could also compare the sentiment of the reviews, which range from -1 to +1, to identify what people like most. Python TextBlob or Vader could be used for this.
* 😠 And lastly, one could look at the worst restaurants to learn from their mistakes. What do people dislike? Why? 

### Closing thoughts
---
In this project, I explored Yelp's open business data to find insights about restaurants, customer preferences, and business performance. I cleaned and prepared the dataset, then used SQL to answer questions like which restaurants have the highest ratings, which cuisines are most popular, and how review counts relate to customer satisfaction.

This showed me I can complete a data project from start to finish — from raw JSON files to a published GitHub portfolio. I became more proficient in SQL, using filtering, aggregation, joins, and subqueries to turn raw data into valuable insights.

Overall, I improved my SQL skills, strengthened my data cleaning abilities, and developed a business owner mindset.
