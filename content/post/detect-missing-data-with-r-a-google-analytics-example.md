---
title: "Detect missing data in GA with R"
summary: "How to compare data from GA to another dataset to see if there are any discrepencies"
date: 2019-03-01
---
## Introduction
This post is based on a project where we needed to see how many transactions we were actually missing in Google Analytics.

## The how to
This process is quite simple,
we will be pulling out transaction id's and then make it into a list. After that we compare that list with another

```r
ga_id <- "yourgaid"

data <- google_analytics(ga_id, 
                         date_range = c("2019-01-23", "2019-02-13"), 
                         metrics = "users", 
                         dimensions = c("transactionId"),
                         max = -1)

#Your transaction ids
rowa <- as.list(data$transactionId)

#the other list of transaction ids
rowb <- as.list(sfData$ti)

#make them into characters
rowa <- as.character(rowa)
rowb <- as.character(rowb)

#Only get the ones that doesn't match
list1 <- setdiff(rowb, rowa)

#Make a dataframe from only those who are missing
df <- sfData[sfData$ti %in% list1, ]
```

## Conclusion
This is a very short tutorial, it is not formatted very pretty, but it should get the trick done in terms of validating if anything should be missing in terms of transactions in Google Analytics. Please leave a comment should you need further elaboration on this progress!