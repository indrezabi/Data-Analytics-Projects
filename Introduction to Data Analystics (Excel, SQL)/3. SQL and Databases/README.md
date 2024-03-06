# Sprint: SQL and Databases
The task involves exploring the AdventureWorks 2005 database, identifying the necessary data, and determining the appropriate methods to retrieve and merge it with other tables within the same database using Google BigQuery.
## Task 1 : An overview of Products
### Task 1.1
You’ve been asked to extract the data on products from the Product table where there exists a product subcategory. And also include the name of the ProductSubcategory.

- Columns needed: ProductId, Name, ProductNumber, size, color, ProductSubcategoryId, Subcategory name.
- Order results by SubCategory name.

### Solution 1.1
```
  SELECT 
        product.ProductID, 
        product.Name, 
        product.ProductNumber, 
        product.Size, 
        product.Color, 
        product_subcategory.ProductSubcategoryID, 
        product_subcategory.Name SubCategory
FROM `adwentureworks_db.product` product
JOIN `adwentureworks_db.productsubcategory` product_subcategory
ON product.ProductSubcategoryID=product_subcategory.ProductSubcategoryID
ORDER BY SubCategory;
```

### Task 1.2
In 1.1 query you have a product subcategory but see that you could use the category name.

- Find and add the product category name.
- Afterwards order the results by Category name.

### Solution 1.2
```
SELECT 
        product.ProductID, 
        product.Name, 
        product.ProductNumber, 
        product.Size, 
        product.Color, 
        product_subcategory.ProductSubcategoryID, 
        product_subcategory.Name SubCategory,
        product_category.Name Category
FROM `adwentureworks_db.product` product
JOIN `adwentureworks_db.productsubcategory` product_subcategory
ON product.ProductSubcategoryID=product_subcategory.ProductSubcategoryID
JOIN `adwentureworks_db.productcategory` product_category
ON product_subcategory.ProductCategoryID=product_category.ProductCategoryID
ORDER BY Category;
```
### Task 1.3
Use the established query to select the most expensive (price listed over 2000) bikes that are still actively sold (does not have a sales end date)

- Order the results from most to least expensive bike.
### Solution 1.3
```
SELECT 
        product.ProductID, 
        product.ListPrice,
        product.Name, 
        product.ProductNumber, 
        product.Size, 
        product.Color, 
        product_subcategory.ProductSubcategoryID, 
        product_subcategory.Name SubCategory,
        product_category.Name Category,
        product.SellEndDate
FROM `adwentureworks_db.product` product
JOIN `adwentureworks_db.productsubcategory` product_subcategory
ON product.ProductSubcategoryID=product_subcategory.ProductSubcategoryID
JOIN `adwentureworks_db.productcategory` product_category
ON product_subcategory.ProductCategoryID=product_category.ProductCategoryID
WHERE   product_category.Name = 'Bikes' 
        AND
        product.ListPrice >2000 
        AND 
        SellEndDate IS NULL
ORDER BY product.ListPrice DESC;
```

## Task 2. Reviewing work orders
### Task 2.1 
Create an aggregated query to select the:

- Number of unique work orders.
- Number of unique products.
- Total actual cost.
For each location Id from the 'workoderrouting' table for orders in January 2004.

### Solution 2.1
```
SELECT LocationId location_id,
        COUNT(DISTINCT WorkOrderID) no_work_orders,
        COUNT(DISTINCT ProductId) no_unique_products,
        SUM(ActualCost) actual_cost,
FROM `adwentureworks_db.workorderrouting`
WHERE DATE(ActualStartDate) BETWEEN '2004-01-01' AND '2004-01-31'
GROUP BY location_id
ORDER BY no_work_orders DESC;
```
### Task 2.2
Update your 2.1 query by adding the name of the location and also add the average days amount between actual start date and actual end date per each location.
### Solution 2.2
```
SELECT wor.LocationId location_id,
        loc.Name location,
        COUNT(DISTINCT wor.WorkOrderID) no_work_orders,
        COUNT(DISTINCT wor.ProductId) no_unique_products,
        SUM(wor.ActualCost) actual_cost,
       ROUND(AVG(DATE_DIFF(wor.ActualEndDate, wor.ActualStartDate, DAY)), 2) AS avg_days_diff
FROM `adwentureworks_db.workorderrouting` wor
JOIN `adwentureworks_db.location` loc
ON wor.LocationID=loc.LocationID
WHERE DATE(wor.ActualStartDate) BETWEEN '2004-01-01' AND '2004-01-31'
GROUP BY wor.LocationId, loc.Name 
ORDER BY no_work_orders DESC;
```
### Task 2.3
Select all the expensive work Orders (above 300 actual cost) that happened throught January 2004.
### Solution 2.3
```
SELECT WorkOrderID work_order_id,
       SUM(ActualCost) actual_cost
FROM `adwentureworks_db.workorderrouting` 
WHERE DATE(ActualStartDate) BETWEEN '2004-01-01' AND '2004-01-31'
GROUP BY WorkOrderID
HAVING actual_cost >300;
```
## Task 3 : Query validation
Below you will find 2 queries that need to be fixed/updated.
### Task 3.1
Your colleague has written a query to find the list of orders connected to special offers. The query works fine but the numbers are off, investigate where the potential issue lies.

Query below:
```
SELECT sales_detail.SalesOrderId
      ,sales_detail.OrderQty
      ,sales_detail.UnitPrice
      ,sales_detail.LineTotal
      ,sales_detail.ProductId
      ,sales_detail.SpecialOfferID
      ,spec_offer_product.ModifiedDate
      ,spec_offer.Category
      ,spec_offer.Description

FROM `tc-da-1.adwentureworks_db.salesorderdetail`  as sales_detail

left join `tc-da-1.adwentureworks_db.specialofferproduct` as spec_offer_product
on sales_detail.productId = spec_offer_product.ProductID

left join `tc-da-1.adwentureworks_db.specialoffer` as spec_offer
on sales_detail.SpecialOfferID = spec_offer.SpecialOfferID

order by LineTotal desc
```
### Solution 3.1
```
SELECT sales_detail.SalesOrderId
      ,sales_detail.OrderQty
      ,sales_detail.UnitPrice
      ,sales_detail.LineTotal
      ,sales_detail.ProductId
      ,sales_detail.SpecialOfferID
      ,spec_offer_product.ModifiedDate
      ,spec_offer.Category
      ,spec_offer.Description

FROM `tc-da-1.adwentureworks_db.salesorderdetail`  AS sales_detail

LEFT JOIN `tc-da-1.adwentureworks_db.specialofferproduct` AS spec_offer_product
ON sales_detail.productId = spec_offer_product.ProductID
AND sales_detail.SpecialOfferID = spec_offer_product.SpecialOfferID

LEFT JOIN `tc-da-1.adwentureworks_db.specialoffer` AS spec_offer
ON sales_detail.SpecialOfferID = spec_offer.SpecialOfferID

ORDER BY LineTotal DESC;
```
### Task 3.2 
Your colleague has written this query to collect basic Vendor information. The query does not work, look into the query and find ways to fix it. Can you provide any feedback on how to make this query be easier to debug/read?
```
SELECT a.VendorId as Id,vendor_contact.ContactId, b.ContactTypeId, a.Name, a.CreditRating, a.ActiveFlag, c.AddressId,d.City

FROM tc-da-1.adwentureworks_db.Vendor as a

left join tc-da-1.adwentureworks_db.vendorcontact as vendor_contact on vendor.VendorId = vendor_contact.VendorId left join tc-da1.adwentureworks_db.vendoraddress as c on a.VendorId = c.VendorId

left join tc-da-1.adwentureworks_db.address as address on vendor_address.VendorId = d.VendorId
```
### Solution 3.2
```
SELECT Vendor.VendorId,
      Vendor.Name,
      VendorContact.ContactId, 
      VendorContact.ContactTypeId,
      Vendor.CreditRating, 
      Vendor.ActiveFlag, 
      VendorAddress.AddressId,
      Address.City

FROM `adwentureworks_db.vendor` AS Vendor

LEFT JOIN `adwentureworks_db.vendorcontact` AS VendorContact
ON Vendor.VendorId = VendorContact.VendorId 

LEFT JOIN `adwentureworks_db.vendoraddress` AS VendorAddress 
ON Vendor.VendorId = VendorAddress.VendorId

LEFT JOIN `adwentureworks_db.address` AS Address
ON VendorAddress.AddressId = Address.AddressID

ORDER BY Vendor.VendorId;
```
