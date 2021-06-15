## Introduction

Mary is a very successful portfolio manager specializing in technology stocks. Because she is so successful in selecting winners, she is now trading for several clients each with their own separate account. Each client will have a different amount of money invested with Mary (E.g. investor John has $50,000 while investor Sarah has $150,000 in her account). Furthermore each client will have different risk appetite for their portfolio (for example, John likes risk and wants 70% of his portfolio to be comprised of highly volatile stocks, while risk-adverse Sarah only wants 20% of her portfolio to be allocated to high risk stocks). 

Whenever Mary buys a new stock (for example 100 shares of GOOGLE), she cannot evenly assign 50 shares to John's account and 50 shares to Sarah's account. Mary must split those shares amongst all her client's accounts in accordance with the size of their account while conforming to her client's risk preferences. Mary has been doing all the calculations in Excel spreadsheets and has asked you to automate the process by writing a Java program to implement the rules for allocating her trades. 


## What you will Implement

You will create a Java application that will compute trade allocations across multiple accounts for Mary's BUY and SELL trades. The application and all of it's code should be checked into either your own Bitbucket or Github account and must be clone-able. After compiling your code, your main class should be runnable like so:

```bash
java com.test.allocation.TradeAllocator trades.csv capital.csv holdings.csv ratios.csv  allocations.csv
```

The above command will ingest several input files [trades.csv](#trades.csv), [capital.csv](#capital.csv), [holdings.csv](#holdings.csv), [targets.csv](#targetscsv) and outputs the file: [allocations.csv](#allocations.csv). 

You will implement the requirements specified in [Allocation Logic BASIC](#allocation-logic-basic). The goal of this exercise is to showcase what you believe a well-designed Java application should like so treat this as if it were code that you would be proud to promote to production.   

Additional rules:

1. Include a README.md file in your project. Indicate whether you've chosen to implement [Allocation Logic BASIC](#allocation-logic-basic). You can explain your approach, describe known issues etc... Also include instructions on how to compile and run your project locally.
2. Your code must be compilable and runnable in Java 11. 
3. Check in your executable jar. 
4. You may use any packages found in maven central. 
5. Assume the input files (capital.csv, trades.csv, etc...) are well formed csv's and data types are consistent and correct. For example there will not be some rows that are 5 columns and some with 3, or alphanumeric text in number fields. Furthermore you can expect that "account" and "stock" to be spelled and cased consistently throughtout (i.e. you will not see "Google" spelled differently in different files, like "GOOGLE" or "Google, Inc"). However do not assume that every account will have a target allocation for a stock, nor that every account will have a position for that stock. 

## Trade Allocation Overview

This section will detail the logic to allocate stock trades across multiple client accounts. 

#### trades.csv
These are the trades Mary completed today that need to be allocated

| Stock | Type | Quantity | Price |
|------|---------|--------|-------|
| GOOGLE | BUY | 100 | $20 |
| APPLE | SELL | 20 | $10 |


#### capital.csv
The amount of money each client has invested with Mary: 

| Account | Capital |
|---------|---------|
| John | $50,000 |
| Sarah | $150,000 |
 
#### holdings.csv
These are the stocks held in each client's account, the price and the current market value **(Quantity * Price)** of each holding. 
For the sake of simplicity, the "Price" in holdings will be the same as the "Price" in the Trades file. 

| Account | Stock | Quantity | Price | Market Value |
|---------|-------|----------|-------|--------------|
| John | GOOGLE | 50 | $20 | $1,000 |
| Sarah | GOOGLE | 10 | $20 | $200 |
| John | APPLE | 25 | $10 | $250 |
| Sarah | APPLE | 50 | $10 | $500 |


#### targets.csv
Every stock that Mary can trade will have an allocation target percent assigned to the stock and client accounts.

| Stock | Account | target_percent |
|------|---------|--------|
| GOOGLE | John | 4 |
| GOOGLE | Sarah | 1 |
| APPLE | John | 1.5 |
| APPLE | Sarah | 2 |
  
The target for the stock and account represent the **MAXIMUM PERCENT** Market Value of the stock that can be held in an account. Let's take a look at John's GOOGLE holdings. He has invested [$50,000](#capital.csv) with Mary, so the maximum value of GOOGLE for John is

```
$50,000 * 4% = $2000. 
```

The MAX_SHARES John's account can hold based on the current price of $20 is :

```
$2000 / $20 = 100 shares. 
```

Since John already owns 50 shares and his MAX_SHARES is 100, he can own 50 more GOOGLE shares.

Let's do the same math for Sarah's GOOGLE holdings. 
```
// Sarah's maximum GOOGLE holdings:
$150,000 * 1% = $1,500 
// Maximum Shares for Sarah:
$1,500 / $20 = 75 shares
// How many additional shares Sarah can own of GOOGLE? 
75 - 10 = 65 shares
```
 
John can own an additional 50 shares and Sarah can own an additional 65 shares, so Mary is entitled to buy up to 115 shares of GOOGLE. If Mary bought exactly 115 shares of Google, 50 shares gets allocated to John's account and 65 gets allocated to Sarah. 
 
However, Mary has bought only 100 shares of GOOGLE not 115, so how do we split these 100 shares equitably between them? This is governed by the [allocation math](#allocation_math) described below. 
 

## Allocation MATH
Data points from the above "Trade Allocation Overview" csv tables will be repeated below for clarity. We will use Mary's BUY 100 GOOGLE shares to illustrate how we will split them across John and Sarah's accounts. 

**TARGET_MARKET_VALUE** = Target_Percent * .01 * Account Capital
```
Ex. 
John's TARGET_MARKET_VALUE  = (4 for Google) * .01 * ($50,000) = 2,000
Sarah's TARGET_MARKET_VALUE  = (1 for Google) * .01 * $150,000 = 1,500
```

**MAX_SHARES** (the maximum number of shares for that stock amd account) = TARGET_MARKET_VALUE / Stock Price
```
Ex. John 2,000 / $20 for Google = 100
```

**ALL_IN_POSITION** sum(current quantity held) +  New Trade Qty
```
Ex. John (50 Held) + Sarah (10 Held) + New Trade (100 Shares) = 160
```
**SUGGESTED_FINAL_POSITION** (what the account's holding should look like after allocation) = ( ALL_IN_POSITION) * TARGET_MARKET_VALUE ) / sum (TARGET_MARKET_VALUE)
```
Ex. John:  
Sum (TARGET_MARKET_VALUE) =  ( 2,000 (John) + 1,500 (Sarah) ) = 3,500 
160 * 2,000 (John) / 3,500 = 91.43
```

**SUGGESTED_TADE_ALLOCATION** = Suggested FINAL Position - Quantity Held
```
Ex. John 91.43 - 50 Held = 41.43
```

In one table: 

| Account | [Capital](#capital.csv) | Stock | [Quantity Held](#holdings.csv) | [Target](#targets.csv) | TARGET_MARKET_VALUE | MAX_SHARES | ALL_IN_POSITION  | SUGGESTED_FINAL_POSITION | SUGGESTED_TRADE_ALLOCATION |
|---------|---------|-------|---------------|-------| -------------|-----------------|------------| -------|----------------------|
| John | $50,000 | GOOGLE | 50 | 4 | 2,000 |  100 | 160  | 91.43 | 41.43 |
| Sarah | $150,000 | GOOGLE | 10 | 1 | 1,500 | 75 | 160 |   68.57 | 58.57 |

## Allocation Logic BASIC

#### Allocation Assignment
With the Allocation MATH complete, we can now determine the actual shares to be attributed. 

1. Starting with the smallest absolute (SUGGESTED_TRADE_ALLOCATION), we will round up or down the computed SUGGESTED_TRADE_ALLOCATION, then compute the "Remaining Shares to be Attributed":
Round (41.43) = 41 shares Allocated to John's account. 
Remaining Shares to be Allocated = Remaining Shares to be Allocated - Allocation = 100 - 41 = 59

2. Repeat Step 1 until there is only one account left to be allocated. That last account is assigned the "Remaining Shares to be Allocated" value. We do this because rounding can cause the sum of the attributions to be greater or smaller than the actual amount traded. 
Remaining Shares to be Allocated" computed in step 1. So 59 shares go to Sarah's account. 

#### allocations.csv
| Account | Stock | Quantity |
|---------|-------|-----|
| John | GOOGLE |  +41 |
| Sarah | GOOGLE |  +59 |

#### Additional rules for BASIC
BUY or SELL trades allocated across multiple accounts must conform to the following rules:  

1. Trades must be split across accounts in accordance to the Allocation MATH.
2. BUY trades INCREASE an account's holdings. In Allocations.csv QUANTITY must be POSITIVE.
3. SELL trades DECREASE an account's holdings. In Allocations.csv QUANTITY must be NEGATIVE. 
4. **ERROR Condition:** SUGGESTED_FINAL_POSITION < 0 
5. **ERROR Condition:** SUGGESTED_FINAL_POSITION > MAX_SHARES
6. **ERROR Condition:** SUGGESTED_FINAL_POSITION < Currently Held Quantity when trade is a BUY.  
7. **ERROR Condition:** SUGGESTED_FINAL_POSITION > Currently Held Quantity when trade is a SELL.  

If an **ERROR Condition** has occurred (defined in rules 4-7), set **ALL** the accounts to quantity = 0 for that trade. For example, if we are allocating the APPLE trade across John and Sarah's account and only John's account has encountered an **ERROR Condition**, the allocations.csv file would look like this:  

| Account | Stock | Quantity |
|---------|-------|-----|
| John | GOOGLE |  +41 |
| Sarah | GOOGLE |  +59 |
| John | APPLE |  0 |
| Sarah | APPLE |  0 |
