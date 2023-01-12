# Sales-Report

## Data Gathering :  
Assemble a sales reports with different visuals to best show the Sales Insights in one page Dashboard. 

  1-Sales (folder by year)
  2-Categories (Excel)
  3-Geography (Excel)
  4-Product (CSV / Database)
  5-SalesRep (Excel)
  6-SubCategories (Excel)
We loaded all the files from the sales folder in our model and then we merged the queries to get a single fact table.

## Data Transformation :  

First we split the location column in the geography table and the sales table to country and city so we could change its data type to allow Geo Maps.
Then we created a unique geokey to create the relationship between the two tables.

We also created a date table with this DAX formula :
```  
Date = CALENDAR(FIRSTDATE(Sales[Date]),LASTDATE(Sales[Date]))
```  

And we made some basic transformations to the rest of our columns.

## Data modeling :  



### DimDate:      
```   
SELECT 
    DateKey,
    FullDateAlternateKey AS Full_Date,
    EnglishDayNameOfWeek AS DayName,
    EnglishMonthName AS MonthName,
    LEFT(EnglishMonthName,3) AS MonthShort,--Useful for the graphs 
    MonthNumberOfYear AS MonthNumber,
    CalendarQuarter AS Quarter,
    CalendarYear AS Year
FROM DimDate
WHERE CalendarYear IN (2013,2014) -- analyzing only the data from 2013 and 2014
```     
### DimCustomer:   
```  
SELECT  [CustomerKey]
      ,[dc].[GeographyKey]
      ,[CustomerAlternateKey]
      ,[FirstName]
      ,[MiddleName]
      ,[LastName]
      ,[FirstName]+ ' ' +[LastName] AS [FullNAME]
      ,[BirthDate]
      ,CASE upper([MaritalStatus])
        WHEN 'M' Then 'Married'
        WHEN 'S' Then 'Single'
        END AS [MaritalStatus]
      ,CASE upper([Gender])
        WHEN 'M' Then 'Male'
        WHEN 'F' then 'Female'
        END AS Gender
      ,[EmailAddress]
      ,[YearlyIncome]
      ,[TotalChildren]
      ,[NumberChildrenAtHome]
      ,[EnglishEducation] AS Education
      ,[EnglishOccupation] AS Occupation
      ,[HouseOwnerFlag]
      ,[NumberCarsOwned]
      ,[AddressLine1]
      ,[AddressLine2]
      ,[Phone]
      ,[DateFirstPurchase]
      ,[CommuteDistance]
      ,[dg].[City] AS City
  FROM [dbo].[DimCustomer] AS dc
  left join DimGeography AS dg 
  on dc.GeographyKey=dg.GeographyKey
  ORDER BY CustomerKey ASC
```     
### DimProduct:    
```     
SELECT [ProductKey]
      ,[ProductAlternateKey]
      ,[dps].[ProductSubcategoryKey] AS ProductSubCategoryKey
      ,[dps].[EnglishProductSubcategoryName] AS ProductSubCategory
      ,[dpc].ProductCategoryKey AS ProductCategoryKey
      ,[dpc].[EnglishProductCategoryName] AS ProductCategory
      ,[EnglishProductName] AS Product
      ,[SafetyStockLevel]
      ,[ReorderPoint]
      ,[DaysToManufacture]
      ,[EnglishDescription] AS [Description]
      ,[StartDate]
      ,[EndDate]
      ,ISNULL([Status],'Outdated') AS [Status]
  FROM [dbo].[DimProduct] AS dp 
  left join [dbo].[DimProductSubcategory] AS dps
  on [dp].[ProductSubcategoryKey]=[dps].[ProductCategoryKey]
  left join [dbo].[DimProductCategory] AS dpc 
  on [dps].[ProductCategoryKey]=[dpc].[ProductCategoryKey]
```        
### FactInternetSales:    
```  
SELECT [ProductKey]
      ,[OrderDateKey]
      ,[DueDateKey]
      ,[ShipDateKey]
      ,[CustomerKey]
      --,[CurrencyKey]
      ,[OrderQuantity]
      ,[UnitPrice]
      ,[ExtendedAmount]
      ,[UnitPriceDiscountPct]
      ,[DiscountAmount]
      ,[ProductStandardCost]
      ,[TotalProductCost]
      ,[SalesAmount]
      ,[TaxAmt]
      ,[OrderDate]
      ,[DueDate]
      ,[ShipDate]
  FROM [dbo].[FactInternetSales]
  WHERE LEFT (OrderDateKey, 4) IN (2013,2014) -- bringing only data of 2013 and 2014
```      
## Data model :   
After the cleaning process we exported the results into csv files and imported them to power BI to create the star schema data model below      
![screenshot](data_model.PNG)       

## Measures :   
For our dashboard we created a simple measure to calculate the  total sales amount over all rows in the context   
 and for the top 10 customer  and the top 10 products we used the same DAX formula below       
 ```   
 Top 10 Customers = 
    var customers=VALUES(DimCustomer[FullNAME])
return 
    CALCULATE(FactInternetSales[Sales],
        TOPN(10,ALL(DimCustomer[FullNAME]),FactInternetSales[Sales]),
        customers)
```   

## The final dahsboard :   
The final one page sales management dashboard with a tooltip page created for the products chart meets the acceptance criteria, as you can see it shows sales over time, per customer and per product   
![screenshot](final_dash.PNG)
    


