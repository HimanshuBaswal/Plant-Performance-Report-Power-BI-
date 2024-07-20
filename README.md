# Power-BI-Plant-Report

## Objectives/Goal
Create a condensed, dynamic performance report that utilizes SWITCH measures and conditional formatting.

## About the Data 
The File contains 3 sheets:
1) Plant_FACT - Product_id,Sales_USD,quantity,Price_USD,COGS_USD,Date_Time,Account_id
2) Accounts - country_code,Account,Master_id,Account_id,latitude,longitude,country,Postal_code,street_name,Street_number
3) Plant_Hierarchy - Product_Family,Product_Family_Id,Product_Group,Product_Group_id,Product_Name,Product_Name_id,Product_Size,Product_Type
   
## Methodology (ETL)
1) Import the excel file using Get data from workbook
2) Change **Plant_Hierarchy** to **Dim_Product**
3) Change **Accounts** to **Dim_Account**
4) Change **Plant_Fact** to **Fact_Sales**
5) Remove any duplicates in product_name_id column in **Dim_Product**
6) Change the column names (if any mistakes in them)

## Model
### Dim_date calendar table
1) Create a **Dim_Date calendar** table - A Date table is best practice when dealing with time based data.
    ```
    Dim_date = CALENDAR (DATE(2022,01,01),DATE(2024,12,31))
    ```
2) In **Dim_Date** create a new column
    ```
    Inpast = 
    VAR lastsalesdate = MAX(FACT_Sales[Date_Time])
    VAR lastsalesdatePY = EDATE(lastsalesdate,-12)
    RETURN
    Dim_Date[Date] <= lastsalesdatePY
    ```
    
### Create a Slc_Values table
1) Sales
2) Gross Profit
3) Quantity

### Create a Measures table
1) Create a measures table (it will contain on a column initially)
```
_Measures = 
```
2) Sales
```
_Sales = SUM(Fact_Sales[Sales_USD]) 
```
3) Quantity
```
_Quantity = SUM(Fact_Sales[quantity]) 
```
4) COGs and Gross Profit 
```
_COGs = SUM(Fact_Sales[COGS_USD]) 
Gross Profit = [Sales] - [COGs]
```
5) PYTD_Sales
```
PYTD_Sales = CALCULATE([Sales], SAMEPERIODLASTYEAR(Dim_Date[Date]),Dim_Date[Inpast]=TRUE)
```
6) PYTD_Quantity
```
PYTD_Quantity = CALCULATE([Quantity], SAMEPERIODLASTYEAR(Dim_Date[Date]),Dim_Date[Inpast]=TRUE)
```
7) PYTD_GrossProfit
```
PYTD_GrossProfit = CALCULATE([GrossProfit], SAMEPERIODLASTYEAR(Dim_Date[Date]),Dim_Date[Inpast]=TRUE)
```
8) YTD_Sales
```
YTD_Sales = TOTALYTD([Sales],Fact_Sales[Date_Time])
```
9) YTD_Quantity
```
YTD_Quantity = TOTALYTD([Quantity],Fact_Sales[Date_Time])
```
10) YTD_GrossProfit
```
YTD_GrossProfit = TOTALYTD([GrossProfit],Fact_Sales[Date_Time])
```
11) S_PYTD (Parameter)
```
VAR selected_value = SELECTEDVALUE(Slc_Values[Values])
Var result = SWITCH(selected_value,
    "Sales",[PYTD_Sales],
    "Quantity",[PYTD_Quantity],
    "Gross Profit",[PYTD_Gross Profit],
    BLANK()
)
RETURN
result
```
12) S_YTD (Parameter)
```
VAR selected_value = SELECTEDVALUE(Slc_Values[Values])
Var result = SWITCH(selected_value,
    "Sales",[YTD_Sales],
    "Quantity",[YTD_Quantity],
    "Gross Profit",[YTD_Gross Profit],
    BLANK()
)
RETURN
result
```
13) YTD vs PYTD
```
[S_YTD] - [S_PYTD]
```
14) GP% (in the measures table>base measures)
```
GP% = DIVIDE([Gross Profit],[Sales])
```
15) _Waterfall title
```
 _Waterfall title = SELECTEDVALUE(Slc_Values[Values]) & " YTD vs PYTD | Month - Country - Product"
```
16) _Column Chart title
```
_Column Chart title = SELECTEDVALUE(Slc_Values[Values]) & " YTD vs PYTD | Month "
```
17) _Scatter title
```
_Scatter title = " Account Profitability Segmentation | GP% and " & SELECTEDVALUE(Slc_Values[Values])
```
18) _Report title
```
_Report title = "Plant Co. " & SELECTEDVALUE(Slc_Values[Values]) & " Performance" & YEAR(SELECTEDVALUE(Dim_Date[Date].[Year]))
```

### Relationships in Model
- Create a PYTD folder - [PYTD_GrossProfit,PYTD_Sales,PYTD_Quantity]
- Create a YTD folder - [YTD_GrossProfit,YTD_Sales,YTD_Quantity]
- Create a Switch Folder - [S_YTD,S_PYTD,YTD vs PYTD]
- Create a Base Measure Folder [COGS,Gross_Profit,Sales.Quantity]
![image](https://github.com/user-attachments/assets/24f42851-10af-43f2-a2c5-80cf61a602d3)

### Create JOINS
- Fact_Sales and Dim_Product on Product_id
- Fact_Sales and Dim_Account on Account_id (MtM)
- Fact_Sales and Dim_Date on Date
![image](https://github.com/user-attachments/assets/4443eea3-4e15-4780-8404-1a96d9d89981)

## Visuals
1) Create a background image or style and Fit in Format page in Canvas Background.
2) Create 4 cards (S_PYTD,S_YTD,YTD vs PYTD,GP%)
3) Values Slicer (gross_profit,sales,quantity)
4) Year filter (year)
5) TreeMap - S_YTD by Country
6) Waterfall - YTD vs PYTD by Month and Country
7) Line and Column Chart - S_YTD and S_PYTD by Date (Q&M), Product_type in Legend
8) Scatter Chart - S_YTD vs GP% by Account with Zoom Slider

## Review
![image](https://github.com/user-attachments/assets/2c38e757-372a-4019-b7a2-18ee24a0a140)
