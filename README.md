# Northwind_Traders_Analysis       

![](selecting_supplier.png)   

## Introduction      
_The Northwind Traders database is originally a fictional sample database used by Microsoft. However, I downloaded this modified version from [Maven Analytics](https://www.mavenanalytics.io/) data playground. It models a fictitious gourmet food supplier which sells various products such as beverages, confections, meat, seafood, etc to customers._     
_A detailed article on the analysis process and insights is up on my [medium]()_     

## Creating the Date Table and DAX Measures             

The date table was created in Power Query;     

    = (StartDate as date, EndDate as date)=>        
    
    let         
    //Capture the date range from the parameters
    StartDate = #date(Date.Year(StartDate), Date.Month(StartDate), 
    Date.Day(StartDate)),
    EndDate = #date(Date.Year(EndDate), Date.Month(EndDate), 
    Date.Day(EndDate)),      

    //Get the number of dates that will be required for the table
    GetDateCount = Duration.Days(EndDate - StartDate),

    //Take the count of dates and turn it into a list of dates
    GetDateList = List.Dates(StartDate, GetDateCount, 
    #duration(1,0,0,0)),

    //Convert the list into a table
    DateListToTable = Table.FromList(GetDateList, 
    Splitter.SplitByNothing(), {"Date"}, null, ExtraValues.Error),

    //Create various date attributes from the date column
    //Add Year Column
    YearNumber = Table.AddColumn(DateListToTable, "Year", 
    each Date.Year([Date])),

    //Add Quarter Column
    QuarterNumber = Table.AddColumn(YearNumber , "Quarter", 
    each "Q" & Number.ToText(Date.QuarterOfYear([Date]))),

    //Add Week Number Column
    WeekNumber= Table.AddColumn(QuarterNumber , "Week Number", 
    each Date.WeekOfYear([Date])),

    //Add Month Number Column
    MonthNumber = Table.AddColumn(WeekNumber, "Month Number", 
    each Date.Month([Date])),

    //Add Month Name Column
    MonthName = Table.AddColumn(MonthNumber , "Month", 
    each Date.ToText([Date],"MMMM")),

     //Add Day of Week Column
    DayOfWeek = Table.AddColumn(MonthName , "Day of Week", 
    each Date.ToText([Date],"dddd"))         

Net Sales;     

    Net Sales =     
    CALCULATE(SUM(order_details[net sales]), USERELATIONSHIP(orders[orderDate], 'Date Table'[Date]) )

Total Orders;

    Total Orders =    
    CALCULATE(COUNT(orders[orderDate]), USERELATIONSHIP('Date Table'[Date], orders[orderDate]) ) 

