---
title: "Raport"
author: "Wojciech Duda"
date: "21 listopada 2018"
output: 
  html_document:
    toc: true 
---

## Bbiblioteki
```{R biblioteki, warning=FALSE, message=FALSE}
library(dplyr)
library(knitr)
library(corrplot)
library(ggplot2)
library(plotly)
library(e1071)
```

## Ustawienie ziarna
```{R ustawienie ziarna}
 set.seed(1); 
```

##Wczytanie wstepnie przetworzonych danych

Csv zosta³ przygotowany w 'Raport-czytanie i czyszczenie'

```{R czytanie danych}
#Data <- read.csv("MyData.csv",  sep=',', comment.char="", na.strings='nan', header=TRUE)
Data <- read.csv("MyData.csv",nrow=3000,  sep=',', comment.char="", na.strings='nan', header=TRUE)

```
Za³adowano `r nrow(Data)` wierszy, które maj¹ `r ncol(Data)` zmiennych.

## Ograniczenie danych do najczêstszych

Wyselekcjonowanie 50 najczêstszych grup
```{R tylko 50}
Data[!duplicated(select(Data, pdb_code, res_name)),] %>% group_by(res_name) %>%summarise(count = n()) %>% arrange(desc(count)) %>% head(50) ->unikalne
Data%>%filter(res_name %in% unikalne$res_name ) -> Data

Data[, c(3:length(Data))] <- sapply(Data[, c(3:length(Data))], as.numeric)
print.data.frame(unikalne)
```

```{R czyszczenie unikalne, hide=TRUE, echo=FALSE}
rm(unikalne)
```

##Podsumowanie danych
```{r przyklad}
kable(summary(Data))
```

# Korelacje miêdzy zmiennymi
Korelacje miêdzy zmiennymi. Pominiêto local_min.

```{R korelacje}
corrplot(cor(Data %>% select(local_res_atom_non_h_count:local_std,local_max:FoFc_max)),method="color", tl.cex = 0.1)
```

# Wykresy rozk³adów liczby atomów oraz elektronów

Rozklad liczby atomów

```{r atomy}
ggplotly(ggplot(Data, aes(x=local_res_atom_non_h_count)) + geom_histogram(col=I("black"),bins = 30) )
```

Rozklad liczby elektronów

```{r elektrony}
ggplotly(ggplot(Data, aes(x=local_res_atom_non_h_electron_sum)) + geom_histogram(col=I("black"),bins = 30))
```

# Najwiêksza niezgodnoœæ

10 klas z najwiêksz¹ niezgodnoœci¹ liczby atomów

```{r roznice}
kable(head(Data%>%select(res_name, local_res_atom_non_h_count,dict_atom_non_h_count)%>%group_by(res_name)%>%summarise_all(funs(sum))%>%mutate(delta = abs(local_res_atom_non_h_count - dict_atom_non_h_count))%>%arrange(desc(delta)),10))
```

# Rozklad wartosci kolumn 'part'
```{r party}
part <- Data %>% select(part_01_shape_segments_count:part_01_density_Z_4_0)
for (i in 1:length(part)) {
  Bez_NA <-  part[,i][!is.na(part[,i])]
  
  avg <- mean(Bez_NA)
  
    p <- ggplot() + aes(Bez_NA) + geom_bar(col=I("black")) + xlab(names(part[i])) + geom_vline(xintercept=avg, color="red") + annotate("text", x=avg,y=-10, label=c(avg), color="red")
  
  print(p)
  
}
```

#Przewidywanie liczby elektronów i atomów na podstawie innych kolumn
```{r part_01_chart}

lm_data <- Data
lm_data[is.na(lm_data)] <- 0
lm_data <- lm_data[sapply(lm_data, is.numeric)]
lm_atom_model <- lm(local_res_atom_non_h_count ~ ., lm_data)
lm_atom_summary <- summary(lm_atom_model)
lm_electron_model <- lm(local_res_atom_non_h_electron_sum ~ ., lm_data)
lm_electron_summary <- summary(lm_electron_model)

```

Miary dla liczby atomów:
R^2: `r lm_atom_summary$r.squared`
RMSE `r lm_atom_summary$sigma`

Miary dla liczby elektronów:
R^2: `r lm_electron_summary$r.squared`
RMSE `r lm_electron_summary$sigma`

#Klasyfikator Bayesa na atrybucie res_name

```{R Klasyfikator}
Klasyfikator<-predict(naiveBayes(res_name ~., data=Data[-1]),Data[-1])
Klasyfikator == Data$res_name
```

Dok³adnoœæ klasyfikatora: `r length(Klasyfikator)/length(Klasyfikator[Klasyfikator==Data$res_name])`
