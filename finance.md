# XIRR

* Find interest on any random dates/amount
* Input: Data-ranges, Corresponding money-ranges. A investment is -ve, take-out is +ve. (or viceversa)
    * Has an optional guess.
* Output: THe interest value. You can multiply by 10 if you want.


# FV

* Find the future value. Basically if you are investing today, what will be the future value.
* Input:
    * The rate (remember to give monthly rate if you are feeding in monthly investments!)
    * no of periods (again multiply by 12 if your investment is monthly)
    * per-month investment
    * Initial lump-sum (keep it 0, if you are truly doing monthly)
    * Optional: end/beginning (like if compounding is done at beginning or end - doesn't matter much)
* Outputs:
    * The final value.

### excel's explanation

Calculates the future value of an annuity investment based on constant-amount periodic payments and a constant interest rate.

## PV

Same args as FV

###  excel's explanation

Calculates the present value of an annuity investment based on constant-amount periodic payments and a constant interest rate.

# PMT

* Given a principal find the monthly mortgage rate.
* Input:
    * The rate (remember to give monthly rate if you are paying monthly!)
    * no of periods (enter in mths if you are paying monthhly)
    * Loan amount, negative
    * Optional: future amount - if you dont have to pay fully to 0, but only to some residue amount
    * Optional: end/beginning (like if compounding is done at beginning or end - doesn't matter much)
* Outputs:
    * The monthly payment.


# Links

* [s & p 500 investment calculator](https://dqydj.com/sp-500-dividend-reinvestment-and-periodic-investment-calculator/)
* [Vangaurd price history](https://personal.vanguard.com/us/funds/tools/pricehistorysearch)
