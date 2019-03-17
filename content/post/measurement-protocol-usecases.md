---
title: "Creating a workflow with measurement protocol and R"
summary: "This is a simple how-to guide on doing basic things with measurement protocol"
date: 2019-03-01
---
{{% toc %}}

## Introduction
This blogpost was inspired by a release to a website which had some last minute changed that when the site when live, caused a javaScript error that blocked the ecommerce transaction data to not populate.

With this error, the client lost 1800+ transactions to the site meaning that we had to upload these in the right currency format to make sure that all the data was still avaliable.

## How the measurement protcol works
Whenever you visit a website, Google Analytic will collect information by sending a long list of query parameters to an endpoint, to which it is stored and modelled to show information in GA. Visiting this blogpost will for an example generate a network call similar to this:
```javaScript
https://www.google-analytics.com/r/collect?v=1&_v=j73&a=1679354847&t=pageview&_s=1&dl=https%3A%2F%2Fwww.mawani.dk%2Fpost%2Fmeasurement-protocol-usecases%2F&ul=en-us&de=UTF-8&dt=Creating%20a%20workflow%20with%20measurement%20protocol%20and%20R%20%7C%20DMH%20Analytics&sd=24-bit&sr=1440x900&vp=1440x256&je=0&_u=QACAAAAB~&jid=651243071&gjid=1319476248&cid=1164110852.1529450664&tid=UA-134673438-1&_gid=106302445.1550868348&_r=1&gtm=2wg241WBTJ7HR&z=1265093896
```

This will tell Google Analytics that:

* The URL is this one
* My browser language is en-us
* what my device id is from the cookie so it can recognize me as a new or returning user
* What tracking ID it should populate the data to
* What my session ID is.

What measurement protocol will do is to emulate the same types of parameters and have it populate the data in GA.

For a more detailed view on measurement protocol I would advise you to visit 
[Optimize Smart for their deepgoing description](https://www.optimizesmart.com/understanding-universal-analytics-measurement-protocol/).

## Usecases
I have used measurement protocol for a various number of things. As you can send in all the supported information, this is a great area to explore in terms of improving your data collection. As for many of my other posts, I will be using another package created by [Mark Edmonson](https://twitter.com/HoloMarkeD) called [googleMeasureR](https://github.com/MarkEdmondson1234/googleMeasureR) to send the hits to Google Analytics, however you can easily transfer the way of doing this to any language of your choice such as Python, javaScript, PHP etc.

### Uploading refunds
To upload refunds you first need a list of all the transactions that are missing. If you are on a site with multiple currencies it is also quite important that you can specify what unit that you are adding data to, so you are sure that it is populated correctly.

First thing that need to be done is to make sure that the data is loaded into R. I have chosen to rename the columns to the GA naming convention for measurement protocol, but it is not important:

{{< figure library="1" src="transaction_example.png" title="" >}}

Once that is made, we need to either have the client id (Google Analytics Cookie ID for identifying people), or create one yourself. This can be done by this code:

```r

#loading the library
library(stringi)
#the count of rows you need to upload
n <- nrow(yourDataframe)

#create the ids
paste("1.2.",stri_rand_strings(n, 10, pattern = "[0-9]"),".",stri_rand_strings(n, 10, pattern = "[0-9]"), sep = "")

#output will look something like this if you print it out in the console

 [1] "1.2.6237558844.6715115260" "1.2.4893337628.0287915875" "1.2.1414860945.6106576917"
 [4] "1.2.8992586956.6450277842" "1.2.2032023475.7135641438" "1.2.1490282359.1139988407"
 [7] "1.2.7613957961.8692578524" "1.2.8481705025.8828767075" "1.2.1331455569.2063291053"
[10] "1.2.9292335596.4531869013"

### add it to the dataframe

yourDataframe$cid <- paste("1.2.",stri_rand_strings(n, 10, pattern = "[0-9]"),".",stri_rand_strings(n, 10, pattern = "[0-9]"), sep = "")

```
Now that a client id is added we are ready to push the refunds into Google Analytics:

```r

####upload
v <- 1 #version
cs <- "measurementprotocol" #source
cm <- "refund" #medium
t <- "event" #type
ec <- "Ecommerce" #event category
ea <- "measurementProtocol" #event action
el <- "refund" #event label
tid <- "UA-1234567-1" #Google Analytics ID
pa <- "refund" #product action
ta <- "measurementprotocol" #transaction affiliation

#for each row in the dataframe send a hit with measurement protocol
for(i in 1:nrow(yourDataframe)) {
  cid <- yourDataframe$cid[i]
  ti <- yourDataframe$ti[i]
  tr <- yourDataframe$tr[i]
  cu <- yourDataframe$cu[i]
  gmr_post(list(v=v,cs=cs,cm=cm,t=t,ec=ec,ea=ea,el=el,tid=tid,cid=cid,pa=pa,ti=ti,tr=tr,cu=cu,ta=ta,ni="1"))

```

And there you have it, the refunds should be uploaded.

### Uploading transactions
This is actually the exact same steps you go through - the primary difference is that you need to change the product action from refund to purchase:

```r

####upload
v <- 1 #version
cs <- "measurementprotocol" #source
cm <- "purchase" #medium
t <- "event" #type
ec <- "Ecommerce" #event category
ea <- "measurementProtocol" #event action
el <- "refund" #event label
tid <- "UA-1234567-1" #Google Analytics ID
pa <- "purchase" #product action
ta <- "measurementprotocol" #transaction affiliation

#for each row in the dataframe send a hit with measurement protocol
for(i in 1:nrow(yourDataframe)) {
  cid <- yourDataframe$cid[i]
  ti <- yourDataframe$ti[i]
  tr <- yourDataframe$tr[i]
  cu <- yourDataframe$cu[i]
  gmr_post(list(v=v,cs=cs,cm=cm,t=t,ec=ec,ea=ea,el=el,tid=tid,cid=cid,pa=pa,ti=ti,tr=tr,cu=cu,ta=ta,ni="1"))

```
## words of caution
When you add data to Google Analytics, it is not possible to remove it again, so remember to test what you are doing beforehand and be sure that what you are doing is correct. 

Below is some areas that you need to remember when doing uploads through measurement protocol:

### 1. Create filters to block views
Create an exclude filter to exclude all event action called "measurementprotocol". This ensures that you data is only send to the views you need.

### 2. Remove the "Exclude all hits from known bots and spiders" checkmark
In some instances I saw that the measurement protocol hits where blocked when this was not checked off. Remember to switch it on the day after so you are secured against people spamming your analytics account with bot traffic.

### 3. Latency 
There is a latency from the the data is send to that it is populated. Unfortunately if you add / remove filters before the day is over it will take in / remove the hits from measurement protocol meaning that you will endanger your datacollection.

## conlusion
This is a simple how-to guide on doing basic things with measurement protocol, there are a lot about datacleaning that could have been included but it would be out of the scope of this blogpost.

In terms of all the parameters added in the call, there are many that could have been excluded, however i choose to add them as it will be easier to filter them out in different instances should you need to do that.

A main issue with measurement protocol, when you don't have the right client id is that you do loose the user behaviour and affect your traffic if you are doing bulk uploads in a day. It is therefore important to consider if this is the right solution for the task, and be sure that the data being send in is correct.
