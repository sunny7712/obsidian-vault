
## Introduction

~~- Oauth 2.0 vs 2.1
~- Add a point about subscription purchase in pre-requisites. Add subscription page link as well. Mention about yearly vs monthly?
~- python 3.9 version. Should we add installation steps of python as well?~
~- Feed rate limit to be added in introduction rate limits.~
- No ga code for unauthorized. wip


## Global changes
~~- Description for everything in annexures in response description. Remove values.~~
- Add a small comment describing method in all snippets.
- Add `you can refer from instrument.csv` in trading symbol description.
- Move `all prices in rupees` in the correct place which is under the response payload.
## Instruments

~~- Format of csv in the snippet is not good. Try to make look properly with good indentation etc.~~
~~- Can we make the response in two lines rather than one and scrolling horizontally.~~
~~- Add a comment around `INSTRUMENT_CSV_URL`~~ 

## Order
~~- typo in the response schema. "MODIFICATION_REQUEST" should be "MODIFICATION_REQUESTED"~~
- Change the description `Trigger price in rupees for the order` to `Trigger price in rupees for the order type LIMIT or STOP_LOSS`. Same goes for price in all descriptions.
~~- Link order status in response of create order. Also cancelled appears two times.~~
~~- Quantity of instruments to order instead of stocks.~~
~~- Add values to segment. Also add segment description.~~
- groww_order_id star coming down in `get_trades_for_order`.
~~- ISIN. full form must be in bracket.~~
~~- Remove auto download statement.~~

## Feed
- connections update
- response description update
- quantity description change

## Portfolio
~~- Remove paise in description fields.~~
~~- Remove stocks everywheere~~
~~- Add all prices in rupees note.~~
~~- segment added in position response but not in docs.~~

~~## Margin
~- the note must be below response payload.~
~- add a note that we can pass multiple in margin for fno.~
~- replace `net_margin` with `net_margin_used` in response description.~~~

## Live data
- Change the description of lower_circuit_limit and upper_circuit_limit. (ask dhwaj)
- Change the response description of ltp and ohlc
- Get ohlc heading description wrong.

~~## Historical data
- ~~add space below or in comment.~~
- ~~~add epoch second comment in response structure.~~~
- ~~update limits.~~~~






For each page
Check descriptions are same
All amounts in rupees wherever required in the right place.




Queries
- Do we remove the enum values in the response descriptions?
- Should I have to add a small comment describing the methods in each snippet?
- Historical data limits confirmation
- n small in FNO ?
- Validity confirmation
- Intro page confirmation
- should I add consume in feed? polling not working, should I include it in feed? ltp description change.
- should I add nest_asyncio hack for notebooks in docs?
