# Peaks DataFrame
Peaks DataFrame is a personal academic project that aims to provide an alternative to SQL statements for processing billions of rows using streaming or in-memory model to accelerate dataframe. The project started coding on February 18th, 2023 in Hong Kong SAR with the goal of achieving real-time processing up to 10 million rows per second. 

Currently, Peaks DataFrame have been innovating and testing a set of algorithms and data structures to support profound acceleration of the dataframe with limited memory. One of the project’s expected outcomes is to solve the data explosion that came with data capture from IoT devices, ERP, internet and data lake. By using a prpper script settings, it can support streaming and in-memory models.
 
## Peaks Framework and Library
Peaks DataFrame comprises of Peaks framework and library that are currently under development. It is working on an end-user driven command flow that enables working with Peaks and third-party libraries. The Peak Library is a high-performance calculation engine that can be configured in either streaming mode or in-memory mode easily. If you use streaming mode with proper settings, you can process billions of rows on your desktop PC with 16GB or above memory. Both framework and library are written in Golang and are considering interface with Python, R and Node.js. Currently, the development and testing environment is using Windows 11 and AMD x86, and will support Linux. Apart from AMD x86, it will also support ARM CPU. For fast-growing RISC V in IoT applications, which is one of my considerations.

## File Format
Currently, Peaks DataFrame supports tables in CSV file format only. Other table formats such as GZIP(CSV), JSON, HTML, XLSX, Parquet, Lance, HDF5, ORC, Feather and etc are under consideration.

## From WebNameSQL to Peaks DataFrame

Peaks Framework is derived by a .net project WebNameSQL. You can see the full spec of "WebNameSQL.pdf" from the repo. Peaks Framework will have an improvement version based on WebNameSQL. Any software can implement this framework to standardise ETL expression similar to HTML5, which benefits for end-users. The author have over 10 years of experience in designing ETL expression covers 4 different designs. WebNameSQL is the best design, so Peaks Framework will adopt this design with some of improvement, particularly to adapt Python code.

WebNameSQL is a C# in-memory databending software that supports accountants using a web browser to interactive with accounting rules and tables for databending. However, this project became obsolete and it is replaced by a new project “Peaks DataFrame” to solve issues arising from real-time processing and big data. During a continuing effort in academic research, it is implemented new algorithms by using Golang which resulted in a performance gain of around 5X ~ 10X.

Commands to be re-implemented in the Peaks DataFrame will not be the same as WebNameSQL. Considering there are too many commands for your learning and practice, further consolidation and improvement is necessary. The use cases are no longer restricted to accounting; for example, some use cases will cover bioinformatics.

## Benchmarking
PeaksBenchmark.xlsx documents some benchmarking results. Currently, we are focusing on comparing Polars and Peaks. The next phase will cover Pandas. For relevant scripts and data, please refer to https://github.com/financialclose/benchmarking.

##  Below Functions has been included in the benchmarking report "PeaksBenchmark.xlsx"

### Distinct Function

Peaks's Script
```
<< Command{Parameters} for Web request, Windows/Linux command line >>

> In-memory model
ReadFile{10MillionRows.csv ~ Table}
Distinct{Ledger, Account, PartNo,Project,Contact,Unit Code, D/C,Currency ~ Table2}
WriteFile{Table2 | * ~ Peaks-Distinct10.csv}
WriteFile{Table | Ledger, Account, PartNo,Project,Contact ~ Peaks-Transaction.csv}

> Streaming model (streaming for reading file only)
CurrentSetting{StreamMB(1000)Thread(100)}
Distinct{1000MillionRows.csv | Ledger, Account, PartNo,Project,Contact,Unit Code, D/C,Currency ~ Table}
WriteFile{Table | * ~ Peaks-Distinct1000M.csv}

<< Tentative Python Code >>

import peaks as hk

Table = hk.ReadFile("10MillionRows.csv")
Table2 = hk.Distinct("Table | Ledger, Account, PartNo,Project,Contact,Unit Code, D/C,Currency")
FilePath = hk.WriteFile("Table2 | * ~ Peaks-Distinct10.csv")
FilePath2 = hk.WriteFile("Table | Ledger, Account, PartNo,Project,Contact ~ Peaks-Transaction.csv")

  where the Python Code (" ") is equvalent to the original syntax {}. 
  And "variable =" is equvalent to the original syntax "~ TableName"
  When output table is a file name instead of in-memory table, the variable  
     will be a string which contain a full file path of the output file.
```

Polar's Python Code

```
> Streaming model
import polars as pl
import time
import pathlib
q = (
     pl.scan_csv("Input/1000MillionRows.csv")      
    .select(["Ledger", "Account", "PartNo", "Contact","Project","Unit Code", "D/C","Currency"]).unique()
    )    

a = q.collect(streaming=True)
path: pathlib.Path = "Output/Polars-Distinct1000M.csv"
a.write_csv(path, separator=",")
e = time.time()
print("Polars GroupBy 1000M Time = {}".format(e-s))
```

### GroupBy Function

Peaks's Script
```
<< Command{Parameters} for Web request, Windows/Linux command line >>

> Streaming model (streaming for reading only)
CurrentSetting{StreamMB(1000)Thread(100)}
GroupBy{1000MillionRows.CSV | Ledger, Account, PartNo,Project,Contact,Unit Code, D/C,Currency 
  =>  Count() Max(Quantity) Min(Quantity) Sum(Quantity) ~ Table}
WriteFile{Table | * ~ Peaks-GroupBy1000M.csv}
```

Polar's Python Code

```
> Streaming model
q = (
     pl.scan_csv("Input/1000MillionRows.csv")      
     .groupby(by=["Ledger", "Account", "PartNo", "Contact","Project","Unit Code", "D/C","Currency"])
    .agg([   
        pl.count('Quantity').alias('Quantity(Count)'),
        pl.max('Quantity').alias('Quantity(Max)'),
        pl.min('Quantity').alias('Quantity(Min)'),
        pl.sum('Quantity').alias('Quantity(Sum)'),        
    ])) 

a = q.collect(streaming=True)
path: pathlib.Path = "Output/Polars-GroupBy1000M.csv"
a.write_csv(path, separator=",")

```

### JoinTable Function 

Peaks's Script
```
<< Command{Parameters} for Web request, Windows/Linux command line>>

> Streaming model (streaming for reading and writing only)
CurrentSetting{StreamMB(500)Thread(100)}
ReadFile{Master.csv ~ Master}
BuildKeyValue{Master | Ledger,Account,Project ~ KeyValue} 
JoinKeyValue{1000MillionRows.csv | Ledger,Account,Project => AllMatch(KeyValue) ~ Peaks-JoinTable1000M.csv} 
```

Polar's Python Code

```
> In-memory model (has requested Polars to support a real streaming model for JoinTable)
transaction = pl.read_csv("Input/1000MillionRows.CSV")            
master = pl.read_csv("Input/Master.CSV") 
joined_table = transaction.join(master, on=["Ledger","Account","Project"], how="inner")
joined_table.write_csv("Output/Polars-JoinTable1000M.csv")

```

## Compare Programming Language

Folder: https://github.com/hkpeaks/peaks-framework/tree/main/CompareProgrammingLanguage

Before deciding to develop the Peaks DataFrame, it is conducted a study to determine which programming language was most suitable for this project. The author had compared CSharp, Golang, and Rust with Pandas, Peaks, and Polars using a benchmark located in the folder ‘CompareProgrammingLanguage’. You can find a readme.pdf file inside the folder that shows a comparison of these languages with Pandas, Peaks, and Polars. This benchmark was prepared on April 20th, 2023.

Testing Machine: Intel i9 8-Cores CPU, 32G RAM, 500GB NVMe SSD

The data structure implemented for the basic programming in a way that is similar to Parquet file format. It is extensively used key-value pairs, for example, use 1, 2, and 3 to represent unique values for each column. However, this extensive use of CPU and memory resources made Peaks DataFrame avoid using it again.

When it comes to data structures, bytearray is one of the most useful and memory-efficient. As for algorithms, parallel streaming for reading/writing files and querying is very powerful and can handle billions of rows even on a desktop PC with only 16GB RAM.

Further Information: https://www.linkedin.com/posts/max01_benchmarking-pandas-github-activity-7054824689273098241-P3VS?utm_source=share&utm_medium=member_desktop

## High Performance Web Pivot Table

Folder: https://github.com/hkpeaks/peaks-framework/tree/main/HighPerformanceWebPivotTable

This is a first .net project before the author using Golang. The author will consider whether to re-implement this into Peak DataFrame or publish this .net project directly.
https://youtu.be/yfJnYQBJ5ZY

[![Web Pivot Table](https://github.com/hkpeaks/peaks-framework/blob/main/HighPerformanceWebPivotTable/WebPivotTable.png)](http://www.youtube.com/watch?v=yfJnYQBJ5ZY "Web Pivot Table")

## Latest News
For latest news about this academic project, please refer to https://www.linkedin.com/in/max01/recent-activity/all/




