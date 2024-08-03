# Project Title: Data Analytics Power BI Reporting Project

## Project Overview

A medium-sized international retailer must take their BI practices to the next level.  They have a large amount of sales data from multiple sources over several years. 

The retailer must transform the data into actionable insights to support improved decision-making.  The objective is to use Power BI to design and develop a comprehensive quarterly report.  This will involve extracting and transforming data from various origins, designing a robust data model rooted in a star-based schema, and then constructing a multi-page report.

The report will present a high-level business summary tailored for C-suite executives, and also give insights into their highest value customers segmented by sales region, provide a detailed analysis of top-performing products categorised by type against their sales targets, and a visually appealing map visual that spotlights the performance metrics of their retail outlets across different territories.

## Milestone 1:

Configuration of the GitHub environment for this project

## Milestone 2: Importing Data into PowerBI

This stage focused on data loading and preparation.  As mentioned, data is being sourced from multiple locations.  This required data to be loaded from:
- Azure SQL database (Orders table)
- Azure Blob storage account (Stores table)
- Web-hosted CSV files (Products table)
- ZIP file containing multiple CSV files (Customers table, created from a combination of three CSV files)

In addition to importing the data a number of transformations were required:

### Orders
- The [Card Number] column was deleted to ensure data privacy compliance.
- [Order Date] and [Shipping Date] columns were split to create separate Date and Time columns.
- Rows with missing and null values were removed from the [Order Date] column
- Columns were renamed (where appropriate) based on standard naming conventions

### Products
- Duplicates were removed from the [product_code] column
- Columns were renamed (where appropriate) based on standard naming conventions

### Stores
- Columns were renamed (where appropriate) based on standard naming conventions

### Customers
- [Full Name] column was created based on combination of [First Name] and [Last Name] columns
- Column referencing the original source file was deleted
- Columns were renamed (where appropriate) based on standard naming conventions

## Milestone 3: Creating the Data Model

### Task 1: Created a date table covering the entire period of time in the dataset

DAX formulae were then used to add the following columns: 
- Day of Week
- Month Number (i.e. Jan = 1, Dec = 12 etc.)
- Month Name
- Quarter
- Year
- Start of Year
- Start of Quarter
- Start of Month
- Start of Week

### Task 2: Built a star schema data model

Added relationships:
- Products[product_code] to Orders[product_code]
- Stores[store code] to Orders[Store Code]
- Customers[User UUID] to Orders[User ID]
- Date[date] to Orders[Order Date]
- Date[date] to Orders[Shipping Date]

The relationship between Orders[Order Date] and Date[date] is the active relationship. All  relationships are one-to-many, with a single filter direction flowing from the dimension table side to the fact table side.

![image](https://github.com/user-attachments/assets/b9537672-e5f7-4656-b52a-30693cb2b5cd)

### Tasks 3 & 4: Created a Measures Table

This includes the following measures and DAX formulae:

- Total Orders        Total Orders = COUNT(Orders[User ID])
- Total Revenue       Total Revenue = SUM(Orders[Total Revenue])    combined with calculated column => Total Revenue = Orders[Product Quantity] * RELATED(Products[Sale Price])
- Total Profit        Total Profit = SUMX(Orders, Orders[Product Quantity] * (RELATED(Products[Sale Price]) - RELATED(Products[Cost Price])))
- Total Customers     Total Customers = DISTINCTCOUNT(Orders[User ID])
- Total Quantity      Total Quantity = COUNT(Orders[Product Quantity])
- Profit YTD          Profit YTD = TOTALYTD('Measures Table'[Total Profit], Orders[Order Date])
- Revenue YTD         Revenue YTD = TOTALYTD(SUM(Orders[Total Revenue]), Orders[Order Date])

### Task 5: Created Date and Geography Hierarchies

- Created a Date Hierarchy with: Start of Year, Start of Quarter, Start of Month, Start of Week, Date
- Created a calculated column in Stores table for Country  
  Country = IF([Country Code]="GB","United Kingdom",IF([Country Code]="US","United States",IF([Country Code]="DE","Germany","N/A")))
- Added a full Geography column combining Region and Country:
  Geography = Stores[Country Region] & ", " & Stores[Country]
- Ensured the correct data category was assigned for Region (Continent), Country (Country), and Country Region (State or Province)
- Added a Geography Hierarchy with: Region, Country and Country Region

