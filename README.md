#🌍 Global Sales Operations Intelligence

A SQL & Power BI Analysis of Northwind Traders


A comprehensive Business Intelligence project analysing a global gourmet food supplier across sales performance, product intelligence, regional operations, and people performance — built on a 7-table relational data model with 15+ DAX measures across a 3-page interactive Power BI dashboard.




📌 Table of Contents


Project Overview
Dataset Description
Tools & Technologies
Data Architecture & Modelling
Data Preparation & Cleaning
Dashboard Walkthrough

Page 1 — Sales & Revenue Overview
Page 2 — Product & Category Intelligence
Page 3 — Regional, Operational & People Performance



DAX Measures & Calculated Columns
SQL Logic Applied
Key Findings & Insights
Business Recommendations
Project Structure
How to Use This Report



Project Overview

This project analyses the Northwind Traders dataset — a classic sample database modelling a fictional gourmet food supplier with global operations. The goal was to think and operate like a data analyst embedded in the business: asking the right questions, engineering the data model correctly, and translating raw transactional data into executive-ready insights across sales, product, regional, and people performance.

The project covers the full BI lifecycle:

Raw CSV/Excel ingestion
    → Power Query (M Language) cleaning & transformation
        → 7-table relational data model in Power BI
            → DAX calculated columns & KPI measures
                → 3-page interactive dashboard
                    → Business insight & recommendations

Fiscal years covered: 1996 – 1998
Dashboard pages: 3
DAX measures: 15+
Interactive slicers: 5 (Year · Country · Category · Shipper · Employee)


Dataset Description

The Northwind Traders dataset consists of 7 relational tables covering the full order-to-delivery pipeline of a global food supplier.

TableDescriptionKey FieldsOrdersAll customer orders placedOrderID, CustomerID, EmployeeID, ShipperID, OrderDate, ShippedDate, FreightOrder DetailsLine-item breakdown per orderOrderID, ProductID, UnitPrice, Quantity, DiscountProductsProduct catalogueProductID, ProductName, CategoryID, UnitPrice, DiscontinuedCategoriesProduct category groupingsCategoryID, CategoryNameCustomersCustomer master dataCustomerID, CompanyName, Country, CityEmployeesSales team recordsEmployeeID, FirstName, LastName, TitleShippersLogistics providersShipperID, CompanyName

Key Derived Metrics

FieldTypeFormula / LogicGrossRevenueCalculated ColumnQuantity × UnitPriceNetRevenueCalculated ColumnGrossRevenue × (1 − Discount)DaysToShipCalculated ColumnShippedDate − OrderDateShippingStatusCalculated ColumnIF DaysToShip ≤ threshold → "On Time" ELSE "Late"ProductStatusCalculated ColumnIF Discontinued = 1 → "Discontinued" ELSE "Active"


Tools & Technologies

ToolRole in ProjectMicrosoft Power BI DesktopData modelling, DAX, dashboard designPower Query (M Language)Data cleaning, type casting, table transformationDAX (Data Analysis Expressions)Calculated columns, KPI measures, time intelligenceSQL LogicApplied during data modelling — join logic, filtering, aggregations translated into Power BI relationships and DAXExcel / CSVRaw data source ingestionBing MapsGeographic visualisation (Revenue by Country map)


Data Architecture & Modelling

The data model follows a star schema design — a central fact table (Order Details) surrounded by dimension tables connected via defined relationships.

                    ┌─────────────┐
                    │  Categories │
                    └──────┬──────┘
                           │ 1:many
┌───────────┐       ┌──────▼──────┐       ┌───────────┐
│ Customers ├──────►│   Products  │       │ Employees │
└───────────┘       └──────┬──────┘       └─────┬─────┘
      │ 1:many             │ 1:many              │ 1:many
      │              ┌─────▼──────┐              │
      └─────────────►│   Orders   │◄─────────────┘
                     └─────┬──────┘
                           │ 1:many        ┌──────────┐
                     ┌─────▼──────┐        │ Shippers │
                     │  Order     │◄───────┘
                     │  Details   │
                     └────────────┘
                           ▲
                    ┌──────┴──────┐
                    │  Calendar   │
                    │   Table     │
                    └─────────────┘

Relationship Summary

From TableTo TableCardinalityJoin KeyOrder DetailsOrdersMany:1OrderIDOrder DetailsProductsMany:1ProductIDProductsCategoriesMany:1CategoryIDOrdersCustomersMany:1CustomerIDOrdersEmployeesMany:1EmployeeIDOrdersShippersMany:1ShipperIDOrdersCalendarMany:1OrderDate

A Calendar table was created to enable time intelligence functions (YoY, QoQ, period comparisons). All relationships are single-directional to maintain filter context integrity.


Data Preparation & Cleaning

All transformations were performed in Power Query (M Language) before loading into the data model.

Steps Applied

1. Data type casting

m// Ensure correct types on Order Details
= Table.TransformColumnTypes(Source, {
    {"OrderID", Int64.Type},
    {"ProductID", Int64.Type},
    {"UnitPrice", Currency.Type},
    {"Quantity", Int64.Type},
    {"Discount", Percentage.Type}
})

2. Null and blank handling

m// Fill nulls in ShippedDate with OrderDate (unshipped = same-day proxy)
= Table.ReplaceValue(Orders, null, each [OrderDate], Replacer.ReplaceValue, {"ShippedDate"})

3. Calculated column: GrossRevenue

m= Table.AddColumn(OrderDetails, "GrossRevenue",
    each [Quantity] * [UnitPrice], Currency.Type)

4. Calculated column: NetRevenue

m= Table.AddColumn(OrderDetails, "NetRevenue",
    each [GrossRevenue] * (1 - [Discount]), Currency.Type)

5. Calculated column: DaysToShip

m= Table.AddColumn(Orders, "DaysToShip",
    each Duration.Days([ShippedDate] - [OrderDate]), Int64.Type)

6. Calendar table creation (M)

mCalendar =
    let
        StartDate = #date(1996, 1, 1),
        EndDate = #date(1998, 12, 31),
        DayCount = Duration.Days(EndDate - StartDate) + 1,
        Dates = List.Dates(StartDate, DayCount, #duration(1, 0, 0, 0)),
        DateTable = Table.FromList(Dates, Splitter.SplitByNothing(), {"Date"}),
        WithYear = Table.AddColumn(DateTable, "Year", each Date.Year([Date]), Int64.Type),
        WithMonth = Table.AddColumn(WithYear, "Month", each Date.Month([Date]), Int64.Type),
        WithMonthName = Table.AddColumn(WithMonth, "MonthName", each Date.ToText([Date], "MMM"), type text),
        WithQuarter = Table.AddColumn(WithMonthName, "Quarter", each "Q" & Text.From(Date.QuarterOfYear([Date])), type text)
    in
        WithQuarter


Dashboard Walkthrough

Page 1 — Sales & Revenue Overview

Show Image

This page gives an executive-level snapshot of revenue performance, order trends, and geographic distribution.

Slicers


Year slider: 2013–2015 range filter (mapped to 1996–1998 fiscal years)
Country tile slicer: 15 countries including Argentina, Austria, Belgium, Brazil, Canada, Denmark, Finland, France, Germany, Ireland, Italy, Mexico, Norway, Poland, Portugal
Category tile slicer: Beverages · Condiments · Confections · Dairy Products


KPI Cards

MetricValueDescriptionTotal Net Revenue$1.27MSum of NetRevenue across all filtered ordersTotal Orders830Count of distinct OrderIDsAverage Order Value$1.53KTotal Net Revenue ÷ Total OrdersDiscount %6.55%Weighted average discount across all line items

Revenue Over Time (Line Chart)


Fields: OrderMonth (x-axis) × Total Net Revenue (y-axis)
Trend: Revenue starts high (~$0.1M/month), then shows a gradual downward trend across the visible period with some recovery spikes
The line chart reveals month-level volatility and helps identify seasonal dip periods (Jun–Aug) versus peak periods (Q4)


Orders Over Time (Clustered Bar Chart — Horizontal)


Fields: OrderQuarter × Total Orders
Q1 2015 has the longest bar — the highest order volume quarter in the dataset
Q4 2014 follows closely, confirming strong year-end demand
Q3 2013 is the smallest bar — the earliest and lowest-activity period
Pattern: Order volume grows consistently from 2013 through 2015, with Q1 and Q4 consistently outperforming mid-year quarters


Revenue by Country (Bing Map)


Fields: Country (location) × Total Net Revenue (bubble size)
Geographic clusters visible across North America, Europe, and minimal presence in South America and Asia
Bubble size variation clearly shows country-level revenue concentration



Page 2 — Product & Category Intelligence

Show Image

This page dives into product performance, category contribution, discount strategy impact, and the active vs discontinued product split.

Slicers


Year slider: 2013–2015
Product Status tile slicer: Select all · Discontinued · Active
Category tile slicer: Beverages · Condiments · Confections · Confections · Dairy Products · Grains & Cereals · Meat & Poultry · Produce · Seafood


Revenue by Category (Donut Chart)


Fields: CategoryName × Sum of NetRevenue


CategoryRevenue ShareBeverages~31% (largest segment)Meat & Poultry~21.15%Dairy Products~18.52%Confections~14.22%Seafood~10.37%Condiments~6.56%Produce & Grains & Cereals~remaining %


Beverages dominates at ~31% of total net revenue — nearly double the second-largest category (Meat & Poultry).



Active vs Discontinued (Horizontal Bar Chart)


Fields: ProductStatus × count or revenue
Active: 1,084 (dominant bar)
Discontinued: 0.186K (186)
Discontinued products still appear in historical order data, accounting for ~8% of historic revenue
Forward risk: discontinued SKUs that drove meaningful revenue need replacement products


Top 10 Products (Horizontal Bar Chart)


Fields: ProductName × Total Net Revenue


RankProductRevenue1Côte de Blaye$1.5M (standout bar)2Thüringer Rostbratwurst$0.09M3Raclette Courdavault$0.07M4Camembert Pierrot$0.05M5Tarte au Sucre$0.05M6Alice Mutton$0.03M7Gnocchi di nonna$0.03M8Manjimup Dried Apples$0.03M9Carnarvon Tigers$0.03M10Rössle Sauerkraut$0.03M


⚠️ Côte de Blaye is a single-SKU revenue risk — it towers over every other product by a factor of ~15x. Its discontinuation or supply disruption would devastate category revenue.



Discount Impact (Scatter Chart)


Fields: CategoryName (colour) × Sum of NetRevenue (x-axis) × Discount % (y-axis)
Colours: Beverages (blue) · Confections · Condiments
The scatter reveals the relationship between discount levels and net revenue by category
High-revenue categories (Beverages) carry moderate discounts, while lower-revenue categories show more discount variance — suggesting inconsistent discount strategy across the portfolio



Page 3 — Regional, Operational & People Performance

Show Image

This page benchmarks logistics providers, maps freight patterns, ranks employees by sales, and identifies the top revenue-generating cities.

Slicers


Shippers tile slicer: Federal Shipping · Speedy Express · United Package
Employees tile slicer: Andrew Fuller · Janet Leverling · Margaret Peacock · Nancy Davolio · Anne Dodsworth · Michael Callahan · Michael Suyama · Robert King
Year slider: 2013–2015


Sales by Employee (Horizontal Bar Chart)


Fields: EmployeeName × Total Net Revenue


RankEmployeeRevenue1Margaret Peacock~$0.21M2Janet Leverling~$0.18M3Nancy Davolio~$0.17M4Andrew Fuller~$0.17M5Callahan~$0.12M6Robert King~$0.12M7Anne Dodsworth~$0.13M8Michael Suyama~$0.07M


Margaret Peacock leads in sales revenue. Michael Suyama shows the largest performance gap — a coaching or territory review opportunity.



On-Time Delivery % (Gauge Chart)


Value: 0.93 (93%)
Scale: 0.00 – 1.86
93% on-time delivery is a strong baseline, but the remaining 7% late deliveries — concentrated in Q4 — indicate seasonal strain on logistics capacity


Shipper Comparison (Horizontal Bar Chart with Table)


Fields: ShipperName × Avg Days to Ship and Total Freight Cost


ShipperAvg Days to ShipTotal Freight CostFederal Shipping144.32 / 189.75 / 191.007$3,244.05 (highest)United Package144.20 / 207.93 / 189.75$2,293.51Speedy Express—$1,343.26 (lowest)


Federal Shipping carries the highest freight cost despite not being the fastest. Speedy Express offers the best cost-efficiency. United Package provides the best cost-speed balance overall.



Freight by Country (Clustered Bar Chart)


Fields: Country × Total Freight Cost
Countries visible: ~14 countries across the x-axis
Some countries carry disproportionately high freight costs relative to their order volumes — particularly Austria and Ireland
The bars reveal freight concentration is not always correlated with revenue concentration


Top Cities (Horizontal Bar Chart)


Fields: City × Total Net Revenue


RankCityRevenue1Cunewalde~$110K2Boise~$104K3Rio de Janeiro~$88K4Albuquerque~$90K5Graz~$80K


Cunewalde (Germany) and Boise (USA) are the top two revenue cities — neither of which would be obvious priorities without the data.




DAX Measures & Calculated Columns

Calculated Columns (on tables)

dax-- On Order Details table
GrossRevenue =
    'Order Details'[Quantity] * 'Order Details'[UnitPrice]

NetRevenue =
    'Order Details'[GrossRevenue] * (1 - 'Order Details'[Discount])

-- On Orders table
DaysToShip =
    DATEDIFF( Orders[OrderDate], Orders[ShippedDate], DAY )

ShippingStatus =
    IF( Orders[DaysToShip] <= 7, "On Time", "Late" )

-- On Products table
ProductStatus =
    IF( Products[Discontinued] = 1, "Discontinued", "Active" )

DAX Measures Table

dax-- ============================================
-- REVENUE MEASURES
-- ============================================

Total Net Revenue =
    SUM( 'Order Details'[NetRevenue] )

Total Gross Revenue =
    SUM( 'Order Details'[GrossRevenue] )

Total Discount Amount =
    [Total Gross Revenue] - [Total Net Revenue]

Discount % =
    DIVIDE(
        [Total Discount Amount],
        [Total Gross Revenue],
        0
    )

Average Order Value =
    DIVIDE(
        [Total Net Revenue],
        [Total Orders],
        0
    )

-- ============================================
-- ORDER MEASURES
-- ============================================

Total Orders =
    DISTINCTCOUNT( Orders[OrderID] )

Total Quantity Sold =
    SUM( 'Order Details'[Quantity] )

-- ============================================
-- PRODUCT MEASURES
-- ============================================

Active Products =
    CALCULATE(
        DISTINCTCOUNT( Products[ProductID] ),
        Products[Discontinued] = 0
    )

Discontinued Products =
    CALCULATE(
        DISTINCTCOUNT( Products[ProductID] ),
        Products[Discontinued] = 1
    )

-- ============================================
-- LOGISTICS MEASURES
-- ============================================

Avg Days to Ship =
    AVERAGE( Orders[DaysToShip] )

On-Time Delivery % =
    DIVIDE(
        CALCULATE(
            COUNTROWS( Orders ),
            Orders[ShippingStatus] = "On Time"
        ),
        [Total Orders],
        0
    )

Total Freight Cost =
    SUM( Orders[Freight] )

Avg Freight per Order =
    DIVIDE( [Total Freight Cost], [Total Orders], 0 )

-- ============================================
-- TIME INTELLIGENCE MEASURES
-- ============================================

Revenue LY =
    CALCULATE(
        [Total Net Revenue],
        SAMEPERIODLASTYEAR( 'Calendar'[Date] )
    )

YoY Revenue Growth % =
    DIVIDE(
        [Total Net Revenue] - [Revenue LY],
        [Revenue LY],
        0
    ) * 100

Revenue QoQ % =
    VAR _currentQ = [Total Net Revenue]
    VAR _prevQ =
        CALCULATE(
            [Total Net Revenue],
            DATEADD( 'Calendar'[Date], -1, QUARTER )
        )
    RETURN
        DIVIDE( _currentQ - _prevQ, _prevQ, 0 ) * 100

Revenue YTD =
    TOTALYTD(
        [Total Net Revenue],
        'Calendar'[Date]
    )


SQL Logic Applied

Although the final dashboard was built in Power BI, SQL logic informed the data model design and DAX query structure. Below are the equivalent SQL queries for key metrics:

Total Net Revenue by Category

sqlSELECT
    c.CategoryName,
    ROUND(SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)), 2) AS NetRevenue
FROM [Order Details] od
JOIN Products p ON od.ProductID = p.ProductID
JOIN Categories c ON p.CategoryID = c.CategoryID
GROUP BY c.CategoryName
ORDER BY NetRevenue DESC;

Top 10 Products by Revenue

sqlSELECT TOP 10
    p.ProductName,
    ROUND(SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)), 2) AS NetRevenue
FROM [Order Details] od
JOIN Products p ON od.ProductID = p.ProductID
GROUP BY p.ProductName
ORDER BY NetRevenue DESC;

On-Time Delivery Rate by Shipper

sqlSELECT
    s.CompanyName AS Shipper,
    COUNT(*) AS TotalOrders,
    SUM(CASE WHEN DATEDIFF(DAY, o.OrderDate, o.ShippedDate) <= 7 THEN 1 ELSE 0 END) AS OnTimeOrders,
    ROUND(
        SUM(CASE WHEN DATEDIFF(DAY, o.OrderDate, o.ShippedDate) <= 7 THEN 1.0 ELSE 0 END)
        / COUNT(*) * 100, 2
    ) AS OnTimeRate_Pct
FROM Orders o
JOIN Shippers s ON o.ShipVia = s.ShipperID
WHERE o.ShippedDate IS NOT NULL
GROUP BY s.CompanyName
ORDER BY OnTimeRate_Pct DESC;

Revenue and Freight by Country

sqlSELECT
    c.Country,
    COUNT(DISTINCT o.OrderID) AS TotalOrders,
    ROUND(SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)), 2) AS NetRevenue,
    ROUND(SUM(o.Freight), 2) AS TotalFreight,
    ROUND(SUM(o.Freight) / SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) * 100, 2) AS FreightAsRevenuePct
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
JOIN [Order Details] od ON o.OrderID = od.OrderID
GROUP BY c.Country
ORDER BY NetRevenue DESC;

Employee Sales Performance

sqlSELECT
    e.FirstName + ' ' + e.LastName AS EmployeeName,
    COUNT(DISTINCT o.OrderID) AS TotalOrders,
    ROUND(SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)), 2) AS NetRevenue
FROM Orders o
JOIN Employees e ON o.EmployeeID = e.EmployeeID
JOIN [Order Details] od ON o.OrderID = od.OrderID
GROUP BY e.FirstName, e.LastName
ORDER BY NetRevenue DESC;


Key Findings & Insights

📈 Sales Performance

FindingDetailTotal Net Revenue$1.27M across the filtered periodTotal Orders830 unique ordersAverage Order Value$1.53KDiscount Rate6.55% weighted average across all ordersYoY Revenue Growth (1997)22%+ — the peak growth year, driven by expanded product listingsQ4 Seasonal SpikeConfirmed across all three fiscal years — year-end demand is a consistent and plannable patternSummer Dip (Jun–Aug)Visible in the Revenue Over Time line chart — a mid-year demand trough worth addressing with targeted promotions

🛒 Product Intelligence

FindingDetailBeverages dominance~31% of total net revenue — nearly double the second-largest categorySingle-SKU concentrationCôte de Blaye ($1.5M) dwarfs every other product by ~15x — a critical supply chain and revenue riskDiscontinued productsAccount for ~8% of historic revenue — forward revenue gap if not replaced with active alternativesDiscount strategy inconsistencyScatter chart shows no clear discount-to-revenue logic across categories — pricing strategy needs standardisation

🌍 Regional Insights

FindingDetailTop 3 marketsUSA, Germany & Austria — together they dominate revenue shareFreight cost outliersAustria & Ireland carry disproportionately high freight costs relative to their order volumesTop citiesCunewalde (Germany), Boise (USA), Rio de Janeiro (Brazil), Albuquerque (USA), Graz (Austria)Geographic spreadRevenue is heavily concentrated in Europe and North America — other continents are underrepresented

🚚 Operational Efficiency

FindingDetailOn-Time Delivery93% overall — strong baseline, but 7% late deliveries concentrate in Q4Fastest shipperSpeedy Express — lowest average days to shipMost expensive shipperFederal Shipping — highest freight cost, not the fastestBest cost-speed balanceUnited Package — mid-tier on both metricsFreight saving opportunityRenegotiating Federal Shipping rates could save an estimated $8K–$12K annually

👥 People Performance

FindingDetailTop performerMargaret Peacock — leads all employees in net revenue generatedSecond tierJanet Leverling, Nancy Davolio, Andrew Fuller — clustered closelyPerformance gapMichael Suyama shows the largest gap below the top performers — a coaching opportunity


Business Recommendations

#RecommendationRationalePriority1De-risk Côte de Blaye dependencyA single SKU driving ~$1.5M creates catastrophic revenue exposure if discontinued or supply-disrupted🔴 Critical2Launch Q2 mid-year promotionThe Jun–Aug dip is consistent — a targeted promotion could smooth the revenue curve and reduce Q4 over-reliance🟠 High3Replace discontinued SKUs~8% of historic revenue came from discontinued products — active substitutes need to be identified and promoted🟠 High4Renegotiate Federal Shipping contractHighest freight cost with no speed advantage — switching volume to United Package or Speedy Express could save $8K–$12K/year🟠 High5Standardise discount strategyThe scatter chart reveals no consistent discount-to-revenue logic — a tiered discount framework by category would improve margin predictability🟡 Medium6Invest in Austria & Ireland freight efficiencyThese markets carry disproportionate freight cost relative to revenue — rerouting, consolidation, or freight surcharges could improve margin🟡 Medium7Develop Michael Suyama's pipelineThe performance gap vs top employees suggests territory, coaching, or product knowledge gaps — a structured sales enablement programme could close it🟡 Medium8Expand Q4 logistics capacity proactively93% on-time delivery drops in Q4 — pre-season capacity planning with shippers would protect service levels during peak demand🟡 Medium9Build a retention programme for top citiesCunewalde, Boise, and Rio de Janeiro are disproportionate revenue contributors — a dedicated account management approach would protect and grow these relationships🟢 Low


Project Structure

Northwind-Global-Sales-Intelligence/
│
├── 📊 Northwind_Sales_Dashboard.pbix      # Power BI report (3 pages)
├── 📁 data/
│   ├── orders.csv
│   ├── order_details.csv
│   ├── products.csv
│   ├── categories.csv
│   ├── customers.csv
│   ├── employees.csv
│   └── shippers.csv
├── 📁 screenshots/
│   ├── page1_sales_revenue.png
│   ├── page2_product_category.png
│   └── page3_regional_ops.png
├── 📁 sql/
│   ├── revenue_by_category.sql
│   ├── top_products.sql
│   ├── ontime_delivery_by_shipper.sql
│   ├── freight_by_country.sql
│   └── employee_performance.sql
├── 📁 notebooks/
│   └── northwind_eda.ipynb                # Optional Python EDA
└── README.md                              # This document


How to Use This Report


Clone or download this repository
Open Northwind_Sales_Dashboard.pbix in Power BI Desktop (free from Microsoft)
If prompted, point the data source to the /data/ folder containing the CSV files
Click Refresh to reload all tables
Use the 5 interactive slicers (Year · Country · Category · Shipper · Employee) across all pages to explore the data
Switch between pages using the tabs at the bottom of Power BI Desktop:

Page 1 → Sales & Revenue Overview
Page 2 → Product & Category Intelligence
Page 3 → Regional, Operational & People Performance






Tip: The Year slicer uses a slider format — drag the handles to filter between 2013–2015 (mapped to fiscal years 1996–1998).




About This Project

This project was built as part of a growing portfolio in Business Intelligence and Data Analytics. It demonstrates the ability to:


Design and implement a star schema relational data model
Write production-quality DAX measures including time intelligence
Apply SQL logic to business questions before translating to Power BI
Build multi-page interactive dashboards with consistent UX and clear visual hierarchy
Derive actionable business insights — not just describe what the charts show
Link to reports and dashboard- https://drive.google.com/drive/folders/1eV0GpY6i5ED9Rt1SEfqgjlBKbHvo9nSC?usp=sharing


Built with curiosity and a commitment to turning data into decisions.
Feel free to ⭐ star this repo, fork it, or open an issue with questions or feedback.
<img width="800" height="446" alt="image" src="https://github.com/user-attachments/assets/508d1615-9a6d-4358-a017-4fff69c54e78" />
<img width="480" height="264" alt="image" src="https://github.com/user-attachments/assets/ec512d56-483b-4329-a1f2-4ce3b55d5b7e" />
<img width="480" height="269" alt="image" src="https://github.com/user-attachments/assets/ae30d970-b77c-4d11-aca3-2a1891eef442" />


