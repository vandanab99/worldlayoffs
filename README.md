# worldlayoffs
This can help to understand the data cleaning in MYSQL

-- checking for the duplicates

use world_layoffs;

select * from layoffs;

-- copy only the structure
create table layoffs_stagging
like layoffs;

-- copies every data into the new stagging table
insert layoffs_stagging
select * from layoffs;

select * from layoffs_stagging;

with duplicate_finder as 
(
select *,
row_number() over(partition by company,location,industry,total_laid_off,percentage_laid_off,
date,stage,country,funds_raised_millions) as row_num
 from layoffs_stagging
)

select * from duplicate_finder 
where row_num>1;

select * from layoffs_stagging
where company in ('Casper','Cazoo','Hibob','Wildlife Studios','Yahoo')
order by company;

-- delete * from duplicate_finder where row_num > 1;
-- we have paste this data into the layoffs staging for that have to replace with new refined table

-- throwing error of not finding in the schema 
delete 
from duplicate_finder
where row_num > 1;


CREATE TABLE `world_layoffs`.`layoffs_staging2` (
`company` text,
`location`text,
`industry`text,
`total_laid_off` INT,
`percentage_laid_off` text,
`date` text,
`stage`text,
`country` text,
`funds_raised_millions` int,
row_num INT
);

insert layoffs_staging2
select *,
row_number() over(partition by company,location,industry,total_laid_off,percentage_laid_off,
date,stage,country,funds_raised_millions) as row_num
 from layoffs_stagging;
 
 select * from layoffs_staging2;
 --where row_num > 1;
 
 
 
 delete 
 from layoffs_staging2
 where row_num > 1;

alter table layoffs_staging2
drop column row_num ;

rollback;

-- standardizing data

select company, trim(company) from layoffs_staging2;

update layoffs_staging2
set company = trim(company) ;

select distinct industry from layoffs_staging2 order by 1;

update layoffs_staging2
set industry = 'Crypto'
where industry like 'Crypto%';

select distinct country
from layoffs_staging2 order by 1;

select * from layoffs_staging2 where country like 'United States.';

update layoffs_staging2
set country = 'United States'
where country = 'United States.';

select * from layoffs_staging2;

select distinct stage from layoffs_staging2;

select `date`, str_to_date(`date`,'%m/%d/%Y') from layoffs_staging2;

update layoffs_staging2
set `date` = str_to_date(`date`,'%m/%d/%Y');

alter table layoffs_staging2
modify column date date;

-- Null, blank values

select * from layoffs_staging2
where company in ('Airbnb','Carvana','Juluu');

select * from layoffs_staging2 t1
join layoffs_staging2 t2
on t1.company = t2.company
where (t1.industry is null or t1.industry = '')
and t2.industry is not null;

update layoffs_staging2 t1
join layoffs_staging2 t2
on t1.company = t2.company
set t1.industry = t2.industry
where (t1.industry is null or t1.industry = '')
and t2.industry is not null
and t1.date <> t2.date;

select * from layoffs_staging2;

select * from layoffs_staging2
where total_laid_off is null
 and percentage_laid_off is null;

delete
from layoffs_staging2
where total_laid_off is null
 and percentage_laid_off is null;
