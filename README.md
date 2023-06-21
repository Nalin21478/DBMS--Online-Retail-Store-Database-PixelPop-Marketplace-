# DBMS--Online-Retail-Store-Database-PixelPop-Marketplace-

# PIXELPOP MARKETPLACE
# Features in our Application 
1) Registration
● To register as a customer, supplier, or delivery executive, click the "Register" button on the homepage.
● Fill in the required details and submit the form.
2) Customer Features
● Click the "Products" tab on the homepage to view the products.
● Enter the product name or category in the search bar to search for a
product.
● To filter products based on quantity or price, use the filter options provided
on the page.
● To add a product to the cart, click the "Add to Cart" button next to the
product.
● After adding the product, the customer can view the cart and place an order
after checking the bill details and successfully paying.


3)Supplier Features
● Click the "Add Product" button on the supplier dashboard to add a new product.
● Fill in the required details, such as product name, description, category, and price.
● To change the price of a product, go to the "Products" tab and click on the "Change Price" button next to the product.
● Enter the new price and submit the form.
● To update the inventory, go to the "Products" tab and click the "Update
Inventory" button next to the product.
● Enter the new stock quantity and submit the form.
4)Delivery Executive Features
● To view your data, just login, and you will be able to see data.
● Here, you can view your personal details.
5)Admin Features
● To access the admin dashboard, log in as an admin.
● Update the salaries of delivery executives.
● View the various products and categories.
● View the suppliers, customers, and delivery executives registered.
● Analyse net revenue and profit across categories and all orders according to
OLAP Queries
  
# OLAP Queries
We have implemented four OLAP queries in our retail store. We have populated our tables with some dummy data to get results for these queries.
1) Our First Olap Query is Top 10 products. We are showing this on the customer login page. On clicking, the customer can view the top 10 selling products based on previous orders and search and filter to find that product on the main page.
  2) Our second query shows the Net Revenue generated by each category. This is visible in the Admin Login in the ‘Manage Inventory -> Manage Category’ option.
 3) Our third query shows the Net Profit from each order. This is available in the admin login again under the ‘Manage Customers’ option. Here we have applied a random formula to generate some profit to show the functionality of our OLAP query.
 
 4) Our fourth OLAP query shows the Net earning of each customer based on their order. This is also in the exact location of our third query under a different button.
 
# Triggers
We have implemented three triggers in our retail store.
1) The first trigger is when a customer enters their payment details. After viewing their
order, they proceed to payment details. If the card's expiry date refers to a date in the past, the details won’t be accepted, and an Error message will be displayed. The user can then enter their card details again with the correct requirements.
  2) The second trigger is when the user tries to add a product to the cart, but the required quantity exceeds the available quantity. When this trigger gives an error, it will be displayed in the top right corner. Otherwise, it will state, ‘Successfully added.’
 
 3) Our Third trigger is based on the updation of the product quantity. It is more of a fallback feature in case our database encounters a negative value. Every time a customer places an order, the product quantity they ordered is subtracted from the inventory. To avoid getting negative values in the database, our trigger checks if the product quantity to be updated is less than equal to zero and then sets it to zero.
Basic Queries
1) We have implemented basic searching and filtering queries everywhere. Each customer and supplier can search for products based on their names. The customer can also sort the products based on quantity, category ID, and price.
2) We have a query in the admin login that allows the salaries of the delivery executives to be updated according to the number of trips they have made.
The query: update sql_users.delivery_executive set salary=salary+1.1*salary where no_of_trips>=150;
3) The details for the customer, supplier, and delivery executive can be changed by updating the tables.
4) The supplier can change the inventory and price of the products based on the user input of the supplier.
  
# Transactions
1) Transaction 1, basically occurs when we try to insert a new product in our database. It inserts into the database, checks if the category it is inserting into is valid and if it is valid it gets inserted into the database, as well as the prod_contains table which is a table that has every combination of valid Product ID and Category ID.
  Schedule 1 - Conflicting Serializable:
T1:
START TRANSACTION
INSERT INTO sql_Inventory.Product (Category_ID, Product_Name, Product_Description, Price, Quantity)
SELECT 21, "fffff", "fdfdfdf", 34, 5
FROM (SELECT 1) AS dummy
WHERE EXISTS (SELECT 1 FROM sql_Inventory.Category WHERE Category_ID = 21);
SET @last_product_id = LAST_INSERT_ID();
T2: INSERT INTO prod_contains (Category_ID, Product_ID) VALUES (21, @last_product_id);
T3: COMMIT;
Solving the conflicting serialzable:
T1:START TRANSACTION
T2: SELECT * FROM sql_Inventory.Category WHERE Category_ID = 21 FOR UPDATE; T3: INSERT INTO sql_Inventory.Product (Category_ID, Product_Name, Product_Description, Price, Quantity)
SELECT 21, "fffff", "fdfdfdf", 34, 5 FROM (SELECT 1) AS dummy

 WHERE EXISTS (SELECT 1 FROM sql_Inventory.Category WHERE Category_ID = 21);
T4: SET @last_product_id = LAST_INSERT_ID(); T5: COMMIT;
T6:
START TRANSACTION;
INSERT INTO prod_contains (Category_ID, Product_ID) VALUES (21, @last_product_id);
T7:COMMIT;
A new SELECT statement (T2) has been added that acquires an exclusive lock on the row in the sql_Inventory.Category table with Category_ID = 21. This lock prevents other transactions from modifying this row until the lock is released. The lock is released when the transaction is committed (T5), which allows the INSERT statement for the prod_contains table (T6) to be executed without conflicts
Schedule 2 - Non-Conflicting Serializable:
T1:
START TRANSACTION ;
INSERT INTO sql_Inventory.Product (Category_ID, Product_Name, Product_Description, Price, Quantity)
SELECT 21, "fffff", "fdfdfdf", 34, 5
FROM (SELECT 1) AS dummy
WHERE EXISTS (SELECT 1 FROM sql_Inventory.Category WHERE Category_ID = 21);
SET @last_product_id = LAST_INSERT_ID();
T2: COMMIT;
T3:
START TRANSACTION;
INSERT INTO prod_contains (Category_ID, Product_ID)
VALUES (21, @last_product_id); T4:COMMIT;

 2) In Transaction 2,It inserts a new record into the assigns table for each successful payment that has not yet been assigned to an agent. The admin_user_id is set to 'mehak', the payment_ID is set to the Payment_ID from the success_payment table, and the agent_id is selected randomly from the sql_users.Delivery_Executive table.It updates the no_of_trips field in the sql_users.Delivery_Executive table for each agent that has been assigned a payment in the previous step.It inserts a new record into the ordered_items table for each successful payment that has not yet been recorded in this table. The order_ID is set to the payment_ID from the success_payment table, the total_cost is set to the corresponding value from the bill_details table, and the products_ordered field is set to a comma-separated list of product IDs from the cart_Details table. It updates the Quantity field in the sql_Inventory.Product table for each product that has been ordered in a successful payment that has not yet been recorded in the ordered_items table. The quantity is decreased by the corresponding value from the cart_Details table.
 
 Conflicting schedule:
T1: START TRANSACTION;
INSERT INTO assigns (admin_user_id, payment_ID, agent_id)
SELECT 'mehak', sp.Payment_ID, (SELECT sql_users.Delivery_Executive.agent_id FROM sql_users.Delivery_Executive ORDER BY RAND() LIMIT 1)
FROM success_payment sp
LEFT JOIN assigns a ON sp.Payment_ID = a.payment_ID
WHERE a.payment_ID IS NULL;
COMMIT;
T2: START TRANSACTION;
UPDATE sql_users.Delivery_Executive de JOIN assigns a ON de.agent_id = a.agent_id SET de.no_of_trips = de.no_of_trips + 1; COMMIT;
T3: START TRANSACTION;
INSERT INTO ordered_items (order_ID,total_cost, products_ordered) SELECT sp.payment_ID,bd.total_cost, GROUP_CONCAT(cd.Product_ID) FROM bill_details bd
JOIN cart_Details cd ON bd.BILL_ID = cd.Cart_ID
JOIN success_payment sp ON cd.Cart_ID = sp.payment_ID
LEFT JOIN ordered_items oi ON sp.payment_ID = oi.order_ID
WHERE oi.order_ID IS NULL
GROUP BY bd.BILL_ID;
COMMIT;
T4: START TRANSACTION;
UPDATE sql_Inventory.Product p
JOIN cart_Details cd ON p.Product_ID = cd.Product_ID
JOIN success_payment sp ON cd.Cart_ID = sp.payment_ID LEFT JOIN ordered_items oi ON sp.payment_ID = oi.order_ID SET p.Quantity = p.Quantity - cd.Product_quantity
WHERE oi.order_ID IS NULL;
COMMIT;
In this schedule, T1 and T2 have conflicting updates on the sql_users.Delivery_Executive table. If T2 executes before T1, it will update the no_of_trips column before the agent is assigned in T1, potentially resulting in incorrect data.

# Solving the conflict serialzable:
Non- T1:
T2:
T3:
●
● ●
● ●
● ●
● ●
T1 acquires a shared lock on the success_payment table and an exclusive lock on the assigns table.
T1 executes its INSERT statement and releases its locks.
T2 acquires a shared lock on the assigns table and an exclusive lock on the sql_users.Delivery_Executive table.
T2 executes its UPDATE statement and releases its locks.
T3 acquires shared locks on the bill_details, cart_Details, and success_payment tables, and an exclusive lock on the ordered_items table.
T3 executes its INSERT statement and releases its locks.
T4 acquires shared locks on the cart_Details, success_payment, and ordered_items tables, and an exclusive lock on the sql_Inventory.Product table.
T4 executes its UPDATE statement and releases its locks.
This schedule ensures that the transactions are executed in a serial order (T1 -> T2 -> T3 -> T4) and that conflicting operations are executed in the same order in all equivalent serial schedules
In the case of the given transactions T1, T2, T3 and T4, using locks ensures that the transactions are executed in a serial order (T1 -> T2 -> T3 -> T4) and that conflicting operations are executed in the same order in all equivalent serial schedules. This helps to prevent conflicts and ensures that the transactions are executed in a way that maintains the consistency and integrity of the data..
Conflicting Schedule:
START TRANSACTION;
INSERT INTO assigns (admin_user_id, payment_ID, agent_id)
SELECT 'mehak', sp.Payment_ID, (SELECT sql_users.Delivery_Executive.agent_id FROM sql_users.Delivery_Executive ORDER BY RAND() LIMIT 1)
FROM success_payment sp
LEFT JOIN assigns a ON sp.Payment_ID = a.payment_ID
WHERE a.payment_ID IS NULL;
COMMIT;
START TRANSACTION;
UPDATE sql_Inventory.Product p
JOIN cart_Details cd ON p.Product_ID = cd.Product_ID
JOIN success_payment sp ON cd.Cart_ID = sp.payment_ID LEFT JOIN ordered_items oi ON sp.payment_ID = oi.order_ID SET p.Quantity = p.Quantity - cd.Product_quantity
WHERE oi.order_ID IS NULL;
COMMIT;
START TRANSACTION;
UPDATE sql_users.Delivery_Executive de

 JOIN assigns a ON de.agent_id = a.agent_id SET de.no_of_trips = de.no_of_trips + 1; COMMIT;
T4: START TRANSACTION;
INSERT INTO ordered_items (order_ID,total_cost, products_ordered) SELECT sp.payment_ID,bd.total_cost, GROUP_CONCAT(cd.Product_ID) FROM bill_details bd
JOIN cart_Details cd ON bd.BILL_ID = cd.Cart_ID
JOIN success_payment sp ON cd.Cart_ID = sp.payment_ID
LEFT JOIN ordered_items oi ON sp.payment_ID = oi.order_ID
WHERE oi.order_ID IS NULL
GROUP BY bd.BILL_ID;
COMMIT;
# To summarise, in our retail store, we have three main users, a Supplier, a Customer, and a Delivery Executive. We also have an admin that oversees the users and the inventory. Our retail store has limited functionality in the sense that our customers can only place one order right now.