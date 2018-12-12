---
title: "Raport-1"
author: "Wojciech Duda"
date: "12 Grudnia 2018"
output: 
  html_document:
    keep_md: yes
    toc: yes
---

## Bbiblioteki

```r
library(dplyr)
library(knitr)
```


##Wczytanie danych z pliku, wstepne czyszczenie danych 

```r
Data <- read.csv2("all_summary.csv", nrows=15000, sep=';', comment.char="", na.strings='nan', header=TRUE)
classes <- sapply(Data, class)
for (i in 1:3) classes[i] <- "NULL"
for (i in 6:12) classes[i] <- "NULL"
classes[14] <- "NULL"
for (i in 16:20) classes[i] <- "NULL"
for (i in 23:64) classes[i] <- "NULL"
for (i in 397:403) classes[i] <- "NULL"
classes[406] <- "NULL"
for (i in 410:412) classes[i] <- "NULL"
Data <- read.csv2("all_summary.csv",sep=';',dec=".", comment.char="", na.strings='nan', header=TRUE, colClasses=classes)
```

##Usuniecie wierszy

```r
Data %>% filter(!(res_name %in% c('UNK', 'UNX', 'UNL', 'DUM', 'N', 'BLOB', 'ALA', 'ARG', 'ASN', 'ASP', 'CYS', 'GLN', 'GLU', 'GLY', 'HIS', 'ILE', 'LEU', 'LYS', 'MET', 'MSE', 'PHE', 'PRO', 'SEC', 'SER', 'THR', 'TRP', 'TYR', 'VAL', 'DA', 'DG', 'DT', 'DC', 'DU', 'A', 'G', 'T', 'C', 'U', 'HOH', 'H20', 'WAT','NAN'))) -> Data
```

##Podsumowanie czystych danych
Zaladowano 585339 wierszy, ktore maja343 zmiennych.

##Zapisanie CVS

```r
write.csv(Data, file = "MyData.csv", row.names=FALSE)
```
