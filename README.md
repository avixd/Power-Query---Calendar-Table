# Power-Query : Calendar Table
Create a calendar/date dimension from scratch using Power Query

## The code sample below has the following functionality:
1. Custom Fiscal Year Index Month Parameter ( Change fiscal year end month : i.e. 4 for 1 April - 31 March etc.)
2. Fiscal Year, Fiscal Quarter and Fiscal Month 
3. Past/Present/Future Flags
4. Weekday/Weekend Flag
5. Day Name, Day of the Year etc.

## Refer to [this youtube video](https://youtu.be/W2C634Q2ltg) with a walk through of the code below

### Code samples of a calendar table using Power Query

```
let
// Parameters
    StartDate = #date(2024,1,1), // Change as required
    EndDate = #date(2027, 1, 1), // Change as required
    FiscalYear_IndexMonth = 3, // Assuming April 1st until 31st March
//Convert (into a table) and Create a list of dates
    Source = List.Dates(StartDate, Duration.Days(EndDate-StartDate)+1, #duration(1,0,0,0) ),
    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Changed Type" = Table.TransformColumnTypes(#"Converted to Table",{{"Column1", type date}}),
    #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"Column1", "Date"}}),
    
// Calendar Year Columns
    #"Inserted Year" = Table.AddColumn(#"Renamed Columns", "Year", each Date.Year([Date]), Int64.Type),
    #"Inserted Start of Year" = Table.AddColumn(#"Inserted Year", "Start of Year", each Date.StartOfYear([Date]), type date),
    #"Inserted End of Year" = Table.AddColumn(#"Inserted Start of Year", "End of Year", each Date.EndOfYear([Date]), type date),
  
// Calendar Month Columns
    #"Inserted Month" = Table.AddColumn(#"Inserted End of Year", "Month", each Date.Month([Date]), Int64.Type),
    #"Inserted Start of Month" = Table.AddColumn(#"Inserted Month", "Start of Month", each Date.StartOfMonth([Date]), type date),
    #"Inserted End of Month" = Table.AddColumn(#"Inserted Start of Month", "End of Month", each Date.EndOfMonth([Date]), type date),
    #"Inserted Days in Month" = Table.AddColumn(#"Inserted End of Month", "Days in Month", each Date.DaysInMonth([Date]), Int64.Type),
    #"Inserted Month Name" = Table.AddColumn(#"Inserted Days in Month", "Month Name", each Date.MonthName([Date]), type text),
  
    #"Inserted Quarter" = Table.AddColumn(#"Inserted Month Name", "Quarter", each Date.QuarterOfYear([Date]), Int64.Type),
// Calendar Quarter Columns
    CalendarQuarter = Table.AddColumn(#"Inserted Quarter", "CalendarQuarter", each "Q"&Text.From([Quarter]) , type text),
    #"Inserted Start of Quarter" = Table.AddColumn(CalendarQuarter, "Start of Quarter", each Date.StartOfQuarter([Date]), type date),
    #"Inserted End of Quarter" = Table.AddColumn(#"Inserted Start of Quarter", "End of Quarter", each Date.EndOfQuarter([Date]), type date),
    #"Inserted Week of Year" = Table.AddColumn(#"Inserted End of Quarter", "Week of Year", each Date.WeekOfYear([Date]), Int64.Type),
    #"Inserted Week of Month" = Table.AddColumn(#"Inserted Week of Year", "Week of Month", each Date.WeekOfMonth([Date]), Int64.Type),
    #"Inserted Start of Week" = Table.AddColumn(#"Inserted Week of Month", "Start of Week", each Date.StartOfWeek([Date]), type date),
    #"Inserted End of Week" = Table.AddColumn(#"Inserted Start of Week", "End of Week", each Date.EndOfWeek([Date]), type date),

// Calendar Day Columns
    #"Inserted Day" = Table.AddColumn(#"Inserted End of Week", "Day", each Date.Day([Date]), Int64.Type),
    #"Inserted Day of Week" = Table.AddColumn(#"Inserted Day", "Day of Week", each Date.DayOfWeek([Date]), Int64.Type),
    #"Inserted Day of Year" = Table.AddColumn(#"Inserted Day of Week", "Day of Year", each Date.DayOfYear([Date]), Int64.Type),
    #"Inserted Day Name" = Table.AddColumn(#"Inserted Day of Year", "Day Name", each Date.DayOfWeekName([Date]), type text),
    #"WeekDay/Weekend" = Table.AddColumn(#"Inserted Day Name", "WeekDay/Weekend", each if [Day Name] ="Saturday" or  [Day Name] ="Sunday" then "Weedend" else "Weekday"),
    
// Fiscal Calendar Columns  
    FiscalYear = Table.AddColumn(#"WeekDay/Weekend", "FiscalYear", each if [Month] < FiscalYear_IndexMonth then [Year]  else [Year] +1 , type number),
    FiscalMonthNumber = Table.AddColumn(FiscalYear, "FiscalMonthNumber", each if [Month] > FiscalYear_IndexMonth then [Month] - FiscalYear_IndexMonth else [Month] + (  12 - FiscalYear_IndexMonth), type number),
    FiscalQuarterNumber = Table.AddColumn(FiscalMonthNumber, "FiscalQuarterNumber", each Number.RoundUp([FiscalMonthNumber]/3,0), type number),
    FiscalQuarter = Table.AddColumn(FiscalQuarterNumber, "FiscalQuarter", each "FQ" & Text.From([FiscalQuarterNumber]) , type text ),
    #"Past/Present/Future Flag" = Table.AddColumn(FiscalQuarter, "Past/Present/Future Flag", each if Date.From( DateTime.LocalNow() ) < [Date] then "Past" else if Date.From( DateTime.LocalNow() ) > [Date] then "Past" else "Present", type text)
in
    #"Past/Present/Future Flag"
```



