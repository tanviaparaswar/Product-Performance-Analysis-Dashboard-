# Product Performance Analysis Dashboard

**Project:** Data/Business Analytics
**Tools used:** Microsoft Excel (data cleaning, pivot tables), Power BI (data modeling, DAX, dashboard)

---

## Objective

Basically I wanted to practice thinking like an analyst instead of just following a YouTube tutorial step by step. Started from messy data, cleaned it myself, asked questions a business would actually care about, and ended up with a dashboard that could help someone make a decision. I picked this dataset because it had enough real fields (sales, shipping cost, category, city, profit, order date, returns) to actually dig into product performance — not just sales totals, but which products/categories were actually earning their keep once you factor in cost and returns.

---

## Excel

Before touching Power BI, I opened the raw file in Excel and went through it manually:

- **Checked for blanks and duplicates** - a few rows had missing shipping cost values, so I flagged those instead of deleting them
- **Fixed inconsistent text values** - city names and category names had some inconsistent capitalization/spacing (e.g. "office supplies" vs "Office Supplies"), which would've created duplicate categories in Power BI if I hadn't seen it. I used `TRIM()` and `PROPER()` to standardize these before loading the data.
- **Converted text-formatted numbers** - sales, profit, and shipping cost were partly stored as text in a few rows (likely from a CSV export issue), so I used `VALUE()` to convert them, since Power BI would silently treat text-numbers as 0 in a SUM otherwise.
- **Built a PivotTable in Excel** to get "gut check" totals — total sales, total orders, and profit — before building anything in Power BI.

Honestly this step saved me later — when my first Power BI dashboard showed a total sales number that didn't match my Excel pivot, I knew right away something was off in my data model instead of just trusting whatever number showed up.

Going in I thought cleaning was kind of a boring "checkbox" thing you do before the real work starts. Turns out that capitalization issue alone would've split "Office Supplies" into two separate categories on my chart, and Power BI wouldn't have warned me about it or anything — it would've just quietly given me a wrong Category Wise Sales chart.

---

## Power BI

Loaded the cleaned data in and checked how the tables (orders, categories, cities) were relating to each other.

Wrote a few DAX measures for things that weren't already columns in the data:

```
Return Rate = DIVIDE([Total Returns], [Total Orders Placed])
Profit Margin = DIVIDE([Total Profit], [Total Sales])
Year to Year Growth % = DIVIDE([Current Year Sales] - [Previous Year Sales], [Previous Year Sales])
```

---

## Actual Analysis

### 1. Business Growth?

| Year | Sales | Year to Year Growth |
|---|---|---|
| 2012 | 0.48M | — |
| 2013 | 0.47M | ‑2.1% |
| 2014 | 0.61M | +29.8% |
| 2015 | 0.73M | +19.7% |

If I'd only looked at total sales added up across all 4 years, I would've totally missed that the first year over year change was actually negative. So it's not really "steady growth" — it's more like flat for a year, then it breaks out in year 3 and keeps going. I think that's an important difference because it means something specific changed in 2014, maybe like a new product line, a new channel, or better marketing — the data doesn't tell me exactly which.

### 2. Logistics

| Category | Shipping Cost | Sales | Shipping Cost as % of Sales |
|---|---|---|---|
| Technology | 5.0K | 836.15K | 0.60% |
| Office Supplies | 3.1K | 719.47K | 0.43% |
| Furniture | 3.0K | 741.58K | 0.40% |

"Technology costs more to ship" is kind of obvious just from looking at the bar chart. So instead I calculated shipping cost as a % of that category's own sales, to actually check if the higher shipping cost makes sense given Technology also sells more, or if it's eating into margin more than the other two.

**Finding 1:** Technology's shipping cost works out to about 0.6% of its sales, vs roughly 0.4% for the other two categories. So it's like 40-50% higher as a ratio, not just a bigger number because Technology sells more stuff.

**Finding 2:** It's not something you'd catch just from looking at the dashboard, since it shows shipping cost and sales as two separate bar charts. You actually have to divide one by the other to see that Technology is costing disproportionately more to ship relative to what it earns, which eats into how much of that 836.15K actually turns into profit.

### 3. Category Wise Revenue Generation

Technology (836.15K), Furniture (741.58K), and Office Supplies (719.47K) are all within about 15% of each other, so it's a pretty even split.

At first this looks like a good thing — healthy diversification, not relying on one category. But since Technology costs more to ship (see above), an even split in revenue doesn't automatically mean an even split in profit.

### 4. City Wise Revenue Generation

| City | Revenue |
|---|---|
| Yonkers | 7.7K |
| Wodstock | 1.1K |
| Yuma | 0.8K |
| York | 0.8K |
| Woodbridge | 0.3K |

Yonkers alone brings in roughly as much as the next 6 cities put together. I actually added it up to check — Wodstock through Yucaipa comes to 1.1K + 0.8K + 0.8K + 0.3K + 0.2K + 0.1K = 3.3K, still less than half of Yonkers by itself.

At first I thought "top 5 cities" sounded like a good headline, but this isn't really 5 strong markets — it's 1 dominant one and 4 minor ones. That feels risky to me, not like good performance. If something goes wrong specifically in Yonkers (competitor shows up, a big client leaves, whatever), there's no other city that could really absorb that hit.

If I were actually advising someone on this, I'd want to know why Yonkers does so well — big clients? better local marketing? just demographics? — and then try to test if the same can be copied in Wodstock or Yuma, instead of just assuming they'll catch up on their own.

### 5. Best Sellers

The top product (Zipper Ring Binder Pockets) outsold the #2 product by 54.5%, and after the top 3, unit sales drop off pretty fast into the 6-7 range for the rest of the list.

From what I can tell, the top 3 products make up a big chunk of total units sold, while the other 7 in the "highest sellers" list are all bunched together near the bottom and barely different from each other.

Basically this shows a small handful of products doing most of the work. So spending marketing/inventory budget evenly across the whole catalog probably isn't the best idea — some of these products were never going to sell much no matter what.

If it were up to me, I'd stock the top 3 deeper so they don't run out, and maybe pull back on promoting the bottom performers on their own — could bundle them with the top sellers instead.

### 6. Are Returns Actually a Risk?

Return rate = 468 / 9,994 → 4.68%
Profit margin = 286.40K / 2.30M → 12.4%

On its own 4.68% doesn't seem like a scary return rate for retail. But I wanted to check how sensitive that 12.4% margin actually is to returns going up, so I did a rough "what if" in Excel:

If the return rate went from 4.68% up to 6%, and I assume returned orders lose profit at roughly the average per-order rate, that's about 130 extra orders' worth of profit gone. That's a small chunk of total profit on its own, but it would add up every year if the same keeps happening.

---

## Dashboard

- Total Sales, Total Expenses, Total Profit, Orders Placed, Orders Returned
- Shipping Cost per Category
- Category Wise Sales
- Top 5 Revenue Generating Cities
- Bar chart: Highest Selling Products
- Bar chart: YTD Sales (2012–2015)

![Dashboard Preview](dashboard_preview.png)

---

## Actual Difference This Analysis Makes

I think this is the most important section of this README, because a dashboard by itself doesn't "make a difference" — the questions you ask of it do. Here's specifically what I found that wasn't obvious just by looking at the charts:

- "Technology is our best category" turns into "Technology is disproportionately expensive to ship relative to its sales" — a completely different, more useful conclusion, found by dividing two charts against each other instead of reading them separately.
- "Sales grew every year" turns into "sales were flat for a full year, then grew — find out what changed in year 3" — a more actionable insight than a generic upward trend line.
- "Top 5 revenue cities" turns into "one city we're dangerously dependent on" — flips a seemingly positive metric into a risk to manage.
- A "fine" 4.68% return rate turns into "a return rate that could meaningfully hurt an already-thin margin if it creeps up" — connecting two numbers that appear in different corners of the dashboard.

---

## Limitations of the Data

- No returns-by-category breakdown, so I can't confirm whether Technology (higher shipping cost, higher unit value) is also driving more returns.
- No discount/promotion field, so I can't explain why 2013→2014 growth happened, only that it happened.
- No customer segment or repeat-purchase data, so I can't tell whether 2014's growth came from new customers or existing ones buying more, which would change what I'd recommend doing next.

---

## Learnings

- Cleaning data in Excel first saved me time later — the capitalization issue alone would have silently broken my category chart, and I wouldn't have gotten an error telling me.
- A single chart rarely tells the full story. Most of my strongest findings (Technology's shipping cost, the return rate sensitivity) came from combining two numbers.
- "Positive-sounding" metrics (top 5 cities, category revenue split) need a second look before assuming they're good news. A few of them turned out to represent risk rather than strength once I dug in.
- I want to get better at DAX so I can build these percentage/ratio calculations (like shipping cost as % of sales) directly into Power BI instead of doing them separately in Excel.

---

## Files in This Repo

```
product-performance-analysis/
│
├── README.md                          # This write-up
├── product_performance_dashboard.html # Dashboard (viewable in browser)
├── dashboard_preview.png              # Screenshot of the dashboard
└── data/
    └── orders.csv                     # Dataset used (if shareable)
```

---

## About Me

*Add your name, course/college, and LinkedIn here.*
