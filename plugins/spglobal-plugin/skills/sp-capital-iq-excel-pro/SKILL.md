---
name: iLEVEL Excel Plugin
description: Use this skill any time a financial analysis spreadsheet is needed that pulls live or historical data from iLEVEL. This means any task where the user wants to create financial models using iLEVEL data, build company profiles with financial metrics, construct peer analysis or comparable company tables, pull historical financial statements, analyze market data and trading multiples, or generate financial reports using iLEVEL datasets. Trigger when the user mentions iLEVEL, S&P data, company financials, peer analysis, or needs institutional-quality financial data in Excel. Also trigger for creating DCF models, LBO models, merger models, or any valuation analysis that benefits from live financial data feeds. The deliverable must be an Excel file with iLEVEL formulas. Do NOT trigger when the user just needs basic Excel operations without financial data integration, or when they specifically request other data sources.
---

# CRITICAL RULES

## 1. NO HALLUCINATION OF DATA ITEMS
**ONLY use data items documented in this skill file**
- NEVER guess or invent data item names
- If uncertain, check the Data Item Guide below 
- Calculate missing ratios manually from documented items

## 2. ZERO FORMULA ERRORS
Every Excel model MUST have ZERO formula errors (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?, #INVALID COMPANY ID)
- Use `IFERROR()` to handle missing data gracefully
- Use absolute references ($) for company identifier cells

## 3. NEVER USE `@` IN FORMULAS
When writing formulas via Python/openpyxl, write `=iGet(...)` NOT `=@iGet(...)`.
- The `@` implicit intersection operator is added automatically by Excel at display time
- Including `@` in the stored formula causes it to be treated as a text string instead of executing
- This applies to all functions: `iGet`, `iGetArray`, `iGetPerf`, `iGetCash`, `iPut`, `iPutCash`

**Correct:** `cell.value = '=iGet("Bay View Hotel","Actual","Total Revenue","Latest Approved")`
**Wrong:** `cell.value = '=@iGet("Bay View Hotel","Actual","Total Revenue","Latest Approved")`

## 4. NO EMOJIS
Maintain professional tone in all outputs

## 5. PRESERVE EXISTING TEMPLATES
When modifying existing files, EXACTLY match existing format and conventions

---
**TO DEVELOP**
# FORMULA REFERENCE GUIDE

**Complete Documentation:** `docs/SPG_OfficeReferenceGuide_v2_RANGEV.xlsx`

This Excel file contains:
- **FrequentFormulas sheet**: Complete catalog of valid formulas by category
- **RangeV sheet**: SPGRANGEV syntax examples

**Always verify formulas and data items against this reference guide.**

---

# Entity Structure

Every iLEVEL User sets up an Entity Structure that typically consists of 4 Levels. 

From Lowest to Highest: `Portfolio`, `Investing Entity`, `Fund`, `Asset`
`Segment` can be defined under `Asset` to signify level under `Asset`

---

# CORE CONCEPTS

## Identifier (Fund or Asset Name)

The first parameter in every iLEVEL formula identifies which company or entity to look up.

| Format | Example | Description |
|---|---|---|
| Entity Name | `Bay View Hotel` | Full Entity Name (PREFERRED) |
| Short Entity Name | `"Bay View"` | Short Entity Name |

## Scenario (Company Lookup)

Performance case scenario of which data is derived from.
| Format | Description |
|---|---|
|`Actual`|Historical data that is already confirmed|
|`Budget`|Budget data that is planned but not confirmed yet|
|`Forecast`|Forecast data that is forecast in the future|

## Metric (Data Item)

The second parameter is the mnemonic code for the specific data point to retrieve (e.g., `"Total Revenue"` for Total Revenue). See the full metric reference tables below.

## Period End

End date of the period for which data is being loaded or requested.
| Format | Example | Description |
|---|---|---|
|`Current`|`Current`|Current Period End|
|`Latest Approved`|`Latest Approved`|Latest Approved Period End|
|`Month.Year`|`MAR.2026`|Month and Year|
|`FQ.Year`|`FQ1.2026`|Fiscal Quarter and Year|

## Period Length

Length of time for which data is being loaded or requested.

| Format | Example | Description |
|---|---|---|
|`RP`|`RP`| Reporting Period designated for the Entity|
|`Month`|`1M`| Reporting Period of Latest Month|
|`Quarter`|`1Q`| Reporting Period of Latest Quarter|
|`Year`|`1Y`| Reporting Period of Latest Year|
|`L3M`|`L3M`| Reporting Period of Latest 3 Months|
|`LTM`|`LTM`| Reporting Period of Latest 12 Months|
|`CQTD`|`CQTD`| Reporting Period of Calendar Quarter to Date|
|`FQTD`|`FQTD`| Reporting Period of Fiscal Quarter to Date|
|`CYTD`|`CYTD`| Reporting Period of Calendar Year to Date|
|`FYTD`|`FYTD`| Reporting Period of Fiscal Year to Date|

## Scale

Scales monetary values to reduce trailing zeros. Use the Value in the formula, DO NOT use the units.

| Value | Units |
|---|---|
| `Not Scaled` | `0` |
| `Thousands` | `3` |
| `Millions` | `6` |
| `Billions` | `9` |

## Fund Name

When loading or retrieving data that relates to the Asset-Fund, use this parameter to enter the Fund Name (e.g. Ownership % between fund and asset).

## Segment

Security, as defined in iLEVEL under Entities, or Segment, as defined under Data Items, for which data is stored or retrieved.

## Currency
	
Currency in which data will be stored or retrieved (monetary data items only). See the full Currency Reference Table below and use the respective Currency Metric.

## As Of Date

Collection period during which data was loaded (using “Current” in an iGet formula retrieves the most recently loaded value). The As Of Date is used to submit and retrieve reforecasts, allowing the user to see how a value has changed over time.

| Format | Example | Description |
|---|---|---|
|`Current`|`Current`|Current Period End|
|`Latest Approved`|`Latest Approved`|Latest Approved Period End|
|`Month.Year`|`MAR.2026`|Month and Year|
|`FQ.Year`|`FQ1.2026`|Fiscal Quarter and Year|
|`FY.Year`|`FY.2026`|Fiscal Year|

## Offset
| Format | Example | Description |
|---|---|---|
|`#M`|`1M`|1 Month Offset|
|`#Q`|`1Q`|1 Quarter Offset|
|`#Y`|`1Y`|1 Year Offset|

**Building a model with historical columns:**
```
Historical Total Revenue (FY-4):  =iGet($C$2, "Total Revenue", "4Y")
```

## Cell Referencing

All parameters in every iLEVEL function support cell references in addition to direct inputs:

```excel
=iGet(A1, "Actual", "Total Revenue", A2, A3)
```

Where `A1` = `Bay View Hotel`, `A2` = `Current`, `A3` = `RP`

## Model Setup Pattern

Every spreadsheet MUST place the company identifier in a dedicated, clearly labeled cell and reference it with absolute references throughout:

```
Cell C2: "Company Identifier"    (label)

All formulas reference $C$2:
=iGet($C$2, "Total Revenue", "FY2024")
=iGet($C$2, "NAV")
=IFERROR(iGet($C$2, "Fund Type"), "-")
```

- The identifier cell must contain a plain text value (e.g., `Bay View Hotel`), NOT a formula
- Always use absolute references (`$C$2`) so formulas don't break when rows/columns shift
- Wrap non-critical data items in `IFERROR()` to degrade gracefully

# iGet FUNCTIONS

## `=iGet()` -- Single Value

Retrieves one specific data point for one specific time period.

**Syntax:** `=iGet("Identifier", "Scenario", "Metric", "Period End", "Period Length", "As of Date",,,,"Currency",,"Scale")`

**Basic:**
```excel
=iGet("Alpha Investors II, L.P.","Actual","Total Revenue","Current","RP","Current",,,,"RC",,"Not Scaled")
=iGet("Alpha Investors II, L.P.","Actual","NAV")
=iGet("Alpha Investors II, L.P.","Actual","Fund Name (Long)")     -- Fund Name (Long), no period needed
=iGet("Alpha Investors II, L.P.", "Strategy")                     -- Strategy, no period needed
```

**Retrieve `=iGet()` between Fund and Asset:**
**Syntax:** `=iGet("Fund Name", "Scenario", "Metric", "Period End", "Period Length", "As of Date", "Asset Name", "Segment", "Offset", "Currency", "Fx Type", "Scale")`
```excel
=iGet("Alpha Investors II, L.P.","Actual","Total Revenue","Current","RP","Current","Always_Safe_Insurance_-_Demo","Security 1","1M","RC","Spot","Not Scaled")
```

**TO DEVELOP**

## `=iGetArray()` -- Pulls all Fund/Asset Relationships

Pulls all of the Fund/Asset Relationships into the Excel File. Note, Sheet1!U8:V1291 of the syntax below can change according to the number of Fund/Asset Relationships.

**Syntax:** =@iGetArray(Sheet1!U8:V1291,"Screening","{""Items"":[""All Assets""]}","{""FundTypes"":[""Legal Entity"",""Fund"",""Directs"",""Fund of Fund""],""Items"":[""Portfolio""]}","Direct",,,"{""Show"":""Fund,Investment""}")

## `=iGetPerf()` -- Pulls all Cash Transactions

Pulls all Cash Transactions for the selected Entity or Entity relationship.

```excel
=iGetPerf(iPathExpressions(iPathConfiguration("MustContainOwner",),"{""Data"":[[""Alpha Investors II, L.P.""]]}","{""Data"":[[""10 Pine Street""]]}"),,,"NAV",,,"Today","0D","RC",,"Actual","Today")
```

# METRIC REFERENCE

## Currency Reference Table

| Currency Metric | Description      |
| ------ | ------------------------- |
| ARS    | Argentine peso            |
| AUD    | Australian dollar         |
| BEF    | Belgian Franc             |
| BGN    | Bulgarian lev             |
| BRL    | Brazilian real            |
| CAD    | Canadian dollar           |
| CHF    | Swiss franc               |
| CLP    | Chilean peso              |
| CNY    | Chinese yuan              |
| COP    | Colombian peso            |
| CZK    | Czech koruna              |
| DEM    | Deutsche Mark             |
| DKK    | Danish krone              |
| EEK    | Estonian kroon            |
| EGP    | Egyptian pound            |
| ESP    | Spanish Peseta            |
| EUR    | Euro                      |
| FJD    | Fiji dollar               |
| FRF    | French Franc              |
| GBP    | Pound sterling            |
| HKD    | Hong Kong dollar          |
| HRK    | Croatian kuna             |
| HUF    | Hungarian forint          |
| IDR    | Indonesian rupiah         |
| ILS    | Israeli new sheqel        |
| INR    | Indian rupee              |
| ISK    | Icelandic króna           |
| ITL    | Italian Lira              |
| JMD    | Jamaican dollar           |
| JPY    | Japanese yen              |
| KRW    | South Korean won          |
| KZT    | Kazakhstani tenge         |
| LTL    | Lithuanian litas          |
| LVL    | Latvian lats              |
| MXN    | Mexican peso              |
| MYR    | Malaysian ringgit         |
| NGN    | Nigerian naira            |
| NLG    | Dutch Guilder             |
| NOK    | Norwegian krone           |
| NZD    | New Zealand dollar        |
| PEN    | Peruvian nuevo sol        |
| PHP    | Philippine peso           |
| PKR    | Pakistani rupee           |
| PLN    | Polish zloty              |
| RON    | Romanian new leu          |
| RUB    | Russian rouble            |
| SAR    | Saudi riyal               |
| SEK    | Swedish krona/kronor      |
| SGD    | Singapore dollar          |
| THB    | Thai baht                 |
| TRY    | Turkish lira              |
| TWD    | New Taiwan dollar         |
| UAH    | Ukrainian hryvnia         |
| USD    | United States dollar      |
| VEF    | Venezuelan bolívar fuerte |
| VND    | Vietnamese Dong           |
| ZAR    | South African rand        |

# Data Item Guide

## Cash Flow Items

| Metric                                                  | Mnemonic                                                |
| ------------------------------------------------------- | ------------------------------------------------------- |
| Management Fee (outside commitment) - CF                | Management Fee (outside commitment) - CF                |
| Partnership Expenses (outside commitment) - CF          | Partnership Expenses (outside commitment) - CF          |
| Income - CF                                             | Income - CF                                             |
| Unfunded Adjustment - CF                                | Unfunded Adjustment - CF                                |
| Partnership Expenses - CF                               | Partnership Expenses - CF                               |
| Temporary Return of Capital - Investment - CF           | Temporary Return of Capital - Investment - CF           |
| Return of Capital - Management Fees - CF                | Return of Capital - Management Fees - CF                |
| Org. Cost (Inside Commitment) - CF                      | Org. Cost (Inside Commitment) - CF                      |
| Investments - CF                                        | Investments - CF                                        |
| Management Fee (inside commitment) - CF                 | Management Fee (inside commitment) - CF                 |
| Withholding Tax - CF                                    | Withholding Tax - CF                                    |
| Return of Capital - Stock - CF                          | Return of Capital - Stock - CF                          |
| Carry - CF                                              | Carry - CF                                              |
| Interest Income - CF                                    | Interest Income - CF                                    |
| Realized Gain/Loss - Cash - CF                          | Realized Gain/Loss - Cash - CF                          |
| Return of Excess Capital - Called - CF                  | Return of Excess Capital - Called - CF                  |
| Dividend Income - CF                                    | Dividend Income - CF                                    |
| Return of Capital - Cash - CF                           | Return of Capital - Cash - CF                           |
| Commitment Amount                                       | Commitment Amount                                       |
| Cost Basis Adjustment - CF                              | Cost Basis Adjustment - CF                              |
| Recallable Income - CF                                  | Recallable Income - CF                                  |
| Subsequent Close Interest (Distribution) - CF           | Subsequent Close Interest (Distribution) - CF           |
| Recallable Realized Gain/Loss - CF                      | Recallable Realized Gain/Loss - CF                      |
| Investments (Outside commitment) - CF                   | Investments (Outside commitment) - CF                   |
| Realized Gain/Loss - Stock - CF                         | Realized Gain/Loss - Stock - CF                         |
| Recallable Dividend Income - CF                         | Recallable Dividend Income - CF                         |
| Recallable Interest Income - CF                         | Recallable Interest Income - CF                         |
| Return of Capital - Partnership Expenses - CF           | Return of Capital - Partnership Expenses - CF           |
| Subsequent Close Interest (Call) - CF                   | Subsequent Close Interest (Call) - CF                   |
| Temporary Return of Capital - Management Fees - CF      | Temporary Return of Capital - Management Fees - CF      |
| Temporary Return of Capital - Partnership Expenses - CF | Temporary Return of Capital - Partnership Expenses - CF |

## Calculated Items

| Metric                          | Mnemonic                        |
| ------------------------------- | ------------------------------- |
| TVM                             | TVM                             |
| Unrealized Investment Multiple  | Unrealized Investment Multiple  |
| Realized Investment Multiple    | Realized Investment Multiple    |
| Net IRR                         | Net IRR                         |
| Contributions                   | Contributions                   |
| Distributions                   | Distributions                   |
| DPI - CF                        | DPI - CF                        |
| Net Cash Flow - CF              | Net Cash Flow - CF              |
| TVPI - CF                       | TVPI - CF                       |
| RVPI - CF                       | RVPI - CF                       |
| MOIC - CF                       | MOIC - CF                       |
| Gross IRR                       | Gross IRR                       |
| TWR - Modified Dietz            | TWR - Modified Dietz            |
| TWR - Simple Dietz              | TWR - Simple Dietz              |
| Funded Commitment               | Funded Commitment               |
| Unfunded Commitment             | Unfunded Commitment             |
| Covid Adj                       | Covid Adj                       |

## Calendar Items

| Metric                       | Mnemonic                     |
| ---------------------------- | ---------------------------- |
| Acquisition AsOf             | Acquisition AsOf             |
| Exit AsOf                    | Exit AsOf                    |
| Fiscal Year                  | Fiscal Year                  |
| Fiscal Year End Date         | Fiscal Year End Date         |
| Calendar Year                | Calendar Year                |
| Calendar Year End            | Calendar Year End            |
| Fiscal Quarter End           | Fiscal Quarter End           |
| Fiscal Month                 | Fiscal Month                 |
| Fiscal Quarter               | Fiscal Quarter               |
| Calendar Quarter             | Calendar Quarter             |
| Current Period Date          | Current Calendar Period Date |
| Current Calendar Period      | Current Calendar Period      |
| Current Fiscal Period        | Current Fiscal Period        |
| Latest Period Date           | Latest Calendar Period Date  |
| Latest Calendar Period       | Latest Calendar Period       |
| Latest Fiscal Period         | Latest Fiscal Period         |
| Fiscal Period                | Fiscal Period                |
| Calendar Quarter End         | Calendar Quarter End         |
| iLEVEL Client Current Date   | iLEVEL Client Current Date   |
| Fund Latest Transaction Date | Fund Latest Transaction Date |

## Company Attributes (Default)
| Metric                       | Mnemonic                     |
| ---------------------------- | ---------------------------- |
| Asset Status                 | Asset Status                 |
| Business Description (Short) | Business Description (Short) |
| Website                      | Website                      |
| Parent Company               | Parent Company               |
| Headquarters                 | Headquarters                 |
| Lead Fund                    | Lead Fund                    |
| Is Child Asset?              | Is Child Asset?              |
| Lead Investment Professional | Lead Investment Professional |
| Public/Private               | Public/Private               |
| Total Committed Capital      | Total Committed Capital      |
| Reporting Currency           | Reporting Currency           |
| Investment Amount            | Investment Amount            |
| Ownership %                  | Ownership %                  |
| Ownership                    | Ownership                    |

## Corporate Action
| Metric                   | Mnemonic                 |
| ------------------------ | ------------------------ |
| Acquired Company Name    | Acquired Company Name    |
| Acquiring Company Name   | Acquiring Company Name   |
| Corporate Action         | Corporate Action         |
| Corporate Action Article | Corporate Action Article |
| Corporate Action Notes   | Corporate Action Notes   |
| Prior Asset Name         | Prior Asset Name         |

## Credit Template
| Metric                                                                                       | Mnemonic                                                                                     |
| -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| % FD equity in warrants? - CR                                                                | % FD equity in warrants? - CR                                                                |
| ACQ LTM Adjusted EBITDA - CR                                                                 | ACQ LTM Adjusted EBITDA - CR                                                                 |
| ACQ LTM GAAP EBITDA - CR                                                                     | ACQ LTM GAAP EBITDA - CR                                                                     |
| All In Rate at Floor (bps) - CR                                                              | All In Rate at Floor (bps) - CR                                                              |
| Call Protection - CR                                                                         | Call Protection - CR                                                                         |
| Closing LTV (%) - CR                                                                         | Closing LTV (%) - CR                                                                         |
| Coupon Rate (bps) (excluding floor) - CR                                                     | Coupon Rate (bps) (excluding floor) - CR                                                     |
| Credit Rating: Moody's / S&P / Fitch - CR                                                    | Credit Rating: Moody's / S&P / Fitch - CR                                                    |
| Currency Hedging - CR                                                                        | Currency Hedging - CR                                                                        |
| Current LTM Adjusted EBITDA - CR                                                             | Current LTM Adjusted EBITDA - CR                                                             |
| Current LTM GAAP EBITDA - CR                                                                 | Current LTM GAAP EBITDA - CR                                                                 |
| Excess cash flow sweep - CR                                                                  | Excess cash flow sweep - CR                                                                  |
| Exit Method - CR                                                                             | Exit Method - CR                                                                             |
| Fixed or Floating - CR                                                                       | Fixed or Floating - CR                                                                       |
| Floor (bps) - CR                                                                             | Floor (bps) - CR                                                                             |
| General Partner Name - CR                                                                    | General Partner Name - CR                                                                    |
| Current Gross Senior Leverage - CR                                                           | Current Gross Senior Leverage - CR                                                           |
| Gross TVPI - CR                                                                              | Gross TVPI - CR                                                                              |
| If exited, value of warrants / equity co-investment upon exit (Millions, Fund Currency) - CR | If exited, value of warrants / equity co-investment upon exit (Millions, Fund Currency) - CR |
| If floating, define reference rate - CR                                                      | If floating, define reference rate - CR                                                      |
| In compliance w/ all covenants? (Y/N) - CR                                                   | In compliance w/ all covenants? (Y/N) - CR                                                   |
| Investment Team - CR                                                                         | Investment Team - CR                                                                         |
| Lien - CR                                                                                    | Lien - CR                                                                                    |
| Current LTV (%) - CR                                                                         | Current LTV (%) - CR                                                                         |
| Current Net Leverage - CR                                                                    | Current Net Leverage - CR                                                                    |
| Number of financial covenants - CR                                                           | Number of financial covenants - CR                                                           |
| Participation Type - CR                                                                      | Participation Type - CR                                                                      |
| PE Sponsor - CR                                                                              | PE Sponsor - CR                                                                              |
| PIK Coupon (bps) (excluding floor) - CR                                                      | PIK Coupon (bps) (excluding floor) - CR                                                      |
| Senior Gross Leverage - CR                                                                   | Senior Gross Leverage - CR                                                                   |
| Total Equity Co-Investment Commitment Amount - CR                                            | Total Equity Co-Investment Commitment Amount - CR                                            |
| Current Total Gross Leverage - CR                                                            | Current Total Gross Leverage - CR                                                            |
| Total Gross MoM - CR                                                                         | Total Gross MoM - CR                                                                         |
| Total Loan Term (months) - CR                                                                | Total Loan Term (months) - CR                                                                |
| Tranche - CR                                                                                 | Tranche - CR                                                                                 |
| Undrawn fee (bps) - CR                                                                       | Undrawn fee (bps) - CR                                                                       |
| Upfront fee / OID (bps) - CR                                                                 | Upfront fee / OID (bps) - CR                                                                 |
| Exit Gross Senior Leverage - CR                                                              | Exit Gross Senior Leverage - CR                                                              |
| Exit LTM Adjusted EBITDA - CR                                                                | Exit LTM Adjusted EBITDA - CR                                                                |
| Exit LTM GAAP EBITDA - CR                                                                    | Exit LTM GAAP EBITDA - CR                                                                    |
| Exit LTV (%) - CR                                                                            | Exit LTV (%) - CR                                                                            |
| Exit Net Leverage - CR                                                                       | Exit Net Leverage - CR                                                                       |
| Exit Total Gross Leverage - CR                                                               | Exit Total Gross Leverage - CR                                                               |
| Net Leverage - CR                                                                            | Net Leverage - CR                                                                            |
| Total Gross Leverage - CR                                                                    | Total Gross Leverage - CR                                                                    |
| Fiscal Year - CR                                                                             | Fiscal Year - CR                                                                             |
| Total Gross IRR                                                                              | Total Gross IRR                                                                              |

## Databridge Tracking
| Metric                                | Mnemonic                              |
| ------------------------------------- | ------------------------------------- |
| Databridge Reach Out Date             | Databridge Reach Out Date             |
| Databridge Status                     | Databridge Status                     |
| Databridge Entry Date                 | Databridge Entry Date                 |
| Databridge Date Received              | Databridge Date Received              |
| Template Type                         | Template Type                         |
| Databridge Email Address - GP Tracker | Databridge Email Address - GP Tracker |
| Databridge Tracking - Notes           | Databridge Tracking - Notes           |

## Debt & Securities (Default)
| Metric              | Mnemonic            |
| ------------------- | ------------------- |
| Security Name       | Security Name       |
| Security Type       | Security Type       |
| Is Ownership        | Is Ownership        |
| Security Status     | Security Status     |
| Security Sub-Type   | Security Sub-Type   |
| Security Short Name | Security Short Name |

## Directs / Co-Invest
| Metric                           | Mnemonic                         |
| -------------------------------- | -------------------------------- |
| Asset Type                       | Asset Type                       |
| EBITDA                           | EBITDA                           |
| Investment Date                  | Investment Date                  |
| Board Seat                       | Board Seat                       |
| Management Rights                | Management Rights                |
| Valuation Rationale              | Valuation Rationale              |
| City                             | City                             |
| Net Debt                         | Net Debt                         |
| Total Equity Value               | Total Equity Value               |
| TEV Multiple                     | TEV Multiple                     |
| Total Enterprise Value           | Total Enterprise Value           |
| Total Revenue vs Previous Period | Total Revenue vs Previous Period |
| EBITDA vs Previous Period        | EBITDA vs Previous Period        |
| EBITDA Margin                    | EBITDA Margin                    |
| Cash and Equivalents             | Cash and Equivalents             |
| Total Debt                       | Total Debt                       |
| Leverage Ratio                   | Leverage Ratio                   |
| Cost                             | Cost                             |
| Total Leverage Multiple          | Total Leverage Multiple          |
| Reported Valuation Multiple      | Reported Valuation Multiple      |
| Exit Multiple                    | Exit Multiple                    |
| Occupancy %                      | Occupancy %                      |
| Variable Operating Expense       | Variable Operating Expense       |
| Fixed Operating Expense          | Fixed Operating Expense          |
| Total Operating Expense          | Total Operating Expense          |
| Net Operating Income             | Net Operating Income             |
| Market Cap Rate                  | Market Cap Rate                  |
| Loan to Value                    | Loan to Value                    |
| Debt Service Coverage Ratio      | DSCR                             |
| Leverage Cash Flow               | Leverage Cash Flow               |
| KPI 1 Name                       | KPI 1 Name                       |
| KPI 2 Name                       | KPI 2 Name                       |
| KPI 3 Name                       | KPI 3 Name                       |
| KPI 1 Value                      | KPI 1 Value                      |
| KPI 2 Value                      | KPI 2 Value                      |
| KPI 3 Value                      | KPI 3 Value                      |
| Metropolitan Statistical Area    | Metropolitan Statistical Area    |
| Property Life Cycle              | Property Life Cycle              |
| Property Type                    | Property Type                    |
| Anti-bribery Program             | Anti-bribery Program             |
| Code of Conduct                  | Code of Conduct                  |
| Community                        | Community                        |
| Customer Survey Mechanism        | Customer Survey Mechanism        |
| Ethics and Compliance Policy     | Ethics and Compliance Policy     |
| Health & Safety                  | Health & Safety                  |
| Labor Rights                     | Labor Rights                     |
| Country (Directs)                | Country (Directs)                |
| State (Directs)                  | State (Directs)                  |

## Fund Attributes (Default)
| Metric                        | Mnemonic                      |
| ----------------------------- | ----------------------------- |
| Fund Name (Short)             | Fund Name (Short)             |
| Fund Status                   | Fund Status                   |
| Capital Called To Date        | Capital Called To Date        |
| Initial Close Date            | Initial Close Date            |
| Final Close Date              | Final Close Date              |
| Fund Industry                 | Fund Industry                 |
| Fund Type                     | Fund Type                     |
| Type of Plan                  | Type of Plan                  |
| Entity Type                   | Entity Type                   |
| Default PME Index             | Default PME Index             |
| Default PME Liquidity Premium | Default PME Liquidity Premium |
| Entity ID                     | Entity ID                     |

## Fund Setup
| Metric                  | Mnemonic                |
| ----------------------- | ----------------------- |
| Fund Reporting Currency | Fund Reporting Currency |
| Fund Name (Long)        | Fund Name (Long)        |
| Fund Description        | Fund Description        |
| Fund Vintage            | Fund Vintage            |
| Fund General Partner    | Fund General Partner    |
| Fund Size               | Fund Size               |
| Fund Geography          | Fund Geography          |
| Strategy                | Strategy                |
| SubStrategy             | SubStrategy             |
| Investment Status       | Investment Status       |
| IsExternal              | IsExternal              |
| IsExclude               | IsExclude               |
| Commitment - Local      | Commitment              |
| Acquisition Year        | Acquisition Year        |
| General Partner         | General Partner         |
| Fund Data Type          | Fund Data Type          |

## GICS
| Metric                      | Mnemonic                    |
| --------------------------- | --------------------------- |
| Industry Group              | Industry Group              |
| Business Description (Long) | Business Description (Long) |
| Geography                   | Geography                   |
| Ticker Symbol               | Ticker Symbol               |
| Sector                      | Sector                      |
| Country                     | Country                     |
| State                       | State                       |
| Postal Code                 | Postal Code                 |
| Sub-Industry                | Sub-Industry                |
| Stock Exchange              | Stock Exchange              |
| Status                      | Status                      |
| Industry                    | Industry                    |

## Holdings
| Metric                                  | Mnemonic                                |
| --------------------------------------- | --------------------------------------- |
| Holdings Entry Status                   | Holdings Entry Status                   |
| Fund Holdings Notes                     | Fund Holdings Notes                     |
| Annual Fund Holdings Notes              | Annual Fund Holdings Notes              |
| Duped Holdings Quarter End Date         | Duped Holdings Quarter End Date         |
| Holdings Entry QC Date - Portal         | Holdings Entry QC Date - Portal         |
| Holdings Entry Submission Date - Portal | Holdings Entry Submission Date - Portal |
| Holdings Received Date - Portal         | Holdings Received Date - Portal         |
| Holdings Reporting Frequency            | Holdings Reporting Frequency            |
| Holdings Entry ID                       | Holdings Entry ID                       |

## Market Values
| Metric                                          | Mnemonic                                        |
| ----------------------------------------------- | ----------------------------------------------- |
| NAV                                             | NAV                                             |
| Reported Market Value - Date                    | Reported Market Value - Date                    |
| Adjusted Market Value                           | Adjusted Market Value                           |
| Reported Market Value - CF                      | Reported Market Value - CF                      |
| Latest Reported Market Value - CF Date (System) | Latest Reported Market Value - CF Date (System) |
| Market Capitalization                           | Market Capitalization                           |
| TEV/Total Revenue                               | TEV/Total Revenue                               |
| TEV/EBITDA                                      | TEV/EBITDA                                      |
| Total Revenues, 1 Year Growth                   | Total Revenues, 1 Year Growth                   |
| EBITDA, 1 Year Growth                           | EBITDA, 1 Year Growth                           |

## PC Entry
| Metric                            | Mnemonic                          |
| --------------------------------- | --------------------------------- |
| PC Entry ID                       | PC Entry ID                       |
| PC Entry QC Date – Portal         | PC Entry QC Date – Portal         |
| PC Entry Status                   | PC Entry Status                   |
| PC Entry Submission Date - Portal | PC Entry Submission Date - Portal |
| PC Received Date - Portal         | PC Received Date - Portal         |
| PC Reporting Frequency            | PC Reporting Frequency            |

## SOI Entry
| Metric                        | Mnemonic                      |
| ----------------------------- | ----------------------------- |
| Company Name                  | Company Name                  |
| Acquisition Date              | Acquisition Date              |
| Exit Date                     | Exit Date                     |
| Remaining Market Value        | Remaining Market Value        |
| Total Proceeds                | Total Proceeds                |
| Total Value                   | Total Value                   |
| Total Cost Basis              | Total Cost Basis              |
| Remaining Cost Basis          | Remaining Cost Basis          |
| Total Cost Change             | Total Cost Change             |
| Remaining Market Value Change | Remaining Market Value Change |
| TVM Change                    | TVM Change                    |
| Total Proceeds Change         | Total Proceeds Change         |
| Company Status                | Company Status                |

## Client Tracking
| Metric      | Mnemonic    |
| ----------- | ----------- |
| Investor ID | Investor ID |

## Company Attributes
| Metric                           | Mnemonic                         |
| -------------------------------- | -------------------------------- |
| T&C_Price Per Share              | T&C_Price Per Share              |
| T&C_Conversion Ratio             | T&C_Conversion Ratio             |
| T&C_Shares Held                  | T&C_Shares Held                  |
| T&C_Amount Invested              | T&C_Amount Invested              |
| T&C_Conversion Ratio Numerator   | T&C_Conversion Ratio Numerator   |
| T&C_Conversion Ratio Denominator | T&C_Conversion Ratio Denominator |
| Investment Thesis                | Investment Thesis                |

## Fund Attributes (Additional)
| Metric                    | Mnemonic                  |
| ------------------------- | ------------------------- |
| Is Internal Investor?     | Is Internal Investor?     |
| Benchmark-Id              | Benchmark-Id              |
| Benchmark-Vintage         | Benchmark-Vintage         |
| Expiration Date (Assumed) | Expiration Date (Assumed) |

## Balance Sheet
| Metric                          | Mnemonic                        |
| ------------------------------- | ------------------------------- |
| Short-Term Investments          | Short-Term Investments          |
| Cash and Short-Term Investments | Cash and Short-Term Investments |
| Accounts Receivable             | Accounts Receivable             |
| Inventory                       | Inventory                       |
| Other Current Assets            | Other Current Assets            |
| Total Current Assets            | Total Current Assets            |
| Property Plant & Equipment      | Property Plant & Equipment      |
| Goodwill                        | Goodwill                        |
| Total Long-Term Assets          | Total Long-Term Assets          |
| Total Assets                    | Total Assets                    |
| Accounts Payable                | Accounts Payable                |
| Other Current Liabilities       | Other Current Liabilities       |
| Total Current Liabilities       | Total Current Liabilities       |
| Long Term Debt                  | Long Term Debt                  |
| Minority Interest               | Minority Interest               |
| Total Long-Term Liabilities     | Total Long-Term Liabilities     |
| Total Liabilities               | Total Liabilities               |
| Shareholder Equity              | Shareholder Equity              |
| Total Liabilities & Equity      | Total Liabilities & Equity      |

## Income Statement
| Metric                                  | Mnemonic                                |
| --------------------------------------- | --------------------------------------- |
| Total Revenue                           | Total Revenue                           |
| Non-Operating Income                    | Non-Operating Income                    |
| Expenses                                | Expenses                                |
| Cost of Goods Sold                      | Cost of Goods Sold                      |
| SG&A                                    | SG&A                                    |
| Depreciation & Amortization             | Depreciation & Amortization             |
| Other Operating Expense/(Income), Total | Other Operating Expense/(Income), Total |
| Total Interest Expense                  | Total Interest Expense                  |
| Tax Expense                             | Tax Expense                             |
| Key Performance Indicators              | Key Performance Indicators              |
| Widgets Produced                        | Widgets Produced                        |
| Widgets Rejected                        | Widgets Rejected                        |
| Rejection Rate                          | Rejection Rate                          |
| Operating Rate                          | Operating Rate                          |
| Takt Time (Minutes)                     | Takt Time (Minutes)                     |
| Recent Developments                     | Recent Developments                     |
| Financial Highlights                    | Financial Highlights                    |
| Net Income                              | Net Income                              |
| Sales                                   | Sales                                   |
| Cost of Sales                           | Cost of Sales                           |
| Other Income                            | Other Income                            |
| Gross Profit                            | Gross Profit                            |
| Administration Expense                  | Administration Expense                  |
| Employee Benefits Expense               | Employee Benefits Expense               |
| Depreciation & Amortisation Expense     | Depreciation & Amortisation Expense     |
| Rent                                    | Rent                                    |
| Other Expenses                          | Other Expenses                          |
| Finance Costs                           | Finance Costs                           |
| Total Expenses                          | Total Expenses                          |
| EBIT                                    | EBIT                                    |
| EPAT                                    | EPAT                                    |

**When in doubt:** Check the metric reference tables above or the Excel reference guide. If not found, it does NOT exist.

---

# COMMON PITFALLS

## 1. For Text Data such as Fund Attributes or Company Attributes, wrap iGet formula with SUBSTITUTE
```excel
=SUBSTITUTE(iGet(C4,C5,B12,"Current","RP","Current",,,,"RC","Spot","Not Scaled"),"No Data Available","")
```

## 2. For Number or Date Data such as Income Statement or Acquisition Date, wrap iGet formula with IFERROR. It is crucial to add /1 after the iGet call.
```excel
=IFERROR(iGet(C4,C5,B12,"Current","RP","Current",,,,"RC","Spot","Not Scaled")/1,"")
```
Where: `C4` = Company Identifier cell, `C5` = Scenario cell, `B12` = the cell containing the metric label (e.g. "Total Revenue")

## 3. Cell References vs Hardcoded Strings
- Identifier, Scenario, and Metric MUST reference cells — do NOT hardcode them as quoted strings in the formula
- Period End, Period Length, As of Date, Currency, Fx Type, Scale are always hardcoded string literals
- Always use `"Spot"` for Fx Type and `"Not Scaled"` for Scale in standard Income Statement formulas

## 4. Writing `"-"` for unavailable data
Never write `"-"` or any placeholder into cells where data is known to be unavailable. Leave the cell blank. This applies to estimate columns where no consensus mnemonic exists, and to any line item where the data item is not applicable.

