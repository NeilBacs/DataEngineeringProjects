# Data Modeling in SQL

#### I use SQL Server as the relational database management system
#### Start by creating database

```sql
CREATE DATABASE MyChessDotComClub;
```

#### Create the three tables and implement primary key and foreign keys to create the relationships between the tables
```sql
USE MyChessDotComClub;

CREATE TABLE Members
(
    member_id INT PRIMARY KEY,
    username  VARCHAR(50),
);


CREATE TABLE Joined_Details
(
    id INT,
    joined_date DATE,
    CONSTRAINT FK_Joined_Details_id FOREIGN KEY (id) REFERENCES Members(member_id) ON UPDATE CASCADE ON DELETE CASCADE
    
);

CREATE TABLE Ratings
(
    id INT ,
    rapid INT,
    tactics INT,
    CONSTRAINT FK_Ratings_id FOREIGN KEY (id) REFERENCES Members(member_id) ON UPDATE CASCADE ON DELETE CASCADE
);
```

#### Insert the csv files we created in python
```sql
BULK INSERT MyChessDotComClub..Members
FROM 'C:\Users\MyPath\Portfolio_DataEngineering\df_users.csv'
WITH (
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    FIRSTROW = 2
);

BULK INSERT MyChessDotComClub..Joined_Details
FROM 'C:\Users\MyPath\Portfolio_DataEngineering\df_joined_members.csv'
WITH (
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    FIRSTROW = 2
);


BULK INSERT MyChessDotComClub..Ratings
FROM 'C:\Users\MyPath\Portfolio_DataEngineering\df_stats_of_members.csv'
WITH (
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    FIRSTROW = 2
);
```

#### Inserting data in relational database is complete. We can now perform queries that we want
#### Now the next step is create visualizations in PowerBI
