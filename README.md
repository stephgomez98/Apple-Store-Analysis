# Apple-Store-Analysis 📱
An app developer needs data driven insights to decide what type of app to build.
# Context
In this project the app developers are identified as the stakeholders. App developers often rely on data-driven insights to determine what type of app to build, understanding user preferences, market trends, and the competitive landscape. For this project I will be helped the app developer gain these insights in order to be more strategic in their decision-making process, leading to the development of apps that are aligned with user preferences, market trends, and have a competitive advantage in the marketplace.
#Dataset Source
https://www.kaggle.com/datasets/gauthamp10/apple-appstore-apps. 
# Objective
The project's results and insights will contribute to the app developers' ability to make informed decisions about app development strategies, prioritize features and functionalities based on user demand, optimize marketing efforts, and ultimately enhance the success and profitability of their apps in the competitive app marketplace.
# Column Descriptions:
| Column Name          | Description                                                                                   |
| --------------------  | --------------------------------------------------------------------------------------------- |
| app_name             | The name of the mobile application.                                                            |
| genre                | The category or genre of the app, e.g., Games, Productivity, Social Networking, etc.          |
| content_rating       | The content rating of the app, such as "Everyone," "Teen," or "Mature 17+."                    |
| size_bytes           | The size of the app in bytes.                                                                  |
| required_ios_version | The minimum iOS version required to run the app.                                              |
| released_date        | The date when the app was initially released to the App Store.                                 |
| updated_date         | The date when the app was last updated on the App Store.                                      |
| version              | The version number of the app, which typically follows a format like "1.2.3".                  |
| price                | The price of the app, which can be in various currencies, or it can be "Free" for free apps.    |
| free                 | If the app is free it has the value of true for the row    |
| developer_id         | The unique identifier for the app's developer or development team.                             |
| developer_name       | The name of the developer or development team responsible for the app.                         |
| average_user_rating  | The average user rating of the app, often displayed as a number out of 5 stars.                |
| number_of_reviews    | The total number of user reviews and ratings for the app.                                     |
# Cleaning Data
- Setting proper naming conventions:

The dataset originally used Pascal Snake Case naming conventions for each column. I decided to switch to Snake Case naming conventions for enhanced readability. Snake_case not only improves readability but also makes easier to identify when claues and functions are used.
````sql
EXEC sp_rename 'data.App_Name', 'app_name', 'COLUMN';
EXEC sp_rename 'data.Primary_Genre', 'genre', 'COLUMN';
EXEC sp_rename 'data.Content_Rating', 'content_rating', 'COLUMN';
EXEC sp_rename 'data.Size_Bytes', 'size_bytes', 'COLUMN';
EXEC sp_rename 'data.Required_IOS_Version', 'required_ios_version', 'COLUMN';
EXEC sp_rename 'data.Released', 'released_date', 'COLUMN';
EXEC sp_rename 'data.Updated', 'updated_date', 'COLUMN';
EXEC sp_rename 'data.Version', 'version', 'COLUMN';
EXEC sp_rename 'data.Price', 'price', 'COLUMN';
EXEC sp_rename 'data.Currency', 'currency', 'COLUMN';
EXEC sp_rename 'data.Free', 'free', 'COLUMN';
EXEC sp_rename 'data.DeveloperId', 'developer_id', 'COLUMN';
EXEC sp_rename 'data.Developer', 'developer_name', 'COLUMN';
EXEC sp_rename 'data.Average_User_Rating', 'average_user_rating', 'COLUMN';
EXEC sp_rename 'data.Number_of_Reviews', 'number_of_reviews', 'COLUMN';
````
- Setting proper data type and size:
I've enchanced the integrity of the dataset by correcting the datatypes, making sure that data types the CHAR, VARCHAR, and TEXT, and being applied to only textual information. This practice also helps prevent potentional erros when attempting arithmetic operations on data meant for text. In these alter queries I also changed the size of the columns to optimize data storage and increase the speed of data retrievel and query performance since smaller column sizes result in faster data access and processing. This change allows for less memory to be taken up as well.


````sql
ALTER TABLE data
ALTER COLUMN [size_bytes] BIGINT;

ALTER TABLE data
ALTER COLUMN [app_name] VARCHAR(350);

ALTER TABLE data
ALTER COLUMN [genre] CHAR(25);

ALTER TABLE data
ALTER COLUMN [required_ios_version] VARCHAR(10);

ALTER TABLE data
ALTER COLUMN [version] VARCHAR(100);
````
- Removing duplicates for app name:
````sql
WITH CTE AS
(
SELECT app_name, genre, content_rating, size_bytes, required_ios_version, 
	released_date, updated_date, version, price, currency, free, developer_id, 
	developer_name, average_user_rating, number_of_reviews,
	ROW_NUMBER() OVER(PARTITION BY app_name ORDER BY (SELECT NULL)) AS RowNum
FROM [dbo].[data]
)
DELETE FROM CTE
WHERE RowNum>1;
````
- Removing rows containing null values:
````sql
DELETE FROM [dbo].[data]
WHERE
    [app_name] IS NULL	OR
	[genre] IS NULL OR
	[content_rating] IS NULL OR
	[size_bytes] IS NULL OR 
	[required_ios_version] IS NULL OR
	[released_date] IS NULL OR
	[updated_date]IS NULL OR 
	[version] IS NULL OR
	[price] IS NULL OR
	[currency]IS NULL OR
	[free] IS NULL OR
	[developer_id] IS NULL 
	[developer_name] IS NULL OR 
	[average_user_rating] IS NULL OR 
	[number_of_reviews] IS NULL OR
	[content_rating] IS NULL ;
  ````
- Removing rows containing empty strings:
````sql
	DELETE FROM [dbo].[data]
	WHERE [genre] ='' OR
		  [price] ='' OR
		  [average_user_rating] ='' OR
		  [number_of_reviews] ='' OR
		  [currency] ='' OR
		  [developer_id] ='' OR
		  [developer_name] ='' OR
		  [app_name] ='';
  ````
- Changing price and average user rating to round to second decimal:
````sql
	UPDATE data
	SET Price = ROUND(Price, 2),
	average_user_rating = ROUND(average_user_rating, 2);
````
# Descriptive Statitics 
Descriptive statistics are used to summarize, simplify, and understand datasets. They provide a quick overview of data's central tendencies, spread, and distribution, helping analysts identify patterns, trends, and outliers. 
- Central tendency: mean, median, mode

Price: Understanding the average price can help determine pricing strategies for your app and assess user willingness to pay.

mean
````sql
SELECT AVG(price) AS mean
FROM table_name;
````
    Genre and Content Rating:This information helps identify popular genres and preferred content ratings, aiding your decision on app type and age-appropriateness.
````sql
mode for genre
WITH CountValues AS (
    SELECT genre, COUNT(*) AS value_count
    FROM data
    GROUP BY genre
),
MaxCount AS (
    SELECT MAX(value_count) AS max_count
    FROM CountValues
)
SELECT genre
FROM CountValues
WHERE value_count = (SELECT max_count FROM MaxCount);

mode for content rating
WITH CountValues AS (
    SELECT content_rating, COUNT(*) AS value_count
    FROM data
    GROUP BY content_rating
),
MaxCount AS (
    SELECT MAX(value_count) AS max_count
    FROM CountValues
)
SELECT content_rating
FROM CountValues
WHERE value_count = (SELECT max_count FROM MaxCount);
````
Size Bytes: Knowing the average app size helps you plan for storage, user experience, and understand user preferences regarding app size.
````sql
mean
SELECT AVG(size_bytes) AS mean
FROM table_name;
````
          Average User Rating: These statistics help you gauge the overall user satisfaction with apps in the dataset, which can guide your app development.
````sql
mean
SELECT AVG(average_user_rating) AS mean
FROM table_name;
median
mode
WITH CountValues AS (
    SELECT average_user_rating, COUNT(*) AS value_count
    FROM data
    GROUP BY average_user_rating
),
MaxCount AS (
    SELECT MAX(value_count) AS max_count
    FROM CountValues
)
SELECT average_user_rating
FROM CountValues
WHERE value_count = (SELECT max_count FROM MaxCount);
````
Number of Reviews: Understanding review counts helps you assess app popularity and user engagement.
````sql
mean
SELECT AVG(number_of_reviews) AS mean
FROM table_name;
median
mode
WITH CountValues AS (
    SELECT number_of_reviews, COUNT(*) AS value_count
    FROM data
    GROUP BY number_of_reviews
),
MaxCount AS (
    SELECT MAX(value_count) AS max_count
    FROM CountValues
)
SELECT number_of_reviews
FROM CountValues
WHERE value_count = (SELECT max_count FROM MaxCount);
````
Required iOS Version: Knowing the average and median required iOS version helps you target a suitable iOS user base for your app.
````sql
mean
SELECT AVG(required_ios_version) AS mean
FROM table_name;
median
````
# Variability Analysis 
Variability analysis is crucial for understanding the spread and dispersion of data in your dataset. 

- Variability: varince, standard deviation, max, and min

Standard Deviation of User Ratings by Genre:

This query calculates the standard deviation of user ratings for different app genres. High standard deviations indicate a wide range of user ratings, which may suggest more variability in user preferences within that genre. Understanding this variability can help developers target specific genres with more or less rating consistency.
````sql
SELECT genre, AVG(average_user_rating) AS avg_user_rating, STDEV(average_user_rating) AS rating_std_dev
FROM data
GROUP BY genre
ORDER BY rating_std_dev DESC;
````
Price Variability by Genre:

This query explores the variability in app prices within different genres. It calculates the standard deviation of prices for each genre, helping developers identify genres with varying pricing strategies and user expectations. High price standard deviations may indicate a wide range of prices in a genre.
````sql
SELECT genre, MAX(price) AS max_price, MIN(price) AS min_price, STDEV(price) AS price_std_dev
FROM data
GROUP BY genre
ORDER BY price_std_dev DESC;
````
Variability in Number of Reviews by Genre:

This query assesses the variability in the number of reviews within different app genres. High standard deviations in review counts may signify variability in user engagement and the competitive nature of the genre.
````sql
SELECT genre, MAX(number_of_reviews) AS max_reviews, MIN(number_of_reviews) AS min_reviews, STDEV(number_of_reviews) AS reviews_std_dev
FROM data
GROUP BY genre
ORDER BY reviews_std_dev DESC;
````
Release Date Variability by Genre:

Analyzing the variability in release dates within genres helps developers understand how genres evolve over time. High standard deviations in release dates may indicate dynamically changing genres or inconsistent app updates.

````sql
SELECT genre, MAX(released_date) AS latest_release, MIN(released_date) AS earliest_release, STDEV(released_date) AS release_date_std_dev
FROM data
GROUP BY genre
ORDER BY release_date_std_dev DESC;
````
Version Update Frequency by Genre:

This query explores the variability in app update frequency (measured in days between updates) within genres. High standard deviations in update frequencies may indicate irregular update patterns, which can be important for competitive analysis and user satisfaction.

````sql
SELECT genre, AVG(updated_date - released_date) AS avg_days_between_updates, STDEV(updated_date - released_date) AS update_freq_std_dev
FROM data
GROUP BY genre
ORDER BY update_freq_std_dev DESC;
````
Content Rating Variability by Genre:

This query calculates the standard deviation of user ratings within each genre and content rating combination. It helps identify genres and content ratings with varying user preferences, which can be important for building apps that align with audience expectations.

````sql
SELECT genre, content_rating, COUNT(*) AS app_count, STDEV(average_user_rating) AS rating_std_dev
FROM data
GROUP BY genre, content_rating
HAVING COUNT(*) > 10
ORDER BY genre, rating_std_dev DESC;
````
# Explore Categorical Data

Exploring categorical data in data analysis is a fundamental practice because it unveils critical insights within datasets.It aids in tailored decision-making, such as choosing the right type of app to build, aligning content with age-appropriate ratings, and determining pricing strategies. Additionally, it ensures user satisfaction and app quality by analyzing user ratings, reviews, and engagement. Categorical data exploration also helps developers stay up-to-date with market trends, identify successful peers, explore collaboration opportunities, and maintain compliance with content guidelines. In summary, exploring categorical data is essential for data analysis as it equips developers with the insights needed to create user-focused, successful apps and adapt to the ever-changing app market.

Explore App Genres: 

This query helps app developers identify popular and in-demand app genres. Knowing which genres have a higher number of apps can inform developers about potential opportunities in those genres and help them make informed decisions about the type of app to build.
````sql
SELECT DISTINCT [genre], COUNT(*) AS app_count
FROM [dbo].[data]
GROUP BY [genre]
ORDER BY app_count DESC;
````
Explore Content Ratings:

Understanding the distribution of content ratings allows developers to ensure that their apps are appropriately rated for their target audience. It helps in adhering to age restrictions and creating age-appropriate content, which is crucial for user satisfaction and compliance.
````sql
SELECT DISTINCT [content_rating], COUNT(*) AS app_count
FROM [dbo].[data]
GROUP BY [content_rating]
ORDER BY app_count DESC;
````
Identify Top Developers:

Identifying top developers by the number of apps they've developed can help developers identify potential collaborators, learn from successful developers, or assess the competitive landscape in the app store.
````sql
SELECT developer_name, COUNT(*) AS app_count
FROM [dbo].[data]
GROUP BY developer_name
ORDER BY app_count DESC;
````
Analze User Rating:

This query helps developers evaluate the average user rating for each genre. It provides insights into which genres tend to have higher user satisfaction, enabling developers to make genre-specific decisions and understand user preferences.
````sql
SELECT genre, AVG(average_user_rating) AS avg_user_rating
FROM [dbo].[data]
GROUP BY genre
ORDER BY avg_user_rating DESC;
````
User Engagement:

By counting the number of reviews for apps in each genre, developers can gauge the level of user engagement and feedback. Genres with more reviews may indicate competitive markets and user interaction.
````sql
SELECT genre, SUM(number_of_reviews) AS total_reviews
FROM [dbo].[data]
GROUP BY genre
ORDER BY total_reviews DESC;
````
Age-Appropriate Genre: 

This query helps developers explore the intersection of genre and content rating, ensuring that apps are suitable for the intended audience. It assists in aligning content with age-appropriate categories.
````sql
SELECT content_rating, genre, COUNT(*) AS app_count
FROM [dbo].[data]
GROUP BY content_rating, genre
ORDER BY content_rating, app_count DESC;
````
Developer Performance:

Finding top developers by the number of apps and their average user ratings provides insights into successful developers. Developers can consider collaboration opportunities, benchmark against successful developers, and assess the competitive landscape.
````sql
SELECT developer_name, COUNT(*) AS app_count, AVG(average_user_rating) AS avg_user_rating
FROM [dbo].[data]
GROUP BY developer_name
ORDER BY app_count DESC;
````
App Version Trends: 

Analyzing app version trends by genre helps developers understand the update frequency and track the latest versions. Frequent updates may signify an active developer and a responsive app, important for maintaining user engagement.
````sql
SELECT genre, MAX(version) AS latest_version, MIN(released_date) AS first_release_date
FROM [dbo].[data]
GROUP BY genre
ORDER BY genre;
````
App Lifecycle: 

Examining the app lifecycle by genre and the average time between updates provides insights into how long apps remain relevant in specific genres. Developers can identify trends and tailor their update strategies.
````sql
SELECT genre, DATEDIFF(day, MIN(released_date), MAX(updated_date)) AS average_days_between_updates
FROM [dbo].[data]
GROUP BY genre
ORDER BY genre;
````
# Exploratory Data Analysis
EDA is a crucial step in data analysis that provides a comprehensive understanding of the data, uncovers insights, identifies data quality issues, and sets the stage for more advanced analyses. It is a data exploration process that supports better decision-making and informed data analysis.

Genre Analysis for User Ratings and Engagement:

This query is helpful because it reveals which app genres have the highest average user rating and the most user reviews provides actionable insights for app developers. It assists in genre selection, user engagement strategies, and competitive positioning. Understanding user preferences and satisfaction is key to building successful apps and maintaining a competitive edge in the market.
````sql
SELECT genre, 
       AVG(average_user_rating) AS avg_rating, 
       SUM(number_of_reviews) AS total_reviews
FROM data
WHERE average_user_rating IS NOT NULL
GROUP BY genre
ORDER BY avg_rating DESC, total_reviews DESC;
````
Content Rating Analysis for User Satisfaction:

This query helps app developers make data-informed decisions about content, targeting specific content ratings that lead to high user satisfaction and ensuring compliance with guidelines. It contributes to the development of apps that are well-received by users and can guide content strategy and improvement efforts.

````sql
SELECT content_rating, 
       AVG(average_user_rating) AS avg_Rating, 
       SUM(number_of_reviews) AS total_Reviews
FROM data
WHERE average_user_rating IS NOT NULL
GROUP BY content_rating
ORDER BY avg_rating DESC, total_reviews DESC;
````
Price Analysis for User Ratings and Engagement:

this query is helpful because it assists app developers in making informed decisions about app pricing, user satisfaction, and engagement. It provides valuable insights for shaping pricing strategies, optimizing resource allocation, and positioning apps effectively in the competitive app market.

````sql
SELECT
    price,
    AVG(average_user_rating) AS avg_rating,
    SUM(number_of_reviews) AS total_reviews
FROM data
WHERE average_user_rating IS NOT NULL
GROUP BY price
ORDER BY avg_rating DESC, total_reviews DESC;
````
Top Developers Analysis for App Inspiration and Insights:

this query is helpful because it provides a source of inspiration, insights, and valuable information for app developers. It offers a window into the strategies and success of top developers, enabling others to learn from their experiences and make informed decisions about their own app development projects.
````sql
SELECT TOP 10 developer_name, COUNT(*) AS App_Count
FROM data
GROUP BY developer_name
ORDER BY App_Count DESC;
````
App Size Analysis for User Ratings Impact:

this query is helpful as it provides valuable insights into the relationship between app size and user ratings, allowing developers to optimize resource allocation, enhance user satisfaction, and align their app development strategies with market expectations.
````sql
SELECT
    CASE
        WHEN size_bytes < 10000000 THEN 'Small'
        WHEN size_bytes < 50000000 THEN 'Medium'
        ELSE 'Large'
    END AS App_Size,
    AVG(average_user_rating) AS avg_rating
FROM data
WHERE average_user_rating IS NOT NULL
GROUP BY
    CASE
        WHEN size_bytes < 10000000 THEN 'Small'
        WHEN Size_bytes < 50000000 THEN 'Medium'
        ELSE 'Large'
    END;
````
iOS Version Analysis for User Ratings Optimization:

This query is helpful because it provides insights into the relationship between the required iOS version and user ratings, allowing app developers to optimize user satisfaction, target the right iOS versions, and make informed decisions about resource allocation and market strategy

````sql

SELECT
    Required_IOS_Version,
    AVG(Average_User_Rating) AS Avg_Rating
FROM data
WHERE Average_User_Rating IS NOT NULL
GROUP BY Required_IOS_Version
ORDER BY Avg_Rating DESC;
````
Popularity Analysis of Paid Apps in User Ratings and Reviews:

this query is helpful because it provides insights into the popularity of paid apps in terms of user engagement, user satisfaction, and competitiveness. It assists app developers in optimizing their app development, pricing, and marketing strategies for paid apps

````sql
SELECT
    AVG(Average_User_Rating) AS Avg_Rating,
    SUM(Number_of_Reviews) AS Total_Reviews
FROM data
WHERE price>0
    AND Average_User_Rating IS NOT NULL
GROUP BY Price
ORDER BY total_reviews DESC;
````
# Ad Hoc Queries

Ad hoc queries allows the flexibility to address unforeseen questions, adapt to changing requirements, and facilitate agile decision-making. They empower analysts to delve deeper into the data and uncover valuable insights that may not have been part of the initial analysis plan. Below I have come up with ad hoc query questions that departments or individuals may be asking an analyst from this dataset.

This ad hoc query retrieves apps that belong to the "Games" genre and have a "Teen" content rating.

````sql
SELECT
    app_name,
    genre,
    content_rating
FROM data
WHERE genre = 'games' AND content_rating >12;
````
Query to find apps with file sizes greater than a specific threshold
````sql
SELECT
    app_name,
    size_bytes
FROM data
WHERE size_bytes > 100000000;
````
Query to identify apps with both high user ratings and a substantial number of reviews.
````sql
SELECT
    app_name,
    average_user_rating,
    number_of_reviews
FROM data
WHERE average_user_rating >= 4.5 AND number_of_reviews > 1000
ORDER BY average_user_rating DESC, number_of_reviews DESC;
````
This query to find the top 10 most expensive paid apps.
````sql
SELECT TOP 10
    app_name,
    price
FROM data
WHERE price > 10
ORDER BY price DESC
````
# Final Recommendations using POWER BI


https://github.com/stephgomez98/Apple-Store-Analysis/assets/129236241/3df175a0-4ee8-4bca-97d5-79c5a9dca9d8

