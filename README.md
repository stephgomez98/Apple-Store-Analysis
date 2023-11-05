# Apple-Store-Analysis 📱
An app developer needs data driven insights to decide what type of app to build.
# Context
In this project the app developers are identified as the stakeholders. App developers often rely on data-driven insights to determine what type of app to build, understanding user preferences, market trends, and the competitive landscape.For a recent project aimed at aiding this decision-making process, I leveraged a valuable resource: an extensive app store dataset obtained from Kaggle. This dataset, accessible at https://www.kaggle.com/datasets/gauthamp10/apple-appstore-apps. 
# Objective
By analyzing and drawing insights from this comprehensive dataset, I aimed to help app developers make data-informed choices when deciding on the type of app to develop. This involved assessing market demands, identifying successful genres, understanding user preferences, and considering competitive factors. The dataset from Kaggle served as a foundational resource, facilitating a data-centric approach to app development and enabling developers to create apps that resonate with their target audience. The project's results and insights will contribute to the app development community, , offering valuable information for strategizing, planning, and ultimately creating successful and impactful mobile applications
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
| developer_id         | The unique identifier for the app's developer or development team.                             |
| developer_name       | The name of the developer or development team responsible for the app.                         |
| average_user_rating  | The average user rating of the app, often displayed as a number out of 5 stars.                |
| number_of_reviews    | The total number of user reviews and ratings for the app.                                     |
# Cleaning Data
- Setting proper naming conventions:

The dataset originally used Pascal Snake Case naming conventions for each column, but I decided to switch to Snake Case naming conventions for enhanced readability. Snake_case not only improves readability but also makes it more straightforward to identify and understand the operations applied to the data.
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
  
I've made a significant enhancement to the dataset by aligning columns with the proper data types.For instance, I've ensured that text-based data types like CHAR, VARCHAR, and TEXT, designed for storing textual information, are employed solely for their intended purpose. This measure helps prevent potential errors stemming from attempting arithmetic operations on data primarily meant for text, a practice that can lead to inaccuracies.Furthermore, I've optimized data storage by employing data types and sizes that are a precise fit for the information they represent. By avoiding overly large data types, we mitigate the risk of squandering storage space, particularly vital in the context of expansive databases. Moreover, this meticulous attention to the appropriate sizes contributes to the overall efficiency of data retrieval and query performance. Smaller data types not only process faster but also demand less memory resources, resulting in expedited and more resource-efficient data operations.
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
Descriptive statistics are used to summarize, simplify, and understand datasets. They provide a quick overview of data's central tendencies, spread, and distribution, helping analysts identify patterns, trends, and outliers. Descriptive statistics aid in data cleaning, comparison, quality assurance, and decision-making. They also serve as a foundation for data visualization and effective communication of results, making complex data more accessible and actionable.
- Central tendency: mean, median, mode
````sql
mean
SELECT AVG(column_name) AS mean
FROM table_name;

median
````
# Explore Categorical Data
````sql
SELECT DISTINCT [genre], COUNT(*) AS app_count
FROM [dbo].[data]
GROUP BY [genre]
ORDER BY app_count DESC;


SELECT DISTINCT [content_rating], COUNT(*) AS app_count
FROM [dbo].[data]
GROUP BY [content_rating]
ORDER BY app_count DESC;
````
# Exploratory Data Analysis
