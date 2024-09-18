# Retail Data Warehouse Project

This project demonstrates the creation of a simple data warehouse for retail data using SQL. It includes staging, dimension, and fact tables, as well as scripts for loading data from a Kaggle dataset.

## Data Source

The data for this project is sourced from a Kaggle retail dataset. You can download the dataset [here]( https://www.kaggle.com/datasets/aditisaxena20/superstore-sales-dataset).

## Project Setup

1. **Database Setup:**
   - Create a database in your SQL environment (e.g., SQL Server).
   
2. **Run the SQL Scripts:**

### **Creating Staging Table**

```sql
CREATE TABLE dbo.StagingRetail (
    Row_ID int,
    Order_ID nvarchar(50) NULL,
    Order_Date date NULL,
    Ship_Date date NULL,
    Ship_Mode nvarchar(50) NULL,
    Customer_ID nvarchar(50) NULL,
    Customer_Name nvarchar(50) NULL,
    Segment nvarchar(50) NULL,
    Country nvarchar(50) NULL,
    City nvarchar(50) NULL,
    State nvarchar(50) NULL,
    Postal_Code int NULL,
    Region nvarchar(50) NULL,
    Product_ID nvarchar(50) NULL,
    Category nvarchar(50) NULL,
    Sub_Category nvarchar(50) NULL,
    Product_Name nvarchar(150) NULL,
    Sales float NULL
);
```
### **Creating Dimension and Fact Tables**

```sql
-- Dimension Tables
CREATE TABLE dbo.DimCustomer (
    Customer_ID NVARCHAR(50) PRIMARY KEY,
    Customer_Name NVARCHAR(50),
    Segment NVARCHAR(50)
);

CREATE TABLE dbo.DimProduct (
    Product_ID NVARCHAR(50) PRIMARY KEY,
    Product_Name NVARCHAR(150),
    Category NVARCHAR(50),
    Sub_Category NVARCHAR(50)
);

CREATE TABLE dbo.DimRegion (
    Region_ID INT IDENTITY(1,1) PRIMARY KEY,
    Country NVARCHAR(50),
    City NVARCHAR(50),
    State NVARCHAR(50),
    Postal_Code INT,
    Region NVARCHAR(50)
);

-- Fact Table
CREATE TABLE dbo.FactSales (
    FactSales_ID INT IDENTITY(1,1) PRIMARY KEY,
    Order_ID NVARCHAR(50),
    Order_Date DATE,
    Ship_Date DATE,
    Customer_ID NVARCHAR(50),
    Product_ID NVARCHAR(50),
    Ship_Mode NVARCHAR(50),
    Sales FLOAT,
    Region_ID INT,
    CONSTRAINT FK_FactSales_CustomerID FOREIGN KEY (Customer_ID) REFERENCES dbo.DimCustomer(Customer_ID),
    CONSTRAINT FK_FactSales_ProductID FOREIGN KEY (Product_ID) REFERENCES dbo.DimProduct(Product_ID),
    CONSTRAINT FK_FactSales_RegionID FOREIGN KEY (Region_ID) REFERENCES dbo.DimRegion(Region_ID)
);

```
### **Inserting Data into Staging Table**
```sql
INSERT INTO dbo.StagingRetail
SELECT * FROM RetailData;
```
### **Loading Data into Dimension Tables**
```sql
--Load Data into DimCustomer
INSERT INTO dbo.DimCustomer (Customer_ID, Customer_Name, Segment)
SELECT DISTINCT Customer_ID, Customer_Name, Segment
FROM dbo.StagingRetail;

--Load Data into DimProduct
INSERT INTO dbo.DimProduct (Product_ID, Product_Name, Category, Sub_Category)
SELECT Product_ID, 
       MIN(Product_Name) AS Product_Name, 
       MIN(Category) AS Category, 
       MIN(Sub_Category) AS Sub_Category
FROM dbo.StagingRetail
WHERE Product_ID IS NOT NULL
GROUP BY Product_ID;

--Load Data into DimRegion
INSERT INTO dbo.DimRegion (Country, City, State, Postal_Code, Region)
SELECT DISTINCT Country, City, State, Postal_Code, Region
FROM dbo.StagingRetail;
```
### **Loading Data into Fact Table**
```sql
--Load Data into FactSales
INSERT INTO dbo.FactSales (Order_ID, Order_Date, Ship_Date, Customer_ID, Product_ID, Ship_Mode, Sales, Region_ID)
SELECT 
    sr.Order_ID,
    sr.Order_Date,
    sr.Ship_Date,
    sr.Customer_ID,
    sr.Product_ID,
    sr.Ship_Mode,
    sr.Sales,
    dr.Region_ID
FROM dbo.StagingRetail sr
JOIN dbo.DimRegion dr
ON sr.Country = dr.Country 
AND sr.City = dr.City 
AND sr.State = dr.State 
AND sr.Postal_Code = dr.Postal_Code
AND sr.Region = dr.Region;

```

