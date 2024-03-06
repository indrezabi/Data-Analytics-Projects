# Sprint: Advanced SQL and Databases
The task involves creating queries to address specific business questions using the AdventureWorks 2005 database. This includes exploring the database, identifying the necessary data, and determining the appropriate methods to retrieve and merge it with other tables within the database using BigQuery in order to provide solutions to the given business questions.
### Task 1.1
You’ve been tasked to create a detailed overview of all individual customers (these are defined by customerType = ‘I’ and/or stored in an individual table). Write a query that provides:

- Identity information : CustomerId, Firstname, Last Name, FullName (First Name & Last Name).
- An Extra column called addressing_title i.e. (Mr. Achong), if the title is missing - Dear Achong.
- Contact information : Email, phone, account number, CustomerType.
- Location information : City, State & Country, address.
- Sales: number of orders, total amount (with Tax), date of the last order.
Copy only the top 200 rows from your written select ordered by total amount (with tax).