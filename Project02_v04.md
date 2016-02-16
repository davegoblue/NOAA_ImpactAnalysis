---
title: "Impact of US Weather Events in 1996-2010"
author: "davegoblue"
date: "February 16, 2016"
output: html_document
---

## Synopsis  
This analysis investigates the economic and health impact of US weather events based on an NOAA database spanning 1950-2011.  Weather event tracking in this NOAA database significantly expanded in the mid-1990s and the analysis focuses on 15 of the more recent years (1996-2010) with more complete data.  A January 2006 California flood event mis-coded in the NOAA data as $115 billion impact was excluded.

Economic impact (as reflected by property/crop damage) is primarily caused by hurricanes, storm surges, and floods.  These three events collectively drive ~60% of property/crop damage.  

Human health impact (reflected by injuries/fatalities, with fatalities weighted more heavily) is heavily driven by extreme heat and tornadoes.  These two events account for ~40% of injuries and fatalities.

While there is occasional overlap among top causes, it appears that different weather events tend to create large economic impacts vs. large health impacts.  The NOAA dataset attributes relatively few deaths and injuries to hurricanes (and their derivatives such as storm surges), which may underly this finding.

## Data Processing  

### Initial Data Acquisition and Loading
Data for this project was obtained from [Storm Data Download](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2).  The raw file is downloaded to the working directory, write-protected, and loaded using R.  


```r
myCSV <- read.csv("repdata-data-StormData.csv.bz2",stringsAsFactors = FALSE,na.strings=c("NA",""))
str(myCSV)
```

```
## 'data.frame':	902297 obs. of  37 variables:
##  $ STATE__   : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_DATE  : chr  "4/18/1950 0:00:00" "4/18/1950 0:00:00" "2/20/1951 0:00:00" "6/8/1951 0:00:00" ...
##  $ BGN_TIME  : chr  "0130" "0145" "1600" "0900" ...
##  $ TIME_ZONE : chr  "CST" "CST" "CST" "CST" ...
##  $ COUNTY    : num  97 3 57 89 43 77 9 123 125 57 ...
##  $ COUNTYNAME: chr  "MOBILE" "BALDWIN" "FAYETTE" "MADISON" ...
##  $ STATE     : chr  "AL" "AL" "AL" "AL" ...
##  $ EVTYPE    : chr  "TORNADO" "TORNADO" "TORNADO" "TORNADO" ...
##  $ BGN_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BGN_AZI   : chr  NA NA NA NA ...
##  $ BGN_LOCATI: chr  NA NA NA NA ...
##  $ END_DATE  : chr  NA NA NA NA ...
##  $ END_TIME  : chr  NA NA NA NA ...
##  $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
##  $ COUNTYENDN: logi  NA NA NA NA NA NA ...
##  $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ END_AZI   : chr  NA NA NA NA ...
##  $ END_LOCATI: chr  NA NA NA NA ...
##  $ LENGTH    : num  14 2 0.1 0 0 1.5 1.5 0 3.3 2.3 ...
##  $ WIDTH     : num  100 150 123 100 150 177 33 33 100 100 ...
##  $ F         : int  3 2 2 2 2 2 2 1 3 3 ...
##  $ MAG       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ PROPDMGEXP: chr  "K" "K" "K" "K" ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ CROPDMGEXP: chr  NA NA NA NA ...
##  $ WFO       : chr  NA NA NA NA ...
##  $ STATEOFFIC: chr  NA NA NA NA ...
##  $ ZONENAMES : chr  NA NA NA NA ...
##  $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
##  $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
##  $ LATITUDE_E: num  3051 0 0 0 0 ...
##  $ LONGITUDE_: num  8806 0 0 0 0 ...
##  $ REMARKS   : chr  NA NA NA NA ...
##  $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
```

Inspection of the file shows there are 902,297 observations across 37 variables.

### Filtering to maintain only relevant columns
Some columns are not relevant for this analysis.  We are particularly interested in event date "BGN_DATE"; event type "EVTYPE"; health impacts "FATALITIES" and "INJURIES"; and economic impacts "PROPDMG" and "CROPDMG".  Note that economic impacts are reported such that magnitude is contained in separate columns "PROPDMGEXP" and "CROPDMGEXP".

We also keep the STATE and REFNUM columns in case there are useful for follow-on analysis.


```r
myVars <- c("REFNUM","BGN_DATE","STATE","EVTYPE","FATALITIES","INJURIES",
            "PROPDMG","CROPDMG","PROPDMGEXP","CROPDMGEXP")
myFiltered <- myCSV[,myVars]
```

### Calculating the event year/decade and checking the resulting data
We also create a year and decade variable for future analysis and inspect the file contents.


```r
myFiltered$year <- as.numeric(format(as.Date(myFiltered$BGN_DATE,format="%m/%d/%Y"),"%Y"))
myFiltered$decade <- 10 * (myFiltered$year %/% 10)
str(myFiltered)
```

```
## 'data.frame':	902297 obs. of  12 variables:
##  $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
##  $ BGN_DATE  : chr  "4/18/1950 0:00:00" "4/18/1950 0:00:00" "2/20/1951 0:00:00" "6/8/1951 0:00:00" ...
##  $ STATE     : chr  "AL" "AL" "AL" "AL" ...
##  $ EVTYPE    : chr  "TORNADO" "TORNADO" "TORNADO" "TORNADO" ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ PROPDMGEXP: chr  "K" "K" "K" "K" ...
##  $ CROPDMGEXP: chr  NA NA NA NA ...
##  $ year      : num  1950 1950 1951 1951 1951 ...
##  $ decade    : num  1950 1950 1950 1950 1950 1950 1950 1950 1950 1950 ...
```

```r
summary(myFiltered[,c(5,6,7,8,11,12)])
```

```
##    FATALITIES          INJURIES            PROPDMG       
##  Min.   :  0.0000   Min.   :   0.0000   Min.   :   0.00  
##  1st Qu.:  0.0000   1st Qu.:   0.0000   1st Qu.:   0.00  
##  Median :  0.0000   Median :   0.0000   Median :   0.00  
##  Mean   :  0.0168   Mean   :   0.1557   Mean   :  12.06  
##  3rd Qu.:  0.0000   3rd Qu.:   0.0000   3rd Qu.:   0.50  
##  Max.   :583.0000   Max.   :1700.0000   Max.   :5000.00  
##     CROPDMG             year          decade    
##  Min.   :  0.000   Min.   :1950   Min.   :1950  
##  1st Qu.:  0.000   1st Qu.:1995   1st Qu.:1990  
##  Median :  0.000   Median :2002   Median :2000  
##  Mean   :  1.527   Mean   :1999   Mean   :1994  
##  3rd Qu.:  0.000   3rd Qu.:2007   3rd Qu.:2000  
##  Max.   :990.000   Max.   :2011   Max.   :2010
```

```r
table(myCSV$PROPDMGEXP,myCSV$CROPDMGEXP,useNA="always")
```

```
##       
##             ?      0      2      B      k      K      m      M   <NA>
##   -         0      0      0      0      0      0      0      0      1
##   ?         0      0      0      0      0      0      0      0      8
##   +         0      0      0      0      0      0      0      0      5
##   0         0      0      0      0      0      4      0      1    211
##   1         0      0      0      0      0      0      0      0     25
##   2         0      0      0      0      0      0      0      0     13
##   3         0      0      0      0      0      1      0      0      3
##   4         0      0      0      0      0      0      0      0      4
##   5         0      0      0      0      0      1      0      1     26
##   6         0      0      0      0      0      0      0      0      4
##   7         0      0      0      0      0      0      0      0      5
##   8         0      0      0      0      0      0      0      0      1
##   B         0      0      0      2      0     11      0     11     16
##   h         0      0      0      0      0      0      0      0      1
##   H         0      0      0      0      0      0      0      0      6
##   K         4     16      0      3     21 274690      0    864 149067
##   m         0      0      0      0      0      0      1      0      6
##   M         1      0      0      0      0   3260      0    674   7395
##   <NA>      2      3      1      4      0   3865      0    443 461616
```

The summary statistics suggest a succesful conversion to numeric year/decade, with 50% of the records being from 1995-2007.  There are no NA records in the numeric fields that we plan to use.  We maintained all of the 902,297 observations along with the 10 key variables and 2 new derived variables (year and decade).

### Calculating property/crop damage per event  
#### _Identify and plan around anomalous magnitudes for crop and property damage_  
Inspection of the PROPDMGEXP and CROPDMGEXP table above, along with exploration of the associated records, suggests K/k is for thousands, M/m is for millions, and B is for billions (e.g., huge hurricane).

There are 321 records with PROPDMGEXP symbols that cannot be interpreted and 27 records with CROPDMGEXP symbols that cannot be interpreted.  Manual inspection of a sample of these records shows them to be unremarkable and they are all treated as having $0 of damage.  These anomalies occured primarily in 1995.

Further, there are 76 records where PROPDMGEXP is NA while PROPDMG > 0; and 3 records where CROPDMGEXP is NA while CROPDMG > 0.  These anomalies all occur in 1993-1995 and inspection of a sample of these records shows them to be unremarkable.  These anomalies will be treated as having $0 of damage.

#### _Convert property and crop damage to dollars based on k/K or m/M or B_  
We convert all damage to millions of dollars for ease of future reporting.  If the PROPDMGEXP or CROPDMGEXP variable is not interpretable, it will stick with the default zero multiplier.  


```r
myFiltered$multProp <- 0
myFiltered$multProp[which(myFiltered$PROPDMGEXP %in% c("k","K"))] <- .001
myFiltered$multProp[which(myFiltered$PROPDMGEXP %in% c("m","M"))] <- 1
myFiltered$multProp[which(myFiltered$PROPDMGEXP %in% c("b","B"))] <- 1000

myFiltered$multCrop <- 0
myFiltered$multCrop[which(myFiltered$CROPDMGEXP %in% c("k","K"))] <- .001
myFiltered$multCrop[which(myFiltered$CROPDMGEXP %in% c("m","M"))] <- 1
myFiltered$multCrop[which(myFiltered$CROPDMGEXP %in% c("b","B"))] <- 1000

myFiltered$dollarProp <- myFiltered$PROPDMG * myFiltered$multProp
myFiltered$dollarCrop <- myFiltered$CROPDMG * myFiltered$multCrop

dim(myFiltered)
```

```
## [1] 902297     16
```

```r
colnames(myFiltered)
```

```
##  [1] "REFNUM"     "BGN_DATE"   "STATE"      "EVTYPE"     "FATALITIES"
##  [6] "INJURIES"   "PROPDMG"    "CROPDMG"    "PROPDMGEXP" "CROPDMGEXP"
## [11] "year"       "decade"     "multProp"   "multCrop"   "dollarProp"
## [16] "dollarCrop"
```

```r
set.seed(216160957)
myFiltered[sample(nrow(myFiltered),15,replace=FALSE),][c(1,8,10,13,15),]  ## Rows of some interest
```

```
##        REFNUM          BGN_DATE STATE      EVTYPE FATALITIES INJURIES
## 833635 833565 9/17/2010 0:00:00    NE        HAIL          0        0
## 802694 802624 5/23/2010 0:00:00    NV  DUST DEVIL          0        0
## 849334 849334 4/23/2011 0:00:00    MO        HAIL          0        0
## 893588 893539 8/28/2011 0:00:00    VT       FLOOD          0        0
## 530112 531181 5/23/2004 0:00:00    IL FLASH FLOOD          0        0
##        PROPDMG CROPDMG PROPDMGEXP CROPDMGEXP year decade multProp multCrop
## 833635       0       0          K          K 2010   2010    0.001    0.001
## 802694      50       0          K          K 2010   2010    0.001    0.001
## 849334      50       0          K          K 2011   2010    0.001    0.001
## 893588     500     500          M          K 2011   2010    1.000    0.001
## 530112       0       0       <NA>       <NA> 2004   2000    0.000    0.000
##        dollarProp dollarCrop
## 833635      0e+00        0.0
## 802694      5e-02        0.0
## 849334      5e-02        0.0
## 893588      5e+02        0.5
## 530112      0e+00        0.0
```

```r
myFiltered[myFiltered$multProp==1000,][sample(40,6,replace=FALSE),]
```

```
##        REFNUM          BGN_DATE STATE            EVTYPE FATALITIES
## 298088 298057 4/18/1997 0:00:00    ND             FLOOD          0
## 834674 834634 10/5/2010 0:00:00    AZ              HAIL          0
## 529498 529446 9/13/2004 0:00:00    FL HURRICANE/TYPHOON          7
## 860386 860355 4/27/2011 0:00:00    AL           TORNADO         44
## 529363 529311 8/13/2004 0:00:00    FL         HIGH WIND          4
## 366694 366653 9/15/1999 0:00:00    NC         HURRICANE          0
##        INJURIES PROPDMG CROPDMG PROPDMGEXP CROPDMGEXP year decade multProp
## 298088        0     3.0       0          B       <NA> 1997   1990     1000
## 834674        1     1.8       0          B          K 2010   2010     1000
## 529498        0     4.0      25          B          M 2004   2000     1000
## 860386      800     1.5       0          B          K 2011   2010     1000
## 529363        0     1.3       0          B       <NA> 2004   2000     1000
## 366694        0     3.0     500          B          M 1999   1990     1000
##        multCrop dollarProp dollarCrop
## 298088    0.000       3000          0
## 834674    0.001       1800          0
## 529498    1.000       4000         25
## 860386    0.001       1500          0
## 529363    0.000       1300          0
## 366694    1.000       3000        500
```

As expected, myFiltered still contains 902,297 observations.  The original 10 key variables plus year/decade remain.  Four new variables for calculating/reporting the dollar value of damage have been added.  

The 2011 flood in Vermont correctly converts 500M to 500, while records with k/K correctly divide by 1000.  A sampling of records with billions of reported property damage reveals catastrophic events commonly associated with reports of widespread property destruction.  Conversion to millions of dollars appears as intended in these samples.

Lastly, we examine the database totals for the key numeric variables (recall that the conversions for dollarCrop and dollarProp made each of these represent millions of dollars).  


```r
colSums(myFiltered[,c(5:8,15:16)])
```

```
##  FATALITIES    INJURIES     PROPDMG     CROPDMG  dollarProp  dollarCrop 
##    15145.00   140528.00 10884500.01  1377827.32   427318.64    49104.19
```

Of note, the estimated $49 billion of crop damage and $427 billion of property damage over ~60 years is far more plausible than the unconverted $11 million of property damage and $1.4 million of crop damage.

### Aggregating event types for further analysis
#### _Declare an event type by finding key words in EVTYPE_

The EVTYPE variable is a bit messy, and we clean it using key word searches.  Exploratory analysis was run to understand key words associated with EVTYPES causing the strong majority of economic and health impacts.  The exploratory analysis is not shown for purposes of brevity.

A strategy to convert EVTYPE to eventType was developed, with the default being to declare eventType as "All Other" unless a specific keyword can be found.  The priority of the assignments is the reverse order of the variables declared in myShortEvent.  That is to say that if EVTYPE were "Thunderstorm with high wind due to tornado", then it would map to TORNADO and not to WIND or THUNDERSTORM.  The general intent was to prioritize more descriptive/severe event names over more general event names.


```r
myShortEvent <- c("WINTER","WIND","FREEZE","COLD","HEAT","RAIN","SNOW","ICE","HAIL","FLD","FLOOD",
                  "TSUNAMI","SURGE","DROUGHT","FOG","DUST","TSTM","THUNDERSTORM","LIGHTNING","FIRE",
                  "BLIZZARD","AVALANCHE","FLASH","SURF","RIP","TYPHOON","HURRICANE","TROPICAL","TORNADO"
                  )

myFiltered$eventType <- "All Other"

for (myMatch in myShortEvent) {
    myFiltered$eventType[grep(myMatch,myFiltered$EVTYPE,ignore.case=TRUE)] <- myMatch
}
```

Further, a few event types were consolidated as they appear highly related to one another.


```r
myFiltered$eventType[myFiltered$eventType %in% c("HURRICANE","TYPHOON")] <- "TROPICAL"
myFiltered$eventType[myFiltered$eventType=="SURF"] <- "RIP"
myFiltered$eventType[myFiltered$eventType=="TSTM"] <- "THUNDERSTORM"
myFiltered$eventType[myFiltered$eventType %in% c("FLD")] <- "FLOOD"
myFiltered$eventType[myFiltered$eventType %in% c("TSUNAMI")] <- "FLASH"
myFiltered$eventType[myFiltered$eventType=="FREEZE"] <- "COLD"
myFiltered$eventType[myFiltered$EVTYPE %in% c("NON TSTM WIND","NON-TSTM WIND")] <- "WIND"
```

#### _Declare an event class by further grouping eventType_
The event types were further aggregated to simplify some of the downstream reporting.


```r
myFiltered$masterType <- "Other"

myFiltered[myFiltered$eventType %in% c("HAIL","ICE","LIGHTNING","RAIN","SNOW",
                                       "THUNDERSTORM","TORNADO","TROPICAL","WINTER"),]$masterType <- "Storm"

myFiltered[myFiltered$eventType %in% c("COLD","DROUGHT","HEAT"),]$masterType <- "Temp/Dew Point"
myFiltered[myFiltered$eventType %in% c("BLIZZARD","WIND","DUST"),]$masterType <- "Wind"
myFiltered[myFiltered$eventType %in% c("FLASH","FLOOD","RIP","SURGE"),]$masterType <- "Water"
myFiltered[myFiltered$eventType %in% c("AVALANCHE","FIRE","FOG"),]$masterType <- "Assorted"
```

### Calculating a single number for economic impact, health impact, and total impact   
The focus of this analysis is to understand the impact of weather events on the economy and human health.  Additional variables are created to assess these overall impacts:  

* econImpact is the straight sum of crop damage and property damage, in millions of dollars
* healthImpact is an indexed aggregate of fatalities and injuries; the value of a human life lost is indexed as $5 million while the value of a human injury is indexed as $100,000; the sum of these indices is defined as healthImpact, in millions of dollars
* totalImpact is the straight sum of econImpact and healthImpact, in millions of dollars


```r
myFiltered$econImpact <- myFiltered$dollarProp + myFiltered$dollarCrop ## in millions
myFiltered$healthImpact <- 0.1 * myFiltered$INJURIES + 5 * myFiltered$FATALITIES ## in millions
myFiltered$totalImpact <- myFiltered$econImpact + myFiltered$healthImpact ## in millions
dim(myFiltered)
```

```
## [1] 902297     21
```

```r
colSums(myFiltered[,c(5:6,15:16,19:21)])
```

```
##   FATALITIES     INJURIES   dollarProp   dollarCrop   econImpact 
##     15145.00    140528.00    427318.64     49104.19    476422.83 
## healthImpact  totalImpact 
##     89777.80    566200.63
```

```r
print(paste0("Property damage represents ",round(100*sum(myFiltered[,15])/sum(myFiltered[,19]),0),
             "% of the $",round(sum(myFiltered[,19])/1000,0)," billion of indexed economic impact"
             )
      )
```

```
## [1] "Property damage represents 90% of the $476 billion of indexed economic impact"
```

```r
print(paste0("Fatalities represent ",round(5*100*sum(myFiltered[,5])/sum(myFiltered[,20]),0),
             "% of the $",round(sum(myFiltered[,20])/1000,0)," billion of indexed health impact"
             )
      )
```

```
## [1] "Fatalities represent 84% of the $90 billion of indexed health impact"
```

```r
print(paste0("Economic impact represents ",round(100*sum(myFiltered[,19])/sum(myFiltered[,21]),0),
             "% of the $",round(sum(myFiltered[,21])/1000,0)," billion of indexed total impact"
             )
      )
```

```
## [1] "Economic impact represents 84% of the $566 billion of indexed total impact"
```

myFiltered continues to have all 902,297 observations.  In addition to the 16 variables previously carried, we have added descriptors for each event (eventType, masterType) and indexed millions of dollars of impact for each category (econImpact, healthImpact, totalImpact).  There are now 21 variables.

While there are many more injuries than fatalities, the assumption that loss of human life is of much greater impact than injury drives fatalities to represent 84% of the indexed health impact.  Property damage is of order of magnitude greater than crop damage, representing 90% of the indexed economic impact.  Economic impact is generally much greater than human impact, as expected of weather impacting highly-developed areas.  These relative impacts are sensitive to indexing assumptions for fatality at $5 million and injury at $100,000.  


## Results

### Can we use all years of NOAA data for our analysis?
#### _What is the pattern by year and impact type?_
In preparation for graphing, we aggregate the data by year and masterType.  Further, we brew a qualitative color palette to help with interpretation.  The RColorBrewer library is required.


```r
yearTotals <- aggregate(cbind(econImpact,healthImpact,totalImpact) ~ year + masterType,
                        data=myFiltered,FUN=sum
)

yearGraph <- reshape(yearTotals[,c(1,2,5)],v.names="totalImpact",timevar="masterType",
                     idvar="year",direction="wide")

yearGraph <- yearGraph[order(yearGraph$year),]

library(RColorBrewer)
myCol <- brewer.pal(6,"Accent")
```

A stacked bar chart reveals the data underwent a significant transition in 1993 when additional event types began to be tracked.


```r
par(las=2)
par(mar=c(5,5,4,2))
barplot(t(yearGraph[,c(4,6,5,7,2,3)])/1000,names.arg=yearGraph$year,col=myCol[6:1],cex.names=0.5)
legend("topleft",legend=c("Other","Misc","Wind","T/D","Water","Storm"),fill=myCol)
title(main="Index of Total Weather Impact by Year",ylab="Indexed Impact ($ billions)",xlab="Year")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)

There is significant spikiness to the total impact data, with pronounced peaks in:

* 2005 - mix of storm and water, revealed by further exploration to be driven by massive destruction attributed to Hurricane Katrina's August 2005 landfall on the LA and MS coastlines (with associated storm surge)
* 2006 - revealed by further exploration to be almost entirely driven by a single flooding event in CA in January 2006

The Katrina impact in this data is in the ballpark of the ~$100 billion of damage commonly associated to the storm.  Many of the ~2,000 deaths commonly attributed to Katrina are not in the database.  NOAA documentation notes death tracking can be finnicky, particularly when caused directly vs. indirectly by weather events.

The 2006 spike is obvious data error introduced by REFNUM 605943.  There was a flood in California, but with impact of ~$100 million.  Due to data entry error, this was coded as a $115 billion event.  Excluding  this record, the balance of 2006 has $13.8 billion of total impact, in line with other years.

#### _What do we do about it?_  
The remainder of this analysis will focus only on the years 1996-2010.  Data before 1993 is incomplete as per the above bar chart.  This document previously noted some anomalous characters for PROPDMGEXP and CROPDMGEXP in 1993-1995.  These three years not being vital to answering key questions, we exclude the years as possible teething issues associated with the methodology change.  Further, this database is of vintage November 2011, with attendant uncertainty as to the completeness and validation of records reported for 2011.

Further, REFNUM 605943 is deleted as it is clear data entry error with very large analysis impact.


```r
yearAnalyze <- subset(myFiltered,c(year >= 1996 & year <= 2010 & REFNUM != 605943) )
dim(yearAnalyze)
```

```
## [1] 591355     21
```

There are 591,335 observations remaining for analysis, including the 21 variables described previously.

Each REFNUM for the top-24 events by total-impact was examined in the raw myCSV file.  There is no other obvious outlier error and most of the data match to large tropical storms and/or major floods, both known to cause extensive property damage.  I am somewhat skeptical about reported impact in the billions of dollars for AZ hail event 834634, NM fire event 398999, CA fire event 488004, and TN flooding event 808257.  I lack domain expertise to delete these and therefore retained them for this analysis.

Please use caution in interpreting the remainder of this report.  The NOAA data may need further scrubbing to refine these initial findings.


```r
## REFNUM associated with the top-24 impact-causing events
yearAnalyze[order(-yearAnalyze$totalImpact),]$REFNUM[1:24]
```

```
##  [1] 577616 577615 581535 569288 581537 581533 529299 444407 529384 529446
## [11] 739515 577623 366653 298057 525145 598472 347811 834634 808257 569065
## [21] 398999 529311 488004 529307
```

### What events have the greatest impact?
#### _Final pre-processing and aggregation_  
The data were aggregated by eventType.  A mapping file was also applied to give more descritpive names to the weather event types.


```r
causeSum <- aggregate(cbind(FATALITIES,INJURIES,dollarProp,dollarCrop,econImpact,healthImpact,totalImpact) ~
                      eventType,data=yearAnalyze,FUN=sum)

myMap <- data.frame(origName = c("All Other","AVALANCHE","BLIZZARD","COLD",
                                 "DROUGHT","DUST","FIRE","FLASH",
                                 "FLOOD","FOG","HAIL","HEAT",
                                 "ICE","LIGHTNING","RAIN","RIP",
                                 "SNOW","THUNDERSTORM","TORNADO","TROPICAL",
                                 "WIND","WINTER","SURGE"),
                    modName = c("All Other","Avalanche","Blizzard","Extreme Cold",
                                "Drought","Dust Storm","Wildfire","Flash Flooding",
                                "Flood (non-flash)","Fog","Hail","Excessive Heat",
                                "Ice Storm","Lightning Strike","Rain","Rip Current/Surf",
                                "Snow","Thunderstorm","Tornado","Hurricane or TS",
                                "Wind","Wintry Mess","Storm Surge"),
                    stringsAsFactors=FALSE
)

for (strNames in myMap$origName){
    causeSum$descName[causeSum$eventType == strNames] <- myMap$modName[myMap$origName==strNames]
}
```

#### _Greatest economic impacts_  
The top-10 events by total economic impact are calculated, with a bar-plot produced to show the proportion of total 1996-2010 economic (crop and property damge) impact associated to that event.

Plainly, hurricanes, storm surges, and flooding cause the strong majority of economic impact.  While Katrina plays a big role in driving this, it is not uncommon for the US to experience major hurricance landfalls and extensive floods.  An insurance company or public safety agency would rightly see all as significant hazards.


```r
myPlot <- causeSum[order(-causeSum$econImpact),]

par(las=1)
par(mar=c(6,8,4,2))
barplot(myPlot$econImpact[10:1]/sum(myPlot$econImpact),
        names.arg=myPlot$descName[10:1],horiz=TRUE,
        col="blue",xlim=c(0,0.4)
)

title(main="Impact of top-10 (by economics) weather events in 1996-2010",
      xlab="% of economic impact caused (sum of crop damage and property damage)"
)
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15-1.png)

```r
colSums(myPlot[1:10,2:8])/colSums(myPlot[,2:8])
```

```
##   FATALITIES     INJURIES   dollarProp   dollarCrop   econImpact 
##    0.4102199    0.6595461    0.9647383    0.8808209    0.9539518 
## healthImpact  totalImpact 
##    0.4388721    0.8810658
```

These top-10 economic events account for ~95% of the economic impact but less than 50% of the health impact.

% Caused | Fatality | Injury | Property | Crop   | Economic | Health
---------|----------|--------|----------|--------|----------|--------
top-10   |  41%     |  66%   |   96%    |  88%   |   95%    |  44%


#### _Greatest health impacts_  
The top-10 events by total health impact are calculated, with a bar-plot produced to show the proportion of total 1996-2010 health (indexed fatality plus injury) impact associated to that event.

Excessive heat, tornadoes, and flash floods are the largest drivers of total health impact due to weather in 1996-2010.  Notably, of the top-3 economic drivers (hurricane, storm surge, flood), only floods make the top-10 for health impact.  This is likely related to the comment above about few deaths in this NOAA dataset being attributed to Katrina and similar hurricanes.

As a reminder, "flash flooding" is broken out separately from "flooding" (widespread river floods) and "storm surge".  Despite this, "flash flooding" ranks third in causing human health impact.


```r
myPlot <- causeSum[order(-causeSum$healthImpact),]

par(las=1)
par(mar=c(6,8,4,2))
barplot(myPlot$healthImpact[10:1]/sum(myPlot$healthImpact),
        names.arg=myPlot$descName[10:1],horiz=TRUE,
        col="orange",xlim=c(0,0.4)
)

title(main="Impact of top-10 weather events (by human health) in 1996-2010",
      xlab="% of injury-equivalents caused (each fatality treated as 50 injury-equivalents)"
)
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-16-1.png)

```r
colSums(myPlot[1:10,2:8])/colSums(myPlot[,2:8])
```

```
##   FATALITIES     INJURIES   dollarProp   dollarCrop   econImpact 
##    0.8648124    0.8536556    0.2850750    0.3241655    0.2900995 
## healthImpact  totalImpact 
##    0.8635303    0.3712425
```

These top-10 health events account for ~85% of the health impact but less than 30% of the economic impact.

% Caused | Fatality | Injury | Property | Crop   | Economic | Health
---------|----------|--------|----------|--------|----------|---------
top-10   |  86%     |  85%   |   29%    |  32%   |   29%    |  86%

  
  
## Addendum #1 - EVTYPE mapping  
This addendum is to provide further description of the implication of aggregating raw EVTYPE in to more descriptive eventType. See below for the EVTYPE associated to each aggregate description.  Only EVTYPE totalling at least $99 million over 1996-2010 are shown.    


```r
testValue <- aggregate(cbind(totalImpact,econImpact,healthImpact,dollarProp,dollarCrop,FATALITIES,INJURIES) ~
                             eventType + EVTYPE,data=yearAnalyze,FUN=sum
                       )

for (strNames in myMap$origName){
    testValue$descName[testValue$eventType == strNames] <- myMap$modName[myMap$origName==strNames]
}

testValue <- testValue[order(testValue$descName,-testValue$totalImpact),]

testValue$totalImpact <- round(testValue$totalImpact,1)
testValue$dollarProp <- round(testValue$dollarProp,1)
testValue$dollarCrop <- round(testValue$dollarCrop,1)

testValue[testValue$totalImpact>=99,c(10,2:3,6:9)]
```

```
##              descName                  EVTYPE totalImpact dollarProp
## 206         All Other               LANDSLIDE       508.6      303.4
## 16          Avalanche               AVALANCHE      1088.5        3.7
## 23           Blizzard                BLIZZARD       908.5      522.9
## 63            Drought                 DROUGHT     14382.7     1046.0
## 74         Dust Storm              DUST STORM        99.9        4.6
## 81     Excessive Heat          EXCESSIVE HEAT      9929.3        6.6
## 147    Excessive Heat                    HEAT       932.8        1.5
## 89       Extreme Cold            EXTREME COLD      1881.6       19.8
## 122      Extreme Cold            FROST/FREEZE      1084.6        3.9
## 90       Extreme Cold EXTREME COLD/WIND CHILL       619.0        1.6
## 52       Extreme Cold         COLD/WIND CHILL       373.6        1.9
## 108      Extreme Cold                  FREEZE       146.4        0.2
## 98     Flash Flooding             FLASH FLOOD     19344.0    13838.2
## 447    Flash Flooding                 TSUNAMI       263.4       90.5
## 102 Flood (non-flash)                   FLOOD     28469.4    21227.2
## 473 Flood (non-flash)    URBAN/SML STREAM FLD       214.7       58.3
## 34  Flood (non-flash)           COASTAL FLOOD       212.3      202.2
## 303 Flood (non-flash)          River Flooding       134.3      106.2
## 37  Flood (non-flash)        COASTAL FLOODING       102.5       97.5
## 106               Fog                     FOG       384.3       13.1
## 142              Hail                    HAIL     16640.7    14143.8
## 185   Hurricane or TS       HURRICANE/TYPHOON     72361.2    69305.8
## 183   Hurricane or TS               HURRICANE     14842.8    11802.3
## 430   Hurricane or TS          TROPICAL STORM      8455.6     7503.7
## 448   Hurricane or TS                 TYPHOON       601.6      600.2
## 196         Ice Storm               ICE STORM      4091.8     3634.4
## 224  Lightning Strike               LIGHTNING      4222.6      696.1
## 154              Rain              HEAVY RAIN      1768.4      573.1
## 300  Rip Current/Surf             RIP CURRENT      1573.2        0.0
## 301  Rip Current/Surf            RIP CURRENTS      1039.6        0.2
## 171  Rip Current/Surf               HIGH SURF       476.8       83.3
## 167  Rip Current/Surf    HEAVY SURF/HIGH SURF       224.7        9.9
## 161              Snow              HEAVY SNOW      1294.2      618.3
## 342       Storm Surge             STORM SURGE     43207.2    43193.5
## 343       Storm Surge        STORM SURGE/TIDE      4656.8     4600.5
## 434      Thunderstorm               TSTM WIND      6599.8     4478.0
## 421      Thunderstorm       THUNDERSTORM WIND      3742.0     3000.8
## 444      Thunderstorm          TSTM WIND/HAIL       143.5       44.3
## 426           Tornado                 TORNADO     21119.8    14797.3
## 496          Wildfire                WILDFIRE      4820.5     4110.3
## 495          Wildfire        WILD/FOREST FIRE      3223.1     3001.8
## 177              Wind               HIGH WIND      7057.4     5205.9
## 346              Wind             STRONG WIND       697.6      158.2
## 92               Wind       EXTREME WINDCHILL       103.3        0.8
## 498              Wind                    WIND       101.0        2.3
## 507       Wintry Mess            WINTER STORM      2605.7     1514.6
## 509       Wintry Mess          WINTER WEATHER       223.3       19.0
## 511       Wintry Mess      WINTER WEATHER/MIX       153.6        6.4
##     dollarCrop FATALITIES INJURIES
## 206       20.0         36       52
## 16         0.0        214      148
## 23         7.1         68      385
## 63     13336.3          0        4
## 74         3.1         11      372
## 81       492.4       1761     6253
## 147        0.2        174      611
## 89      1289.0        113       79
## 122     1080.7          0        0
## 90         0.0        123       23
## 52         0.6         74       11
## 108      146.2          0        0
## 98      1246.5        819     1644
## 447        0.0         32      129
## 102     4787.4        356     6748
## 473        8.5         28       79
## 34         0.0          2        1
## 303       28.0          0        1
## 37         0.0          1        0
## 106        0.0         60      712
## 142     2393.7          7      682
## 185     2607.9         64     1275
## 183     2730.9         61       46
## 430      653.2         53      337
## 448        0.8          0        5
## 196       15.6         82      318
## 224        6.8        625     3947
## 154      707.5         93      229
## 300        0.0        311      182
## 301        0.0        202      294
## 171        0.0         76      135
## 167        0.0         42       48
## 161       71.1        107      698
## 342        0.0          2       37
## 343        0.8         11        5
## 434      553.9        241     3629
## 421      258.5         76     1027
## 444       64.7          5       95
## 426      252.1        924    14504
## 496      285.7         69      795
## 495      106.8         12      545
## 177      589.3        231     1072
## 346       49.9         93      245
## 92        17.0         17        5
## 498        0.3         18       84
## 507       11.9        190     1292
## 509       15.0         31      343
## 511        0.0         28       72
```

```r
colSums(testValue[testValue$totalImpact>=99,c(3,6:9)])/colSums(testValue[testValue$totalImpact>=0,c(3,6:9)])
```

```
## totalImpact  dollarProp  dollarCrop  FATALITIES    INJURIES 
##   0.9952317   0.9990311   0.9937128   0.9758085   0.9803718
```

The EVTYPE shown above account for 97%-99% of the impact of the various events.


## Addendum #2 - Top-10 events for each type of individual impact  
The raw data are such that the indexing and aggregation methodologies lead crop damage and injuries to take a back-seat to property damage and fatalities.

Below are the top-10 EVTYPE for each of dollarCrop, INJURIES, dollarProp, and FATALITIES.  Drought in particular drives a lot of crop damage, but crop damage is just 10% of economic damge and since drought causes negligible property damage, it never appears as a top cause in this report.


```r
for (myString in c("dollarCrop","INJURIES","dollarProp","FATALITIES")) {
    myLoc <- grep(myString,colnames(testValue))
    testValue <- testValue[order(-testValue[,myLoc]),]
    print("--------------------")
    print(paste0("----- Data for top-10 drivers of: ",myString))
    print(testValue[1:10,c(2,10,3,6:9)])
    print("--------------------")
    print(paste0("These top 10 EVTYPE for: ",myString," represent the below proportions of total"))
    print(colSums(testValue[1:10,c(3,6:9)])/colSums(testValue[,c(3,6:9)]))
}
```

```
## [1] "--------------------"
## [1] "----- Data for top-10 drivers of: dollarCrop"
##                EVTYPE          descName totalImpact dollarProp dollarCrop
## 63            DROUGHT           Drought     14382.7     1046.0    13336.3
## 102             FLOOD Flood (non-flash)     28469.4    21227.2     4787.4
## 183         HURRICANE   Hurricane or TS     14842.8    11802.3     2730.9
## 185 HURRICANE/TYPHOON   Hurricane or TS     72361.2    69305.8     2607.9
## 142              HAIL              Hail     16640.7    14143.8     2393.7
## 89       EXTREME COLD      Extreme Cold      1881.6       19.8     1289.0
## 98        FLASH FLOOD    Flash Flooding     19344.0    13838.2     1246.5
## 122      FROST/FREEZE      Extreme Cold      1084.6        3.9     1080.7
## 154        HEAVY RAIN              Rain      1768.4      573.1      707.5
## 430    TROPICAL STORM   Hurricane or TS      8455.6     7503.7      653.2
##     FATALITIES INJURIES
## 63           0        4
## 102        356     6748
## 183         61       46
## 185         64     1275
## 142          7      682
## 89         113       79
## 98         819     1644
## 122          0        0
## 154         93      229
## 430         53      337
## [1] "--------------------"
## [1] "These top 10 EVTYPE for: dollarCrop represent the below proportions of total"
## totalImpact  dollarProp  dollarCrop  FATALITIES    INJURIES 
##   0.5807880   0.6040589   0.9054312   0.2025873   0.2200745 
## [1] "--------------------"
## [1] "----- Data for top-10 drivers of: INJURIES"
##                EVTYPE          descName totalImpact dollarProp dollarCrop
## 426           TORNADO           Tornado     21119.8    14797.3      252.1
## 102             FLOOD Flood (non-flash)     28469.4    21227.2     4787.4
## 81     EXCESSIVE HEAT    Excessive Heat      9929.3        6.6      492.4
## 224         LIGHTNING  Lightning Strike      4222.6      696.1        6.8
## 434         TSTM WIND      Thunderstorm      6599.8     4478.0      553.9
## 98        FLASH FLOOD    Flash Flooding     19344.0    13838.2     1246.5
## 507      WINTER STORM       Wintry Mess      2605.7     1514.6       11.9
## 185 HURRICANE/TYPHOON   Hurricane or TS     72361.2    69305.8     2607.9
## 177         HIGH WIND              Wind      7057.4     5205.9      589.3
## 421 THUNDERSTORM WIND      Thunderstorm      3742.0     3000.8      258.5
##     FATALITIES INJURIES
## 426        924    14504
## 102        356     6748
## 81        1761     6253
## 224        625     3947
## 434        241     3629
## 98         819     1644
## 507        190     1292
## 185         64     1275
## 177        231     1072
## 421         76     1027
## [1] "--------------------"
## [1] "These top 10 EVTYPE for: INJURIES represent the below proportions of total"
## totalImpact  dollarProp  dollarCrop  FATALITIES    INJURIES 
##   0.5685398   0.5806990   0.3173448   0.6839586   0.8248012 
## [1] "--------------------"
## [1] "----- Data for top-10 drivers of: dollarProp"
##                EVTYPE          descName totalImpact dollarProp dollarCrop
## 185 HURRICANE/TYPHOON   Hurricane or TS     72361.2    69305.8     2607.9
## 342       STORM SURGE       Storm Surge     43207.2    43193.5        0.0
## 102             FLOOD Flood (non-flash)     28469.4    21227.2     4787.4
## 426           TORNADO           Tornado     21119.8    14797.3      252.1
## 142              HAIL              Hail     16640.7    14143.8     2393.7
## 98        FLASH FLOOD    Flash Flooding     19344.0    13838.2     1246.5
## 183         HURRICANE   Hurricane or TS     14842.8    11802.3     2730.9
## 430    TROPICAL STORM   Hurricane or TS      8455.6     7503.7      653.2
## 177         HIGH WIND              Wind      7057.4     5205.9      589.3
## 343  STORM SURGE/TIDE       Storm Surge      4656.8     4600.5        0.8
##     FATALITIES INJURIES
## 185         64     1275
## 342          2       37
## 102        356     6748
## 426        924    14504
## 142          7      682
## 98         819     1644
## 183         61       46
## 430         53      337
## 177        231     1072
## 343         11        5
## [1] "--------------------"
## [1] "These top 10 EVTYPE for: dollarProp represent the below proportions of total"
## totalImpact  dollarProp  dollarCrop  FATALITIES    INJURIES 
##   0.7652467   0.8905932   0.4481713   0.3270375   0.5250782 
## [1] "--------------------"
## [1] "----- Data for top-10 drivers of: FATALITIES"
##             EVTYPE          descName totalImpact dollarProp dollarCrop
## 81  EXCESSIVE HEAT    Excessive Heat      9929.3        6.6      492.4
## 426        TORNADO           Tornado     21119.8    14797.3      252.1
## 98     FLASH FLOOD    Flash Flooding     19344.0    13838.2     1246.5
## 224      LIGHTNING  Lightning Strike      4222.6      696.1        6.8
## 102          FLOOD Flood (non-flash)     28469.4    21227.2     4787.4
## 300    RIP CURRENT  Rip Current/Surf      1573.2        0.0        0.0
## 434      TSTM WIND      Thunderstorm      6599.8     4478.0      553.9
## 177      HIGH WIND              Wind      7057.4     5205.9      589.3
## 16       AVALANCHE         Avalanche      1088.5        3.7        0.0
## 301   RIP CURRENTS  Rip Current/Surf      1039.6        0.2        0.0
##     FATALITIES INJURIES
## 81        1761     6253
## 426        924    14504
## 98         819     1644
## 224        625     3947
## 102        356     6748
## 300        311      182
## 434        241     3629
## 177        231     1072
## 16         214      148
## 301        202      294
## [1] "--------------------"
## [1] "These top 10 EVTYPE for: FATALITIES represent the below proportions of total"
## totalImpact  dollarProp  dollarCrop  FATALITIES    INJURIES 
##   0.3254818   0.2609744   0.2328219   0.7353169   0.7656178
```

