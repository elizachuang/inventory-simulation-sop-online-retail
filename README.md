# (In progress) Mini S&OP and inventory simulation for Online Retail in the UK

## Business question

How much stock should a shop hold for each product? Too little means empty shelves and lost sales. Too much means cash stuck in the warehouse and a risk of discounts. Can a simple forecast and a stock rule find the middle? And what would we decide each month?

## Why it matters to the business

- **Lost sales:** a product that is out of stock earns nothing.
- **Cash:** every unit in the warehouse is money that cannot be used elsewhere.
- **One shared plan:** sales, purchasing and finance need the same numbers in the monthly meeting.

**S&OP** (Sales and Operations Planning) is the monthly meeting where sales and supply plans are matched.

- **Audience:** a supply chain or S&OP manager.
- **Skills it covers:** S&OP, stock management, promotions and launches, forecasting and forecast KPIs.
- **Prerequisite:** build [Project 5](https://github.com/elizachuang/retail-demand-forecasting) first. This project reuses its methods.

## Data sources

| Source | What it gives you | Link |
|---|---|---|
| UCI Online Retail II | 1,067,371 transaction lines from a UK online retailer (mainly gift-ware, many wholesale customers), from 1 December 2009 to 9 December 2011. Columns: InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country. Licence CC BY 4.0. One Excel file of 43.5 MB. Invoice numbers that start with C are cancellations. | [UCI dataset page](https://archive.ics.uci.edu/dataset/502/online+retail+ii) |

**What the data does not have:** stock levels, lead times, costs and a promotion calendar. So the stock part is a simulation. Every assumption goes on a visible **Assumptions** tab.

**Size warning:** the file has 1,067,371 rows and about 8.5 million cells. That is too big for one Excel sheet (1,048,576 rows). It is also too big for Google Sheets (about 10 million cells). A database handles it easily. So all the cleaning and counting is done in SQL. Only small result tables go to Tableau.

## Tools: how the pieces fit

- **Google Colab** is a free notebook in the browser. It needs no install. Notebook: Project 6 notebook in Google Colab *(link to be added)*.
- **DuckDB** is a small database that runs inside Colab. As far as I know it is free and fast on big tables. The logic is written in SQL. Python only loads the file and runs the SQL.
- **Tableau Public** is free and public. As far as I know, charts can be built in the browser, which suits a Chromebook. Tableau announced this in 2021; check that it still works before relying on it. This data is open (CC BY 4.0), so making it public is fine.
- **GitHub** stores the notebook, the `.sql` files and the README.

### Honest view on SQL for this project

- **Good fit:** cleaning, weekly totals, forecasts (window functions), forecast scores and KPIs. This is what analyst jobs ask for.
- **Harder part:** the week-by-week stock simulation. Each week depends on the last one. In SQL this needs a recursive query. It is the hardest step. Python or a spreadsheet would be easier. SQL is still possible, and it is good practice.
- **Not done in SQL:** the charts. Tableau does those.

## Method

1. **Clean.** Flag each row with one status: cancellation, zero or negative quantity or price, not a product code, or keep. Count the rows per status. This is the cleaning log.
2. **Explore.** Look at sales by month, top products and quiet weeks.
3. **Weekly demand.** Add up units per SKU per week. A SKU is one product code. Build a full grid, so weeks with no sales show as zero.
4. **Pick SKUs.** Choose 5 to 10 SKUs that sell in most weeks and show different patterns: steady, seasonal and lumpy.
5. **Forecast** with four methods, using window functions: naive, 4-week average, 12-week average and seasonal naive (52 weeks back).
6. **Choose** the method with the lowest WAPE on the validation weeks.
7. **Stock rule.** Lead time L = 4 weeks. Service level 95% (z = 1.65). Order-up-to level = forecast demand over the next 5 weeks + safety stock.
8. **Simulate** the 13 test weeks, one week at a time, with a recursive query. Run it for the chosen rule and for a simple baseline rule.
9. **Promotion scenario.** Demand rises for 4 weeks. Compare a forecast that knows about it with one that does not.
10. **Optional: new-product ramp-up.** Forecast a new SKU from the first weeks of earlier new SKUs.
11. **S&OP pack.** Build it in Tableau from the result tables.

### The three periods

| Period | Weeks | Purpose |
|---|---|---|
| Training | First full week up to the week of 28 February 2011 | Fit the methods |
| Validation | Weeks starting 7 March to 29 August 2011 (26 weeks) | Pick the method and measure the error |
| Test | Weeks starting 5 September to 28 November 2011 (13 weeks) | Run the simulation |

## KPIs

1. **WAPE** and **bias**, as defined in Project 5.
2. **Service level** = share of weeks with no stock-out.
3. **Fill rate** = units sold ÷ units demanded.
4. **Weeks of cover** = closing stock ÷ average weekly demand.
5. **Stock-out weeks** and **overstock weeks** (more than 8 weeks of cover).

## Repository structure

```
data/    raw download (not committed if too large) and small result tables
sql/     cleaning, weekly demand, forecast, simulation and KPI queries
docs/    assumptions, S&OP pack and notes
images/  charts and screenshots for this README
```
