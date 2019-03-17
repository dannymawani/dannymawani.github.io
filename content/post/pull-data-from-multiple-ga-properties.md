---
title: "Pull data from multiple Google Analytics Properties with R"
summary: "Benchmarking your data can be rough. In this post we go through how to pull data from multiple GA accounts"
date: 2019-03-01
---
{{% toc %}}

## Background
Being in an agency, we often have to do benchmarks, reports etc. that require us to pull data from multiple Google Analytics properties.

## Usecase
One of our client have multiple properties each containing 3 business units, a B2B, B2C and B2G (Business to government). To make sure that we could reproduce the reporting for them, we build an R script that:

* Pulls Google Analytics data
* Adds business unit and country to the dataframe as additional variables / columns
* Merges the data into another dataframe

## The code
To extract the data, use the ```googleAnalyticsR```and ```googleAuthR```package made by [Mark Edmonson](https://twitter.com/HoloMarkeD). To see more information about installing the packages and using the libraries please check out the site made explaining how the package work [here](http://code.markedmondson.me/googleAnalyticsR/)

### 1. Connect To Google Analytics
To begin with we must first authenticate to Google Analytics.

```r 
library(googleAnalyticsR)
googleAnalyticsR::ga_auth()
```
Next you will be asked to log in with your Google Account. Once that is done, we are ready to do the rest of the script.

{{% alert note %}}
Remember to authenticate with the account you want access to
{{% /alert %}}

### 2. Setting up the script to pull from multiple properties
To combine different properties we are fest specifying the views we need and adding them to a list. Once that is done we will create 2 corresponding lists with business units and countries.

```r
#this are the view links which can be find under view settings or through the URL of the view - If you are used to working with this package, you can also do an extraction of all your views directly from R. The below views are fake.

#Danish
dk_b2b <- "123213213"
dk_b2c <- "543454533"
dk_b2g <- "173714215"

#Finnish
fi_b2b <- "345345435"
fi_b2c <- "345435345"
fi_b2g <- "234234232"

#French
fr_b2b <- "655464555"
fr_b2c <- "989834589"
fr_b2g <- "039485309"

#adding all the views to a list
views <- c(dk_b2b,dk_b2c,dk_b2g,fi_b2b,fi_b2c,fi_b2g,fr_b2b,fr_b2c,fr_b2g)

#countries should be in the same order as your list above, we will use this to add attributes to the dataframe
countries <- c("DK","DK","DK","FI","FI","FI","FR","FR","FR")

#same approach for business units
BU <- c("B2B","B2C","B2G","B2B","B2C","B2G","B2B","B2C","B2G")

#Set a start and enddate
startDate <- "2018-01-01"
endDate <- as.character(Sys.Date()-1)

#Set dimensions and metrics
dimensions <- c("year","sourceMedium","campaign")
metrics <- c("sessions","transactions","transactionRevenue")

#create an empty dataframe
upload <- data.frame()

#pulling the data
for(i in seq_along(views)) {
  data <- google_analytics(views[i], date_range = c(startDate,endDate), metrics = metrics,
                   dimensions = dimensions,max = -1)

data$country <- countries[i]
data$bu <- BU[i]

upload <- rbind(upload,data)

}

```
The end states that for each element in the view list, then run the analytics script, apply the data and business unit to the corresponding list item to the dataframe "data", and then add it to the empty dataframe.

## Conclusion
Now that the dataframe has been made it is up to you what to do with it. you can either upload it to [bigQuery](https://bigquery.cloud.google.com), do some statistics with R and some plot with [ggplot2](https://ggplot2.tidyverse.org/) or just write it down to a CSV file with ```r write.csv2(upload,"yourFileName.csv")``` and work with it in another tool like Excel, Tableau or powerBI.


