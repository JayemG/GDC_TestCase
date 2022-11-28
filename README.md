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
```
CREATE DATABASE GDC_TestCase1 ;

USE GDC_TestCase1 ;
```

--Create Table
```
SET sql_mode = '';
SET sql_mode = 'IGNORE_SPACE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

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
```


-- Import data from csv file
```
LOAD DATA INFILE 'C:/Users/xavir/Documents/GDC_TestCase/ads.csv' 
INTO TABLE GDC_TestCase1.ads
FIELDS
  TERMINATED BY ','
  ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
IGNORE 1 ROWS (AA,owner_id,title,category,price,city,created_at,deleted_at,id);
```

### ... But also linking the tables.

-- Add Foreign Keys once all the tables are set
```
ALTER TABLE referrals  
ADD FOREIGN KEY (referrer_user_id) REFERENCES users(id);
```


## --- IV. First treatement : in SQL ---

While the previous chapter is generic enough to apply to all .csv files, I did manipulate the table once imported to do a first clean-up and understand what was in them on a very basic level. Here are a few example of things I did that are relevant. It is in no way an exhaustive list of the manipulations made.

### All tables :

-- Set Nulls
```
UPDATE users
SET age=NULL
WHERE age='0';

UPDATE users
SET birthdate=NULL
WHERE birthdate='0000-00-00';

UPDATE users
SET city=NULL
WHERE city='';
```

### Table ads :
-- Check if 1st and last column identical, Make 1st column the id column
```
SELECT AA, id
from ads
WHERE AA <> id ;

ALTER TABLE ads
DROP id;

ALTER TABLE ads
RENAME COLUMN AA TO id ;
```

-- Standardize fields
```
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
```

## --- V. First analysis : in SQL ---

Here are a few notes I took while looking into the structure and content of the tables. 
They contain remarks as to what to look further into, questions that arose, and a few basic suggestions as to improvements.
NB : some remarks are quite obvious, maybe even unnecessary. I just wanted to lay out my whole process.

### Ads : 
- 40 000 rows
- id : 40 000 distinct
 	!! No errors here
- owner_id : 2 '0', 14 313 distinct
	!!  '0' is normal.
- title : 0 NULL, 39 distinct
	!! 39 for 40 000 rows : So Selection from a list. 
 	!! Check if fields clean. 
 	!! Check in list clean. 
- category : 0 NULL, 4 distinct
	!! 3 real estate poorly orthographied. How did that happen? (Historical? other?)
 	!! Why only 2 categories (once cleaned) ? 2 existing? Or others poorly advertised or accessible?
- price : 0 NULL, 21 135 distinct
- city : 0 NULL, 6 561 distinct
 	!! Check if clean
- created_at : 0 NULL, 972 distinct
	!! If it was the full DB, it would be a 3-4 years old product. Checks out with the dates (2016-2020)
- deleted_at : ALL NULL
 	!! Why none deleted?

### ads_transaction
- 6 645 rows
- id : 6 645 distinct
  !! No errors here
- ad_owner_id : 0 null, 5 361 distinct
- buyer_user_id : 0 null, 6 645 distinct
- ad_id : 0 null, 6 645 distinct
- sold_price : 0 null, 6 645 distinct
- created_at : 0 null, 999 distinct
  !! Hehe :D 

- ad_owner_id = buyer_user_id : 0


### Users
- 30 000 rows
- age : 0 null
- bday : 0 null
- city : 0 null
- created_at : 0 null
- sex: 3052 null ; F,M, M., Mr
	!! do we want it compulsory? why do we even want it at all?
	!! Block keybord entries : list (if we keep)
- lat : 6000 null
- lon : 6000 null
- password : 12 nulls 2017-2020
	!! Find where it comes from. Contact them or delete them.
  !! Block creation of account without passport
- utm_source : 5985 null ; fb, mailing
- utm_medium : 6966 null ; ad, email, social
- utm_campaign : 5942 null
- firstname : 0 null
- lastname : 0 null
- user_agent : 0 null
- misc : 0 null
  
- !! deleted? add field deleted_at DATE (for tracking interest and usage)


### Referals
- 303 958 rows
- id : 303 958 distinct
- referrer_user_id : 0 null, 23 543 distinct
- referree_user_id : 0 null, 23 523 distinct
- created_at : 0 null
- deleted_at : All null
  !! Odd

- referrer_user_id=referree_user_id = 40
	!! Look into how it is possible. Make sure it gets blocked.



## --- VI. Data Analysis 
For this part of the case, I chose to use a tool I am really comfortable with : MyReport (Data and Page).
In this way you will be able to see what I can achieve with a software I have been using for a while, which I will be able to do with whatever tools you are using, given a little time to get into it.

You can see the visual analysis here :
https://github.com/JayemG/GDC_TestCase/blob/main/Data%20Analysis.pdf


Here are summed up my observations, and **what I would have like to look into, given more time.**

### Overview :
- A third of the users are not active. We can see that you can vouch for people without being active.
- A good proportion of users are referrers. That seems to work well.
- When you take the inactive users out of the count, users without ad go from 52% to 43%.  Still a lot and worth looking into.
- When you take the inactive users out of the count, users who never did a purchase go from 80% to 74%.  
- The distribution by age does not change sensibly.
- In 4 years, only 2.8 ads per user who actually posted an ad is quite low. 
- The conversion rates of the ads is extremely low : less than 1 on 5. 
- I looked into the ages relative to type of activity, and we can see that : 
		1/ for selling it's more the "over 30 yo"
		2/ for buying it's more evenly distributed, with a peak over 50 yo.
		3/ and those most active in both at once are between 40 and 50 yo.

- **Why is the conversion rate of the ads so low? Can it be boosted?**
- **Look into how to appeal to the half of the users with no ads whatsoever?**
- **How to appeal to the younger generations?**
- **Is referral an issue?**

### Users :
- No marked evolution as to ages at time of subscription or ages of users over time, nor of the proportion of women relative to men.
- Twice as many women as men. 
- Most women are over 50 yo. Type of items : half and half (unrealistic : made-up data). Evolution of subscription and their origins? Too clean.

- **I would like to look into what appeals to younger generations, as they are fewer in numbers.**
- **Proportion of women : Reasons? Maybe what attracts them could be used in ad campaign to attract a wider public?**
- **Women over 50 : Cities or countryside? Why no more ads after 2019?**
- **Add a map of the activity, overall and by category, and by year, and maybe by year/category if it feels relevant after a first enquiry.**

### Referrals :
- The seniority shows that all referrals have been made during the same year and month.  
- The “Status” graph shines a light on how difficult it is to climb that ladder.

- **Is the yearmonth of the referrals date a bug?**
- **Look into the average seniority of each status. And for each sub-category (see here ) too.**
- **Has it been set randomly? If not, how?**

### Other :
- **Count number of connexions and analyse patterns of behaviour based on them (number of ads, purchases, referrals around those dates)**


## --- VII. Business and product conclusions ---

The product exists since 2016, but it really took off in 2017 and interest has been stable until 2020.
All subscriptions, ads etc stopped around april 2020. Either it was closed, or the Covid restrictions had a big impact on it.
The referral system works well. The status system is too hard to be truly enticing.
It attracts a lot of middle-aged women. And overall twice as many women as men. Probably the sense of safety that comes with the vouching system, but maybe something else too : women are notoriously good at networking in a way that is relevant to this product.
There is about twice as many users over 50 than under 50. If we want it to have a long life, we need to work on making it more attractive to the younger generations.


## --- VIII. Conclusion : Suggestions for improvements ---

**Technicalities :**
- Make password compulsory.
- Make gender compulsory, from list. Add a "Non binary".
- Delete self-referees. Make it impossible.
- Make city names as list. Add Lat and Lon automatically.
- Add an automatic deletion of ads.
- Block accents from first and last names.


**Product quality :**
- Add more categories.
- Use last suggestion (pattern of behaviour based on dates and numbers of connexion) to make the product more user-friendly. (Shortcuts to most used features, what feature is accessible from which page...)
- Assess what makes it enticing to people under 50 yo, and act on it. Can be crossed with the previous point. Cand be built on from Women over 50 analysis.







