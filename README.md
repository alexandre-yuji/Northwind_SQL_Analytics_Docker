# Advanced Reports in SQL Northwind

## Objective

This repository aims to present advanced reports built in SQL. The analyses provided here can be applied to companies of all sizes that wish to become more analytical. Through these reports, organisations will be able to extract valuable insights from their data, helping them to make strategic decisions.

## Reports we will create

1. **Revenue Reports**
    
    * What was the total revenue in 1997?

    ```sql
    CREATE VIEW total_revenues_1997_view AS
    SELECT SUM((order_details.unit_price) * order_details.quantity * (1.0 - order_details.discount)) AS total_revenues_1997
    FROM order_details
    INNER JOIN (
        SELECT order_id 
        FROM orders 
        WHERE EXTRACT(YEAR FROM order_date) = '1997'
    ) AS ord 
    ON ord.order_id = order_details.order_id;
    ```

    * Perform a monthly growth analysis and YTD calculation

    ```sql
    CREATE VIEW view_accumulated_revenue AS
    WITH MonthlyIncome AS (
        SELECT
            EXTRACT(YEAR FROM orders.order_date) AS Ano,
            EXTRACT(MONTH FROM orders.order_date) AS Mes,
            SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS Monthly_Income
        FROM
            orders
        INNER JOIN
            order_details ON orders.order_id = order_details.order_id
        GROUP BY
            EXTRACT(YEAR FROM orders.order_date),
            EXTRACT(MONTH FROM orders.order_date)
    ),
    AccumulatedRevenue AS (
        SELECT
            Ano,
            Mes,
            Monthly_Income,
            SUM(Monthly_Income) OVER (PARTITION BY Ano ORDER BY Mes) AS Revenue_YTD
        FROM
            MonthlyIncome
    )
    SELECT
        Ano,
        Mes,
        Monthly_Income,
        Monthly_Income - LAG(Monthly_Income) OVER (PARTITION BY Ano ORDER BY Mes) AS Monthly_Diff,
        Revenue_YTD,
        (Monthly_Income - LAG(Monthly_Income) OVER (PARTITION BY Ano ORDER BY Mes)) / LAG(Monthly_Income) OVER (PARTITION BY Ano ORDER BY Mes) * 100 AS Percentual_Mudanca_Mensal
    FROM
        AccumulatedRevenue
    ORDER BY
        Ano, Mes;
    ```

2. **Customer segmentation**
    
    * What is the total amount each customer has paid so far?

    ```sql
    CREATE VIEW view_total_revenues_per_customer AS
    SELECT 
        customers.company_name, 
        SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total
    FROM 
        customers
    INNER JOIN 
        orders ON customers.customer_id = orders.customer_id
    INNER JOIN 
        order_details ON order_details.order_id = orders.order_id
    GROUP BY 
        customers.company_name
    ORDER BY 
        total DESC;
    ```

    * Separate customers into 5 groups according to the amount paid per customer.

    ```sql
    CREATE VIEW view_total_revenues_per_customer_group AS
    SELECT 
    customers.company_name, 
    SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total,
    NTILE(5) OVER (ORDER BY SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) DESC) AS group_number
    FROM 
        customers
    INNER JOIN 
        orders ON customers.customer_id = orders.customer_id
    INNER JOIN 
        order_details ON order_details.order_id = orders.order_id
    GROUP BY 
        customers.company_name
    ORDER BY 
        total DESC;
    ```


    * Now only customers in groups 3, 4, and 5 will undergo a special marketing analysis.

    ```sql
    CREATE VIEW clients_to_marketing AS
    WITH clientes_para_marketing AS (
        SELECT 
        customers.company_name, 
        SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total,
        NTILE(5) OVER (ORDER BY SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) DESC) AS group_number
    FROM 
        customers
    INNER JOIN 
        orders ON customers.customer_id = orders.customer_id
    INNER JOIN 
        order_details ON order_details.order_id = orders.order_id
    GROUP BY 
        customers.company_name
    ORDER BY 
        total DESC
    )

    SELECT *
    FROM clientes_para_marketing
    WHERE group_number >= 3;
    ```

3. **Top 10 Best-Selling Products**
    
    * Identify the top 10 best-selling products.

    ```sql
    CREATE VIEW top_10_products AS
    SELECT products.product_name, SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS sales
    FROM products
    INNER JOIN order_details ON order_details.product_id = products.product_id
    GROUP BY products.product_name
    ORDER BY sales DESC;
    ```

4. **UK customers who paid over $1,000**
    
    * Which UK customers paid over $1,000?

    ```sql
    CREATE VIEW uk_clients_who_pay_more_then_1000 AS
    SELECT customers.contact_name, SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount) * 100) / 100 AS payments
    FROM customers
    INNER JOIN orders ON orders.customer_id = customers.customer_id
    INNER JOIN order_details ON order_details.order_id = orders.order_id
    WHERE LOWER(customers.country) = 'uk'
    GROUP BY customers.contact_name
    HAVING SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) > 1000;
    ```

## Context

The `Northwind` database contains sales data for a company called `Northwind Traders`, which imports and exports specialty foods from around the world.

The Northwind database is an ERP database containing data on customers, orders, inventory, purchases, suppliers, shipments, employees, and accounting.

The Northwind dataset includes sample data for the following:

* **Suppliers:** Northwind's suppliers and vendors
* **Customers:** Customers who purchase products from Northwind
* **Employees:** Details of Northwind Traders' employees
* **Products:** Product information
* **Shippers:** Details of shippers who ship products from traders to end customers
* **Orders and Order Details:** Sales order transactions occurring between customers and the company

The `Northwind` database includes 14 tables, and the relationships between the tables are shown in the following entity relationship diagram.

![northwind](https://github.com/lvgalvao/Northwind-SQL-Analytics/blob/main/pics/northwind-er-diagram.png?raw=true)

## Objective

The objective of this 

## Initial Configuration

### Manually

Use the provided SQL file, `nortwhind.sql`, to populate your database.

### With Docker and Docker Compose

**Pre-requisite**: Install Docker and Docker Compose

* [Getting started with Docker](https://www.docker.com/get-started)
* [Installing Docker Compose](https://docs.docker.com/compose/install/)

### Steps for configuration with Docker:

1. **Start Docker Compose** Run the command below to start the services:
    
    ```
    docker-compose up
    ```
    
    Wait for configuration messages, such as:
    
    ```csharp
    Creating network "northwind_psql_db" with driver "bridge"
    Creating volume "northwind_psql_db" with default driver
    Creating volume "northwind_psql_pgadmin" with default driver
    Creating pgadmin ... done
    Creating db      ... done
    ```
       
2. **Connect to PgAdmin** Access PgAdmin via the URL: [http://localhost:5050](http://localhost:5050), using the password `postgres`. 

Configure a new server in PgAdmin:
    
    * **General tab**:
        * Name: db
    * **Connection tab**:
        * Host name: db
        * Username: postgres
        * Password: postgres 
    Then select the ‘northwind’ database.

3. **Stop Docker Compose** Stop the server started by the `docker-compose up` command using Ctrl-C and remove the containers with:
    
    ```
    docker-compose down
    ```
    
4. **Files and Persistence** Your modifications to the Postgres databases will be persisted in the Docker volume `postgresql_data` and can be recovered by restarting Docker Compose with `docker-compose up`. To delete the database data, run:
    
    ```
    docker-compose down -v
    ```
