# Data Dictionary — Online Retail II (Raw Dataset)

Source file: `data/raw/online_retail_II.xlsx` (2 sheets, merged: 1,067,371 rows total)

| Column Name | Data Type | Meaning | Example | Missing Values | How It Will Be Used |
|---|---|---|---|---|---|
| Invoice | Text (object) | Unique identifier for a transaction. Invoices starting with 'C' are cancellations; starting with 'A' are accounting adjustments. | `536365`, `C489449` | 0 | Identify orders, detect cancellations, count number of orders per customer |
| StockCode | Text (object) | Product/item code. Some codes represent non-product administrative entries (postage, fees, manual entries) rather than real products. | `85123A`, `POST` | 0 | Identify products, count unique products purchased, filter out non-product rows |
| Description | Text (object) | Human-readable product name. | `WHITE HANGING HEART T-LIGHT HOLDER` | 4,382 | Label top-selling products in EDA charts and tables |
| Quantity | Integer | Number of units of the product in this order line. Negative values represent cancellations or returns. | `6`, `-2` | 0 | Calculate revenue (with Price), detect cancellations/invalid rows |
| InvoiceDate | Datetime | Date and time the transaction was recorded. | `2010-12-01 08:26:00` | 0 | Build date-part features (Year, Month, Day, DayOfWeek, Hour), define the feature/target time split for modeling |
| Price | Float | Unit price of the product (currency as recorded in source, GBP). Values ≤ 0 indicate free items, samples, or data errors. | `2.55` | 0 | Calculate revenue (`TotalAmount = Quantity x Price`) |
| Customer ID | Float (numeric identifier) | Unique identifier for the customer who placed the order. Missing where the retailer did not capture a customer account for the transaction. | `17850.0` | 243,007 (22.8%) | Build customer-level features and the ML target; rows with missing Customer ID are excluded from customer-level and ML analysis |
| Country | Text (object) | Country of the customer/shipping destination. | `United Kingdom`, `Germany` | 0 | Country-level revenue/customer analysis, `IsUK` model feature |

## Derived Columns Created During Cleaning and Feature Engineering

| Column Name | Created In | Meaning |
|---|---|---|
| IsCancelled | Notebook 2 | True if `Invoice` starts with 'C' |
| IsAdjustment | Notebook 2 | True if `Invoice` starts with 'A' (accounting adjustment, not a real sale) |
| IsAdminCode | Notebook 2 | True if `StockCode` is a non-product administrative code (POST, DOT, M, BANK CHARGES, AMAZONFEE, PADS, CRUK, C2, D) |
| IsInvalidQty | Notebook 2 | True if `Quantity` ≤ 0 (outside cancellations) |
| IsInvalidPrice | Notebook 2 | True if `Price` ≤ 0 |
| MissingCustomerID | Notebook 2 | True if `Customer ID` is null |
| TotalAmount | Notebook 2 | `Quantity x Price` — revenue for this order line |
| Year / Month / MonthName / Day / DayOfWeek / Hour | Notebook 2 | Extracted from `InvoiceDate` |
| TotalSpend, NumOrders, NumItemsPurchased, NumUniqueProducts, AvgOrderValue, Recency, PurchaseFrequency | Notebook 2 | Full-period customer-level aggregates, used for descriptive EDA only |
| HistTotalSpend, HistNumOrders, HistNumItems, HistNumUniqueProducts, HistAvgOrderValue, HistRecencyDays, HistLifespanDays, HistPurchaseFrequency, IsUK | Notebook 3 | Historical (feature-period-only) customer features used as ML model inputs |
| TargetPeriodSpend | Notebook 3 | Customer's spend during the target period; used only to build the label, never as a feature |
| HighValue | Notebook 3 | ML target: 1 if `TargetPeriodSpend` ≥ 75th percentile among feature-period customers, else 0 |
