# Key DAX Measures
This document outlines the core DAX logic used in the **Service Incentive Monitor** dashboard. These measures handle the calendar (date), first time fix, total bonus paid in year 2024 and the previous year, dynamic scenario planning, and security filtering.

## 1. Calendar (Date)
Created dynamically to ensure continuous dates for Time Intelligence functions.
#### Calendar Table
```dax
Dim_Calendar = CALENDAR(MIN(Fact_Tickets[Ticket_Date]), MAX(Fact_Tickets[Ticket_Date]))
```
#### Month
```dax
Month = FORMAT('Dim_Calendar'[Date], "mmm")
```
#### Month Number
```dax
Month Number = MONTH('Dim_Calendar'[Date])
```
#### Quarter

```dax
Quarter = FORMAT('Dim_Calendar'[Date], "\QQ")
```
#### Year
```dax
Year = FORMAT('Dim_Calendar'[Date], "yyyy")
```

## 2. First Time Fix %
A standard efficiency metric calculating the percentage of tickets resolved on the first visit.
```dax
First Time Fix % = 
DIVIDE(
CALCULATE(COUNTROWS('Fact_Tickets'), 'Fact_Tickets'[First_Time_Fix] = 1),
COUNTROWS('Fact_Tickets'), 0
)
```

## 3. Adjusted Gender Pay Gap (Iterative Logic)

To ensure pay equity compliance, i calculated the gap by iterating through specific Job Levels rather than using a global average, which can be misleading due to representation differences.
```dax
Gender Pay Gap % = 
VAR MaleAvg = CALCULATE(AVERAGE('Dim_Technicians'[Base_Salary_PLN]), 'Dim_Technicians'[Gender] = "Male")
VAR FemaleAvg = CALCULATE(AVERAGE('Dim_Technicians'[Base_Salary_PLN]), 'Dim_Technicians'[Gender] = "Female")
RETURN 
DIVIDE(MaleAvg - FemaleAvg, MaleAvg, 0)
```

## 4. Actual Bonus Payout
This measure applies the business logic for the "Outcome-Based Pay" model. It checks the technician's fix rate and assigns a multiplier (1.2x, 1.0x, or 0.5x) to their target bonus.
```dax
Actual Bonus Payout = 
VAR CurrentTechFixRate = [First Time Fix %]
VAR BaseSalary = SELECTEDVALUE('Dim_Technicians'[Base_Salary_PLN])
VAR TargetPct = SELECTEDVALUE('Dim_Technicians'[Target_Bonus_Pct])
VAR TargetBonusAmount = BaseSalary * TargetPct

RETURN
IF(
    ISBLANK(BaseSalary), 0,
    IF(
        CurrentTechFixRate >= 0.85, TargetBonusAmount * 1.2,
        IF(
            CurrentTechFixRate >= 0.75, TargetBonusAmount * 1.0,
            TargetBonusAmount * 0.5
        )
    )
)
```

## 5. Total Bonus Paid
Since Actual Bonus Payout relies on row-level logic, I used SUMX to ensure totals calculate correctly at the Department/Country level.

```dax
Total Bonus Paid = SUMX('Dim_Technicians', [Actual Bonus Payout])
```

## 6. Total Bonus PY (Time Intelligence)
Calculates the bonus spend for the same period in the previous year for Year-over-Year (YoY) comparison.
```dax
Total Bonus PY = CALCULATE([Total Bonus Paid], SAMEPERIODLASTYEAR('Dim_Calendar'[Date]))
```

## 7. Dynamic Budget Forecasting (What-If Parameter)
This measure calculates the projected total bonus spend based on the user's selection in the "Bonus Adjustment" slider. It uses a disconnected parameter table to multiply the actual values without altering the underlying data.
Parameter Table Generation:
```dax
Bonus Adjustment = GENERATESERIES(-0.5, 0.5, 0.05)
```
Forecast Measure:
```dax
Projected Bonus Cost = 
VAR Adjustment = 'Bonus Adjustment'[Bonus Adjustment Value]
RETURN
[Total Bonus Paid] * (1 + Adjustment)
```

## 8. Machine Age
Calculated column to determine the asset's lifecycle stage (used to identify the "Year 6" drop).
```dax
= DATEDIFF(Dim_Machines[Install_Date], TODAY(), YEAR)
```

## 9. Average Downtime Hours
This calculates the downtime hours of the machine. A metric used to identify high-risk assets (e.g., MRI Scanners).
```dax
Average Downtime Hours = AVERAGE(Fact_Tickets[Downtime_Hours])

```
## 10. Row-Level Security (RLS) Filter
This logic is applied to the Dim_Technicians table to restrict data access. A specific role was created for the Warsaw Regional Manager.
```text
// Filter expression applied to Dim_Technicians table
[Location] = "Warsaw"
```