# HR Analytics Dashboard - Atlas Labs

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-00599C?style=for-the-badge&logo=microsoft&logoColor=white)

**Author:** Darius Chiosa Chiosa  
**Tech Stack:** Power BI, DAX, Data Modeling, Data Visualization.

## Problem Statement
Atlas Labs (a fictional company from a data case study) needed a clear view of their workforce dynamics. 
My goal for this project wasn't just to report basic headcount numbers, but to dive deep into the data to figure out why employees were leaving (attrition), how performance aligns with satisfaction, and what the overall demographic landscape looks like to help HR make data-driven decisions.

## How I did it (Personal workflow)
I went hands-on with the dataset from start to finish:

1. **Data Validation & Modeling:** First things first, I built a robust Star Schema, connecting the core `FactPerformanceRating` table with dimension tables to ensure data integrity and optimize complex filtering.
2. **Writing DAX:** I got to work building out the core measures needed for the analysis, like the exact Attrition Rate, isolating active employees, and handling complex date relationships using `USERELATIONSHIP`.
3. **Slicing and Dicing:** To uncover real insights, I created a fully dynamic custom Calendar table (`DimDate`) instead of relying on Power BI's heavy Auto Date/Time feature. This allowed me to segment the workforce accurately across different fiscal periods.
4. **Telling the Story:** Finally, I designed a 4-page interactive dashboard. I wanted to make sure stakeholders could easily follow the narrative, starting from a high-level overview down to the nitty-gritty details of why employees were leaving.

## A Quick Technical Highlight: Keeping DAX Clean
When I needed to calculate the `NextReviewDate` for an employee, the quick-and-dirty way would have been to repeat the `MAX` functions multiple times within a standard `IF` statement. 

However, I always try to write code that is scalable and easy to read. So, I opted to use Variables (`VAR` and `RETURN`) instead. It calculates the logic once, stores it in memory, and makes the code look and work so much better:

```dax
NextReviewDate = 
VAR reviewOrHire =
    IF (
        MAX ( FactPerformanceRating[ReviewDate] ) = BLANK (),
        MAX ( DimEmployee[HireDate] ),
        MAX ( FactPerformanceRating[ReviewDate] )
    )
RETURN
    reviewOrHire + 365
```
📈 What did the data actually say?
After putting it all together, three main things stood out:

Satisfaction directly impacts retention: There is a clear correlation between low Environment and Job Satisfaction scores and the likelihood of an employee leaving the company. When satisfaction drops, flight risk skyrockets.

The Tenure Risk Profile: Employees with fewer years at the company or stuck in the same role for a long time show a noticeably higher attrition rate compared to the rest of the staff.

Performance Discrepancies: By comparing Self Ratings with Manager Ratings, we identified specific areas where employees feel undervalued compared to how their managers actually rate them.

Dashboard Previews
Overview
<img width="1920" height="1031" alt="1" src="https://github.com/user-attachments/assets/b69a55d6-0551-45bb-88f4-5cb4e518bdc5" />
Demographics
<img width="1915" height="1080" alt="2" src="https://github.com/user-attachments/assets/6470dc0c-f1af-4df8-9975-f17e5094a163" />
Performance & Satisfaction
<img width="1920" height="1080" alt="3" src="https://github.com/user-attachments/assets/532c8029-193d-40ed-9256-ff7071573fce" />
Attrition Analysis
<img width="1920" height="1080" alt="4" src="https://github.com/user-attachments/assets/38f5de0d-ebd9-421a-b580-0a3155005349" />
Model View
<img width="1304" height="813" alt="Model View" src="https://github.com/user-attachments/assets/c01c8145-06c5-4286-9113-27ed8015e46e" />
🗄️ Appendix: Advanced DAX Measures Repository
<details>
<summary>👉 <b>Click here to expand and see the core DAX formulas used in this project</b></summary>

Dynamic Calendar Table
```dax
DimDate = 
VAR _minYear = YEAR(MIN(DimEmployee[HireDate]))
VAR _maxYear = YEAR(MAX(DimEmployee[HireDate]))
VAR _fiscalStart = 4 

RETURN
ADDCOLUMNS(
    CALENDAR(DATE(_minYear,1,1), DATE(_maxYear,12,31)),
    "Year",YEAR([Date]),
    "Year Start",DATE( YEAR([Date]),1,1),
    "YearEnd",DATE( YEAR([Date]),12,31),
    "MonthNumber",MONTH([Date]),
    "MonthStart",DATE( YEAR([Date]), MONTH([Date]), 1),
    "MonthEnd",EOMONTH([Date],0),
    "DaysInMonth",DATEDIFF(DATE( YEAR([Date]), MONTH([Date]), 1),EOMONTH([Date],0),DAY)+1,
    "YearMonthNumber",INT(FORMAT([Date],"YYYYMM")),
    "YearMonthName",FORMAT([Date],"YYYY-MMM"),
    "DayNumber",DAY([Date]),
    "DayName",FORMAT([Date],"DDDD"),
    "DayNameShort",FORMAT([Date],"DDD"),
    "DayOfWeek",WEEKDAY([Date]),
    "MonthName",FORMAT([Date],"MMMM"),
    "MonthNameShort",FORMAT([Date],"MMM"),
    "Quarter",QUARTER([Date]),
    "QuarterName","Q"&FORMAT([Date],"Q"),
    "YearQuarterNumber",INT(FORMAT([Date],"YYYYQ")),
    "YearQuarterName",FORMAT([Date],"YYYY")&" Q"&FORMAT([Date],"Q"),
    "QuarterStart",DATE( YEAR([Date]), (QUARTER([Date])*3)-2, 1),
    "QuarterEnd",EOMONTH(DATE( YEAR([Date]), QUARTER([Date])*3, 1),0),
    "WeekNumber",WEEKNUM([Date]),
    "WeekStart", [Date]-WEEKDAY([Date])+1,
    "WeekEnd",[Date]+7-WEEKDAY([Date]),
    "FiscalYear",if(_fiscalStart=1,YEAR([Date]),YEAR([Date])+ QUOTIENT(MONTH([Date])+ (13-_fiscalStart),13)),
    "FiscalQuarter",QUARTER( DATE( YEAR([Date]),MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1,1) ),
    "FiscalMonth",MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1
)
```
Core HR Metrics
```dax
TotalEmployees = DISTINCTCOUNT(DimEmployee[EmployeeID])

ActiveEmployees = CALCULATE([TotalEmployees], FILTER(DimEmployee, DimEmployee[Attrition] = "No"))

InactiveEmployees = CALCULATE([TotalEmployees], FILTER(DimEmployee, DimEmployee[Attrition] = "Yes"))

% Attrition Rate = DIVIDE([InactiveEmployees], [TotalEmployees])

AverageSalary = AVERAGE(DimEmployee[Salary])
```
Time Intelligence & Inactive Relationships
```dax
TotalEmployeesDate = 
CALCULATE (
    [TotalEmployees],
    USERELATIONSHIP ( DimEmployee[HireDate], DimDate[Date] )
)

InactiveEmployeeDate = 
CALCULATE(
    [InactiveEmployees], 
    USERELATIONSHIP(DimEmployee[HireDate], DimDate[Date])
)

% Attrition Rate Date = [InactiveEmployeeDate]/[TotalEmployeesDate]
```
Performance & Satisfaction Metrics
```dax
EnvironmentSatisfaction = 
CALCULATE (
    MAX ( FactPerformanceRating[EnvironmentSatisfaction] ),
    USERELATIONSHIP ( FactPerformanceRating[EnvironmentSatisfaction], DimSatisfiedLevel[SatisfactionID] )
)

JobSatisfaction = MAX(FactPerformanceRating[JobSatisfaction])

RelationshipSatisfaction = 
CALCULATE (
    MAX ( FactPerformanceRating[RelationshipSatisfaction] ),
    USERELATIONSHIP ( FactPerformanceRating[RelationshipSatisfaction], DimSatisfiedLevel[SatisfactionID] )
)

WorkLifeBalance = 
CALCULATE (
    MAX ( FactPerformanceRating[WorkLifeBalance] ),
    USERELATIONSHIP ( FactPerformanceRating[WorkLifeBalance], DimSatisfiedLevel[SatisfactionID] )
)

ManagerRating = 
CALCULATE (
    MAX ( FactPerformanceRating[ManagerRating] ),
    USERELATIONSHIP ( FactPerformanceRating[ManagerRating], DimRatingLevel[RatingID] )
)

SelfRating = 
CALCULATE (
    MAX ( FactPerformanceRating[SelfRating] ),
    USERELATIONSHIP ( FactPerformanceRating[SelfRating], DimRatingLevel[RatingID] )
)
```
Employee Data Formatting
```dax
FullName = COMBINEVALUES(" ", DimEmployee[FirstName], DimEmployee[LastName])

LastReviewDate = 
IF (
    MAX ( FactPerformanceRating[ReviewDate] ) = BLANK (),
    "No Review Yet",
    MAX ( FactPerformanceRating[ReviewDate] )
)
```
