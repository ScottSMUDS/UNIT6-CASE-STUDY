---
title: "CASE STUDY  OF GDP RANK & ED STATS DATASETS"
author: "Marvin Scott"
date: "June 15, 2016"
output: 
md_document:
variant: markdown_github
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE)
```

### **Introduction**
The objective of this document is to illustrate via R programming  the data munging and analytical processes invloved between datasets listed below. GDP Rank dataset consists of countries by there GDP value and ED Stats dataset categorizes Countryies by the Income Group.  

#####        1) GDP Rank
#####        2) Ed Stats

### **Dowload Process**

#####        1) GDP Rank

```{r echo=TRUE}
######LOAD FILE INTO WORKSPACE AFTER SAVING INTO DIRECTORY 
gdp <-read.csv("GDP.csv", stringsAsFactors = TRUE, skip= 5, nrows = 231,  header  =FALSE)

```

#####        2) Ed Stats

```{r echo=TRUE}
######LOAD FILE INTO WORKSPACE AFTER SAVING INTO DIRECTORY 
edstat <-read.csv("Edstats_Country.csv", stringsAsFactors = TRUE,   header  =TRUE)

```

```{r echo=TRUE}
######LOAD PACKAGES
library(plyr)
library(dplyr)
library (ggplot2)
```
### **Tidying Process**
#####        1) GDP Rank


```{r echo=TRUE}
######NAME VARIBLES
 names(gdp)<- c("COUNTRYCODE", "RANKING", "JUNK1", "ECONOMY","US_DOLLARS_MILLIONS", "JUNK2", "JUNK3", "JUNK4", "JUNK5", "JUNK6")

######SPECIFIY VARIABLE SELECTION REQ'D FOR ANALYSIS
grossx<-select(gdp, COUNTRYCODE,ECONOMY ,RANKING,  US_DOLLARS_MILLIONS)
###### remove/replace "," with blanks 
grossx$US_DOLLARS_MILLIONS <- as.numeric(gsub("[^[:digit:]]","", grossx$US_DOLLARS_MILLIONS))
###### remove NAs from US_DOLLARS_MILLIONS and RANKING
grossx_sub<-subset(grossx , !is.na(US_DOLLARS_MILLIONS) & !is.na(RANKING) )

```

#####        2) Ed Stats
```{r echo=TRUE}

######NAME vARIBLES
names(edstat)[2]<- "SHORTNAME"
names(edstat)[9]<- "INCOMEGROUP"
names(edstat)[1]<- "COUNTRYCODE"

######SPECIFIY VARIABLE SELECTION REQ'D FOR ANALYSIS
edstat_gdp<-select(edstat, COUNTRYCODE,INCOMEGROUP, SHORTNAME)

edstat_sub<-subset(edstat_gdp , INCOMEGROUP != "" )

```


```{r echo=TRUE}

######CHECK VALUES/NAs
summary(edstat_sub)
summary(edstat_gdp)
######Analysis/data types
str(edstat_sub)
str(edstat_gdp)

######CHECK VALUES/NAs
summary(grossx)
summary(grossx_sub)
######Analysis/data types
str(grossx)
str(grossx_sub)

```

### **Merging Process For 1) GDP Rank & 2)Ed Stats DATASETS**

```{r echo=TRUE}

######MERGE DATA BY "**COUNTRYCODE**"
mergedata1<-merge(x=edstat_sub, y= grossx_sub , by = "COUNTRYCODE", all= TRUE)
head(mergedata1)

mergedata1_sub<-subset(mergedata1 , !is.na (US_DOLLARS_MILLIONS) )
```


####**Q&A's**
######**Q1.**	  *Match the data based on the country shortcode. How many of the IDs match?* 

######**ANS1.** *Based merge procedure between both datasets by countrycode resulted in total of 195, merge dataset was filtered for missing*                 *values on US_DOLLARS_MILLIONS , INCOMEGROUP and RANKING*

######**Q2.**	  *Sort the data frame in descending order by GDP rank (so United States is last). What is the 13th country in the resulting data frame?*

######**ANS2.** *The 13th country is **The Gambia**
```{r echo=TRUE}

######descending order by GDP RANKING
order_desc<-arrange (mergedata1_sub, desc(RANKING))
order_desc
```


######**Q3.**	  *What are the average GDP rankings for the "High income: OECD" and "High income: nonOECD" groups?* 

######**ANS3.** *Average GDP Ranking for **High income: nonOECD** 100.9655 and Average GDP Ranking for **High income: OECD** 34.35484*
```{r echo=TRUE}

######create dataset filtered by "High income: nonOECD"
HINOECD<-mergedata1_sub[mergedata1_sub$INCOMEGROUP=="High income: nonOECD" ,]
######create dataset filtered by "High income: OECD" 
HIOECD<-mergedata1_sub[mergedata1_sub$INCOMEGROUP=="High income: OECD" ,] 
mean(HINOECD$RANKING)
mean(HIOECD$RANKING)
```
######**Q4.**		*Plot the GDP for all of the countries. Use ggplot2 to color your plot by Income Group*.

######**ANS4.**
```{r echo=TRUE}

plot(mergedata1_sub$COUNTRYCODE, mergedata1_sub$US_DOLLARS_MILLIONS, xlab="COUNTRYCD", ylab="GDP MILLIONS")

#######GGPLOT
ggplot(data=mergedata1_sub, aes(x=mergedata1_sub$INCOMEGROUP, fill = INCOMEGROUP)) +
geom_density()
```

######**Q5.**  *Cut the GDP ranking into 5 separate quantile groups. Make a table versus Income.Group. How many countries are Lower middle income but among the 38 nations with highest GDP?*


######**ANS5.** *Total of 4 countries are lower middle income and among the 38 nations with highest GDP* 
```{r echo=TRUE}
#####transform ranking into quantiles of 5 add column "qtl"
mergedata1_sub_quan <- mergedata1_sub %>% transform(qtl= ntile(RANKING, 5))

###### create vector table 
ddply(mergedata1_sub_quan, c("qtl", "INCOMEGROUP"))

#### order ranking by ascending order 
order_asc<-arrange (mergedata1_sub_quan, (RANKING))
```


##### **Conclusion**
*To Summarize, the GDP Ranking 0f the 190 countries is categorize by five different Income Groups and based in US dollars. The US has the highest *GDP while Tuvalu has the least.*   