# Cleaning-Layoffs-Dataset 👨🏻‍💻⛔️
![layoff-staff](https://github.com/user-attachments/assets/883bf229-3530-4d04-844d-369e682ebce0)


# Project Overview

This project involves cleaning a dataset containing information about company layoffs. The data cleaning process includes removing duplicates, standardizing data, handling null or blank values, and removing unnecessary columns or rows to ensure the dataset is accurate and ready for analysis.


# Tools
MySQL


# Steps Involved in Data Cleaning
1. Load the Initial Data

SELECT * 
FROM layoffs;

![lay](https://github.com/user-attachments/assets/f725a9b4-f5fb-4789-ad2d-43b7c1de0301)


Retrieve the original data from the layoffs table to understand the structure and contents.


2. Remove Duplicates
Create a Staging Table

CREATE TABLE layoffs_staging 
LIKE layoffs;

Create a new table layoffs_staging with the same structure as layoffs.
Insert Data into the Staging Table

INSERT INTO layoffs_staging 
SELECT * 
FROM layoffs;

Populate the staging table with the data from the original layoffs table.
Identify Duplicates

WITH duplicate_cte AS (
    SELECT *,ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
    FROM layoffs_staging
)
SELECT * 
FROM duplicate_cte WHERE row_num > 1;

Use the ROW_NUMBER() window function to identify duplicate rows based on several columns.
Create a Second Staging Table

CREATE TABLE layoffs_staging2 (
  `company` TEXT,
  `location` TEXT,
  `industry` TEXT,
  `total_laid_off` INT DEFAULT NULL,
  `percentage_laid_off` TEXT,
  `date` TEXT,
  `stage` TEXT,
  `country` TEXT,
  `funds_raised_millions` INT DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

Create layoffs_staging2 to store the cleaned data with duplicates removed.
Populate Staging Table 2 and Remove Duplicates

INSERT INTO layoffs_staging2
SELECT *, ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

DELETE FROM layoffs_staging2 WHERE row_num > 1;

Insert data into layoffs_staging2, using ROW_NUMBER() to flag duplicates, and then delete these duplicates.

3. Standardize the Data

Remove Leading and Trailing Spaces

UPDATE layoffs_staging2 
SET company = TRIM(company);

Ensure consistency in the company column by removing any extra spaces.

Standardize Industry Names

UPDATE layoffs_staging2 
SET industry = 'Crypto' 
WHERE industry LIKE 'Crypto%';

Standardize the industry names, changing any variations of "Crypto" to a single consistent value.

Clean Up Country Names

UPDATE layoffs_staging2 
SET country = TRIM(TRAILING '.' FROM country) 
WHERE country LIKE 'United States%';

Standardize country names by trimming unwanted characters and ensuring uniformity.

Convert Date Format

UPDATE layoffs_staging2 
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2 
MODIFY COLUMN `date` DATE;

Convert the date column to a standard DATE format for consistency.

4. Handle Null or Blank Values

Update or Remove Null Values

UPDATE layoffs_staging2 
SET industry = NULL WHERE industry = '';

DELETE 
FROM layoffs_staging2 
WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

Replace blank industry values with NULL and remove rows where both total_laid_off and percentage_laid_off are null.

Fill Missing Industry Data

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2 ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL AND t2.industry IS NOT NULL;

Use available data from other rows to fill in missing industry values based on company names.

5. Remove Unnecessary Columns or Rows

Drop Unneeded Columns

ALTER TABLE layoffs_staging2 
DROP COLUMN row_num;

Remove the row_num column since it's no longer needed after deduplication.


# Conclusion
The dataset has been successfully cleaned through the removal of duplicates, standardization of data, handling of null values, and elimination of unnecessary columns. The layoffs_staging2 table now contains a clean and consistent dataset, ready for further analysis or reporting.

