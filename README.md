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

As for the Telegraf conf attached, appart from the timings and number of records to write, the input part is the most important.
