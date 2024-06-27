# AdventureWorks
Performance analysis of a global manufacturing company that produces cycling equipment and accessories.

## Overview
The AdventureWorks dataset provided by microsoft, offers sample data with information about a fictitious company with transactions, returns, products, customers and sales territories.

The idea is to analyse this dataset in order to understand the company KPI's (sales, revenue, profit, returns), compare regional performance, analyze product-level trends and identify high-value customers.

## The Data Model
After transform, clean and load the data needed through power query the model achieved is a snowflake with two fact tables ('Sales Data' and 'Returns Data') and six dimentions tables.
Data flows from the lookups tables to the fact tables in a 1 to many relationship.

The calendar table was created following the steps below:
![image](https://github.com/PatrickNAquino/AdventureWorks/assets/118391206/96aec943-0524-4675-8a9c-372c4ffba5c3)
![image](https://github.com/PatrickNAquino/AdventureWorks/assets/118391206/b46f9bda-78fd-4aa2-beb7-c4d39a77a615)


#### Tables
Dimentions
- Calendar Lookup
- Customer Lookup
- Product Lookup
- Product Categories Lookup
- Product Subcategories Lookup
- Territory Lookup

Fact
- Sales Data
- Returns Data

![Data model image](https://github.com/PatrickNAquino/AdventureWorks/assets/118391206/a7bcd7e2-acbd-4fb9-85c3-c0741b8a5633)

## Measures (DAX)
- Total Orders
  ```
  Total Orders = 
  DISTINCTCOUNT(
      'Sales Data'[OrderNumber]
  )
  ```

- Total Revenue
  ```
  Total Revenue = 
    SUMX(
        'Sales Data',
        'Sales Data'[OrderQuantity] *
        RELATED(
            'Product Lookup'[ProductPrice]
        )
    )
  ```
  
- Total Cost
  ```
  Total Cost = 
    SUMX(
        'Sales Data',
        'Sales Data'[OrderQuantity] *
        RELATED(
            'Product Lookup'[ProductCost]
        )
    )
  ```

- Total Profit
  ```
  Total Profit = 
    [Total Revenue]-[Total Cost]
  ```

- Total Returns
  ```
  Total Returns = 
    COUNT(
        'Returns Data'[ReturnQuantity]
    )
  ```

- Quantity Sold
  ```
  Quantity Sold = 
    SUM(
        'Sales Data'[OrderQuantity]
    )
  ```

- Quantity Returned
  ```
  Quantity Returned = 
    COUNT(
        'Returns Data'[ProductKey]
    )
  ```

- Return Rate
  ```
  Return Rate = 
    DIVIDE(
        [Quantity Returned],
        [Quantity Sold],
        "No Sales"
    )
  ```

- Average Retail Price
  ```
  Average Retail Price = 
    AVERAGE(
        'Product Lookup'[ProductPrice]
    )
  ```

- Overall Average Price
  ```
  Overall Average Price = 
    CALCULATE(
        [Average Retail Price],
        ALL(
            'Product Lookup'
        )
    )
  ```

- High Ticket Orders
  ```
  High Ticket Orders = 
    CALCULATE(
        [Total Orders],
        FILTER(
            'Product Lookup',
            'Product Lookup'[ProductPrice] > [Overall Average Price]
        )
    )
  ```

- YTD Revenue
  ```
  YTD Revenue = 
    CALCULATE(
        [Total Revenue],
        DATESYTD(
            'Calendar Lookup'[Date]
        )
    )
  ```

- Previews Month Orders
  ```
  Previews Month Orders = 
    CALCULATE(
        [Total Orders],
        DATEADD(
            'Calendar Lookup'[Date],
            -1,
            MONTH
        )
    )
  ```

- Previous Month Revenue
  ```
  Previous Month Revenue = 
    CALCULATE(
        [Total Revenue],
        DATEADD(
            'Calendar Lookup'[Date],
            -1,
            MONTH
        )
    )
  ```

- Previews Month Profit
  ```
  Previews Month Profit = 
    CALCULATE(
        [Total Profit],
        DATEADD(
            'Calendar Lookup'[Date],
            -1,
            MONTH
        )
    )
  ```

- Previews Month Returns
  ```
  Previews Month Returns = 
    CALCULATE(
        [Total Returns],
        DATEADD(
            'Calendar Lookup'[Date],
            -1,
            MONTH
        )
    )
  ```

- Total Customers
  ```
  Total customers = 
    DISTINCTCOUNT(
        'Sales Data'[CustomerKey]
    )
  ```
- Full Name (Customer Detail)
  ```
  Full Name (Customer Detail) = 
    IF(
        HASONEVALUE(
            'Customer Lookup'[CustomerKey]
        ),
        MAX(
            'Customer Lookup'[FullName]
        ),
        "Multiple Customers"
    )
  ```

- Total Revenue (customer Detail)
  ```
  Total Revenue (Customer Detail) = 
    IF(
        HASONEVALUE(
            'Customer Lookup'[CustomerKey]
        ),
        [Total Revenue],
        "-"
    )
  ```

- Average Revenue per Customer
  ```
  Average Revenue per Customer = 
    DIVIDE(
        [Total Revenue],
        [Total customers]
    )
  ```

- Total Orders (Customer Detail)
  ```
  Total Orders (Customer Detail) = 
    IF(
        HASONEVALUE(
            'Customer Lookup'[CustomerKey]
        ),
        [Total Orders],
        "-"
    )
  ```

## Analysis

#### Revenue
- With a total of $ 24.9M the company revenue is increasing over time, with the best result in jun/2022 ($ 1.83M) even with a small drop in sales of -0.88% compared with the previous month;
- 'Bikes' categorie represents the highest slice of the company revenue with $ 23.6M and best-seller product 'Mountain-200 Black';
- USA leads the market with $ 7.9M representing 32% of total revenue.

![image](https://github.com/PatrickNAquino/AdventureWorks/assets/118391206/701b63c9-a661-458a-ada5-f63f9ef7bd3c)


#### Orders
- The United States is the country with the highest number of sales, representing 34.5% of company market share;
- The category 'Accessories' has 17K products selled representing 44.82% of sales. In this category the best seller product is 'Water Bottle - 30 oz' with 3983 orders;

![image](https://github.com/PatrickNAquino/AdventureWorks/assets/118391206/19d3d48f-ac6e-4aa1-ae6a-eda8ab3ba66c)

#### Returns
- Started trending up since jul-2021, resuting in a 2075% increase.
- Accessories had the highest returns accounted for 61.64%, being 'Watter Bottle - 30oz' the product most returned (149 returns - 1.9% return rate)
- USA also leads product returns with a return rate of 2.1% (624 items)

![image](https://github.com/PatrickNAquino/AdventureWorks/assets/118391206/73296b8c-82b7-46ea-8bda-7179081fff74)

#### Clients
- The number of customers remains almost constant since jan-2020 and started to grow in jul-2021.
- The company has 17K customers and an average revenue per customer of $ 1.43K.
- The majority of customers risides in USA with 41.54% (7.2K clients) while Canada has the lowest number 1499;
- Revenue per Customer trended down, resulting in a 72.48% decrease between January-2020 and June-2022.
- Mr. Maurice Shan is the top customer by revenue contributing with $ 12.4K;

![image](https://github.com/PatrickNAquino/AdventureWorks/assets/118391206/26e3c3f2-f1c2-4e7a-8d1b-4ac4deebe0be)

