# MySql_layoff-_data-cleaning
the dataset name is layoff and in this repository I have used mysql to clean the data in it ... I have another repositroy where I have used the same dataset - 'layoffs' for data - analysis using MySql
Below is the Mysql Code .. iave also uploaded the MySql File 
------------------------------------------------------------------------------------------------------------------------------------------------------
******************************************************************************************************************************************************
use data_cleaning;
select * from layoffs;
-- Counting the no. of rows
select count(*) from layoffs;
------------------------------------------------------------------------------------
-- make a copy of the original table 
create table layoff_stage
Like layoffs;
-----------------------------------------------------------------------------------
-- Now look at the structure of the table created above 
select * from layoff_stage;
-- Insert the data into the newly created table 
insert layoff_stage
select * from layoffs;
-- Check at the data in the new table 
select * from layoff_stage;
-- counting the no. of enrty in the new table 
select count(*) from layoff_stage;

----------------------------------------------------------------------------------
-- HANDLING DUPLICATE VALUES
----------------------------------------------------------------------------------
select *, 
ROW_NUMBER() OVER(
				  Partition By company, location, industry, total_laid_off, percentage_laid_off, `date`,
							   stage,country, funds_raised_millions
				 ) AS row_num
from layoff_stage;

-- Create a CTE and look for rows where the row_num is greater than 2
with duplicate_cte AS
(
	select *,
    ROW_NUMBER() OVER(
					  Partition By company, location, industry, total_laid_off, percentage_laid_off, `date`,
								    stage, country, funds_raised_millions) AS row_num
	from layoff_stage
)
select * from duplicate_cte where row_num>1;
-- Check one of the entry shown as a result of the above querry 
select *, 
ROW_NUMBER() OVER(
				  Partition By company, location, industry, total_laid_off, percentage_laid_off, `date`,
							   stage,country, funds_raised_millions
				 ) AS row_num
from layoff_stage where company = 'Casper';
------------------------------------------------------------------------------------
-- Now to drop these duplicate rows - for this  we create a new table 
CREATE TABLE `layoff_stage2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
select * from layoff_stage2;

-- Inserting the data into the new table from where we delete all the duplicate rows

insert into layoff_stage2
select *, 
ROW_NUMBER() OVER(
				  Partition By company, location, industry, total_laid_off, percentage_laid_off, `date`,
							   stage,country, funds_raised_millions
				 ) AS row_num
from layoff_stage;
select * from layoff_stage2 where row_num >1 ;

-- Delete all the duplicate rows

delete from layoff_stage2 where row_num>1;
select count(*) from layoff_stage2;

------------------------------------------------------------------------------------------------------
-- Now that we have deleted all the duplicate rows we turncate the table layoff_stage and 

-- insert the data from layoff_stage2 to the mentioned table 
truncate table layoff_stage;

select * from layoff_stage;

-- Now insert the data from layoff_stage2 to layoff_stage

insert layoff_stage
select company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
from layoff_stage2;

select count(*) from layoff_stage;
-----------------------------------------------------------------------------------------------------------------------
-- Standerdize the data 

-- STEP 1 - First we look at the column company

select distinct (company) from layoff_stage order by 1;

-- We can see that there are some extra spaces  in the company name 

select distinct(company), trim(company) from layoff_stage;
update layoff_stage
set company = trim(company) ;

-- Now we look at the Industry column in the table layoff_stage

select distinct(industry) from layoff_stage order by 1;

select * from layoff_stage where industry like 'Fin%' order by 1;
update layoff_stage
set industry = 'Finance' 
where industry like 'Fin%';
select distinct (industry) from layoff_stage order by 1;

select * from layoff_stage where industry like 'Crypto%';
update layoff_stage
set industry = 'Crypto'
where industry like 'Crypto%';
select distinct(industry) from layoff_stage order by 1;

select distinct (country) from layoff_stage order by 1;
-- The united states have a 2 entries so we need to correct it 
update layoff_stage
set country = TRIM(trailing '.' from country)
WHERE country like 'United States%';
-- Relook at the country column again and see if the correction has been made 
select distinct(country) from layoff_stage order by 1; -- Corrections have been made
 
 
 -- Handling time data for time series analysis
 describe layoff_stage;
 -- As we can see that the date column is in the text format
 -- For doing this we use str_to_date() function
 select `date`,
 str_to_date(`date`,'%m/%d/%Y') from layoff_stage;
 -- Next we update the date column to date format 
 update layoff_stage
 set `date`= str_to_date(`date`,'%m/%d/%Y');
 -- Now that we have upadted the date column to date format .. however the column is stil showing a text d-type 
 -- So to update it the date column to date d-type we have to modify the column 
 alter table layoff_stage
 modify column `date` date;
 describe layoff_stage;
select `date` from layoff_stage order by 1; -- Even though we still have null values in the data set we will do them seperatly 

-- STEP 3 - Handling null and blank values

-- While making changes and modifing the data-set we see that there are null values in various columns
-- INDUSTRY column 
select * from layoff_stage
where industry is NULL 
OR
industry = '';
-- the entries we get from running the above querry we can see some entries in the indistry column is nul and some are missing
-- lets check each entry one by one 
-- company - Airbnb
select * from layoff_stage 
where company = 'Airbnb';
-- As we can see some entries in the data set have correct entries in the industry column while some are missing 
-- lets update the rows with ccorrect entries
select t1.company,t1.industry,t2.industry
from layoff_stage t1
join layoff_stage t2
on
t1.company = t2.company
where (t1.industry is NULL  or t1.industry ='')
and t2.industry is Not NULL;
-- With the above querry we can see that some entries in the data set with the same countries are missing so we subsititute them 
-- based in the results given by this querry
update layoff_stage t1
join  layoff_stage t2 
on t1.company = t2.company
set t1.industry = t2.industry
where t1.industry is null
and t2.industry is not NULL;
-- the above querry is working fine but now rows has been affected. 
--  So for changing the entries we change the empty quewes with null values
update layoff_stage
set industry = null
where industry ='';
-- with this querry we get null values in empty spaces and now we can run the previous querries for the desired results 

-- Let's check the if there are any more  empty or null entries in the industry column 
select * from layoff_stage
where industry is NULL 
OR
industry = '';
-- Here we can see that there is a row with null value in industry column. ..now lets check how many rows are there the under said co.
select * from layoff_stage
where company like 'bally%';
-- since we do not have any refrence values for filling the industry column above entry we drop the rows
delete from layoff_stage
where company like 'bally%';



