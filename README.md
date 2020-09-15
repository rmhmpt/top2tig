# top2tig
Script to prepare a VMWare esxtop csv export to be imported by Telegraf into InfluxDB 

go run main.go --help

  
  -h string
        String to remove from header ie: '-s localhost' will remove \\localhost\ (default "localhost")
  
  -i string
        File path to read from (default "input.csv")
  
  -o string
        File path to write to (default "output.csv")
  
  -s int
        Number of csv files to create. Same number of rows, a fraction of the columns (default 1)
        
I've also added to this git the telegraf config file I've used to import the CSV.

The code will execute the following steps:

#Process header
1) Remove the trail "," left at the end of the line. Replaces it with a \n
2) Remove the hostname from the header ie: "\\localhost\Memory\Memory Overcommit" by default will become "Memory\Memory Overcommit". Step 2 is not necessarily needed, but I found that it improved readibility a lot when importing it to Grafana
3) Remove all '\' characters from the header. This is important because when Grafana generates the queries to grab the data from influx, it'll send "\M" or "\r" which will cause the queries to fail. Replaces them all with '-'

#Process values
1) Remove the trail "," left at the end of the line. Replaces it with a \n
2) Swap around the month and the day, so Telegraf can export to Influx
3) Removes all the '"' from the line. 

#Split file into multiple CSVs
If the -s option was used, then the code will go through the processed file and split it into multiple files. The number of rows will be the same, but the number of columns will be a fraction.
I've included this feature because I was getting timeouts during the export to Influx because the CSVs were massive and, although I don't have proof other than the behaviour I saw, I believe writing the headers with their lenghty strings was generating massive HTTP resquests. 
This part of the code is inneficient, as it will run through the file multiple times. This could be optimised by writing multiple files at the same time and running through the file once.

As for the Telegraf conf attached, appart from the timings and number of records to write, the input part is the most important:

#INPUT PLUGINS#

##I've used tail, because file would cycle through the CSV constantly importing the same values. With tail it'll import the values once
[[inputs.tail]]
  files = ["/home/esxtop/split12proc.csv"]

##Read file from beginning.
  from_beginning = true

##The dataformat to be read from files
  data_format = "csv"
  
##Indicates how many rows to treat as a header. By default, the parser assumes
##there is no header and will parse the first row as data. If set to anything more
##than 1, column names will be concatenated with the name listed in the next header row.
##If `csv_column_names` is specified, the column names in header will be overridden.
csv_header_row_count = 1

##Indicates the number of rows to skip before looking for header information.
csv_skip_rows = 0

##Indicates the number of columns to skip before looking for data to parse.
##These columns will be skipped in the header as well.
csv_skip_columns = 0
 
##The character reserved for marking a row as a comment row
##Commented rows are skipped and not parsed
csv_comment = "@"

##The column to extract time information for the metric
##`csv_timestamp_format` must be specified if this is used
csv_timestamp_column = "(PDH-CSV 4.0) (UTC)(0)"

##The format of time data extracted from `csv_timestamp_column`
##this must be specified if `csv_timestamp_column` is specified
csv_timestamp_format = "02/01/2006 15:04:05"

csv_delimiter=","
