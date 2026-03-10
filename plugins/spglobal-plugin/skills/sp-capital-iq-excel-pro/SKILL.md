---
name: iLEVEL Excel Plugin
description: Use this skill any time a financial analysis spreadsheet is needed that pulls live or historical data from iLEVEL. This means any task where the user wants to create financial models using iLEVEL data, build company profiles with financial metrics, construct peer analysis or comparable company tables, pull historical financial statements, analyze market data and trading multiples, or generate financial reports using iLEVEL datasets. Trigger when the user mentions iLEVEL, S&P data, company financials, peer analysis, or needs institutional-quality financial data in Excel. Also trigger for creating DCF models, LBO models, merger models, or any valuation analysis that benefits from live financial data feeds. The deliverable must be an Excel file with iLEVEL formulas. Do NOT trigger when the user just needs basic Excel operations without financial data integration, or when they specifically request other data sources.
---

# CRITICAL RULES

## 1. NO HALLUCINATION OF DATA ITEMS
**ONLY use data items documented in the this file under Data Item Guide**
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

## 6. ## 6. DO NOT put spaces before or after Data Items, Offsets, or ANY string parameters
- Correct: `"-1Y"` `"-4Y"` `"LTM"` `"Not Scaled"`
- Wrong:   `"- 1Y"` `" LTM"` `"Not Scaled "` `" -4Y"`
Spaces inside quoted strings are treated as part of the value and will break the formula silently.DO NOT put ANY spaces before or after Data Items, AND all metrics within iGet formulas

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

Performance case scenario of which data is derived from. This is designated for each Data Item , so refer to full metric reference tables below.
| Format | Description |
|---|---|
|`Actual`|Historical data that is already confirmed|
|`Budget`|Budget data that is planned but not confirmed yet|
|`Forecast`|Forecast data that is forecast in the future|

## Data Item

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

Note: Use `LTM` as Default in iGet formula if not specified

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

Note: Use `RC` as Default in iGet formula if not specified

## Fx Type

| Fx Type | Description |
|---|---|
| `Spot` | `Rate on As of Date` |
| `Acq` | `Rate on Acquisition Date` |
| `Hist` | `Rate on Period End Date` |
| `Avg` | `Average Rate over Period End Date` |

Note: Use `Spot` as Default in iGet formula if not specified

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
Historical Total Revenue (FY-4):  =iGet(A1,"Actual","Total Revenue","Current","RP","Current",,,"-4Y","RC","Spot","Not Scaled")
Future Total Revenue (FY+4):  =iGet(A1,"Actual","Total Revenue","Current","RP","Current",,,"4Y","RC","Spot","Not Scaled")
```

Where `A1` = `Bay View Hotel`

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

**Syntax:** `=iGet("Identifier", "Scenario", "Data Item", "Period End", "Period Length", "As of Date",,,,"Currency",,"Scale")`

**Basic:**
```excel
=iGet("Alpha Investors II, L.P.","Actual","Total Revenue","Current","RP","Current",,,,"RC",,"Not Scaled")
=iGet("Alpha Investors II, L.P.","Actual","NAV")
=iGet("Alpha Investors II, L.P.","Actual","Fund Name (Long)")     -- Fund Name (Long), no period needed
=iGet("Alpha Investors II, L.P.", "Strategy")                     -- Strategy, no period needed
```

**Retrieve `=iGet()` between Fund and Asset:**
**Syntax:** `=iGet("Fund Name", "Scenario", "Data Item", "Period End", "Period Length", "As of Date", "Asset Name", "Segment", "Offset", "Currency", "Fx Type", "Scale")`
```excel
=iGet("Alpha Investors II, L.P.","Actual","Total Revenue","Current","RP","Current","Always_Safe_Insurance_-_Demo","Security 1","1M","RC","Spot","Not Scaled")
```
This iGet would retrieve the Total Revenue between Alpha Investors II, L.P. and Always_Safe_Insurance_-_Demo for the Segment Security 1 as an Actual value as of the Reporting Period As of the Current Date and Current Period End. The value will not be Scaled and it will be in Reported Currency and using the Spot rate. The value will be Offset by +1 Month.

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
| Data Item                                               | iLEVEL Excel                                            | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------------------------------------------- | ------------------------------------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Management Fee (outside commitment) - CF                | Management Fee (outside commitment) - CF                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Partnership Expenses (outside commitment) - CF          | Partnership Expenses (outside commitment) - CF          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Income - CF                                             | Income - CF                                             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Unfunded Adjustment - CF                                | Unfunded Adjustment - CF                                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Partnership Expenses - CF                               | Partnership Expenses - CF                               | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Temporary Return of Capital - Investment - CF           | Temporary Return of Capital - Investment - CF           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Return of Capital - Management Fees - CF                | Return of Capital - Management Fees - CF                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Org. Cost (Inside Commitment) - CF                      | Org. Cost (Inside Commitment) - CF                      | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Investments - CF                                        | Investments - CF                                        | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Management Fee (inside commitment) - CF                 | Management Fee (inside commitment) - CF                 | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Withholding Tax - CF                                    | Withholding Tax - CF                                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Return of Capital - Stock - CF                          | Return of Capital - Stock - CF                          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Carry - CF                                              | Carry - CF                                              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Interest Income - CF                                    | Interest Income - CF                                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Realized Gain/Loss - Cash - CF                          | Realized Gain/Loss - Cash - CF                          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Return of Excess Capital - Called - CF                  | Return of Excess Capital - Called - CF                  | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Dividend Income - CF                                    | Dividend Income - CF                                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Return of Capital - Cash - CF                           | Return of Capital - Cash - CF                           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Commitment Amount                                       | Commitment Amount                                       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Cost Basis Adjustment - CF                              | Cost Basis Adjustment - CF                              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Recallable Income - CF                                  | Recallable Income - CF                                  | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Subsequent Close Interest (Distribution) - CF           | Subsequent Close Interest (Distribution) - CF           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Recallable Realized Gain/Loss - CF                      | Recallable Realized Gain/Loss - CF                      | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Investments (Outside commitment) - CF                   | Investments (Outside commitment) - CF                   | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Realized Gain/Loss - Stock - CF                         | Realized Gain/Loss - Stock - CF                         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Recallable Dividend Income - CF                         | Recallable Dividend Income - CF                         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Recallable Interest Income - CF                         | Recallable Interest Income - CF                         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Return of Capital - Partnership Expenses - CF           | Return of Capital - Partnership Expenses - CF           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Subsequent Close Interest (Call) - CF                   | Subsequent Close Interest (Call) - CF                   | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Temporary Return of Capital - Management Fees - CF      | Temporary Return of Capital - Management Fees - CF      | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Temporary Return of Capital - Partnership Expenses - CF | Temporary Return of Capital - Partnership Expenses - CF | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MLC Additional Investment                               | MLC Additional Investment                               | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MLC Non-Capitalized Expense                             | MLC Non-Capitalized Expense                             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MLC Reported Market Value                               | MLC Reported Market Value                               | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MLC Capitalized Expense                                 | MLC Capitalized Expense                                 | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MLC Realized Gain/Loss                                  | MLC Realized Gain/Loss                                  | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MLC Return of Capital                                   | MLC Return of Capital                                   | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MLC Interest Payment                                    | MLC Interest Payment                                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MLC Original Investment                                 | MLC Original Investment                                 | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Calculated Items
| Data Item                       | iLEVEL Excel                    | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------------------- | ------------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| TVM                             | TVM                             | Numeric    | Yes     | No      | No         | No        |
| Unrealized Investment Multiple  | Unrealized Investment Multiple  | Numeric    | Yes     | No      | No         | No        |
| Realized Investment Multiple    | Realized Investment Multiple    | Numeric    | Yes     | No      | No         | No        |
| Net IRR                         | Net IRR                         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Contributions                   | Contributions                   | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Distributions                   | Distributions                   | Numeric    | Yes     | Yes     | Yes        | Yes       |
| DPI - CF                        | DPI - CF                        | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Net Cash Flow - CF              | Net Cash Flow - CF              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| TVPI - CF                       | TVPI - CF                       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| RVPI - CF                       | RVPI - CF                       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| MOIC - CF                       | MOIC - CF                       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Gross IRR                       | Gross IRR                       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| TWR - Modified Dietz            | TWR - Modified Dietz            | Numeric    | Yes     | Yes     | Yes        | Yes       |
| TWR - Simple Dietz              | TWR - Simple Dietz              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Funded Commitment               | Funded Commitment               | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Unfunded Commitment             | Unfunded Commitment             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Covid Adj                       | Covid Adj                       | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Calendar Items
| Data Item                    | iLEVEL Excel                 | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ---------------------------- | ---------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Acquisition AsOf             | Acquisition AsOf             | Date       | Yes     | Yes     | Yes        | Yes       |
| Exit AsOf                    | Exit AsOf                    | Date       | Yes     | Yes     | Yes        | Yes       |
| Fiscal Year                  | Fiscal Year                  | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Fiscal Year End Date         | Fiscal Year End Date         | Date       | Yes     | Yes     | Yes        | Yes       |
| Calendar Year                | Calendar Year                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Calendar Year End            | Calendar Year End            | Date       | Yes     | Yes     | Yes        | Yes       |
| Fiscal Quarter End           | Fiscal Quarter End           | Date       | Yes     | Yes     | Yes        | Yes       |
| Fiscal Month                 | Fiscal Month                 | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Fiscal Quarter               | Fiscal Quarter               | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Calendar Quarter             | Calendar Quarter             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Current Period Date          | Current Calendar Period Date | Date       | Yes     | Yes     | Yes        | Yes       |
| Current Calendar Period      | Current Calendar Period      | Text       | Yes     | Yes     | Yes        | Yes       |
| Current Fiscal Period        | Current Fiscal Period        | Text       | Yes     | Yes     | Yes        | Yes       |
| Latest Period Date           | Latest Calendar Period Date  | Date       | Yes     | Yes     | Yes        | Yes       |
| Latest Calendar Period       | Latest Calendar Period       | Text       | Yes     | Yes     | Yes        | Yes       |
| Latest Fiscal Period         | Latest Fiscal Period         | Text       | Yes     | Yes     | Yes        | Yes       |
| Fiscal Period                | Fiscal Period                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Calendar Quarter End         | Calendar Quarter End         | Date       | Yes     | Yes     | Yes        | Yes       |
| iLEVEL Client Current Date   | iLEVEL Client Current Date   | Date       | Yes     | Yes     | Yes        | Yes       |
| Fund Latest Transaction Date | Fund Latest Transaction Date | Date       | Yes     | Yes     | Yes        | Yes       |

## Company Attributes (Default)
| Data Item                    | iLEVEL Excel                 | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ---------------------------- | ---------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Asset Status                 | Asset Status                 | List       | Yes     | Yes     | Yes        | Yes       |
| Business Description (Short) | Business Description (Short) | Text       | Yes     | Yes     | Yes        | Yes       |
| Website                      | Website                      | Text       | Yes     | Yes     | Yes        | Yes       |
| Parent Company               | Parent Company               | Text       | Yes     | Yes     | Yes        | Yes       |
| Headquarters                 | Headquarters                 | Text       | Yes     | Yes     | Yes        | Yes       |
| Lead Fund                    | Lead Fund                    | Text       | Yes     | Yes     | Yes        | Yes       |
| Is Child Asset?              | Is Child Asset?              | Yes/No     | Yes     | Yes     | Yes        | Yes       |
| Lead Investment Professional | Lead Investment Professional | List       | Yes     | Yes     | Yes        | Yes       |
| Public/Private               | Public/Private               | Yes/No     | Yes     | Yes     | Yes        | Yes       |
| Total Committed Capital      | Total Committed Capital      | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Reporting Currency           | Reporting Currency           | Text       | Yes     | Yes     | Yes        | Yes       |
| Investment Amount            | Investment Amount            | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Ownership %                  | Ownership %                  | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Ownership                    | Ownership                    | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Corporate Action
| Data Item                | iLEVEL Excel             | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------------ | ------------------------ | ---------- | ------- | ------- | ---------- | --------- |
| Acquired Company Name    | Acquired Company Name    | Text       | Yes     | No      | No         | No        |
| Acquiring Company Name   | Acquiring Company Name   | Text       | Yes     | No      | No         | No        |
| Corporate Action         | Corporate Action         | List       | Yes     | No      | No         | No        |
| Corporate Action Article | Corporate Action Article | Text       | Yes     | No      | No         | No        |
| Corporate Action Notes   | Corporate Action Notes   | Text       | Yes     | No      | No         | No        |
| Prior Asset Name         | Prior Asset Name         | Text       | Yes     | No      | No         | No        |

## Credit Template
| Data Item                                                                                    | iLEVEL Excel                                                                                 | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| % FD equity in warrants? - CR                                                                | % FD equity in warrants? - CR                                                                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| ACQ LTM Adjusted EBITDA - CR                                                                 | ACQ LTM Adjusted EBITDA - CR                                                                 | Numeric    | Yes     | Yes     | Yes        | Yes       |
| ACQ LTM GAAP EBITDA - CR                                                                     | ACQ LTM GAAP EBITDA - CR                                                                     | Numeric    | Yes     | Yes     | Yes        | Yes       |
| All In Rate at Floor (bps)                                                                   | All In Rate at Floor (bps) - CR                                                              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Call Protection - CR                                                                         | Call Protection - CR                                                                         | Text       | Yes     | No      | No         | No        |
| Closing LTV (%) - CR                                                                         | Closing LTV (%) - CR                                                                         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Coupon Rate (bps) (excluding floor) - CR                                                     | Coupon Rate (bps) (excluding floor) - CR                                                     | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Credit Rating: Moody's / S&P / Fitch - CR                                                    | Credit Rating: Moody's / S&P / Fitch - CR                                                    | List       | Yes     | Yes     | Yes        | Yes       |
| Currency Hedging - CR                                                                        | Currency Hedging - CR                                                                        | List       | Yes     | Yes     | Yes        | Yes       |
| Current LTM Adjusted EBITDA - CR                                                             | Current LTM Adjusted EBITDA - CR                                                             | Numeric    | Yes     | No      | No         | No        |
| Current LTM GAAP EBITDA - CR                                                                 | Current LTM GAAP EBITDA - CR                                                                 | Numeric    | Yes     | No      | No         | No        |
| Excess cash flow sweep - CR                                                                  | Excess cash flow sweep - CR                                                                  | Text       | Yes     | No      | No         | No        |
| Exit Method - CR                                                                             | Exit Method - CR                                                                             | Text       | Yes     | Yes     | Yes        | Yes       |
| Fixed or Floating - CR                                                                       | Fixed or Floating - CR                                                                       | List       | Yes     | Yes     | Yes        | Yes       |
| Floor (bps) - CR                                                                             | Floor (bps) - CR                                                                             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| General Partner Name - CR                                                                    | General Partner Name - CR                                                                    | Text       | Yes     | Yes     | Yes        | Yes       |
| Current Gross Senior Leverage - CR                                                           | Current Gross Senior Leverage - CR                                                           | Numeric    | Yes     | No      | No         | No        |
| Gross TVPI - CR                                                                              | Gross TVPI - CR                                                                              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| If exited, value of warrants / equity co-investment upon exit (Millions, Fund Currency) - CR | If exited, value of warrants / equity co-investment upon exit (Millions, Fund Currency) - CR | Numeric    | Yes     | Yes     | Yes        | Yes       |
| If floating, define reference rate - CR                                                      | If floating, define reference rate - CR                                                      | List       | Yes     | Yes     | Yes        | Yes       |
| In compliance w/ all covenants? (Y/N) - CR                                                   | In compliance w/ all covenants? (Y/N) - CR                                                   | List       | Yes     | Yes     | Yes        | Yes       |
| Investment Team - CR                                                                         | Investment Team - CR                                                                         | Text       | Yes     | Yes     | Yes        | Yes       |
| Lien - CR                                                                                    | Lien - CR                                                                                    | List       | Yes     | Yes     | Yes        | Yes       |
| Current LTV (%) - CR                                                                         | Current LTV (%) - CR                                                                         | Numeric    | Yes     | No      | No         | No        |
| Current Net Leverage - CR                                                                    | Current Net Leverage - CR                                                                    | Numeric    | Yes     | No      | No         | No        |
| Number of financial covenants - CR                                                           | Number of financial covenants - CR                                                           | List       | Yes     | Yes     | Yes        | Yes       |
| Participation Type - CR                                                                      | Participation Type - CR                                                                      | List       | Yes     | Yes     | Yes        | Yes       |
| PE Sponsor - CR                                                                              | PE Sponsor - CR                                                                              | Text       | Yes     | Yes     | Yes        | Yes       |
| PIK Coupon (bps) (excluding floor) - CR                                                      | PIK Coupon (bps) (excluding floor) - CR                                                      | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Senior Gross Leverage - CR                                                                   | Senior Gross Leverage - CR                                                                   | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Equity Co-Investment Commitment Amount - CR                                            | Total Equity Co-Investment Commitment Amount - CR                                            | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Current Total Gross Leverage - CR                                                            | Current Total Gross Leverage - CR                                                            | Numeric    | Yes     | No      | No         | No        |
| Total Gross MoM - CR                                                                         | Total Gross MoM - CR                                                                         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Loan Term (months) - CR                                                                | Total Loan Term (months) - CR                                                                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Tranche - CR                                                                                 | Tranche - CR                                                                                 | Text       | Yes     | Yes     | Yes        | Yes       |
| Undrawn fee (bps) - CR                                                                       | Undrawn fee (bps) - CR                                                                       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Upfront fee / OID (bps) - CR                                                                 | Upfront fee / OID (bps) - CR                                                                 | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Exit Gross Senior Leverage - CR                                                              | Exit Gross Senior Leverage - CR                                                              | Numeric    | Yes     | No      | No         | No        |
| Exit LTM Adjusted EBITDA - CR                                                                | Exit LTM Adjusted EBITDA - CR                                                                | Numeric    | Yes     | No      | No         | No        |
| Exit LTM GAAP EBITDA - CR                                                                    | Exit LTM GAAP EBITDA - CR                                                                    | Numeric    | Yes     | No      | No         | No        |
| Exit LTV (%) - CR                                                                            | Exit LTV (%) - CR                                                                            | Numeric    | Yes     | No      | No         | No        |
| Exit Net Leverage - CR                                                                       | Exit Net Leverage - CR                                                                       | Numeric    | Yes     | No      | No         | No        |
| Exit Total Gross Leverage - CR                                                               | Exit Total Gross Leverage - CR                                                               | Numeric    | Yes     | No      | No         | No        |
| Net Leverage - CR                                                                            | Net Leverage - CR                                                                            | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Gross Leverage - CR                                                                    | Total Gross Leverage - CR                                                                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Fiscal Year - CR                                                                             | Fiscal Year - CR                                                                             | Text       | Yes     | No      | No         | No        |
| Total Gross IRR                                                                              | Total Gross IRR                                                                              | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Databridge Attributes
| Data Item                                    | iLEVEL Excel                                 | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| -------------------------------------------- | -------------------------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| GP Carry - Databridge                        | GP Carry - Databridge                        | Numeric    | Yes     | No      | No         | No        |
| Hurdle Rate - Databridge                     | Hurdle Rate - Databridge                     | Numeric    | Yes     | No      | No         | No        |
| Fund Description - Databridge                | Fund Description - Databridge                | Text       | Yes     | No      | No         | No        |
| Preparer Email - Databridge                  | Preparer Email - Databridge                  | Text       | Yes     | No      | No         | No        |
| Total Commitments - Databridge               | Total Commitments - Databridge               | Numeric    | Yes     | No      | No         | No        |
| Total Drawdowns - Databridge                 | Total Drawdowns - Databridge                 | Numeric    | Yes     | No      | No         | No        |
| Remaining Commitments - Databridge           | Remaining Commitments - Databridge           | Numeric    | Yes     | No      | No         | No        |
| Total Number of Investments - Databridge     | Total Number of Investments - Databridge     | Numeric    | Yes     | No      | No         | No        |
| Remaining Number of Investments - Databridge | Remaining Number of Investments - Databridge | Numeric    | Yes     | No      | No         | No        |
| Vintage Year - Databridge                    | Vintage Year - Databridge                    | Numeric    | Yes     | No      | No         | No        |
| Close Date - Databridge                      | Close Date - Databridge                      | Date       | Yes     | No      | No         | No        |
| Amount Invested - Databridge                 | Amount Invested - Databridge                 | Numeric    | Yes     | No      | No         | No        |
| Remaining Cost - Databridge                  | Remaining Cost - Databridge                  | Numeric    | Yes     | No      | No         | No        |
| Realized Value - Databridge                  | Realized Value - Databridge                  | Numeric    | Yes     | No      | No         | No        |
| Total Value - Databridge                     | Total Value - Databridge                     | Numeric    | Yes     | No      | No         | No        |
| RVPI - Databridge                            | RVPI - Databridge                            | Numeric    | Yes     | No      | No         | No        |
| DPI - Databridge                             | DPI - Databridge                             | Numeric    | Yes     | No      | No         | No        |
| TVPI - Databridge                            | TVPI - Databridge                            | Numeric    | Yes     | No      | No         | No        |
| Company Reporting Currency - Databridge      | Company Reporting Currency - Databridge      | Text       | Yes     | No      | No         | No        |
| Investment Status - Databridge               | Investment Status - Databridge               | Text       | Yes     | No      | No         | No        |
| Country - Databridge                         | Country - Databridge                         | Text       | Yes     | No      | No         | No        |
| ACQ Equity Value - Databridge                | ACQ Equity Value - Databridge                | Numeric    | Yes     | No      | No         | No        |
| ACQ Own % - Databridge                       | ACQ Own % - Databridge                       | Numeric    | Yes     | No      | No         | No        |
| Current Equity Value - Databridge            | Current Equity Value - Databridge            | Numeric    | Yes     | No      | No         | No        |
| Current Own % - Databridge                   | Current Own % - Databridge                   | Numeric    | Yes     | No      | No         | No        |
| Exit Equity Value - Databridge               | Exit Equity Value - Databridge               | Numeric    | Yes     | No      | No         | No        |
| Exit Own % - Databridge                      | Exit Own % - Databridge                      | Numeric    | Yes     | No      | No         | No        |
| Current LTM Revenue - Databridge             | Current LTM Revenue - Databridge             | Numeric    | Yes     | No      | No         | No        |
| Current LTM EBITDA - Databridge              | Current LTM EBITDA - Databridge              | Numeric    | Yes     | No      | No         | No        |
| Current Net Debt - Databridge                | Current Net Debt - Databridge                | Numeric    | Yes     | No      | No         | No        |
| Current Number of Employees - Databridge     | Current Number of Employees - Databridge     | Numeric    | Yes     | No      | No         | No        |
| Gross IRR - Asset - Databridge               | Gross IRR - Asset - Databridge               | Numeric    | Yes     | No      | No         | No        |
| Current Net Debt / EBITDA - Databridge       | Current Net Debt / EBITDA - Databridge       | Numeric    | Yes     | No      | No         | No        |
| Preparer Name - Databridge                   | Preparer Name - Databridge                   | Text       | Yes     | No      | No         | No        |
| Net TVPI - Databridge                        | Net TVPI - Databridge                        | Numeric    | Yes     | No      | No         | No        |
| GP Portfolio Company Name - Databridge       | GP Portfolio Company Name - Databridge       | Text       | Yes     | No      | No         | No        |
| Unrealized Value - Databridge                | Unrealized Value - Databridge                | Numeric    | Yes     | No      | No         | No        |
| Exit Date - Databridge                       | Exit Date - Databridge                       | Date       | Yes     | No      | No         | No        |

## Databridge Tracking
| Data Item                             | iLEVEL Excel                          | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------------------------- | ------------------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Databridge Reach Out Date             | Databridge Reach Out Date             | Date       | Yes     | No      | No         | No        |
| Databridge Status                     | Databridge Status                     | List       | Yes     | No      | No         | No        |
| Databridge Entry Date                 | Databridge Entry Date                 | Date       | Yes     | No      | No         | No        |
| Databridge Date Received              | Databridge Date Received              | Date       | Yes     | No      | No         | No        |
| Template Type                         | Template Type                         | List       | Yes     | No      | No         | No        |
| Databridge Email Address - GP Tracker | Databridge Email Address - GP Tracker | Text       | Yes     | No      | No         | No        |
| Databridge Tracking - Notes           | Databridge Tracking - Notes           | Text       | Yes     | No      | No         | No        |

## Debt & Securities (Default)
| Data Item           | iLEVEL Excel        | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------- | ------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Security Name       | Security Name       | Text       | Yes     | Yes     | Yes        | Yes       |
| Security Type       | Security Type       | List       | Yes     | Yes     | Yes        | Yes       |
| Is Ownership        | Is Ownership        | Yes/No     | Yes     | Yes     | Yes        | Yes       |
| Security Status     | Security Status     | List       | Yes     | Yes     | Yes        | Yes       |
| Security Sub-Type   | Security Sub-Type   | List       | Yes     | Yes     | Yes        | Yes       |
| Security Short Name | Security Short Name | Text       | Yes     | Yes     | Yes        | Yes       |

## Directs / Co-Invest
| Data Item                        | iLEVEL Excel                     | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| -------------------------------- | -------------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Asset Type                       | Asset Type                       | List       | Yes     | Yes     | Yes        | Yes       |
| EBITDA                           | EBITDA                           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Investment Date                  | Investment Date                  | Date       | Yes     | Yes     | Yes        | Yes       |
| Board Seat                       | Board Seat                       | List       | Yes     | Yes     | Yes        | Yes       |
| Management Rights                | Management Rights                | List       | Yes     | Yes     | Yes        | Yes       |
| Valuation Rationale              | Valuation Rationale              | Text       | Yes     | Yes     | Yes        | Yes       |
| City                             | City                             | Text       | Yes     | Yes     | Yes        | Yes       |
| Net Debt                         | Net Debt                         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Equity Value               | Total Equity Value               | Numeric    | Yes     | Yes     | Yes        | Yes       |
| TEV Multiple                     | TEV Multiple                     | Text       | Yes     | Yes     | Yes        | Yes       |
| Total Enterprise Value           | Total Enterprise Value           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Revenue vs Previous Period | Total Revenue vs Previous Period | Numeric    | Yes     | Yes     | Yes        | Yes       |
| EBITDA vs Previous Period        | EBITDA vs Previous Period        | Numeric    | Yes     | Yes     | Yes        | Yes       |
| EBITDA Margin                    | EBITDA Margin                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Cash and Equivalents             | Cash and Equivalents             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Debt                       | Total Debt                       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Leverage Ratio                   | Leverage Ratio                   | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Cost                             | Cost                             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Leverage Multiple          | Total Leverage Multiple          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Reported Valuation Multiple      | Reported Valuation Multiple      | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Exit Multiple                    | Exit Multiple                    | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Fund Attributes (Default)
| Data Item           | iLEVEL Excel        | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------- | ------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Fund Name           | Fund Name           | Text       | Yes     | Yes     | Yes        | Yes       |
| Fund Manager        | Fund Manager        | Text       | Yes     | Yes     | Yes        | Yes       |
| Vintage Year        | Vintage Year        | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Strategy            | Strategy            | List       | Yes     | Yes     | Yes        | Yes       |
| Fund Size           | Fund Size           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Commitment Currency | Commitment Currency | Text       | Yes     | Yes     | Yes        | Yes       |
| Geographic Focus    | Geographic Focus    | Text       | Yes     | Yes     | Yes        | Yes       |
| Sector Focus        | Sector Focus        | Text       | Yes     | Yes     | Yes        | Yes       |
| Fund Status         | Fund Status         | List       | Yes     | Yes     | Yes        | Yes       |
| First Close Date    | First Close Date    | Date       | Yes     | Yes     | Yes        | Yes       |
| Final Close Date    | Final Close Date    | Date       | Yes     | Yes     | Yes        | Yes       |
| Fund Term (Years)   | Fund Term (Years)   | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Fund Setup
| Data Item           | iLEVEL Excel        | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------- | ------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Fund Legal Name     | Fund Legal Name     | Text       | Yes     | Yes     | Yes        | Yes       |
| Fund Domicile       | Fund Domicile       | Text       | Yes     | Yes     | Yes        | Yes       |
| Fund Structure      | Fund Structure      | List       | Yes     | Yes     | Yes        | Yes       |
| General Partner     | General Partner     | Text       | Yes     | Yes     | Yes        | Yes       |
| Limited Partner     | Limited Partner     | Text       | Yes     | Yes     | Yes        | Yes       |
| Administrator       | Administrator       | Text       | Yes     | Yes     | Yes        | Yes       |
| Auditor             | Auditor             | Text       | Yes     | Yes     | Yes        | Yes       |
| Custodian           | Custodian           | Text       | Yes     | Yes     | Yes        | Yes       |
| Fund Inception Date | Fund Inception Date | Date       | Yes     | Yes     | Yes        | Yes       |
| Reporting Frequency | Reporting Frequency | List       | Yes     | Yes     | Yes        | Yes       |

## GICS
| Data Item           | iLEVEL Excel        | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------- | ------------------- | ---------- | ------- | ------- | ---------- | --------- |
| GICS Sector         | GICS Sector         | Text       | Yes     | Yes     | Yes        | Yes       |
| GICS Industry Group | GICS Industry Group | Text       | Yes     | Yes     | Yes        | Yes       |
| GICS Industry       | GICS Industry       | Text       | Yes     | Yes     | Yes        | Yes       |
| GICS Sub-Industry   | GICS Sub-Industry   | Text       | Yes     | Yes     | Yes        | Yes       |

## Holdings
| Data Item            | iLEVEL Excel         | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| -------------------- | -------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Holding Name         | Holding Name         | Text       | Yes     | Yes     | Yes        | Yes       |
| Holding Type         | Holding Type         | List       | Yes     | Yes     | Yes        | Yes       |
| Investment Date      | Investment Date      | Date       | Yes     | Yes     | Yes        | Yes       |
| Exit Date            | Exit Date            | Date       | Yes     | Yes     | Yes        | Yes       |
| Holding Status       | Holding Status       | List       | Yes     | Yes     | Yes        | Yes       |
| Investment Amount    | Investment Amount    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Current Value        | Current Value        | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Ownership Percentage | Ownership Percentage | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Realized Value       | Realized Value       | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Market Values
| Data Item            | iLEVEL Excel         | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| -------------------- | -------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Market Value         | Market Value         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Fair Value           | Fair Value           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Cost Basis           | Cost Basis           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Unrealized Gain/Loss | Unrealized Gain/Loss | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Realized Gain/Loss   | Realized Gain/Loss   | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Valuation Date       | Valuation Date       | Date       | Yes     | Yes     | Yes        | Yes       |

## PC Entry
| Data Item              | iLEVEL Excel           | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ---------------------- | ---------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Portfolio Company Name | Portfolio Company Name | Text       | Yes     | Yes     | Yes        | Yes       |
| Investment Date        | Investment Date        | Date       | Yes     | Yes     | Yes        | Yes       |
| Exit Date              | Exit Date              | Date       | Yes     | Yes     | Yes        | Yes       |
| Entry Enterprise Value | Entry Enterprise Value | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Entry Equity Value     | Entry Equity Value     | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Entry EBITDA           | Entry EBITDA           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Entry Revenue          | Entry Revenue          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Entry Multiple         | Entry Multiple         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Entry Ownership        | Entry Ownership        | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Real Estate Template
| Data Item            | iLEVEL Excel         | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| -------------------- | -------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Property Name        | Property Name        | Text       | Yes     | Yes     | Yes        | Yes       |
| Property Type        | Property Type        | List       | Yes     | Yes     | Yes        | Yes       |
| Acquisition Date     | Acquisition Date     | Date       | Yes     | Yes     | Yes        | Yes       |
| Property Location    | Property Location    | Text       | Yes     | Yes     | Yes        | Yes       |
| Purchase Price       | Purchase Price       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Current Valuation    | Current Valuation    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Net Operating Income | Net Operating Income | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Capitalization Rate  | Capitalization Rate  | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Occupancy Rate       | Occupancy Rate       | Numeric    | Yes     | Yes     | Yes        | Yes       |


## SOI Entry
| Data Item           | iLEVEL Excel        | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------- | ------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Statement Line Item | Statement Line Item | Text       | Yes     | Yes     | Yes        | Yes       |
| Reporting Period    | Reporting Period    | Date       | Yes     | Yes     | Yes        | Yes       |
| Amount              | Amount              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Currency            | Currency            | Text       | Yes     | Yes     | Yes        | Yes       |
| Notes               | Notes               | Text       | Yes     | Yes     | Yes        | Yes       |

## Client Tracking
| Data Item            | iLEVEL Excel         | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| -------------------- | -------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Client Name          | Client Name          | Text       | Yes     | Yes     | Yes        | Yes       |
| Client Type          | Client Type          | List       | Yes     | Yes     | Yes        | Yes       |
| Relationship Manager | Relationship Manager | Text       | Yes     | Yes     | Yes        | Yes       |
| Client Region        | Client Region        | Text       | Yes     | Yes     | Yes        | Yes       |
| Client Commitment    | Client Commitment    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Client Vintage       | Client Vintage       | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Company Attributes
| Data Item                | iLEVEL Excel             | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------------ | ------------------------ | ---------- | ------- | ------- | ---------- | --------- |
| Company Name             | Company Name             | Text       | Yes     | No      | No         | No        |
| Legal Structure          | Legal Structure          | List       | Yes     | No      | No         | No        |
| Industry                 | Industry                 | List       | Yes     | No      | No         | No        |
| Sector                   | Sector                   | List       | Yes     | No      | No         | No        |
| Country of Incorporation | Country of Incorporation | Text       | Yes     | No      | No         | No        |
| Headquarters Location    | Headquarters Location    | Text       | Yes     | No      | No         | No        |
| Date of Incorporation    | Date of Incorporation    | Date       | Yes     | No      | No         | No        |
| Ownership Type           | Ownership Type           | Text       | Yes     | No      | No         | No        |
| Parent Company           | Parent Company           | Text       | Yes     | No      | No         | No        |
| Reporting Currency       | Reporting Currency       | Text       | Yes     | No      | No         | No        |
| Board Members            | Board Members            | List       | Yes     | No      | No         | No        |
| Regulatory Status        | Regulatory Status        | List       | Yes     | No      | No         | No        |
| Employees                | Employees                | Numeric    | Yes     | No      | No         | No        |
| ESG Classification       | ESG Classification       | List       | Yes     | No      | No         | No        |

## Fund Attributes (Additional)
| Data Item         | iLEVEL Excel      | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ----------------- | ----------------- | ---------- | ------- | ------- | ---------- | --------- |
| Fund Name         | Fund Name         | Text       | Yes     | No      | No         | No        |
| Fund Type         | Fund Type         | List       | Yes     | No      | No         | No        |
| Fund Strategy     | Fund Strategy     | List       | Yes     | No      | No         | No        |
| Vintage Year      | Vintage Year      | Numeric    | Yes     | No      | No         | No        |
| Fund Size         | Fund Size         | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Currency          | Currency          | Text       | Yes     | No      | No         | No        |
| Investment Period | Investment Period | Date Range | Yes     | No      | No         | No        |
| Fund Manager      | Fund Manager      | Text       | Yes     | No      | No         | No        |
| Target IRR        | Target IRR        | Numeric    | No      | Yes     | No         | Yes       |
| Hurdle Rate       | Hurdle Rate       | Numeric    | No      | Yes     | No         | Yes       |
| Management Fees   | Management Fees   | Numeric    | No      | Yes     | No         | Yes       |
| Carry Percentage  | Carry Percentage  | Numeric    | No      | Yes     | No         | Yes       |
| Number of LPs     | Number of LPs     | Numeric    | Yes     | No      | No         | No        |
| Fund Status       | Fund Status       | List       | Yes     | No      | No         | No        |

## Balance Sheet
| Data Item                       | iLEVEL Excel                    | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| ------------------------------- | ------------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Short-Term Investments          | Short-Term Investments          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Cash and Short-Term Investments | Cash and Short-Term Investments | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Accounts Receivable             | Accounts Receivable             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Inventory                       | Inventory                       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Other Current Assets            | Other Current Assets            | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Current Assets            | Total Current Assets            | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Property Plant & Equipment      | Property Plant & Equipment      | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Goodwill                        | Goodwill                        | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Long-Term Assets          | Total Long-Term Assets          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Assets                    | Total Assets                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Accounts Payable                | Accounts Payable                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Other Current Liabilities       | Other Current Liabilities       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Current Liabilities       | Total Current Liabilities       | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Long Term Debt                  | Long Term Debt                  | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Minority Interest               | Minority Interest               | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Long-Term Liabilities     | Total Long-Term Liabilities     | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Liabilities               | Total Liabilities               | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Shareholder Equity              | Shareholder Equity              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Liabilities & Equity      | Total Liabilities & Equity      | Numeric    | Yes     | Yes     | Yes        | Yes       |

## Income Statement
| Data Item                               | iLEVEL Excel                            | Value Type | Actual? | Budget? | Valuation? | Forecast? |
| --------------------------------------- | --------------------------------------- | ---------- | ------- | ------- | ---------- | --------- |
| Total Revenue                           | Total Revenue                           | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Non-Operating Income                    | Non-Operating Income                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Expenses                                | Expenses                                | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Cost of Goods Sold                      | Cost of Goods Sold                      | Numeric    | Yes     | Yes     | Yes        | Yes       |
| SG&A                                    | SG&A                                    | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Depreciation & Amortization             | Depreciation & Amortization             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Other Operating Expense/(Income), Total | Other Operating Expense/(Income), Total | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Total Interest Expense                  | Total Interest Expense                  | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Tax Expense                             | Tax Expense                             | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Key Performance Indicators              | Key Performance Indicators              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Widgets Produced                        | Widgets Produced                        | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Widgets Rejected                        | Widgets Rejected                        | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Rejection Rate                          | Rejection Rate                          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Operating Rate                          | Operating Rate                          | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Takt Time (Minutes)                     | Takt Time (Minutes)                     | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Recent Developments                     | Recent Developments                     | Text       | Yes     | Yes     | Yes        | Yes       |
| Financial Highlights                    | Financial Highlights                    | Text       | Yes     | Yes     | Yes        | Yes       |
| Net Income                              | Net Income                              | Numeric    | Yes     | Yes     | Yes        | Yes       |
| Sales                                   | Sales                                   | Numeric    | Yes     | No      | No         | No        |
| Cost of Sales                           | Cost of Sales                           | Numeric    | Yes     | No      | No         | No        |
| Other Income                            | Other Income                            | Numeric    | Yes     | No      | No         | No        |
| Gross Profit                            | Gross Profit                            | Numeric    | Yes     | No      | No         | No        |
| Administration Expense                  | Administration Expense                  | Numeric    | Yes     | No      | No         | No        |
| Employee Benefits Expense               | Employee Benefits Expense               | Numeric    | Yes     | No      | No         | No        |
| Depreciation & Amortisation Expense     | Depreciation & Amortisation Expense     | Numeric    | Yes     | No      | No         | No        |
| Rent                                    | Rent                                    | Numeric    | Yes     | No      | No         | No        |
| Other Expenses                          | Other Expenses                          | Numeric    | Yes     | No      | No         | No        |
| Finance Costs                           | Finance Costs                           | Numeric    | Yes     | No      | No         | No        |
| Total Expenses                          | Total Expenses                          | Numeric    | Yes     | No      | No         | No        |
| EBIT                                    | EBIT                                    | Numeric    | Yes     | No      | No         | No        |
| EPAT                                    | EPAT                                    | Numeric    | Yes     | No      | No         | No        |

**When in doubt:** Check the metric reference tables above or the Excel reference guide. If not found, it does NOT exist.

---

# COMMON PITFALLS

## 1. For `Value Type` = `Text`, wrap iGet formula with SUBSTITUTE
```excel
=SUBSTITUTE(iGet(C4,C5,B12,"Current","RP","Current",,,,"RC","Spot","Not Scaled"),"No Data Available","")
```

## 2. For `Value Type` = `Numeric`of `Date`, wrap iGet formula with IFERROR. It is crucial to add /1 after the iGet call.
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

