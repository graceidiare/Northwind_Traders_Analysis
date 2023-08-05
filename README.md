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

DAX Maesures;     

    Net Sales =     
    CALCULATE(SUM(order_details[net sales]), USERELATIONSHIP(orders[orderDate], 'Date Table'[Date]) )   

    Gross Sales =
    CALCULATE(SUM(order_details[gross sales]), USERELATIONSHIP(orders[orderDate], 'Date Table'[Date]))    

    Total Orders =    
    CALCULATE(COUNT(orders[orderDate]), USERELATIONSHIP('Date Table'[Date], orders[orderDate]) )   

    Products Sold =
    CALCULATE(COUNT(order_details[orderID]), ISBLANK(order_details[orderID]) = FALSE(), USERELATIONSHIP('Date Table'[Date], orders[orderDate]) )

    Discounts =
    [Gross Sales] - [Net Sales]

    Shipped Orders =
    CALCULATE(COUNT(orders[shippedDate]), USERELATIONSHIP('Date Table'[Date], orders[shippedDate]) ) 

    Not Shipped =
    CALCULATE(COUNT(orders[orderID]), ISBLANK(orders[shippedDate]) , USERELATIONSHIP(orders[shippedDate], 'Date Table'[Date]))

    Discontinued Products =
    CALCULATE(COUNT(products[discontinued]), products[discontinued] = 1, USERELATIONSHIP('Date Table'[Date], orders[orderDate]))

    Freight Paid =
    CALCULATE(SUM(orders[freight]), USERELATIONSHIP(orders[orderDate], 'Date Table'[Date]) )

    On-time =
    CALCULATE(COUNT(orders[Shipping Delivery]), orders[Shipping Delivery] = "On-time") 

    On-time Delivery Rate =
    DIVIDE([On-time], [Total Orders]) 

    
    

    

