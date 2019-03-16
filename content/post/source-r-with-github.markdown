---
title: "Source R scripts from github through Google Cloud Storage"
summary: "Making changes to your pipelines can be rough if you are multiple people working on the same project. This guide shows you how to use Github to make it easier"
date: 2019-03-01
---
{{% toc %}}

## Background
In my company, we have gone from 1 person working with adhoc assignment with R, to 4 persons within the last year. This set's up a whole new level of requirements to ensure quality, stability and safety when it comes to working with datamodels and API calls. Furthermore, there is also an aspect of reusability that needed to be set up to ensure that we could produce our solutions faster,and better updated.

To do this, we started looking into differen't types of systems to work with our code. In the end we have selected Github as our main source of creating our code and documentation.

To get to know how to do this, I actually used this guide: [Automated Static Website Publishing with Cloud Build
](https://www.google.com), and used most of the principles in the guide to set up our setup.

## Why is sourcing from github smart?
Github allows us to:

* Share code easily
* Scale projects to other customers
* Ensure documentation
* Making updates to code safe and with the possibility to roll-back should something break
* See who have created code
* Source Scripts directly in R

Unfortunately, going into how to use Github is a bit out of the scope for this post, however I recommend going to Github and read their documentation and do a few searches on the net to get started.

## The how-to part
In order to source R scripts from github there are a few things you need to have ready in order to get ready:

### 1. Setting up a github account
One of the first thing you will need is to actually create a [Github account](https://www.google.com)
. This can be done on the frontpage:

{{< figure library="1" src="githubFrontpage_githubR.png" title="" >}}

Once that is done, [create a repository](https://github.com/new):

{{< figure library="1" src="createRepositories_githubR.png" title="" >}}

Recently, it's been possible to [create private repositories for free](https://www.google.com)[^1]. As we are multiple users that are using Github right now, we did manage to get our financaial department to approve the monthly 7$ account fee, however in theory, you could create a shared login to help get you started.  

### 2. Creating a bucket in Google Cloud Storage
1. [Set up billing for GCP (Google Cloud Platform)](https://cloud.google.com/billing/docs/how-to/manage-billing-account)
2. [Create a bucket in Google Cloud Storage)](https://console.cloud.google.com/storage/): <br>
{{< figure library="1" src="createBucket_githubR.png" title="" >}}

### 3. Creating a trigger with Cloud Build
Now, this part is something that is already documented in the GCP documentaion. To avoid copying the same guide in this post, [instead go to this page](https://cloud.google.com/community/tutorials/automated-publishing-cloud-build) and follow the steps from the headling: **Set up automated builds** until you see this line: "Now, create a cloudbuild.yaml". From there, you will not need to follow the rest of the steps to continue this guide.

### 4. Setting up the YALM
YALM stands for "Yet Another Markup Language" and will be added to Github. From here, the YALM file sends the information to Cloud Build that copies the updates on Github into the Cloud Storage Bucket just created.

In your Github repo, click on "Create new file". Now, add the following YALM syntax in the edit section:

```YALM
steps:
  - name: gcr.io/cloud-builders/gsutil
    args: ["-m", "rsync", "-r", "-c", "-d", ".", "gs://yourfolderincloudstorage/ifneededthencreateasubfolder"]
```
save the file as cloudbuild.yaml.

Once that is done, each change made to your repository will be pushed directly to Google Cloud Storage, where it is possible to run R scripts.

### 5. Sourcing scripts from Cloud Storage
Playing around with R and the Google API universe usually means using something that [Mark Edmonson has build](http://code.markedmondson.me), and this post is not an exception.

We will be using the Package called "googleCloudStorageR", which you can find more information about [here](https://cran.r-project.org/web/packages/googleCloudStorageR/vignettes/googleCloudStorageR.html).

To source a script from Google Cloud Storage, use this ```R code```to authenticate and source the script you need:

```R
#install.package("googleCloudStorageR") if you have not installed it yet

#load the library
library(googleCloudStorageR)

#Authenticate
gcs_auth()

googleCloudStorageR::gcs_source('yoursubfolderifyouhaveone/yourscript.R', bucket = 'yourcreatedbucket')
```
And voila. This means that you can always download your R files where ever you need it, build on it, add the information back to github and source the script from a third place without having to have multiple copies of your code laying around.

## Thoughts and conclusion
This post have showed how to use Github to run R scripts and keep them updated. I do recommend trying to document and automate as much as possible when working with R, however this should only be used whenever it makes sense. If you are building something on the fly that will only be used for a specific tasks once, then it might not make sense to go through all of this to make your workflow optimal.



[^1]: A repository is a folder to which your content in github is stored. This is usually your code files.