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

![Dataset.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/msedge_EReYpbqCQX.png)

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
This, honestly, was the hardest part. It required a lot of time to clean up the data and create new tables with recursive queries. Data and the tables grew to convert given **JSON files** to cleaned tables.

## ER Diagram
---
ER Diagram was created with an online ER diagram maker.
![ER Diagram.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/ER%20Diagram.png)


## The Queries
---

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

![CuisineType.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/WINWORD_xqZvWXbY7q.png)
![Pricing.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/WINWORD_fZw2ysh1xU.png)

More common cuisines (American, Mexican and Asian) are more beloved than niche (Deli, Mediterranean) ones. 


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
|BEAST Craft BBQ|1.0|I will review on the service because I strongly believe this is as important, if not more, than the food. I am one that goes back for take out rather than sit down if the service is lousy. With the Beast, this will probably be the case moving forward. nHere is my experience. So I and the misses are on our vacation and thought it would be a grand idea to try the Beast BBQ since we were in the area. It was at the tail end of lunch time so not a full house. Mind you this is our first time here so was not sure which end of the building is dining and which was self serve bar type. Went in and was greeted by a young lady believe name is Breanna or something around that name. WnAm not sure cause someone else yelled her attention by name. So anyway, she asked with a friendly greeting "dine in or take out". This should have been when I chose take out but, instead, said,"dine in". She led us to the counter which you have to order before any of your party members sit. No big deal cause we have been to Pappy's in St Louis. ( THIS PLACE BY NO MEANS CAN COMPARE TO PAPPY's). As soon as she gets behind the counter to take our order, the switcharoo personality changes. She became less friendly and more annoyed. Again, this is our first time and not familiar with the menu so we had to scan and think for a moment. She seem annoyed that she had to wait. This was at the tail end of lunch and we are the only ones in line. No one was behind us. I asked a question and received a negative attitude. There's this huge menu behind her and she seemed annoyed that she had to be questioned instead of me reading the menu. The wife could not make up her mind cause the menu look so good. The worker just looked away in disgust. I almost thought she may have rolled her eyes is the reason she looked away. She did not make any recommendation or assisted at all in helping us decide. She kept silent. No smiles!!!!! nnOrdered seated and waited for the meal. I was glad to get away from her. I thought she was gonna grow horns and spit fire at us. Now from that experience came an angel in servant clothing. This lady, much older, brought out our meal with a pleasant smile and checked on us every so often. Okay, she may have been the waitress and was her job to do so but at least she played her role correctly. The one behind the counter forgot she was working as a customer service and a representative of The Beast establishment. nnI hope and pray management reads this and have a talk with her. Shame cause I truly enjoyed the food. Brisket was dry but wife loved the pork steak.|
|BEAST Craft BBQ|1.0|After hearing great things about this place, I thought I'd give it a try today, driving from two towns over. Unfortunately, I left disappointed and hungry. All I wanted was the beef brisket. What kind of barbecue joint runs out of beef brisket at 4:45 on a Sunday afternoon? Can you imagine if Pizza Hut ran out of cheese and pepperoni?  If you're looking something in particular, call ahead...|
|BEAST Craft BBQ|1.0|What's the deal with always out of ribs!nYour BQ rest buy more ribs. There has to be a print out on Sales tickets showing Sales. Your food is Great, but you could sell more Ribs. Been there three times now for Ribs. None ! Sold out! Maybe review the numbers! Just saying!|
|BEAST Craft BBQ|1.0|The food is dry and tasteless. The whole operation appears to be run by teenagers and the food and service show it! Not recommended.|
|Baja Cafe|1.0|We tried to eat at Baja Cafe last night, since all available information said they were open for dinner (their website, yelp, etc.). When we arrived, there were several other annoyed patrons leaving, as the restaurant is clearly not actually open for dinner on Friday and Saturday nights, contrary to what they advertise. This wouldn't be a huge deal, but we drove 35 minutes just to eat here. I emailed the owner that night to let them know, and received an email saying I should have known better somehow (how?!) and they just haven't gotten around to updating their website. She was curt and unapologetic, which makes it clear she doesn't care about lost business. I'll be taking my money and time elsewhere.|
|Baja Cafe|1.0|Ordered the corned beef with eggs. What I was attracted to was the "we crispen our hash browns then add the meat". What I got was raw potatoes or damn near. I guess when the kitchen gets really busy and the dollar signs start rolling the quality dips. Not again.|
|Baja Cafe|1.0|I decided to give Baja Cafe a try because the menu looked different and a bit creative.  Huge mistake!! Our waitress took forever to bring us drinks and forgot the water we'd asked for. We sat on the patio. There turned out to be tons of flies, despite the gross fly paper hanging from the roof. It took forever for the waitress to take our order. The food was awful! I thought to go with a safe choice because the first impression was not good. I mean, how hard is it to get eggs, hash browns and bacon right? I ordered The Tower. I did not expect kraft singles. The eggs were a strange consistency and the hash browns had no flavor. My friend ordered chilaquiles. They looked better, but the taste was no bueno. Also, the coffee was super weak and flavorless. The cafe failed on ambience, food quality, and service. I will NOT be returning.|
|Baja Cafe|1.0|I never write reviews, but we got bad service twice in a row so writing a review felt warranted. I can understand that this place gets super busy, so my husband and I usually order take-out instead of dining in. nnLast week my husband called and ordered food and the person over the phone says that the food will be ready in 20 min. It takes me 25 min to drive there. I actually arrived 30-35 min after the order was placed over the phone and one female said that it wasn't ready yet. I'm thinking no big deal, food should be ready in 5 min, giving them a total of 35-40 min to make a take-out order. I didn't get my food until 30 min later! And the whole time I was standing/sitting there, the workers would look at me and then turn away. No one offered me anything. No one went to see why the order was taking so long. When I got my order, there was no apology, nothing. I told one of the workers who just looked at me to next time tell customers that take-out orders will take at least an hour, and not to lie and say it will take 20 min. Horrible customer service! I would rather have honesty than be lied to and then pretend like nothing's wrong. At least do something for the customer who has been waiting a half hour for take-out and not dine-in. Offer a cup of free coffee, give a free pancake, or just say sorry!nnSo, today we ordered food again over the phone. And this time, I'll just let my husband complain in another post.|


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
The result would look like this:
![RatingsOverYears.png](https://github.com/ronalds141/SQL-Portfolio/blob/main/WINWORD_SwpsB3SlN6.png)

---

# **What I Learned**  
---
- **Advanced SQL Skills**: Learned to clean data, join multiple tables, use subqueries, and apply CTEs for cleaner, more efficient queries, using ROW_Number functions and lastly recursive commands. 
- **Data Visualization & Reporting**: Used Excel and Power Query to clean, transform, and consolidate results from SQL queries, creating interactive charts and dashboards for clear storytelling.  
- **Insight Generation**: Converted millions of raw data points into actionable insights, such as identifying top cuisines, price to performance ratio and client insights to help business owners make the best restaurant.  

---

# **Conclusions**  
---
These results confirmed my hypothesis that:
* Averagely priced and cheaper priced restaurants will have more reviews and will be more attended.
* More well-known cuisines will be more beloved than niche cuisines.
* It's important to listen to the customers, since reviews greatly affect the performance of the restaurants. 

## What else could have  been done
---
* This dataset contains Latitude and Longitude for each business. With this information, one could explore:
  * *Restaurant Density Heatmaps*: Identify “food clusters” in cities using geospatial clustering (e.g., DBSCAN).
  * *Travel Distance Analysis*: Calculate the  average distance between a user’s reviews to see how far they travel for food.
  * *Proximity Impact*: Measure if restaurants near tourist attractions or transport hubs have higher ratings.
* I barely touched the reviews table. One could extract the main keywords. These results will show what people love and hate the most. This can be done using python wordcloud library.
* Secondly, the reviews don't always paint the full picture. One could also compare the sentiment of the reviews, which range from -1 to +1, to identify what people like most. Python TextBlob or Vader could be used for this.
* And lastly, one could look at the worst restaurants to learn from their mistakes. What do people dislike? Why? 

### Closing thoughts
---
In this project, I explored Yelp's open business data to find insights about restaurants, customer preferences, and business performance. I cleaned and prepared the dataset, then used SQL to answer questions like which restaurants have the highest ratings, which cuisines are most popular, and how review counts relate to customer satisfaction.

This showed me I can complete a data project from start to finish — from raw JSON files to a published GitHub portfolio. I became more proficient in SQL, using filtering, aggregation, joins, and subqueries to turn raw data into valuable insights.

Overall, I improved my SQL skills, strengthened my data-cleaning abilities, and developed a business-owner mindset.
