# Mini S&OP and inventory simulation for Online Retail in the UK

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

**Size warning:** 1,067,371 rows is more than one Excel sheet can hold (1,048,576), so the file is expected to be split across two sheets. Check after you download. Google Sheets holds about 10 million cells and this file has about 8.5 million, so do not load the raw file into Sheets. First add up units per SKU per week with SQL or Python, then load only the weekly result into Sheets or Excel.

## Method

1. **Clean.** Remove these rows:
   - cancellation invoices (or net them off, and say which was done)
   - rows with zero or negative quantity or price
   - StockCodes that are not products, such as postage, fees and manual adjustments (for example POST, M, DOT and BANK CHARGES; check the full list in the data)

   CustomerID is not needed for demand.
2. **Aggregate.** Add up units per SKU per week. A SKU is a single product code. Pick 5 to 10 SKUs that sell in most weeks and show different patterns: steady, seasonal and lumpy.
3. **Forecast.** Use the Project 5 methods on weekly data. Train on the first part. Test on the last four months of 2011, which include the Christmas peak.
   > **Limitation:** there is only one earlier Christmas to learn from, so the seasonal methods get just one cycle.
4. **Stock rule.** Every value is an assumption:
   - Lead time L = 4 weeks.
   - Target service level = 95% (the share of weeks with no stock-out; z = 1.65).
   - Review the stock weekly.
   - Safety stock = z × standard deviation of weekly forecast error × √L. This is the extra stock that covers forecast errors.
   - Order-up-to level = forecast demand over L + 1 weeks + safety stock.
5. **Simulate week by week.** Receive the orders that are due. Sell what is in stock. Record the demand that could not be met. Place a new order. Compare the simulated stock with actual weekly sales.
6. **Measure.** Count stock-out weeks, units short, weeks of cover and overstock weeks. Overstock limit (assumption): more than 8 weeks of cover.
7. **Christmas promotion scenario.** Add an assumed uplift to the November and December forecast. Compare stock-outs with and without it. The uplift is estimated from last year's Q4 compared with the months before, and is labelled as an estimate.
8. **New-product ramp-up.** Find SKUs whose first sale comes after the data starts. Average their first 12 weeks into a ramp curve. Then forecast a new SKU's first weeks from that curve, scaled to its first two weeks of sales. Check that there are enough new SKUs before committing to this step.
9. **S&OP pack.** Build a one-page S&OP pack for two or three months in a row. Include:
   - forecast against actual in units, with bias and WAPE
   - stock position against target
   - what changed since last month
   - the top three risks or decisions

## KPIs

1. **WAPE** and **bias**, as defined in Project 5.
2. **Service level** = share of weeks with no stock-out.
3. **Fill rate** = units shipped ÷ units demanded.
4. **Weeks of cover** = stock ÷ average weekly demand.
5. **Stock-out weeks** and **overstock weeks**.

## Repository structure

```
data/    raw download (not committed if too large) and weekly SKU totals
sql/     cleaning and weekly aggregation queries
docs/    assumptions, S&OP packs and notes
images/  charts and screenshots for this README
```
