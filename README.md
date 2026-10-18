# Wayfaie Sales Analysis (2024 vs. 2025)

An end-to-end analysis of an e-commerce home-goods retailer's sales data, covering data cleaning, revenue definition, seasonality, category performance, and return/cancellation risk. The goal is to turn raw transactional data into clear, actionable recommendations for the business.

**Tools:** Python · pandas · NumPy · Matplotlib · Seaborn · Jupyter Notebook

---

## Table of Contents

1. [Business Context](#business-context)
2. [Business Questions](#business-questions)
3. [Dataset](#dataset)
4. [Methodology: What I Did and Why](#methodology-what-i-did-and-why)
5. [Key Findings](#key-findings)
6. [Recommendations](#recommendations)
7. [Limitations and Next Steps](#limitations-and-next-steps)
8. [Repository Structure](#repository-structure)
9. [How to Run](#how-to-run)

---

## Business Context

Wayfaie sells home products (furniture, kitchen, lighting, outdoor, storage, bath, decor and more) online. Leadership wants to understand **how the business grew between 2024 and 2025, when and where revenue is earned, and which product categories carry hidden risk** through returns and cancellations. This analysis answers those questions and is structured to support an executive pitch.

## Business Questions

| # | Question | Why it matters |
|---|----------|----------------|
| 1 | How did net revenue change from 2024 to 2025? | Establishes overall growth and the baseline for planning. |
| 2 | Is there seasonality, and when are the peaks? | Drives inventory, staffing, and marketing calendar decisions. |
| 3 | Which categories generate the most revenue and which are growing fastest? | Shows where to invest and where growth is stalling. |
| 4 | Which categories have the highest Average Order Value (AOV), and what is the return risk attached? | High AOV is only valuable if the revenue is kept. |
| 5 | Which categories suffer the most cancellations and returns? | Identifies operational leakage and product/fulfilment problems. |

## Dataset

Four relational tables covering orders placed from January 2024 to December 2025.

| Table | Rows | Key columns |
|-------|------|-------------|
| `orders.csv` | 9,245 | `order_id`, `customer_id`, `order_date`, `status`, `payment_method`, `discount_code`, `shipping_fee` |
| `order_items.csv` | 15,608 | `order_id`, `product_id`, `quantity`, `unit_price`, `discount_amount` |
| `products.csv` | 90 | `product_id`, `product_name`, `category`, `list_price`, `unit_cost` |
| `customers.csv` | 3,600 | `customer_id`, `signup_date`, `state`, `acquisition_channel` |

Order `status` takes three values: `completed`, `returned`, `canceled`.

---

## Methodology: What I Did and Why

### 1. Data Audit and Cleaning

| Step | What I did | Why |
|------|-----------|-----|
| **Load and validate** | Loaded all four tables with date parsing and printed their shapes. | Confirms row counts and that dates are real datetimes before any time-based analysis. |
| **Duplicate check** | Ran `.duplicated()` on every table. Only `orders` had issues: **38 duplicated rows**. | Duplicate orders would be counted twice and inflate revenue and order counts. |
| **Inspect before fixing** | Filtered the duplicates by `order_id` and reviewed them. They were exact copies, not different orders sharing an ID. | Deleting data should be a decision based on evidence, not a blind rule. |
| **Standardize categories** | Applied `.str.strip().str.title()` to `products.category`. | Prevents the same category from splitting into several groups because of casing or stray whitespace. |
| **Impute missing unit prices** | Left-joined `products.list_price` and filled missing `order_items.unit_price` values with it. | Missing prices would silently turn into zero or NaN revenue. List price is the most defensible substitute. |
| **Compute line-item revenue** | `item_revenue = quantity × unit_price − discount_amount` | Creates one consistent, discount-aware revenue measure at the lowest grain, so it can be rolled up by any dimension. |

![chart](chart1_monthly_revenue.png)

### 2. Metric Definition

Gross sales and realized revenue are not the same thing, so I defined them explicitly:

- **Net Sales Revenue** = sum of `item_revenue` for orders with `status == 'completed'`.
- **Canceled orders** represent zero realized revenue.
- **Returned orders** are refunded, so they are excluded from revenue and tracked separately as operational leakage.

Defining the metric up front keeps every later number consistent and defensible.

### 3. Building the Analytical Dataset

Joined `order_items` → `orders` → `products` with inner joins to produce a single analysis-ready table. I then created two views:

- `df`: all orders, used for **return and cancellation rates** (needs every status in the denominator).
- `df_completed`: completed orders only, used for **revenue and AOV**.

Keeping the two separate avoids the common mistake of calculating return rates on a dataset that has already had returns filtered out.

### 4. Analysis

- **Monthly trend and seasonality:** pivoted revenue by month and year and calculated month-by-month YoY growth.
- **Category performance:** aggregated revenue and unique orders per category and derived `AOV = revenue / orders`.
- **Risk analysis:** calculated return rate and cancellation rate per category as a share of all orders in that category.

### 5. Visualization

Three charts were built for an executive audience: a YoY line chart, a category revenue comparison, and a risk matrix of AOV vs. return rate.

---

## Key Findings

### 1. Strong, consistent growth

Net revenue grew roughly **21% year over year**, from about **$848K in 2024 to about $1.03M in 2025**. Every single month of 2025 beat the same month of 2024.

![Monthly Revenue YoY](/images/chart1_monthly_revenue.png)

### 2. Clear seasonality

- **November is the annual peak** (about $100.5K in 2024 and $129.4K in 2025), consistent with Black Friday / Cyber Monday promotions.
- December is also strong in both years, which points to holiday-season demand and the `HOLIDAY20` discount code.
- Mid-year months (June to August) also run higher than spring and autumn in 2025.

### 3. Furniture and Kitchen lead revenue, Storage leads growth

- **Furniture** (~$322.8K) and **Kitchen** (~$279.7K) are the two biggest revenue drivers.
- **Storage** grew the fastest at **+40.8%** (about $58.0K to $81.6K), followed by **Kitchen** at +26.2%.

![Category Revenue](/images/chart2_category_revenue.png)

### 4. High value does not mean low risk

| Category | AOV | Return / cancel note |
|----------|-----|----------------------|
| **Furniture** | **$376.19** (highest) | **11.72% return rate**, nearly triple the site average of about 4 to 5% |
| Lighting | $208.18 | ~4.0% return rate, healthy |
| Outdoor | $199.21 | ~4.4% return rate, healthy |
| Bath | n/a | **Highest cancellation rate (4.56%)** |

Furniture earns the most per order but also loses the most revenue to returns, which makes it the biggest hidden profitability risk.

![Risk Matrix](/images/chart3_risk_matrix.pngchart3_risk_matrix.png)

---

## Recommendations

1. **Plan the calendar around November and December.** Build inventory, fulfilment capacity, and ad budget ahead of the Black Friday peak, which is growing year over year.
2. **Attack Furniture returns.** Investigate return reasons by product (damage in transit, size or colour mismatch, assembly issues). Improve packaging, add dimension guides and room-scale imagery, and consider a quality review on the worst-performing SKUs. Even a modest drop in return rate protects the highest-AOV category.
3. **Double down on Storage.** It is the fastest-growing category with normal risk. Increase assortment, run bundle offers, and cross-sell with Furniture and Kitchen.
4. **Protect margin in Lighting and Outdoor.** They combine solid AOV with low return rates. They are good candidates for promotion without the refund drag seen in Furniture.
5. **Investigate Bath cancellations.** The highest cancellation rate suggests stock availability, delivery-time, or checkout issues worth checking.
6. **Track returns as a standing KPI.** Report return and cancellation rate next to revenue so growth is always judged on revenue kept, not revenue booked.

---

## Limitations and Next Steps

- **Seasonality drivers are inferred.** Attributing the November spike to Black Friday and the summer lift to outdoor demand is a reasonable hypothesis. Confirm it by breaking revenue down by category and discount code per month.
- **Profitability is not yet modelled.** `unit_cost` is available, so the next step is gross margin by category, which would show whether Furniture's returns erase its AOV advantage.
- **Customer-level analysis.** Using `customers.csv` (acquisition channel, state, signup date) could reveal repeat-purchase behaviour, channel quality, and regional performance.
- **Discount effectiveness.** Compare discounted vs. full-price orders on volume, AOV, and return rate.

---

## Repository Structure

```
.
├── data/
│   ├── customers.csv
│   ├── order_items.csv
│   ├── orders.csv
│   └── products.csv
├── Wayfaie_Sales_Analysis.ipynb
├── images/
│   ├── chart1_monthly_revenue.png
│   ├── chart2_category_revenue.png
│   └── chart3_risk_matrix.png
└── README.md
```

## How to Run

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd <your-repo-folder>

# 2. Install dependencies
pip install pandas numpy matplotlib seaborn jupyter

# 3. Launch the notebook
jupyter notebook Wayfaie_Sales_Analysis.ipynb
```

Run the cells from top to bottom. The charts are saved automatically as PNG files in the project folder.

---

## Author

**Ahmed H. Hewidi**, Data Analyst · [Linkedin](https://www.linkedin.com/in/ahmedhewidi1710/)