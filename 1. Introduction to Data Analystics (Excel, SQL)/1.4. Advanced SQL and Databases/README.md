# Sprint: Advanced SQL and Databases
The task involves creating queries to address specific business questions using the AdventureWorks 2005 database. This includes exploring the database, identifying the necessary data, and determining the appropriate methods to retrieve and merge it with other tables within the database using BigQuery in order to provide solutions to the given business questions.
## Task 1
### Task 1.1
You’ve been tasked to create a detailed overview of all individual customers (these are defined by customerType = ‘I’ and/or stored in an individual table). Write a query that provides:

- Identity information : CustomerId, Firstname, Last Name, FullName (First Name & Last Name).
- An Extra column called addressing_title i.e. (Mr. Achong), if the title is missing - Dear Achong.
- Contact information : Email, phone, account number, CustomerType.
- Location information : City, State & Country, address.
- Sales: number of orders, total amount (with Tax), date of the last order.
Copy only the top 200 rows from your written select ordered by total amount (with tax). 
### Solution 1.1
```
WITH LatestAddress AS (
        SELECT
                customer_address.CustomerID,
                address.StateProvinceID,
                MAX(customer_address.AddressID) AS LatestAddressID,
                address.City,
                address.AddressLine1,
                address.AddressLine2
        FROM `adwentureworks_db.customeraddress` customer_address
        JOIN `adwentureworks_db.address` address
        ON customer_address.AddressID=address.AddressID
        GROUP BY 
                customer_address.CustomerID,
                address.StateProvinceID,
                address.City,
                address.AddressLine1,
                address.AddressLine2
),

        CustomerData AS(
        SELECT
                individual.CustomerID,
                contact.Firstname,
                contact.LastName,
                CONCAT(contact.Firstname, ' ', contact.LastName) FullName,
                CASE 
                        WHEN contact.Title IS NOT NULL THEN CONCAT(contact.Title, ' ', contact.LastName) 
                        ELSE CONCAT('Dear', ' ', contact.LastName) 
                        END AS AddressingTitle,
                contact.EmailAddress,
                contact.Phone,
                customer.AccountNumber,
                customer.CustomerType,
                latest_address.City,
                latest_address.AddressLine1,
                CASE 
                        WHEN latest_address.AddressLine2 IS NULL THEN '' 
                        ELSE latest_address.AddressLine2 
                        END AS AddressLine2,
                state_province.Name State,
                country_region.Name Country
        FROM `adwentureworks_db.individual` individual

        LEFT JOIN `adwentureworks_db.customer` customer
        ON individual.CustomerID=customer.CustomerID

        JOIN `adwentureworks_db.contact` contact
        ON individual.ContactID=contact.ContactId

        JOIN LatestAddress latest_address
        ON customer.CustomerID=latest_address.CustomerID 

        JOIN `adwentureworks_db.stateprovince` state_province
        ON latest_address.StateProvinceID=state_province.StateProvinceID

        JOIN `adwentureworks_db.countryregion` country_region
        ON state_province.CountryRegionCode=country_region.CountryRegionCode
        WHERE customer.CustomerType='I'

),
        CustomerOrders AS(
        SELECT
                CustomerID,
                COUNT(SalesOrderID) AS NumberOrders,
                ROUND(SUM (TotalDue),3) AS TotalAmount,
                MAX(OrderDate) AS DateLastOrder
        FROM `adwentureworks_db.salesorderheader` 
        GROUP BY CustomerID
        )

SELECT 
    CustomerData.CustomerID,
    CustomerData.Firstname,
    CustomerData.LastName,
    CustomerData.FullName,
    CustomerData.AddressingTitle,
    CustomerData.EmailAddress,
    CustomerData.Phone,
    CustomerData.AccountNumber,
    CustomerData.CustomerType,
    CustomerData.City,
    CustomerData.AddressLine1,
    CustomerData.AddressLine2,
    CustomerData.State,
    CustomerData.Country,
    CustomerOrders.NumberOrders,
    CustomerOrders.TotalAmount,
    CustomerOrders.DateLastOrder

FROM CustomerData
JOIN CustomerOrders
ON CustomerData.CustomerID=CustomerOrders.CustomerID

ORDER BY TotalAmount DESC
LIMIT 200; 
```
### Task 1.2
Business finds the original query valuable to analyze customers and now want to get the data from the first query for the top 200 customers with the highest total amount (with tax) who have not ordered for the last 365 days. How would you identify this segment?

Note that the database is old and the current date should be defined by finding the latest order date in the orders table.
### Solution 1.2
```
WITH LatestAddress AS (
        SELECT
                customer_address.CustomerID,
                address.StateProvinceID,
                MAX(customer_address.AddressID) AS LatestAddressID,
                address.City,
                address.AddressLine1,
                address.AddressLine2
        FROM `adwentureworks_db.customeraddress` customer_address
        JOIN `adwentureworks_db.address` address
        ON customer_address.AddressID=address.AddressID
        GROUP BY 
                customer_address.CustomerID,
                address.StateProvinceID,
                address.City,
                address.AddressLine1,
                address.AddressLine2
),

        CustomerData AS(
        SELECT
                individual.CustomerID,
                contact.Firstname,
                contact.LastName,
                CONCAT(contact.Firstname, ' ', contact.LastName) FullName,
                CASE 
                        WHEN contact.Title IS NOT NULL THEN CONCAT(contact.Title, ' ', contact.LastName) 
                        ELSE CONCAT('Dear', ' ', contact.LastName) 
                        END AS AddressingTitle,
                contact.EmailAddress,
                contact.Phone,
                customer.AccountNumber,
                customer.CustomerType,
                latest_address.City,
                latest_address.AddressLine1,
                CASE 
                        WHEN latest_address.AddressLine2 IS NULL THEN '' 
                        ELSE latest_address.AddressLine2 
                        END AS AddressLine2,
                state_province.Name State,
                country_region.Name Country
        FROM `adwentureworks_db.individual` individual

        LEFT JOIN `adwentureworks_db.customer` customer
        ON individual.CustomerID=customer.CustomerID

        JOIN `adwentureworks_db.contact` contact
        ON individual.ContactID=contact.ContactId

        JOIN LatestAddress latest_address
        ON customer.CustomerID=latest_address.CustomerID 

        JOIN `adwentureworks_db.stateprovince` state_province
        ON latest_address.StateProvinceID=state_province.StateProvinceID

        JOIN `adwentureworks_db.countryregion` country_region
        ON state_province.CountryRegionCode=country_region.CountryRegionCode
        WHERE customer.CustomerType='I'

),
        CustomerOrders AS(
        SELECT
                CustomerID,
                COUNT(SalesOrderID) AS NumberOrders,
                ROUND(SUM (TotalDue),3) AS TotalAmount,
                MAX(OrderDate) AS DateLastOrder
        FROM `adwentureworks_db.salesorderheader` 
        GROUP BY CustomerID
),
          LatestOrder AS (
              SELECT
                  CustomerID,
                  MAX(OrderDate) AS LatestOrderDate
              FROM `adwentureworks_db.salesorderheader`
              GROUP BY CustomerID
)


SELECT 
    CustomerData.CustomerID,
    CustomerData.Firstname,
    CustomerData.LastName,
    CustomerData.FullName,
    CustomerData.AddressingTitle,
    CustomerData.EmailAddress,
    CustomerData.Phone,
    CustomerData.AccountNumber,
    CustomerData.CustomerType,
    CustomerData.City,
    CustomerData.AddressLine1,
    CustomerData.AddressLine2,
    CustomerData.State,
    CustomerData.Country,
    CustomerOrders.NumberOrders,
    CustomerOrders.TotalAmount,
    CustomerOrders.DateLastOrder

FROM CustomerData
JOIN CustomerOrders
ON CustomerData.CustomerID=CustomerOrders.CustomerID

JOIN LatestOrder
ON LatestOrder.CustomerID=CustomerData.CustomerID

WHERE (LatestOrder.LatestOrderDate IS NULL OR LatestOrder.LatestOrderDate< DATE_SUB((SELECT MAX(OrderDate) FROM `adwentureworks_db.salesorderheader`), INTERVAL 365 DAY)) 

ORDER BY TotalAmount DESC
LIMIT 200;
```
### Task 1.3
Enrich your original 1.1 SELECT by creating a new column in the view that marks active & inactive customers based on whether they have ordered anything during the last 365 days.
- Copy only the top 500 rows from your written select ordered by CustomerId desc.
### Solution 1.3
```
WITH LatestAddress AS (
        SELECT
                customer_address.CustomerID,
                address.StateProvinceID,
                MAX(customer_address.AddressID) AS LatestAddressID,
                address.City,
                address.AddressLine1,
                address.AddressLine2
        FROM `adwentureworks_db.customeraddress` customer_address
        JOIN `adwentureworks_db.address` address
        ON customer_address.AddressID=address.AddressID
        GROUP BY 
                customer_address.CustomerID,
                address.StateProvinceID,
                address.City,
                address.AddressLine1,
                address.AddressLine2
),

        CustomerData AS(
        SELECT
                individual.CustomerID,
                contact.Firstname,
                contact.LastName,
                CONCAT(contact.Firstname, ' ', contact.LastName) FullName,
                CASE 
                        WHEN contact.Title IS NOT NULL THEN CONCAT(contact.Title, ' ', contact.LastName) 
                        ELSE CONCAT('Dear', ' ', contact.LastName) 
                        END AS AddressingTitle,
                contact.EmailAddress,
                contact.Phone,
                customer.AccountNumber,
                customer.CustomerType,
                latest_address.City,
                latest_address.AddressLine1,
                CASE 
                        WHEN latest_address.AddressLine2 IS NULL THEN '' 
                        ELSE latest_address.AddressLine2 
                        END AS AddressLine2,
                state_province.Name State,
                country_region.Name Country
        FROM `adwentureworks_db.individual` individual

        LEFT JOIN `adwentureworks_db.customer` customer
        ON individual.CustomerID=customer.CustomerID

        JOIN `adwentureworks_db.contact` contact
        ON individual.ContactID=contact.ContactId

        JOIN LatestAddress latest_address
        ON customer.CustomerID=latest_address.CustomerID 

        JOIN `adwentureworks_db.stateprovince` state_province
        ON latest_address.StateProvinceID=state_province.StateProvinceID

        JOIN `adwentureworks_db.countryregion` country_region
        ON state_province.CountryRegionCode=country_region.CountryRegionCode
        
        WHERE customer.CustomerType='I'

),
        CustomerOrders AS(
        SELECT
                CustomerID,
                COUNT(SalesOrderID) AS NumberOrders,
                ROUND(SUM (TotalDue),3) AS TotalAmount,
                MAX(OrderDate) AS DateLastOrder
        FROM `adwentureworks_db.salesorderheader` 
        GROUP BY CustomerID
),
          LatestOrder AS (
              SELECT
                  CustomerID,
                  MAX(OrderDate) AS LatestOrderDate
              FROM `adwentureworks_db.salesorderheader`
              GROUP BY CustomerID
),

          Activity AS(
              SELECT
                  CustomerID,
                  CASE 
                      WHEN (LatestOrderDate IS NULL OR LatestOrderDate< DATE_SUB((SELECT MAX(OrderDate) FROM `adwentureworks_db.salesorderheader`), INTERVAL 365 DAY)) THEN 'Inactive'
                      ELSE 'Active' 
                      END AS ActivityInfo
              FROM LatestOrder
)


SELECT 
    CustomerData.CustomerID,
    CustomerData.Firstname,
    CustomerData.LastName,
    CustomerData.FullName,
    CustomerData.AddressingTitle,
    CustomerData.EmailAddress,
    CustomerData.Phone,
    CustomerData.AccountNumber,
    CustomerData.CustomerType,
    CustomerData.City,
    CustomerData.AddressLine1,
    CustomerData.AddressLine2,
    CustomerData.State,
    CustomerData.Country,
    CustomerOrders.NumberOrders,
    CustomerOrders.TotalAmount,
    CustomerOrders.DateLastOrder,
    Activity.ActivityInfo

FROM CustomerData
JOIN CustomerOrders
ON CustomerData.CustomerID=CustomerOrders.CustomerID

JOIN LatestOrder
ON LatestOrder.CustomerID=CustomerData.CustomerID

JOIN Activity 
ON CustomerData.CustomerID=Activity.CustomerID

ORDER BY CustomerID DESC
LIMIT 500;
```
### Task 1.4
Business would like to extract data on all active customers from North America. Only customers that have either ordered no less than 2500 in total amount (with Tax) or ordered 5 + times should be presented.

In the output for these customers divide their address line into two columns, i.e.:
| AddressLine1       | address_no | Address_st     |
|--------------------|------------|----------------|
| '8603 Elmhurst Lane' | 8603       | Elmhurst Lane  |

Order the output by country, state and date_last_order.
### Solution 1.4
```
WITH LatestAddress AS (
        SELECT
                customer_address.CustomerID,
                address.StateProvinceID,
                MAX(customer_address.AddressID) AS LatestAddressID,
                address.City,
                address.AddressLine1 AddressLine1,
                address.AddressLine2 AddressLine2,
        FROM `adwentureworks_db.customeraddress` customer_address
        JOIN `adwentureworks_db.address` address
        ON customer_address.AddressID=address.AddressID
        GROUP BY 
                customer_address.CustomerID,
                address.StateProvinceID,
                address.City,
                address.AddressLine1,
                address.AddressLine2
),

        CustomerData AS(
        SELECT
                individual.CustomerID,
                contact.Firstname,
                contact.LastName,
                CONCAT(contact.Firstname, ' ', contact.LastName) FullName,
                CASE 
                        WHEN contact.Title IS NOT NULL THEN CONCAT(contact.Title, ' ', contact.LastName) 
                        ELSE CONCAT('Dear', ' ', contact.LastName) 
                        END AS AddressingTitle,
                contact.EmailAddress,
                contact.Phone,
                customer.AccountNumber,
                customer.CustomerType,
                latest_address.City,
                latest_address.AddressLine1,
                REGEXP_EXTRACT(latest_address.AddressLine1, '^[0-9]+') AS Address_no,
                REGEXP_REPLACE(latest_address.AddressLine1, '^[0-9]+(,)?', '') AS Address_st,  
                CASE 
                        WHEN latest_address.AddressLine2 IS NULL THEN '' 
                        ELSE latest_address.AddressLine2 
                        END AS AddressLine2,
                state_province.Name State,
                country_region.Name Country
        FROM `adwentureworks_db.individual` individual

        LEFT JOIN `adwentureworks_db.customer` customer
        ON individual.CustomerID=customer.CustomerID

        JOIN `adwentureworks_db.contact` contact
        ON individual.ContactID=contact.ContactId

        JOIN LatestAddress latest_address
        ON customer.CustomerID=latest_address.CustomerID 

        JOIN `adwentureworks_db.stateprovince` state_province
        ON latest_address.StateProvinceID=state_province.StateProvinceID

        JOIN `adwentureworks_db.countryregion` country_region
        ON state_province.CountryRegionCode=country_region.CountryRegionCode
        
        WHERE customer.CustomerType='I'

),
        CustomerOrders AS(
        SELECT
                CustomerID,
                COUNT(SalesOrderID) AS NumberOrders,
                ROUND(SUM (TotalDue),3) AS TotalAmount,
                MAX(OrderDate) AS DateLastOrder
        FROM `adwentureworks_db.salesorderheader` 
        GROUP BY CustomerID
),
          LatestOrder AS (
              SELECT
                  CustomerID,
                  MAX(OrderDate) AS LatestOrderDate
              FROM `adwentureworks_db.salesorderheader`
              GROUP BY CustomerID
),

          Activity AS(
              SELECT
                  CustomerID,
                  CASE 
                      WHEN (LatestOrderDate IS NULL OR LatestOrderDate< DATE_SUB((SELECT MAX(OrderDate) FROM `adwentureworks_db.salesorderheader`), INTERVAL 365 DAY)) THEN 'Inactive'
                      ELSE 'Active' 
                      END AS ActivityInfo
              FROM LatestOrder
),

    CustomerTotals AS (
        SELECT
            CustomerID,
            SUM(TotalDue) AS Total_amount,
            COUNT(SalesOrderID) AS Order_count 
        FROM `adwentureworks_db.salesorderheader` 
        GROUP BY CustomerID
    )



SELECT 
    CustomerData.CustomerID,
    CustomerData.Firstname,
    CustomerData.LastName,
    CustomerData.FullName,
    CustomerData.AddressingTitle,
    CustomerData.EmailAddress,
    CustomerData.Phone,
    CustomerData.AccountNumber,
    CustomerData.CustomerType,
    CustomerData.City,
    CustomerData.AddressLine1,
    CustomerData.Address_no,
    CustomerData.Address_st, 
    CustomerData.AddressLine2,
    CustomerData.State,
    CustomerData.Country,
    CustomerOrders.NumberOrders,
    CustomerOrders.TotalAmount,
    CustomerOrders.DateLastOrder,
    Activity.ActivityInfo

FROM CustomerData
JOIN CustomerOrders
ON CustomerData.CustomerID=CustomerOrders.CustomerID

JOIN LatestOrder
ON LatestOrder.CustomerID=CustomerData.CustomerID

JOIN Activity 
ON CustomerData.CustomerID=Activity.CustomerID

JOIN CustomerTotals 
ON CustomerData.CustomerID=CustomerTotals.CustomerID

WHERE ActivityInfo = 'Active'
AND CustomerData.Country IN ('Canada','United States')
AND (CustomerOrders.TotalAmount>= 2500 OR CustomerOrders.NumberOrders >=5)

ORDER BY Country, State, DateLastOrder DESC;
```
## Task 2 Reporting Sales’ numbers
Main tables to start from: salesorderheader.
### Task 2.1
Create a query of monthly sales numbers in each Country & region. Include in the query a number of orders, customers and sales persons in each month with a total amount with tax earned. Sales numbers from all types of customers are required.
### Solution 2.1
```
SELECT
      LAST_DAY(DATE(OrderDate)) AS Order_month,
      sales_territory.CountryRegionCode AS Country_region_code,
      sales_territory.name AS Region,
      COUNT(SalesOrderID) AS Number_orders,
      COUNT(DISTINCT CustomerID) AS Number_customers,
      COUNT(DISTINCT SalesPersonID) AS No_sales_persons,
      CAST(SUM(TotalDue) AS INT) AS Total_w_tax

FROM `adwentureworks_db.salesorderheader` sales_order

JOIN `adwentureworks_db.salesterritory` sales_territory
ON sales_order.TerritoryID=sales_territory.TerritoryID

GROUP BY 
      Order_month,
      Country_region_code,
      Region

ORDER BY Country_region_code DESC;
```
### Task 2.2
Enrich 2.1 query with the cumulative_sum of the total amount with tax earned per country & region.
### Solution 2.2
```
WITH MonthlySalesNumbers AS (
        SELECT
            LAST_DAY(DATE(OrderDate)) AS Order_month,
            sales_territory.CountryRegionCode AS Country_region_code,
            sales_territory.name AS Region,
            COUNT(SalesOrderID) AS Number_orders,
            COUNT(DISTINCT CustomerID) AS Number_customers,
            COUNT(DISTINCT SalesPersonID) AS No_sales_persons,
            CAST(SUM(TotalDue) AS INT) AS Total_w_tax
            
      FROM `adwentureworks_db.salesorderheader` sales_order

      JOIN `adwentureworks_db.salesterritory` sales_territory
      ON sales_order.TerritoryID=sales_territory.TerritoryID

      GROUP BY 
            Order_month,
            Country_region_code,
            Region

      ORDER BY Country_region_code DESC
)

SELECT
      Order_month,
      Country_region_code,
      Region,
      Number_orders,
      Number_customers,
      No_sales_persons,
      Total_w_tax,
      SUM(Total_w_tax) OVER (PARTITION BY Country_region_code, Region ORDER BY Order_month) AS Cumulative_sum

FROM MonthlySalesNumbers
ORDER BY Country_region_code;
```
### Task 2.3
Enrich 2.2 query by adding ‘sales_rank’ column that ranks rows from best to worst for each country based on total amount with tax earned each month. I.e. the month where the (US, Southwest) region made the highest total amount with tax earned will be ranked 1 for that region and vice versa.
### Solution 2.3
```
WITH MonthlySalesNumbers AS (
        SELECT
            LAST_DAY(DATE(OrderDate)) AS Order_month,
            sales_territory.CountryRegionCode AS Country_region_code,
            sales_territory.name AS Region,
            COUNT(SalesOrderID) AS Number_orders,
            COUNT(DISTINCT CustomerID) AS Number_customers,
            COUNT(DISTINCT SalesPersonID) AS No_sales_persons,
            CAST(SUM(TotalDue) AS INT) AS Total_w_tax
            
      FROM `adwentureworks_db.salesorderheader` sales_order

      JOIN `adwentureworks_db.salesterritory` sales_territory
      ON sales_order.TerritoryID=sales_territory.TerritoryID

      GROUP BY 
            Order_month,
            Country_region_code,
            Region

      ORDER BY Country_region_code DESC
)

SELECT
      Order_month,
      Country_region_code,
      Region,
      Number_orders,
      Number_customers,
      No_sales_persons,
      Total_w_tax,
      DENSE_RANK() OVER (PARTITION BY Country_region_code ORDER BY Total_w_tax DESC) AS Country_sales_rank,
      SUM(Total_w_tax) OVER (PARTITION BY Country_region_code, Region ORDER BY Order_month) AS Cumulative_sum

FROM MonthlySalesNumbers
WHERE Region= 'France' --filter can be adjusted or removed if needed
ORDER BY Country_sales_rank; 
```
### Task 2.4
Enrich 2.3 query by adding taxes on a country level:
- As taxes can vary in country based on province, the needed column is ‘mean_tax_rate’ -> average tax rate in a country.
- Also, as not all regions have data on taxes, you also want to be transparent and show the ‘perc_provinces_w_tax’ -> a column representing the percentage of provinces with available tax rates for each country (i.e. If US has 53 provinces, and 10 of them have tax rates, then for US it should show 0,19)
- If a state has multiple tax rates, choose the higher one. Do not double count a state in country average rate calculation if it has multiple tax rates.
- Ignore the isonlystateprovinceFlag rate mechanic, it is beyond the scope of this exercise. Treat all tax rates as equal.
### Solution 2.4
```
WITH MonthlySalesNumbers AS (
        SELECT
            LAST_DAY(DATE(OrderDate)) AS Order_month,
            sales_territory.CountryRegionCode AS Country_region_code,
            sales_territory.name AS Region,
            COUNT(SalesOrderID) AS Number_orders,
            COUNT(DISTINCT CustomerID) AS Number_customers,
            COUNT(DISTINCT SalesPersonID) AS No_sales_persons,
            CAST(SUM(TotalDue) AS INT) AS Total_w_tax           
      FROM `adwentureworks_db.salesorderheader` sales_order
      JOIN `adwentureworks_db.salesterritory` sales_territory
      ON sales_order.TerritoryID=sales_territory.TerritoryID
      GROUP BY 
            Order_month,
            Country_region_code,
            Region
      ORDER BY Country_region_code DESC
        ),

        --Some states have multiple tax rates, here I filter the higher ones (as the task asks)--
        MaxStateTaxRates AS (
        SELECT 
            state_province.CountryRegionCode,
            state_province.Name,
            MAX(sale_tax_rate.TaxRate) AS Max_tax_rate
        FROM `tc-da-1.adwentureworks_db.salestaxrate`  sale_tax_rate
        JOIN `adwentureworks_db.stateprovince` state_province
        ON sale_tax_rate.StateProvinceID=state_province.StateProvinceID
        GROUP BY 
            state_province.Name, state_province.CountryRegionCode
        ),

        -- Calculating average tax rate in each country--
        CountryTaxRates AS (
        SELECT
             MaxStateTaxRates.CountryRegionCode,
            ROUND(AVG(MaxStateTaxRates.Max_tax_rate),1) as Mean_tax_rate
        FROM MaxStateTaxRates
        GROUP BY 
            MaxStateTaxRates.CountryRegionCode
        ),

        -- All provinces with available Tax rate for each country--
        ProvincesWithTax AS (
            SELECT
                state_province.CountryRegionCode,
                COUNT(DISTINCT sale_tax_rate.StateProvinceID) AS Number_of_provinces
            FROM `adwentureworks_db.salestaxrate` sale_tax_rate
            JOIN `adwentureworks_db.stateprovince` state_province
            ON sale_tax_rate.StateProvinceID=state_province.StateProvinceID
            GROUP BY 
                state_province.CountryRegionCode
        ),

        -- All provinces with and without Tax rates--
        AllProvinces AS (
            SELECT 
                state_province.CountryRegionCode,
                COUNT(DISTINCT state_province.StateProvinceID) AS Total_Provinces
            FROM `adwentureworks_db.stateprovince` state_province
            GROUP BY 
                state_province.CountryRegionCode 
        )


SELECT
    MonthlySalesNumbers.Order_month,
    MonthlySalesNumbers.Country_region_code,
    MonthlySalesNumbers.Region,
    MonthlySalesNumbers.Number_orders,
    MonthlySalesNumbers.Number_customers,
    MonthlySalesNumbers.No_sales_persons,
    MonthlySalesNumbers.Total_w_tax,
    DENSE_RANK() OVER (PARTITION BY Country_region_code ORDER BY Total_w_tax DESC) AS Country_sales_rank,
    SUM(Total_w_tax) OVER (PARTITION BY Country_region_code, Region ORDER BY Order_month) AS Cumulative_sum,
    CountryTaxRates.Mean_tax_rate,
    ROUND(ProvincesWithTax.Number_of_provinces/AllProvinces.Total_Provinces,2) AS Perc_provinces_w_tax

FROM MonthlySalesNumbers

JOIN CountryTaxRates 
ON MonthlySalesNumbers.Country_region_code = CountryTaxRates.CountryRegionCode 

JOIN ProvincesWithTax
ON MonthlySalesNumbers.Country_region_code = ProvincesWithTax.CountryRegionCode

JOIN AllProvinces
ON MonthlySalesNumbers.Country_region_code = AllProvinces.CountryRegionCode

WHERE MonthlySalesNumbers.Country_region_code = 'US' --filter can be adjusted or removed if needed
ORDER BY Country_sales_rank;
```