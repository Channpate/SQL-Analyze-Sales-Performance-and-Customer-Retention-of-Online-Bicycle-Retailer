# [SQL] Analyze Sales Performance & Customer Retention - Online Bicycle Retailer

**Author:** Mai Ngoc Chau (Chann)

**Date:** 2025-03-12

**Tool used:** SQL (Google BigQuery)

## 📑 Table of Contents
- [📌 Background & Overview](#-background--overview)
- [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
- [⚒️ Process](#-process)
- [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)


## 📌 Background & Overview

### **Objectives:**
This project uses SQL to analyze the AdventureWorks2019 dataset to:
* Evaluate sales performance through metrics like item quantity, order volume, sales value, and YoY growth rate by subcategory and territory.
* Track customer behavior and retention using cohort analysis, shipment status, and order trends.
* Analyze inventory and discount efficiency by calculating stock/sales ratios, MoM stock changes, and seasonal discount costs by product.

### **Who is this project for:**
* Data analysts & business analysts
* Marketing/ Sales/ Purchasing managers,stakeholders

## 📂 Dataset Description & Data Structure

### Data source
* Source:
  * The dataset is taken from **AdventureWorks 2019 Sample Datasets**. The AdventureWorks database supports standard online transaction processing scenarios for a fictitious bicycle manufacturer (Adventure Works Cycles). Scenarios include Manufacturing, Sales, Purchasing, Product Management, Contact Management, and Human Resources.
  * This project will only query on specific tables of the database, as mentioned in the next part.
* Size: 60+ Tables
* Format: The data is in **SQL table** format on BigQuery.

### Data Structure & Relationships
**1. Tables Used:** 8 tables
|Schema | Table Name | Rows | Columns |  
|-------------|----------|-------------|-------------| 
| Sales       | Sales.SalesOrderHeader       |31465  | 26 |
| Sales       | Sales.SalesOrderDetail       | 121317 | 11 |
| Sales       | Sales.SpecialOffer           | 16 | 11 |
| Production  |Production.Product            | 504 | 25 |
| Production  |Production.ProductSubcategory | 37 | 5 |
| Production  |Production.WorkOrder          | 72591 | 10 |
| Purchasing  |Purchasing.PurchaseOrderHeader| 4012 | 13 |
| Purchasing  |Purchasing.PurchaseOrderDetail| 8845 | 11 |

**2. Table Schema & Data Snapshot:**

Detail Data dictionary at: https://drive.google.com/file/d/1bwwsS3cRJYOg1cvNppc1K_8dQLELN16T/view

<details>
<summary>Sales.SalesOrderHeader Schema</summary>

| Column Name              | Data Type        | Description                                                                                                           |
| ------------------------ | ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| `SalesOrderID`           | int              | Primary key. *(Identity / Auto increment)*                                                                            |
| `RevisionNumber`         | tinyint          | Tracks changes to the order. **Default: 0**                                                                           |
| `OrderDate`              | datetime         | Date order was created. **Default: getdate()**                                                                        |
| `DueDate`                | datetime         | Date the order is due to the customer.                                                                                |
| `ShipDate`               | datetime         | Date the order was shipped.                                                                                           |
| `Status`                 | tinyint          | Order status: 1 = In Process; 2 = Approved; 3 = Backordered; 4 = Rejected; 5 = Shipped; 6 = Cancelled. **Default: 1** |
| `OnlineOrderFlag`        | bit              | 0 = Salesperson order; 1 = Online order. **Default: 1**                                                               |
| `SalesOrderNumber`       | nvarchar(25)     | Unique sales order number. **Computed**: `ISNULL('SO' + CONVERT(nvarchar, SalesOrderID), '*** ERROR ***')`            |
| `PurchaseOrderNumber`    | nvarchar(25)     | Customer purchase order reference.                                                                                    |
| `AccountNumber`          | nvarchar(15)     | Financial account reference.                                                                                          |
| `CustomerID`             | int              | Foreign key to `Customer.BusinessEntityID`.                                                                           |
| `SalesPersonID`          | int              | Foreign key to `SalesPerson.BusinessEntityID`.                                                                        |
| `TerritoryID`            | int              | Foreign key to `SalesTerritory.SalesTerritoryID`.                                                                     |
| `BillToAddressID`        | int              | Billing address. FK to `Address.AddressID`.                                                                           |
| `ShipToAddressID`        | int              | Shipping address. FK to `Address.AddressID`.                                                                          |
| `ShipMethodID`           | int              | Foreign key to `ShipMethod.ShipMethodID`.                                                                             |
| `CreditCardID`           | int              | Foreign key to `CreditCard.CreditCardID`.                                                                             |
| `CreditCardApprovalCode` | varchar(15)      | Credit card approval code.                                                                                            |
| `CurrencyRateID`         | int              | Foreign key to `CurrencyRate.CurrencyRateID`.                                                                         |
| `SubTotal`               | money            | Sales subtotal. **Computed**: `SUM(SalesOrderDetail.LineTotal)` **Default: 0.00**                                     |
| `TaxAmt`                 | money            | Tax amount. **Default: 0.00**                                                                                         |
| `Freight`                | money            | Shipping cost. **Default: 0.00**                                                                                      |
| `TotalDue`               | money            | Total amount due. **Computed**: `ISNULL((SubTotal + TaxAmt) + Freight, 0)`                                            |
| `Comment`                | nvarchar(128)    | Sales representative comment.                                                                                         |
| `rowguid`                | uniqueidentifier | Unique row identifier for replication. **Default: newid()**                                                           |
| `ModifiedDate`           | datetime         | Last update timestamp. **Default: getdate()**                                                                         |

</details>

<details>
<summary>Sales.SalesOrderDetail Schema</summary>

| Column Name             | Data Type        | Description                                                                                       |
| ----------------------- | ---------------- | ------------------------------------------------------------------------------------------------- |
| `SalesOrderID`          | int              | Foreign key to `SalesOrderHeader.SalesOrderID`.                                                   |
| `SalesOrderDetailID`    | int              | Primary key. One unique row per product sold. *(Identity / Auto increment)*                       |
| `CarrierTrackingNumber` | nvarchar(25)     | Shipment tracking number from the shipper.                                                        |
| `OrderQty`              | smallint         | Quantity of the product ordered.                                                                  |
| `ProductID`             | int              | Foreign key to `Product.ProductID`.                                                               |
| `SpecialOfferID`        | int              | Foreign key to `SpecialOffer.SpecialOfferID`.                                                     |
| `UnitPrice`             | money            | Selling price of one product unit.                                                                |
| `UnitPriceDiscount`     | money            | Discount amount. **Default: 0.00**                                                                |
| `LineTotal`             | numeric(38,6)    | Subtotal per product. **Computed**: `ISNULL(UnitPrice * (1 - UnitPriceDiscount) * OrderQty, 0.0)` |
| `rowguid`               | uniqueidentifier | Unique row identifier for replication. **Default: newid()**                                       |
| `ModifiedDate`          | datetime         | Last update timestamp. **Default: getdate()**                                                     |

</details>

<details>
<summary>Sales.SpecialOffer Schema</summary>

| Column Name      | Data Type        | Description                                               |
| ---------------- | ---------------- | --------------------------------------------------------- |
| `SpecialOfferID` | int              | Primary key. *(Identity / Auto increment)*                |
| `Description`    | nvarchar(255)    | Description of the discount offer.                        |
| `DiscountPct`    | smallmoney       | Discount percentage. **Default: 0.00**                    |
| `Type`           | nvarchar(50)     | Category of discount (e.g., Seasonal, Reseller, etc.).    |
| `Category`       | nvarchar(50)     | Group the discount applies to (e.g., Customer, Reseller). |
| `StartDate`      | datetime         | Start date of the offer.                                  |
| `EndDate`        | datetime         | End date of the offer.                                    |
| `MinQty`         | int              | Minimum quantity for the discount. **Default: 0**         |
| `MaxQty`         | int              | Maximum quantity allowed for discount.                    |
| `rowguid`        | uniqueidentifier | Unique identifier for replication. **Default: newid()**   |
| `ModifiedDate`   | datetime         | Last update timestamp. **Default: getdate()**             |

</details>

<details>
<summary>Production.Product Schema</summary>
  
| Column Name             | Data Type        | Description                                                                    |
| ----------------------- | ---------------- | ------------------------------------------------------------------------------ |
| `ProductID`             | int              | Primary key for Product records. *(Identity / Auto increment)*                 |
| `Name`                  | nvarchar(50)     | Name of the product.                                                           |
| `ProductNumber`         | nvarchar(25)     | Unique product identification number.                                          |
| `MakeFlag`              | bit              | 0 = Product is purchased, 1 = Product is manufactured in-house. **Default: 1** |
| `FinishedGoodsFlag`     | bit              | 0 = Not salable, 1 = Salable product. **Default: 1**                           |
| `Color`                 | nvarchar(15)     | Product color.                                                                 |
| `SafetyStockLevel`      | smallint         | Minimum inventory quantity.                                                    |
| `ReorderPoint`          | smallint         | Inventory level that triggers a purchase or work order.                        |
| `StandardCost`          | money            | Standard cost of the product.                                                  |
| `ListPrice`             | money            | Selling price of the product.                                                  |
| `Size`                  | nvarchar(5)      | Product size.                                                                  |
| `SizeUnitMeasureCode`   | nchar(3)         | Unit of measure for size.                                                      |
| `WeightUnitMeasureCode` | nchar(3)         | Unit of measure for weight.                                                    |
| `Weight`                | decimal(8,2)     | Product weight.                                                                |
| `DaysToManufacture`     | int              | Number of days required to manufacture the product.                            |
| `ProductLine`           | nchar(2)         | Product line: R = Road, M = Mountain, T = Touring, S = Standard.               |
| `Class`                 | nchar(2)         | Product class: H = High, M = Medium, L = Low.                                  |
| `Style`                 | nchar(2)         | Style: W = Womens, M = Mens, U = Universal.                                    |
| `ProductSubcategoryID`  | int              | Foreign key to `ProductSubCategory.ProductSubCategoryID`.                      |
| `ProductModelID`        | int              | Foreign key to `ProductModel.ProductModelID`.                                  |
| `SellStartDate`         | datetime         | Date the product was available for sale.                                       |
| `SellEndDate`           | datetime         | Date the product was no longer available for sale.                             |
| `DiscontinuedDate`      | datetime         | Date the product was discontinued.                                             |
| `rowguid`               | uniqueidentifier | Unique identifier for merge replication. **Default: newid()**                  |
| `ModifiedDate`          | datetime         | Last updated timestamp. **Default: getdate()**                                 |

</details>

<details>
<summary>Production.ProductSubcategory Schema</summary>

| Column Name            | Data Type        | Description                                                        |
| ---------------------- | ---------------- | ------------------------------------------------------------------ |
| `ProductSubcategoryID` | int              | Primary key for subcategory records. *(Identity / Auto increment)* |
| `ProductCategoryID`    | int              | Foreign key to `ProductCategory.ProductCategoryID`.                |
| `Name`                 | nvarchar(50)     | Subcategory description.                                           |
| `rowguid`              | uniqueidentifier | Unique identifier for replication. **Default: newid()**            |
| `ModifiedDate`         | datetime         | Date and time the record was last updated. **Default: getdate()**  |

</details>

<details>
<summary>Production.WorkOrder Schema</summary>

| Column Name     | Data Type | Description                                                                          |
| --------------- | --------- | ------------------------------------------------------------------------------------ |
| `WorkOrderID`   | int       | Primary key for work order records. *(Identity / Auto increment)*                    |
| `ProductID`     | int       | Foreign key to `Product.ProductID`.                                                  |
| `OrderQty`      | int       | Product quantity to build.                                                           |
| `StockedQty`    | int       | Quantity built and put into inventory. Computed: `ISNULL(OrderQty - ScrappedQty, 0)` |
| `ScrappedQty`   | smallint  | Quantity that failed inspection.                                                     |
| `StartDate`     | datetime  | Work order start date.                                                               |
| `EndDate`       | datetime  | Work order end date.                                                                 |
| `DueDate`       | datetime  | Work order due date.                                                                 |
| `ScrapReasonID` | smallint  | Reason code for inspection failure.                                                  |
| `ModifiedDate`  | datetime  | Date and time the record was last updated. **Default: getdate()**                    |

</details>

<details>
<summary>Purchasing.PurchaseOrderHeader Schema</summary>

| Column Name       | Data Type | Description                                                                         |
| ----------------- | --------- | ----------------------------------------------------------------------------------- |
| `PurchaseOrderID` | int       | Primary key. *(Identity / Auto increment)*                                          |
| `RevisionNumber`  | tinyint   | Tracks changes to the purchase order. **Default: 0**                                |
| `Status`          | tinyint   | Order status: 1 = Pending, 2 = Approved, 3 = Rejected, 4 = Complete. **Default: 1** |
| `EmployeeID`      | int       | Creator of the purchase order. FK to `Employee.BusinessEntityID`.                   |
| `VendorID`        | int       | Vendor with whom the order was placed. FK to `Vendor.BusinessEntityID`.             |
| `ShipMethodID`    | int       | Shipping method. FK to `ShipMethod.ShipMethodID`.                                   |
| `OrderDate`       | datetime  | Date the order was created. **Default: getdate()**                                  |
| `ShipDate`        | datetime  | Estimated shipment date from the vendor.                                            |
| `SubTotal`        | money     | Order subtotal. **Computed**: `SUM(PurchaseOrderDetail.LineTotal)`                  |
| `TaxAmt`          | money     | Tax amount. **Default: 0.00**                                                       |
| `Freight`         | money     | Shipping cost. **Default: 0.00**                                                    |
| `TotalDue`        | money     | Total amount due. **Computed**: `ISNULL((SubTotal + TaxAmt) + Freight, 0)`          |
| `ModifiedDate`    | datetime  | Last update timestamp. **Default: getdate()**                                       |

</details>

<details>
<summary>Purchasing.PurchaseOrderDetail Schema</summary>

| Column Name             | Data Type    | Description                                                                               |
| ----------------------- | ------------ | ----------------------------------------------------------------------------------------- |
| `PurchaseOrderID`       | int          | Foreign key to `PurchaseOrderHeader.PurchaseOrderID`.                                     |
| `PurchaseOrderDetailID` | int          | Primary key. One line per purchased product. *(Identity / Auto increment)*                |
| `DueDate`               | datetime     | Expected receipt date of the product.                                                     |
| `OrderQty`              | smallint     | Quantity ordered.                                                                         |
| `ProductID`             | int          | Foreign key to `Product.ProductID`.                                                       |
| `UnitPrice`             | money        | Vendor's price per product unit.                                                          |
| `LineTotal`             | money        | Subtotal per product. **Computed**: `ISNULL(OrderQty * UnitPrice, 0.00)`                  |
| `ReceivedQty`           | decimal(8,2) | Quantity actually received from the vendor.                                               |
| `RejectedQty`           | decimal(8,2) | Quantity rejected during inspection.                                                      |
| `StockedQty`            | decimal(9,2) | Quantity accepted into inventory. **Computed**: `ISNULL(ReceivedQty - RejectedQty, 0.00)` |
| `ModifiedDate`          | datetime     | Last update timestamp. **Default: getdate()**                                             |

</details>

**3. Data Relationships:**

![ERD](https://github.com/Channpate/SQL-Analyze-Sales-Performance-and-Customer-Retention-of-Online-Bicycle-Retailer/blob/4d49abbad6a561fb8da21ea409811d2c098a8d22/ERD.png)

## ⚒️ Process
### Query 1: Calculate Quantity of items, Sales value & Order quantity by each Subcategory in L12M (Last 12 months)

L12M from ‘2014-06-30’ is ‘2013-07-01’ → DATESUB returns ‘2013-06-30’,DATE_TRUNC wont’t be used as it includes June’s Orders.

#### Code:
``` sql
-- find out L12M
WITH date_range as (
  SELECT DATE_SUB(DATE(MAX(OrderDate)), INTERVAL 12 MONTH) L12M
  FROM `adventureworks2019.Sales.SalesOrderHeader`
)

-- join tables and filter order date
,join_table as ( 
  SELECT sub_cate.Name Category
      , OrderDate
      , ord_head.SalesOrderId
      , OrderQty
      , LineTotal
  FROM `adventureworks2019.Sales.SalesOrderHeader` as ord_head
  LEFT JOIN `adventureworks2019.Sales.SalesOrderDetail` ord_detail
    ON ord_head.SalesOrderID = ord_detail.SalesOrderId
  LEFT JOIN `adventureworks2019.Production.Product` prod
    ON ord_detail.ProductID = prod.ProductID
  LEFT JOIN `adventureworks2019.Production.ProductSubcategory` sub_cate
    ON CAST(prod.ProductSubcategoryID AS INT64) = sub_cate.ProductSubcategoryID
  WHERE DATE(OrderDate) > (SELECT L12M FROM date_range)
)

-- calculate Quantity of items, Sales value & Order quantity, group by date & cate
SELECT FORMAT_DATE('%b %Y',DATE(OrderDate)) period
    , Category as Subcate
    , SUM(OrderQty) qty_item
    , COUNT(DISTINCT SalesOrderId) ord_count
    , ROUND(SUM(LineTotal),4) total_sales
FROM join_table
GROUP BY 1, 2
ORDER BY PARSE_DATE('%b %Y',period), Category ASC
```
#### Result (first 10 rows):

| period   | Subcate           | qty_item | ord_count | total_sales  |
|----------|-------------------|----------|-----------|--------------|
| Jul 2013 | Bib-Shorts        | 2        | 1         | 116.987      |
| Jul 2013 | Bike Racks        | 422      | 75        | 29802.3      |
| Jul 2013 | Bike Stands       | 19       | 19        | 3021.0       |
| Jul 2013 | Bottles and Cages | 845      | 386       | 4705.5904    |
| Jul 2013 | Bottom Brackets   | 155      | 36        | 8787.57      |
| Jul 2013 | Brakes            | 205      | 42        | 13099.5      |
| Jul 2013 | Caps              | 526      | 214       | 3331.7443    |
| Jul 2013 | Chains            | 156      | 32        | 1886.789     |
| Jul 2013 | Cleaners          | 344      | 118       | 1830.3166    |
| Jul 2013 | Cranksets         | 191      | 39        | 36294.954    |

#### Observations & Findings:
* **Bicycles are the top-selling product group**, especially Road Bikes, Mountain Bikes and Touring Bikes, each of which typically generates over $1 million/month with a very high average order value.
* **Accessories such as Tires, Gloves, Helmets have high sales volume but low revenue**, making them suitable for combo or upsell strategies to increase order value.
* **Peak sales fall in March–May**, followed by a sharp decline in June, indicating a clear seasonal sales trend that should be exploited in marketing plans.

### Query 2: Calc % YoY growth rate by SubCategory & release top 3 cat with highest grow rate. 

Can use metric: quantity_item. Round results to 2 decimal

#### Code:
``` sql
WITH join_table AS (
  SELECT sub_cate.Name category
      , EXTRACT(YEAR FROM OrderDate) year
      , SUM(OrderQty) qty_item
  FROM `adventureworks2019.Sales.SalesOrderHeader` ord_head -- get time 
  LEFT JOIN `adventureworks2019.Sales.SalesOrderDetail` ord_detail --get ord quantity
    ON ord_head.SalesOrderID = ord_detail.SalesOrderId
  LEFT JOIN `adventureworks2019.Production.Product` prod -- prod_id to join with subcate
    ON ord_detail.ProductID = prod.ProductID
  LEFT JOIN `adventureworks2019.Production.ProductSubcategory` sub_cate -- get subcate
    ON CAST(prod.ProductSubcategoryID AS INT64) = sub_cate.ProductSubcategoryID
  GROUP BY EXTRACT(YEAR FROM OrderDate), sub_cate.Name
  ORDER BY sub_cate.Name, EXTRACT(YEAR FROM OrderDate)
)
, lag_table AS ( -- align with sale quant last year
  SELECT *
    , LAG(qty_item,1) OVER(PARTITION BY category ORDER BY year) prv_qty
  FROM join_table
)
, calc_yoy AS ( -- yoy growth rate
  SELECT *
    , ROUND((qty_item-prv_qty)*100.0/prv_qty,2) yoy_growth_pct
  FROM lag_table
)
SELECT year -- rank and filter
      , category
      , qty_item
      , prv_qty
      , yoy_growth_pct
FROM (
  SELECT *
    , DENSE_RANK() OVER(PARTITION BY year ORDER BY yoy_growth_pct DESC) dr
  FROM calc_yoy
) AS sub_rank
WHERE dr < 4 AND prv_qty IS NOT NULL
ORDER BY year, category
```
#### Result:
| year | category        | qty_item | prv_qty | yoy_growth_pct |
|------|-----------------|----------|---------|----------------|
| 2012 | Jerseys         | 4263     | 1027    | 315.09         |
| 2012 | Mountain Frames | 3168     | 510     | 521.18         |
| 2012 | Road Frames     | 5564     | 1137    | 389.36         |
| 2013 | Jerseys         | 12104    | 4263    | 183.93         |
| 2013 | Shorts          | 5761     | 1586    | 263.24         |
| 2013 | Socks           | 2724     | 523     | 420.84         |
| 2014 | Bike Stands     | 113      | 136     | -16.91         |
| 2014 | Fenders         | 1033     | 1088    | -5.06          |
| 2014 | Tires and Tubes | 8720     | 9286    | -6.1           |

#### Observations & Findings:
* **2012–2013 saw explosive growth in key products** such as Mountain Frames (+521%), Road Frames (+389%) and Jerseys (+315% → +183% next), indicating a period of strong expansion in the core category.
* **2014 marked a slight decline** in some categories such as Bike Stands, Fenders and Tires and Tubes with YoY declines ranging from -5% to -17%, suggesting that the market is starting to mature or demand has peaked in these products.

### Query 3: Ranking Top 3 TeritoryID with biggest Order quantity of every year. If theres TerritoryID with same quantity in a year, do not skip the rank number 
#### Code:
``` sql
WITH  
sale_info AS (
  SELECT 
      FORMAT_TIMESTAMP("%Y", ord_head.ModifiedDate) AS yr
      , ord_head.TerritoryID
      , SUM(OrderQty) AS order_cnt 
  FROM `adventureworks2019.Sales.SalesOrderDetail` ord_detail 
  LEFT JOIN `adventureworks2019.Sales.SalesOrderHeader` ord_head 
    ON ord_detail.SalesOrderID = ord_head.SalesOrderID
  GROUP BY 1,2
),

sale_rank AS (
  SELECT *
      , DENSE_RANK() OVER(PARTITION BY yr ORDER BY order_cnt DESC) AS rk 
  FROM sale_info 
)

SELECT yr
    , TerritoryID
    , order_cnt
    , rk
FROM sale_rank 
WHERE rk IN(1,2,3) 
```
#### Result:
| yr   | TerritoryID | order_cnt | rk |
|------|-------------|-----------|----|
| 2011 | 4           | 3238      | 1  |
| 2011 | 6           | 2705      | 2  |
| 2011 | 1           | 1964      | 3  |
| 2014 | 4           | 11632     | 1  |
| 2014 | 6           | 9711      | 2  |
| 2014 | 1           | 8823      | 3  |
| 2012 | 4           | 17553     | 1  |
| 2012 | 6           | 14412     | 2  |
| 2012 | 1           | 8537      | 3  |
| 2013 | 4           | 26682     | 1  |
| 2013 | 6           | 22553     | 2  |
| 2013 | 1           | 17452     | 3  |

#### Observations & Findings:
* TerritoryID 4 consistently leads in order count across all years (2011–2014), indicating it is the strongest-performing region.
* All territories show strong growth from 2011 to 2013, especially Territory 4 (from 3,238 to 26,682 orders), reflecting rapid market expansion, before a slight drop in 2014, suggesting potential market stabilization.

### Query 4: Calc Total Discount Cost belongs to Seasonal Discount for each SubCategory
#### Code:
``` sql
SELECT 
    FORMAT_TIMESTAMP("%Y", ModifiedDate) year
    , Name
    , SUM(disc_cost) AS total_cost
FROM (
      SELECT DISTINCT ord_detail.ModifiedDate
      , cate.Name
      , offer.DiscountPct, offer.Type
      , ord_detail.OrderQty * offer.DiscountPct * UnitPrice as disc_cost 
      from `adventureworks2019.Sales.SalesOrderDetail` ord_detail
      LEFT JOIN `adventureworks2019.Production.Product` prod
            ON ord_detail.ProductID = prod.ProductID
      LEFT JOIN `adventureworks2019.Production.ProductSubcategory` cate
            ON cast(prod.ProductSubcategoryID as int) = cate.ProductSubcategoryID
      LEFT JOIN `adventureworks2019.Sales.SpecialOffer` offer
            ON ord_detail.SpecialOfferID = offer.SpecialOfferID
      WHERE LOWER(offer.Type) LIKE '%seasonal discount%' -- ensure all value format 
)
GROUP BY 1,2
```
#### Result:
| year | Name    | total_cost |
|------|---------|------------|
| 2012 | Helmets | 149.71669  |
| 2013 | Helmets | 543.21975  |

#### Observations & Findings:
* Helmet costs surged from 2012 to 2013 (from 149.72 to 543.22), suggesting a significant increase in demand, product upgrades, or pricing changes during that period.

### Query 5: Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)

#### Code:
``` sql
WITH 
info AS ( -- retrieve customer info + count successfully shipped order for each customer 
  SELECT  
      EXTRACT(month FROM ModifiedDate) AS month_no
      , EXTRACT(year FROM ModifiedDate) AS year_no
      , CustomerID
      , COUNT(DISTINCT SalesOrderID) AS order_cnt
  FROM `adventureworks2019.Sales.SalesOrderHeader`
  WHERE FORMAT_TIMESTAMP("%Y", ModifiedDate) = '2014'
  AND Status = 5
  GROUP BY 1,2,3
  ORDER BY 3,1 
),

row_num AS (-- number month customer ordered
  SELECT *
      , ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY month_no) AS row_numb
  FROM info 
), 

first_order AS (   --retrieve the first month of each customer 
  SELECT *
  FROM row_num
  WHERE row_numb = 1
), 

month_gap AS ( -- join info and first_order  
  SELECT 
      a.CustomerID
      , b.month_no AS month_join
      , a.month_no AS month_order
      , a.order_cnt
      , CONCAT('M - ',a.month_no - b.month_no) AS month_diff
  FROM info a
  LEFT JOIN first_order b 
  ON a.CustomerID = b.CustomerID
  ORDER BY 1,3
)

SELECT month_join
      , month_diff 
      , COUNT(DISTINCT CustomerID) AS customer_cnt
FROM month_gap
GROUP BY 1,2
ORDER BY 1,2
```
#### Result:
| month_join | month_diff | customer_cnt |
|------------|------------|--------------|
| 1          | M - 0      | 2076         |
| 1          | M - 1      | 78           |
| 1          | M - 2      | 89           |
| 1          | M - 3      | 252          |
| 1          | M - 4      | 96           |
| 1          | M - 5      | 61           |
| 1          | M - 6      | 18           |
| 2          | M - 0      | 1805         |
| 2          | M - 1      | 51           |
| 2          | M - 2      | 61           |
| 2          | M - 3      | 234          |
| 2          | M - 4      | 58           |
| 2          | M - 5      | 8            |
| 3          | M - 0      | 1918         |
| 3          | M - 1      | 43           |
| 3          | M - 2      | 58           |
| 3          | M - 3      | 44           |
| 3          | M - 4      | 11           |
| 4          | M - 0      | 1906         |
| 4          | M - 1      | 34           |
| 4          | M - 2      | 44           |
| 4          | M - 3      | 7            |
| 5          | M - 0      | 1947         |
| 5          | M - 1      | 40           |
| 5          | M - 2      | 7            |
| 6          | M - 0      | 909          |
| 6          | M - 1      | 10           |
| 7          | M - 0      | 148          |

#### Observations & Findings:
- **Customer acquisition is strongest in the joining month** (M - 0) across all months, with a steep drop in customer activity in subsequent months, highlighting low retention or re-engagement over time.
- Months 1 to 3 show higher long-term engagement (e.g., Month 1 has 252 customers still active at M - 3) compared to later months like Month 5 or 6.
  
### Query 6: Trend of Stock level & MoM diff % by all product in 2011. If %gr rate is null then 0. Round to 1 decimal

#### Code:
``` sql
WITH 
raw_data AS (
  SELECT
      EXTRACT(month from w_ord.ModifiedDate) AS mth 
      , EXTRACT(year from w_ord.ModifiedDate) AS yr 
      , prod.Name
      , SUM(StockedQty) AS stock_qty

  FROM `adventureworks2019.Production.WorkOrder` w_ord
  LEFT JOIN `adventureworks2019.Production.Product` prod on w_ord.ProductID = prod.ProductID
  WHERE FORMAT_TIMESTAMP("%Y", w_ord.ModifiedDate) = '2011'
  GROUP BY 1,2,3
  ORDER BY 1 DESC
)

SELECT  Name
      , mth, yr 
      , stock_qty
      , stock_prv    
      , ROUND(coalesce((stock_qty /stock_prv -1)*100 ,0) ,1) AS diff   
FROM (                                                                 
      SELECT *
      , LEAD(stock_qty) OVER (PARTITION BY Name ORDER BY mth desc) AS stock_prv
      FROM raw_data
      )
ORDER BY 1 ASC, 2 DESC
```
#### Result:
| Name                           | mth | yr   | stock_qty | stock_prv | diff   |
|--------------------------------|-----|------|-----------|-----------|--------|
| BB Ball Bearing                | 12  | 2011 | 8475      | 14544     | -41.7  |
| BB Ball Bearing                | 11  | 2011 | 14544     | 19175     | -24.2  |
| BB Ball Bearing                | 10  | 2011 | 19175     | 8845      | 116.8  |
| BB Ball Bearing                | 9   | 2011 | 8845      | 9666      | -8.5   |
| BB Ball Bearing                | 8   | 2011 | 9666      | 12837     | -24.7  |
| BB Ball Bearing                | 7   | 2011 | 12837     | 5259      | 144.1  |
| BB Ball Bearing                | 6   | 2011 | 5259      | null      | 0.0    |
| Blade                          | 12  | 2011 | 1842      | 3598      | -48.8  |
| Blade                          | 11  | 2011 | 3598      | 4670      | -23.0  |
| Blade                          | 10  | 2011 | 4670      | 2122      | 120.1  |
| Blade                          | 9   | 2011 | 2122      | 2382      | -10.9  |
| Blade                          | 8   | 2011 | 2382      | 3166      | -24.8  |
| Blade                          | 7   | 2011 | 3166      | 1280      | 147.3  |
| Blade                          | 6   | 2011 | 1280      | null      | 0.0    |

#### Observations & Findings (for 2 products in Result):
- BB Ball Bearing stock levels show significant fluctuations throughout 2011, with major restocking spikes in July and October, followed by sharp declines in subsequent months. This suggests irregular inventory replenishment and high variability in demand or supply.
- Blade inventory patterns mirror those of BB Ball Bearing, indicating cyclical or seasonal demand.

### Query 7:  Calc Ratio of Stock / Sales in 2011 by product name, by month

Order results by month desc, ratio desc. Round Ratio to 1 decimal mom yoy

#### Code:
``` sql
WITH  
sale_info AS (
  SELECT 
      EXTRACT(month from ord_detail.ModifiedDate) AS mth 
     , EXTRACT(year from ord_detail.ModifiedDate) AS yr 
     , ord_detail.ProductId
     , prod.Name
     , SUM(ord_detail.OrderQty) AS sales
  FROM `adventureworks2019.Sales.SalesOrderDetail` ord_detail
  LEFT JOIN `adventureworks2019.Production.Product` prod
    ON ord_detail.ProductID = prod.ProductID
  WHERE FORMAT_TIMESTAMP("%Y", ord_detail.ModifiedDate) = '2011'
  GROUP BY 1,2,3,4
), 

stock_info AS (
  SELECT
      EXTRACT(month from ModifiedDate) AS mth 
      , EXTRACT(year from ModifiedDate) AS yr 
      , ProductId
      , SUM(StockedQty) AS stock_cnt
  FROM `adventureworks2019.Production.WorkOrder`
  WHERE FORMAT_TIMESTAMP("%Y", ModifiedDate) = '2011'
  GROUP BY 1,2,3
)

SELECT
      sale.mth
    , sale.yr
    , sale.ProductId
    , sale.Name
    , sale.sales
    , stock.stock_cnt AS stock  --(*)
    , ROUND(COALESCE(stock.stock_cnt,0) / sales,2) AS ratio
FROM sale_info sale 
FULL JOIN stock_info stock
  ON sale.ProductId = stock.ProductId
AND sale.mth = stock.mth 
AND sale.yr = stock.yr
ORDER BY 1 DESC, 7 DESC
```
#### Result (shortlisted):
| mth  | yr   | ProductId | Name                           | sales | stock | ratio |
|------|------|-----------|--------------------------------|-------|-------|-------|
| 12   | 2011 | 745       | HL Mountain Frame - Black, 48  | 1     | 27    | 27.0  |
| 12   | 2011 | 743       | HL Mountain Frame - Black, 42  | 1     | 26    | 26.0  |
| 12   | 2011 | 748       | HL Mountain Frame - Silver, 38 | 2     | 32    | 16.0  |
| 12   | 2011 | 722       | LL Road Frame - Black, 58      | 4     | 47    | 11.75 |
| 12   | 2011 | 747       | HL Mountain Frame - Black, 38  | 3     | 31    | 10.33 |
| 12   | 2011 | 726       | LL Road Frame - Red, 48        | 5     | 36    | 7.2   |
| 12   | 2011 | 738       | LL Road Frame - Black, 52      | 10    | 64    | 6.4   |
| 12   | 2011 | 730       | LL Road Frame - Red, 62        | 7     | 38    | 5.43  |
| 12   | 2011 | 741       | HL Mountain Frame - Silver, 48 | 5     | 27    | 5.4   |
| 12   | 2011 | 725       | LL Road Frame - Red, 44        | 12    | 53    | 4.42  |
| 12   | 2011 | 729       | LL Road Frame - Red, 60        | 10    | 43    | 4.3   |
| 12   | 2011 | 732       | ML Road Frame - Red, 48        | 10    | 16    | 1.6   |
| 12   | 2011 | 750       | Road-150 Red, 44               | 25    | 38    | 1.52  |

#### Observations & Findings: 
- The ratio (stock-to-sales) reflects how many units are in stock per unit sold — a higher ratio (>10) suggests slow-moving or overstocked products, while a lower ratio (<5) indicates faster-selling or better-turnover items, potentially requiring restocking or promotion to meet demand.

### Query 8: No of order and value at Pending status in 2014

#### Code:
``` sql
SELECT EXTRACT(year FROM OrderDate) year
    , COUNT(DISTINCT purs_header.PurchaseOrderID) order_count
    , SUM(LineTotal) value
FROM `adventureworks2019.Purchasing.PurchaseOrderHeader` purs_header
LEFT JOIN `adventureworks2019.Purchasing.PurchaseOrderDetail` purs_detail
  ON purs_header.PurchaseOrderID = purs_detail.PurchaseOrderID
WHERE EXTRACT(year FROM OrderDate) = 2014
  AND Status = 1
GROUP BY 1
```
#### Result:
| year | order_count | value              |
|------|-------------|--------------------|
| 2014 | 224         | 3505501.3665000061 |

#### Observations & Findings:
* In 2014, the company processed 224 orders with a total order value exceeding 3.5 million, indicating high average order value (~15,646 per order)

## 🔎 Final Conclusion & Recommendations
| Insight | Recommendation | 
|------|-------------|
| Customer acquisition is front-loaded but retention quickly drops, a high churn rate and short customer lifecycle | Implement post-onboarding engagement campaigns (e.g., loyalty points, email nurturing) to improve retention. |
| Sales growth varies by product and year | Focus investment and promotions on high-growth categories; re-evaluate underperforming ones for discontinuation or bundle offers. |
| Sales concentration is high in certain regions (TerritoryID 4, 6, 1) | Double down on top-performing territories with regional promotions, while exploring why other areas lag behind.|
| Inventory analysis shows volatility, stock levels fluctuate wildly| Adopt demand-based restocking to minimize overstock and stockouts. Apply tighter stock forecasting.|
