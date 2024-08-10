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

### Task 5: Created Date and Geography Hierarchies:

- Created a Date Hierarchy with: Start of Year, Start of Quarter, Start of Month, Start of Week, Date
- Created a calculated column in Stores table for Country  
  Country = IF([Country Code]="GB","United Kingdom",IF([Country Code]="US","United States",IF([Country Code]="DE","Germany","N/A")))
- Added a full Geography column combining Region and Country:
  Geography = Stores[Country Region] & ", " & Stores[Country]
- Ensured the correct data category was assigned for Region (Continent), Country (Country), and Country Region (State or Province)
- Added a Geography Hierarchy with: Region, Country and Country Region

## Milestone 4: Setting up the Report

Core report structure created, with the following pages:
- Executive Summary
- Customer Detail
- Product Detail
- Stores Map

## Milestone 5: Building Customer Detail Page

- Added Card Visuals to show the total number of unique customers and the average revenue per customer
- Added a donut chart showing the split of customers per country
- Added a bar chart showing the customer split by product category
- Added a line chart showing total customers by date, with the user option to drill down to month level. Incorporated a trend line and forecast for the next 10 periods, using a 95% confidence interval
- Added a table to show the top 20 customers (name, revenue and total number of orders), incorporating a bar chart for total revenue
- Added top customer cards, showing the name of the top customer, their total number of orders and total revenue
- Added a between slicer allowing user to select which years to filter the page

### Finished Customer Detail Page:
<img width="871" alt="image" src="https://github.com/user-attachments/assets/0e659168-e18d-4cb5-a7e2-e8e95af57241">

## Milestone 6: Building Executive Summary Page

- Added card visuals showing Total Revenue, Total Orders and Total Profit (formatted to two decimal places), using the related Measures
- Added a Revenue Trend Line chart, with drill down capability from Year to Quarter to Month
- Added two donut charts showing Total Revenue, broken down by Store[Country] and Store[Store Type]
- Added a clustered bar chart showing Total Orders filtered by Products[Category]
- Added KPI visuals.  This required the creation of new measures:
- - Previous Quarter Profit    Previous Quarter Profit = CALCULATE('Measures Table'[Total Profit], DATEADD('Date'[Date], -1, QUARTER))
  - Previous Quarter Revenue   Previous Quarter Revenue = CALCULATE('Measures Table'[Total Revenue], DATEADD('Date'[Date], -1, QUARTER))
  - Previous Quarter Orders    Previous Quarter Orders = CALCULATE('Measures Table'[Total Orders], DATEADD('Date'[Date], -1, QUARTER))
  Target measures are based on a 5% uplift on the previous quarter:
  - Target Qtly Profit         Target Qtly Profit = 'Measures Table'[Previous Quarter Profit] * 1.05 
  - Target Qtly Revenue        Target Qtly Revenue = 'Measures Table'[Previous Quarter Revenue] * 1.05 
  - Target Qtly Orders         Target Qtly Orders = 'Measures Table'[Previous Quarter Orders] * 1.05

### Finished Executive Summary Page:
<img width="763" alt="image" src="https://github.com/user-attachments/assets/7bd0c586-2059-4c14-bbd8-7e6577856158">

## Milestone 7: Building Product Detail Page

- Added Gauge visuals for Orders, Revenue & Product
  - Defined new measures for the quarterly targets for each metric based on a CEO defined 10% target q-on-q growth
    - Target QTR Orders = 'Measures Table'[Previous Quarter Orders] * 1.1
    - Target QTR Revenue = 'Measures Table'[Previous Quarter Revenue] * 1.1
    - Target QTR Profit = 'Measures Table'[Previous Quarter Profit] * 1.1
  - Maximum value is set to these target measures
  - Value is set to use the previously defined QTD Orders / Revenue / Profit measures:
    - QTD Orders = TOTALQTD([Total Orders], 'Date'[Date])
    - QTD Revenue = TOTALQTD([Total Revenue], 'Date'[Date])
    - QTD Profit = TOTALQTD([Total Profit], 'Date'[Date])
  - Callout value is conditionally formatted to be red if the target is not yet met
- Added Filter State cards to show the filter state for Product Category and Country using the following measures
  - Category Selection = IF(ISFILTERED(Products[Category]), SELECTEDVALUE(Products[Category], "No Selection"), "No Selection")
  - Country Selection = IF(ISFILTERED(Stores[Country]), SELECTEDVALUE(Stores[Country],"No Selection")
- Added an area chart showing how the different product categories are performing in terms of revenue over time:
  - X axis = Dates[Start of Quarter]
  - Y axis = Total Revenue
  - Legend = Products[Category]
- Added a Top 10 products table showing
    - Product Description
    - Total Revenue
    - Total Customers
    - Total Orders
    - Profit Per Order
- Added a Scatter Graph showing Quantity Sold vs Profit per Item
    - Created a new calculated column in Products table called 'Profit per Item':
      Profit per Item = Products[Sale Price] - Products[Cost Price]
      Scatter chart is configured as:
      Values = Products[Description]
      X-Axis = Products[Profit per Item]
      Y-Axis = Orders[Total Quantity]
      Legend = Products[Category]
- Added a slicer toolbar. This required:
    - Filter icon added to top of navigation bar to open slicer toolbar
    - Addition of two slicers for Product Category and Stores Country
    - Addition of a back button on the slicer toolbar
    - Creation of bookmarks to show the slicer Open and slicer Closed states
    - Assignment of actions to the buttons using the bookmarks
  Slicer Toolbar:
  <img width="106" alt="image" src="https://github.com/user-attachments/assets/9fae8ec8-9d0e-411e-b1dc-495201794b16">

Final Product Detail Page:

<img width="452" alt="image" src="https://github.com/user-attachments/assets/b5c61794-e8c4-4b74-83da-f578f2438a05">

## Milestone 8: Building Stores Map Page

- Added a Stores Map pagem based on the defined Geography hierarchy and bubble size = ProfitYTD
- Added a Country Slicer above the map to allow country selection (or Select All)
- Added a Stores Drillthrough page, including:
  - Table showing top 5 products with Description, Profit YTD, Total Orders, Total Revenue
  - A column chart showing Total Orders by product category for the store
  - Gauges for Profit YTD and Revenue YTD, with a Profit/Revenue Goal based on a 20% increase on the previous year's y-t-d profit/revenue, these were based on the following Measures:
    - Revenue Goal = CALCULATE('Measures Table'[Revenue YTD], SAMEPERIODLASTYEAR('Date'[Date])) * 1.2
    - Profit Goal = CALCULATE('Measures Table'[Profit YTD], SAMEPERIODLASTYEAR('Date'[Date])) * 1.2
- Added a Stores Tooltip page

Stores Map page:

<img width="476" alt="image" src="https://github.com/user-attachments/assets/6cf1d8ce-8657-4403-b0a7-acd3be61646a">

Stores Drillthrough page:

<img width="480" alt="image" src="https://github.com/user-attachments/assets/4d62d7b4-8237-46ff-8cde-d1b7737f6450">


## Milestone 9: Cross Filtering & Navigation

- Cross Filtering
  - Executive Summary:
    - Product Category bar chart and Top 10 Products table are set to not filter the card visuals or KPIs
  - Customer Detail:
    - Top 20 Customers table does not filter any other visuals
    - Total Customers by Product chart does not filter the Customer line chart
    - Total Customers by Country donut chart does filter the Product bar chart
  - Product Detail Page
    - Orders/Profitability Scatter Chart does not filter any other visuals
    - Top 10 Products table does not filter any other visuals
- Navigation Bar
  - Icons added to navigation bar to enable navigation between pages
  - Icons set to change colour on hover over
Final Navigation Bar:

<img width="34" alt="image" src="https://github.com/user-attachments/assets/a3ef088d-328b-452a-bfd7-f943c58bc5a3">


