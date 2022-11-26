# GDC_TestCase
Test case for the GDC job interview. 18/11/22
 
 
## --- Table of content ---
 
 
 - I    - Install MariaDB
 - II   - Install DBeaver
 - III  - Example of how to upload a csv file to SQL DB
 - IV   - First treatement : in SQL
 - V    - First analysis : in SQL
 - VI    - Analysis in Python : Anaconda
 - VII   - Business and product conclusions
 - VIII  - Conclusion : Suggestions for improvements
 
 
 
 
## --- I. Install Maria DB ---

###Why MariaDB? 
My goal was to install a relational database management system on my personnal computer, mostly for this exercise, but with the possibility of using it for future projects. I wanted something that worked fast, without fuss, and was good with smaller databases.
A quick research let me to conclude MariaDB was my best option.
Also, it is widely used, so its worth has been soundly proven.


1/ Download it on this page :
https://mariadb.org/download/?t=mariadb&p=mariadb&r=10.10.2&os=windows&cpu=x86_64&pkg=msi&m=icam

2/ Follow this tutorial :
https://www.mariadbtutorial.com/getting-started/install-mariadb/



## --- II. Install DBeaver ---

###Why DBeaver?
I was looking for a SQL client software application and a database administration tool, free and accessible for Windows (my computer's system), as well as Linux and Mac OS as requested.
After a short enquiry I realised there are quite a few options out there. I picked DBeaver for 3 reasons : at first glance I found it user-friendly, it's recommanded by MariaDB, and i like its name. I have a positive bias for people confident enough to have chosen a whimsical name for their product.


1/ Download it on this page :
https://dbeaver.io/

2/ Follow this tutorial :
[https://www.cdata.com/kb/tech/mariadb-jdbc-dbvr.rst](https://github.com/dbeaver/dbeaver/wiki/Create-Connection)





## --- III. Example of how to upload a csv file to SQL DB ---

### Uploading the tables...

--Create DB
CREATE DATABASE GDC_TestCase1 ;

USE GDC_TestCase1 ;


--Create Table
CREATE TABLE ads (
	AA INTEGER,
	owner_id INTEGER,
	title VARCHAR(255),
	category VARCHAR(255),
	price INTEGER,
	city VARCHAR(255),
	created_at DATE,
	deleted_at DATE,
	id INTEGER,
	PRIMARY KEY (id)
);


-- Import data from csv file
LOAD DATA INFILE 'C:/Users/xavir/Documents/GDC_TestCase/ads.csv' 
INTO TABLE GDC_TestCase1.ads
FIELDS
  TERMINATED BY ','
  ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
IGNORE 1 ROWS (AA,owner_id,title,category,price,city,created_at,deleted_at,id);


### ... But also linking the tables.

-- Add Foreign Keys once all the tables are set
ALTER TABLE referrals  
ADD FOREIGN KEY (referrer_user_id) REFERENCES users(id);



## --- IV. First treatement : in SQL ---

While the previous chapter is generic enough to apply to all .csv files, I did manipulate the table once imported to do a first clean-up and understand what was in them on a very basic level. Here are a few example of things I did that are relevant. It is in no way an exhaustive list of the manipulations made.

### All tables :

-- Set Nulls

UPDATE users
SET age=NULL
WHERE age='0';

UPDATE users
SET birthdate=NULL
WHERE birthdate='0000-00-00';

UPDATE users
SET city=NULL
WHERE city='';


### Table ads :
-- Check if 1st and past column identical, Make 1st column the id column
SELECT AA, id
from ads
WHERE AA <> id ;

ALTER TABLE ads
DROP id;

ALTER TABLE ads
RENAME COLUMN AA TO id ;

-- Standardize fields
SELECT DISTINCT category
FROM ads;

ALTER TABLE category
ADD category_clean VARCHAR(255);

UPDATE ads
SET category_clean = CASE
  WHEN category = 'vehicle' THEN 'vehicle'
  WHEN category = 'realestate' OR category = 'REAL_ESTATE' OR category = 'relestate' THEN 'real_estate'
  ELSE null
END ;


## --- V. First analysis : in SQL ---

Here are a few notes I took while looking into the structure and content of the tables. 
They contain remarks as to what to look further into, questions that arose, and a few basic suggestions as to improvements.
NB : some remarks are quite obvious, maybe even unnecessary. I just wanted to lay out my whole process.

### Ads : 
40 000 rows
id : 40 000 distinct
  !! No errors here
owner_id : 2 '0', 14 313 distinct
	!!  '0' is normal.
title : 0 NULL, 39 distinct
	!! 39 for 40 000 rows : So Selection from a list. 
  !! Check if fields clean. 
  !! Check in list clean. 
category : 0 NULL, 4 distinct
	!! 3 real estate poorly orthographied. How did that happen? (Historical? other?)
  !! Why only 2 categories (once cleaned) ? 2 existing? Or others poorly advertised or accessible?
price : 0 NULL, 21 135 distinct
city : 0 NULL, 6 561 distinct
  !! Check if clean
created_at : 0 NULL, 972 distinct
  !! If it was the full DB, it would be a 3-4 years old product
deleted_at : ALL NULL
  !! Why none deleted?

### ads_transactions
6 645 rows
id : 6 645 distinct
  !! No errors here
ad_owner_id : 0 null, 5 361 distinct
buyer_user_id : 0 null, 6 645 distinct
ad_id : 0 null, 6 645 distinct
sold_price : 0 null, 6 645 distinct
created_at : 0 null, 999 distinct
  !! Hehe :D 

ad_owner_id = buyer_user_id : 


### Users
30 000 rows
age : 0 null
bday : 0 null
city : 0 null
created_at : 0 null
sex: 3052 null ; F,M, M., Mr
	!! do we want it compulsory? why do we even want it at all?
	!! Block keybord entries : list (if we keep)
lat : 6000 null
lon : 6000 null
password : 12 nulls 2017-2020
	!! Find where it comes from. Contact them or delete them.
  !! Block creation of account without passport
utm_source : 5985 null ; fb, mailing
utm_medium : 6966 null ; ad, email, social
utm_campaign : 5942 null
firstname : 0 null
lastname : 0 null
user_agent : 0 null
misc : 0 null

	!! deleted? add field deleted_at DATE (for tracking interest and usage)


### Referals
303 958 rows
id : 303 958 distinct
referrer_user_id : 0 null, 23 543 distinct
referree_user_id : 0 null, 23 523 distinct
created_at : 0 null
deleted_at : All null
  !! Odd

referrer_user_id=referree_user_id = 40
	!! Look into how it is possible. Make sure it gets blocked.



## --- VI. Analysis in Python : Anaconda ---


## --- VII. Business and product conclusions ---


## --- VIII. Conclusion : Suggestions for improvements ---


