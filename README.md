# Order-InventoryAnalytics-PowerBI

<img width="1282" height="719" alt="image" src="https://github.com/user-attachments/assets/f998d2d7-a88b-4628-8705-458805175fdc" />
<br>
<br>
In this Power BI assignment, I handled the full data modeling and transformation workflow for fictional technology distributor. The main objective was to enhance the existing datasets with calculated columns, derive fiscal and calendar attributes, and establish proper relationships between fact and dimension tables to support accurate reporting and analytics. Below is an overview of the main tasks and techniques applied during the process.

## Data Modeling and Relationships
I began by establishing relationships between the fact tables (Orders, Inventory) and the relevant dimensional tables (Customers, Products, Fiscal Calendar, etc.). One-to-many relationships were configured wherever appropriate, with filters flowing toward the primary fact tables. This ensured a clean, scalable, and performant data model.

<img width="1219" height="666" alt="image" src="https://github.com/user-attachments/assets/4c7555ba-1b75-45c1-b86d-e3f8f85346a7" />

## Customer Table 
In the [Customers] table, I added a calculated column [Margin%]. The maximum margin was set to 30%, the minimum to 2%, and the margin decreased by 1% for every six months the customer had been with company, based on the customer_since column. This allowed for more accurate revenue and profitability calculations per customer.

```Margin% = VAR MonthsBeingCustomer = DATEDIFF(Customers[customer_since], TODAY(), MONTH)```
```VAR MarginDrop = INT(MonthsBeingCustomer / 6) ```
```VAR Margin = MAX(2, 30-MarginDrop) ```
```RETURN Margin / 100 ```

## Inventory Table
In the [Inventory] table, I created an [Inventory Value] column by multiplying the inventory quantity with each product’s resale price. This provided a clear snapshot of the total inventory value at any given time.

```Value = Inventory[qty] * RELATED( Products[resale_price])```

## Fiscal Calendar 
In the [Fiscal Calendar] table, several time-based columns were added via Power Query:

[CY] – Calendar Year, formatted as YYYY
[CYQR] – Calendar Year + Quarter, formatted as YYYYQ
[CYPD] – Calendar Period, formatted as YYYYPPP

These columns support both fiscal and calendar-based reporting.

## Orders Table
Multiple calculated columns were added to the [Orders] table to enable detailed order tracking:

[Booked] – calculated based on Order Qty, product’s resale price, and customer margin.
```Booked = Orders[Order Qty] * RELATED(Products[resale_price]) * (1 + RELATED( Customers[Margin%]))```

[Remaining_Bill] – showing the difference between the order value and what the customer has already paid.
```Remaining_Bill = Orders[Booked] - Orders[Billed]```

[Remaining_Bill%] – representing the percentage of the remaining bill.
```Remaining_Bill% = DIVIDE(Orders[Remaining_Bill],Orders[Booked],  0)```
       
[Overdue Y/N] – flag indicating whether the order delivery is overdue, based on Requested Date and Delivery Date.
```Overdue Y/N = IF(Orders[Delivery Date] > Orders[Requested Date],  "Y",  "N")```

[CustomerClassification] – classifying customers as Bronze (<40% billed), Silver (40–80% billed), or Gold (>80% billed).
```CustomerClassification = SWITCH( TRUE(),Orders[Billed%] < 0.4, "Bronze", Orders[Billed%] <= 0.8, "Silver", Orders[Billed%] > 0.8, "Gold" )```
   
Fiscal and Calendar attributes [FY], [FYQR], [FYPD], [CY], [CYQR], [CYPD] were also added via Power Query to support time-based analytics.

## Measures 

```Inventory Qty = SUM(Inventory[qty])```

```Inventory Value = SUM(Inventory[Value])```

```Inventory Value in FYQR 20234 = CALCULATE([Inventory Value], 'Fiscal Calendar'[FYQR] = 20234)```

```Order Value = SUMX(Orders, Orders[Order Qty] * RELATED(Products[resale_price]) * (1 + RELATED(Customers[Margin%])))```

```Order Value in FYPD 2023010 = CALCULATE([Order Value], 'Fiscal Calendar'[FYPD] = 2023010)```

```Orders Delivered on Requested Date = CALCULATE(COUNTROWS(Orders), Orders[Delivery Date] = Orders[Requested Date], NOT(ISBLANK(Orders[Delivery Date])))```

```Sum of Overdue Orders in 2021 = CALCULATE(COUNTROWS(Orders), Orders[Overdue Y/N] = "Y", YEAR(Orders[Order Date]) = 2021)```

```Top 10 Order Qty = SUMX(TOPN(10, FILTER(Orders, MONTH(Orders[Order Date]) = 6 && YEAR(Orders[Order Date]) = 2023), Orders[Order Qty], DESC), Orders[Order Qty])```

```Total No. of Orders for Speedboat Helix Dynamics = CALCULATE([Total Orders], Speedboat[boat] = "Helix Dynamics")```

```Total Orders = DISTINCTCOUNT(Orders[Order No.])```

```Unique Customers for Product UBDRM99A1 in Jan 2023 = CALCULATE(COUNTROWS(Orders), Orders[Product] = "UBDRM99A1", YEAR(Orders[Order Date]) = 2021, MONTH(Orders[Order Date]) = 1)```
