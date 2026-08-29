# DAX Measures & Glossary

All measures live in a dedicated `Key Measures` table.

## 📖 Glossary

| Abbreviation | Full Form |
|---|---|
| **OTIF** | On Time In Full |
| **OT** | On Time |
| **IF** | In Full |
| **LIFR** | Line Fill Rate |
| **VOFR** | Volume Fill Rate |
| **COCT** | Customer Order Cycle Time |
| **DID** | Delay in Delivery |
| **dim_** | Dimension table |
| **fact_** | Fact table |
| **DAX** | Data Analysis Expressions |
| **M** | Power Query Formula Language |

## 🧮 DAX Measures

### Order Counts

**Total Orders**
```dax
Total Orders = COUNTROWS(fact_aggregate_final)
```

**On Time Orders**
```dax
On Time Orders = SUM(fact_aggregate_final[on_time])
```

**In Full Orders**
```dax
In Full Orders = SUM(fact_aggregate_final[in_full])
```

**OTIF Orders**
```dax
OTIF Orders = SUM(fact_aggregate_final[otif])
```

### Order-Level Percentages

**On Time %**
```dax
On Time % = DIVIDE([On Time Orders], [Total Orders])
```

**In Full %**
```dax
In Full % = DIVIDE([In Full Orders], [Total Orders])
```

**OTIF %**
```dax
OTIF % = DIVIDE([OTIF Orders], [Total Orders])
```

### Targets

**OT Target %**
```dax
OT Target % = AVERAGE(dim_customers[ontime_target%]) / 100
```

**IF Target %**
```dax
IF Target % = AVERAGE(dim_customers[infull_target%]) / 100
```

**OTIF Target %**
```dax
OTIF Target % = AVERAGE(dim_customers[otif_target%]) / 100
```

### Gaps

**OT Gap**
```dax
OT Gap = [On Time %] - [OT Target %]
```

**IF Gap**
```dax
IF Gap = [In Full %] - [IF Target %]
```

**OTIF Gap**
```dax
OTIF Gap = [OTIF %] - [OTIF Target %]
```

### Line-Level Metrics

**Total Order Lines**
```dax
Total Order Lines = COUNTROWS(fact_order_line_final)
```

**Line Fill Rate %**
```dax
Line Fill Rate % = DIVIDE(SUM(fact_order_line_final[In Full]), [Total Order Lines])
```

**Volume Fill Rate %**
```dax
Volume Fill Rate % = DIVIDE(SUM(fact_order_line_final[delivery_qty]), SUM(fact_order_line_final[order_qty]))
```

### Cycle Time & Delay

**COCT**
```dax
COCT = 
AVERAGEX(
    fact_order_line_final,
    DATEDIFF(fact_order_line_final[order_placement_date], fact_order_line_final[actual_delivery_date], DAY)
)
```

**DID**
```dax
DID = 
AVERAGEX(
    fact_order_line_final,
    DATEDIFF(fact_order_line_final[agreed_delivery_date], fact_order_line_final[actual_delivery_date], DAY)
)
```

### Quantity Measures

**Total Quantity Ordered**
```dax
Total Quantity Ordered = SUM(fact_order_line_final[order_qty])
```

**Total Quantity Delivered**
```dax
Total Quantity Delivered = SUM(fact_order_line_final[delivery_qty])
```

**Total Quantity Undelivered**
```dax
Total Quantity Undelivered = [Total Quantity Ordered] - [Total Quantity Delivered]
```

## 🏷️ Calculated Column

**Delay Bucket** (on `fact_order_line_final`)
```dax
Delay Bucket = 
VAR DaysLate = DATEDIFF(fact_order_line_final[agreed_delivery_date], fact_order_line_final[actual_delivery_date], DAY)
RETURN
    SWITCH(
        TRUE(),
        DaysLate <= 0, "On Time",
        DaysLate = 1, "1 Day Late",
        DaysLate = 2, "2 Days Late",
        DaysLate >= 3, "3+ Days Late"
    )
```

## 🗂️ Data Model

| Table | Type | Key Fields |
|---|---|---|
| `dim_customers` | Dimension | customer_id, customer_name, city, currency, ontime_target%, infull_target%, otif_target% |
| `dim_products` | Dimension | product_id, product_name, category, price_INR, price_USD |
| `dim_date` | Dimension | date, Year, Month, Month Name, Day, Weekday Name |
| `fact_aggregate_final` | Fact (order-level) | order_id, customer_id, order_placement_date, on_time, in_full, otif |
| `fact_order_line_final` | Fact (line-level) | order_id, order_placement_date, customer_id, product_id, order_qty, agreed_delivery_date, actual_delivery_date, delivery_qty, In Full, On Time, On Time In Full |

**Relationships:** `dim_customers`, `dim_products`, and `dim_date` relate one-to-many into both fact tables via `customer_id`, `product_id` (line-level only), and `order_placement_date` → `date`.
