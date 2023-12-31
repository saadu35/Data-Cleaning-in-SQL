# Data-Cleaning-in-SQL
In this project we clean Nashville Housing data in SQL server

## Table of Contents
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [Data Cleaning Tasks](#data-cleaning-tasks)
- [Data Analysis ](#data-analysis)
- [Results](#results)
- [Recommendation ](#recommendation)
- [Limitations](#limitations)
- [References](#references)


## Project Overview
The Nashville Housing Data Cleaning project aims to enhance the quality and consistency of a dataset containing information about property transactions in Nashville. The dataset, stored in the NashvilleHousing table, exhibits various data quality issues, including inconsistent date formats, missing property addresses, and binary indicators represented as 'Y' and 'N'. This project employs SQL queries to address these issues and prepare the data for further analysis.

### Data Source 
The primary dataset used for this analysis is the " Nashville Housing Data for Data Cleaning.xlsx" file, which contains detailed information about UniqueID, ParcelID, LandUse, PropertyAddress, SaleDate, SalePrice, LegalReference, SoldAsVacant, OwnerName, OwnerAddress, Acreage, TaxDistrict, LandValue, BuildingValue, TotalValue, YearBuilt, Bedrooms and Bathrooms.

### Tools 
- SQL Server [Download here](https://microsoft.com)


### Data Cleaning Tasks

In the data preparation, we performed the following tasks: 
1. Standardize Date Format: The initial task involves standardizing the date format in the SaleDate column to ensure consistency across the dataset. The CONVERT function is used to achieve this, followed by altering the column type to DATE.
   
2. Populate Property Address Data: Incomplete property addresses are populated by leveraging information from records with the same ParcelID. This process enhances the dataset by filling in missing property address details.

3. Breaking out Address into Individual Columns: To facilitate more granular analysis, the PropertyAddress column is split into individual columns, namely PropertySplitAddress and PropertySplitCity.

4. Change Y and N to Yes and No in "Sold as Vacant" Field: The binary indicator in the SoldAsVacant field is modified to 'YES' for 'Y' and 'NO' for 'N' to improve readability and consistency.

5. Remove Duplicates: Duplicate records are identified based on specific columns, and only unique entries are retained, eliminating redundancy in the dataset.

6. Delete Unused Columns: Columns deemed unnecessary for further analysis, such as OwnerAddress, TaxDistrict, and SaleDate, are removed to streamline the dataset.

### Data Analysis 

``` sql

--Standardize Date Format

SELECT SaleDateconverted, CONVERT(Date,SaleDate)
FROM NashvilleHousing

ALTER TABLE NashvilleHousing 
ALTER COLUMN SaleDate DATE

UPDATE NashvilleHousing
SET SaleDateconverted = CONVERT(Date,SaleDate)

.............................................
--Populate Property Adress Data

SELECT *
FROM NashvilleHousing
order by ParcelID

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
on a.ParcelID = b.ParcelID
AND a.[UniqueID ] < > b.[UniqueID ]
WHERE a.PropertyAddress is null

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
on a.ParcelID = b.ParcelID
AND a.[UniqueID ] < > b.[UniqueID ]
WHERE a.PropertyAddress is null

.............................................
Breaking out Address into Individual Columns(Address, City, Sate)

SELECT 
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1) as Address
, SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress)) as Address
FROM NashvilleHousing


ALTER TABLE NashvilleHousing 
Add PropertySplitAddress Nvarchar(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1);

ALTER TABLE NashvilleHousing 
Add PropertySplitcity Nvarchar(255)

UPDATE NashvilleHousing
SET PropertySplitcity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) +1, LEN(PropertyAddress));




SELECT
    SUBSTRING(OwnerAddress, 1, CHARINDEX(',', OwnerAddress + ',') - 1) AS FirstPart,
    SUBSTRING(OwnerAddress, CHARINDEX(',', OwnerAddress + ',') + 1, CHARINDEX(',', OwnerAddress + ',', CHARINDEX(',', OwnerAddress + ',') + 1) - CHARINDEX(',', OwnerAddress + ',') - 1), 
    SUBSTRING(OwnerAddress, CHARINDEX(',', OwnerAddress + ',', CHARINDEX(',', OwnerAddress + ',') + 1) + 1, LEN(OwnerAddress)) 
FROM
    NashvilleHousing;



ALTER TABLE NashvilleHousing 
Add OwnerSplitAddress Nvarchar(255);

UPDATE NashvilleHousing
SET OwnerSplitAddress = SUBSTRING(OwnerAddress, 1, CHARINDEX(',', OwnerAddress + ',') - 1);

ALTER TABLE NashvilleHousing 
Add OwnerSplitcity Nvarchar(255)

UPDATE NashvilleHousing
SET OwnerSplitcity = SUBSTRING(OwnerAddress, CHARINDEX(',', OwnerAddress + ',') + 1, CHARINDEX(',', OwnerAddress + ',', CHARINDEX(',', OwnerAddress + ',') + 1) - CHARINDEX(',', OwnerAddress + ',') - 1);

ALTER TABLE NashvilleHousing 
Add OwnerSplitState Nvarchar(255)

UPDATE NashvilleHousing
SET OwnerSplitState = SUBSTRING(OwnerAddress, CHARINDEX(',', OwnerAddress + ',', CHARINDEX(',', OwnerAddress + ',') + 1) + 1, LEN(OwnerAddress));

.............................................
Change Y and N to Yes and No in “Sold as Vacant” Field

SELECT DISTINCT (SoldAsVacant), COUNT(SoldAsVacant)
FROM NashvilleHousing
GROUP BY SoldAsVacant
ORDER BY 2

SELECT 
    SoldAsVacant,
    CASE 
        WHEN SoldAsVacant = 'Y' THEN 'YES'
        WHEN SoldAsVacant = 'N' THEN 'NO'
        ELSE SoldAsVacant
    END
FROM 
    NashvilleHousing;


UPDATE NashvilleHousing
SET SoldAsVacant = CASE 
        WHEN SoldAsVacant = 'Y' THEN 'YES'
        WHEN SoldAsVacant = 'N' THEN 'NO'
        ELSE SoldAsVacant
    END
.............................................
Remove Duplicates 

WITH RownumCTE AS(
Select *,
        ROW_NUMBER() OVER (
		PARTITION BY ParcelID,
		             PropertyAddress,
					 SalePrice,
					 SaleDate,
					 LegalReference
					 ORDER BY 
					    UniqueID
						) row_num

FROM NashvilleHousing
)
Select *
FROM RownumCTE
Where row_num > 1
Order by PropertyAddress




WITH RownumCTE AS(
Select *,
        ROW_NUMBER() OVER (
		PARTITION BY ParcelID,
		             PropertyAddress,
					 SalePrice,
					 SaleDate,
					 LegalReference
					 ORDER BY 
					    UniqueID
						) row_num

FROM NashvilleHousing
)
DELETE 
FROM RownumCTE
Where row_num > 1

.............................................
Delete Unused Columns 

ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress


ALTER TABLE NashvilleHousing
DROP COLUMN SaleDate  

.............................................

```

### Results

The Nashville Housing dataset underwent comprehensive cleaning to address various data quality issues.
- Date formats in the SaleDate column were standardized for consistency using the CONVERT function, and the column type was altered to DATE.
  
- Missing property addresses were populated by merging data from records with the same ParcelID, enhancing the completeness of the dataset.
  
- The PropertyAddress column was split into individual columns (PropertySplitAddress and PropertySplitCity) for more granular analysis.
  
- Binary indicators in the SoldAsVacant field were modified to 'YES' for 'Y' and 'NO' for 'N' to improve readability and ensure consistency.
  
- Duplicate records were identified based on specific columns, and only unique entries were retained, eliminating redundancy in the dataset.
  
- Unnecessary columns, including OwnerAddress, TaxDistrict, and SaleDate, were removed to streamline the dataset.
  
- The cleaned dataset, stored in the NashvilleHousing table, is now standardized, complete, and ready for further exploratory data analysis.


### Recommendation 
Based on the data cleaning, we recommend the following:

1. Automate Data Cleaning Processes: Implement automation tools or scripts to streamline repetitive data cleaning tasks. Automation reduces manual errors, ensures consistency, and allows for efficient processing of large datasets. Tools like Python with pandas or Apache Spark can be beneficial for automating data cleaning procedures.

2. Implement Robust Data Validation: Introduce thorough data validation checks to verify the accuracy of the cleaning processes. Validate the cleaned dataset against external sources or known benchmarks to ensure that data quality is maintained throughout the cleaning pipeline.

3. Explore Advanced Analytics and Machine Learning: Leverage the cleaned dataset for more advanced analytics, such as predictive modeling or machine learning. Explore correlations, patterns, and trends that can provide deeper insights into the factors influencing property transactions and prices in Nashville.

### Limitations

I undertook a comprehensive cleaning of the Nashville Housing dataset, addressing diverse data quality issues. This involved standardizing date formats by converting the SaleDate column to DATE for consistency. I filled missing property addresses by merging data based on ParcelID, significantly enhancing dataset completeness. To facilitate more detailed analysis, I split the PropertyAddress column into individual columns—PropertySplitAddress and PropertySplitCity. I transformed binary indicators in the SoldAsVacant field to 'YES' for 'Y' and 'NO' for 'N,' aiming to improve readability and consistency. I meticulously identified duplicate records based on specific columns, ensuring that only unique entries remained in the dataset. I systematically removed unnecessary columns such as OwnerAddress, TaxDistrict, and SaleDate, streamlining the dataset for clarity. The resulting cleaned dataset, now stored in the NashvilleHousing table, is standardized and complete, poised for my further exploratory data analysis and insights into the Nashville housing market.

### References 
- SQL Server Documentation [View here](https://docs.microsoft.com/en-us/sql/)
- [Youtube](https://www.youtube.com/)
- [stackoverflow](https://stackoverflow.com/)

