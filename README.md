# Peaks Framework
Peaks framework is an end-user driven command flow that enables working with Peaks and third-party libraries.

## Distinct Function

Peaks's Command{Parameters}
```
CurrentSetting{StreamMB(1000)Thread(100)}
Distinct{1000MillionRows.csv | Ledger, Account, PartNo,Project,Contact,Unit Code, D/C,Currency ~ Table}
WriteFile{Table | * ~ Peaks-Distinct1000M.csv}
```

Polar's Python Code

```
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
```
e = time.time()
print("Polars GroupBy 1000M Time = {}".format(e-s))

## GroupBy Function

Peaks's Command{Parameters}
```
CurrentSetting{StreamMB(1000)Thread(100)}
GroupBy{1000MillionRows.CSV | Ledger, Account, PartNo,Project,Contact,Unit Code, D/C,Currency 
  =>  Count() Max(Quantity) Min(Quantity) Sum(Quantity) ~ Table}
WriteFile{Table | * ~ Peaks-GroupBy1000M.csv}
```

Polar's Python Code

```
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



