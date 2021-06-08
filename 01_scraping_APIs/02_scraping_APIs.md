---
title: "SICSS-Oxford, 2021"
subtitle: "Working with Digital Data: APIs"
author:
  name: Christopher Barrie
  affiliation: University of Edinburgh | [SICSS](https://github.com/cjbarrie/sicss_21)
# date: Lecture 6  #"08 June 2021"
output: 
  html_document:
    theme: flatly
    highlight: haddock
    # code_folding: show
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: true
    
bibliography: scrapeAPIs.bib    
---


# Working with Digital Data

The lecture material for today mentioned work by @Freelon2018a, @Bruns2019, @Puschmann2019, and @Lazer2020b as well as a [report](https://www.disinfobservatory.org/download/26541) by SOMA outlining solutions for research data exchange. 

These example tasks use different sources of online data, and here I introduce you to how we might gather data through both screen-scraping (or server-side) techniques as well as API (or client-side) techniques. 

## Tutorial: APIs 

In this tutorial, you will learn how to:

* Get developer access credentials to Twitter
* Get Academic Research Product Track access credentials to Twitter
* Use the <tt>academictwitteR</tt> package to query the Twitter API

## Setup 

In order to use the Twitter Academic Research Product Track you will first need to obtain an authorization token. You will find details about the process of obtaining authorization [here](https://developer.twitter.com/en/solutions/academic-research/application-info). 

**In order to gain authorization you first need a Twitter account.**

First, Twitter will ask for details about your academic profile. Per the documentation linked above, they will ask for the following:

> Your full name as it is appears on your institution’s documentation
> 
>   Links to webpages that help establish your identity; provide one or more of the following:
> 
>   - A link to your profile in your institution’s faculty or student directory
>   - A link to your Google Scholar profile
>   - A link to your research group, lab or departmental website where you are listed
> 
>   Information about your academic institution: its name, country, state, and city
> 
>   Your department, school, or lab name
> 
>   Your academic field of study or discipline at this institution
> 
>   Your current role as an academic (whether you are a graduate student, doctoral candidate,       post-doc, professor, research scientist, or other faculty member)

Twitter will then ask for details of the proposed research project. Here, questions include:

> 1. What is the name of your research project?
>
> 2. Does this project receive funding from outside your academic institution? If yes, please list all your sources of funding.
>
> 3. In English, describe your research project. Minimum 200 characters.
>
> 4. In English, describe how Twitter data via the Twitter API will be used in your research project. Minimum 200 characters.
>
> 5. In English, describe your methodology for analyzing Twitter data, Tweets, and/or Twitter users. Minimum 200 characters.
>
> 6. Will your research present Twitter data individually or in aggregate?
>
> 7. In English, describe how you will share the outcomes of your research (include tools, data, and/or other resources you hope to build and share). Minimum 200 characters.
>
> 8. Will your analysis make Twitter content or derived information available to a government entity?

Once you have gained authorization for your project you will be able to see the new project on your Twitter developer portal. First click on the developer portal as below. 

![](images/twitterdev2.png){width=80%}


Here you will see your new project, and the name you gave it, appear on the left hand side. Once you have associated an App with this project, it will also appear below the name of the project. Here, I have several Apps authorized to query the basic API. I have one App, named "gencap", that is associated with my Academic Research Product Track project. 


![](images/twitterdev3.png){width=80%}


When you click on the project, you will first see how much of your monthly cap of 10m tweets you have spent. You will also see the App associated with your project below the monthly tweet cap usage information.


![](images/twitterdev4.png){width=80%}


By clicking on the Settings icons for the App, you will be taken through to the information about the App associated with the project. Here, you will see two options listed, for "Settings" and "Keys and Tokens."


![](images/twitterdev5.png){width=80%}


Beside the panel for Bearer Token, you will see an option to Regenerate the token. You can do this if you have not stored the information about the token and no longer have access to it. It is important to store information on the Bearer Token to avoid having to continually regenerate the Bearer Token information.


![](images/twitterdev6.png){width=80%}


Once you have the Bearer Token, you are ready to use `academictwitteR`!

##  Load data and packages 

Before proceeding, we'll load the remaining packages we will need for this tutorial.


```r
library(tidyverse) # loads dplyr, ggplot2, and others
library(academictwitteR) # to query the Academic Research Product Track Twitter v2 API endpoint in R
```

## The Twitter Academic Research Product Track

The Academic Research Product Track permits the user to access larger volumes of data, over a far longer time range, than was previously possible. From the Twitter [documentation](https://developer.twitter.com/en/solutions/academic-research/application-info):

> "The Academic Research product track includes full-archive search, as well as increased access and other v2 endpoints and functionality designed to get more precise and complete data for analyzing the public conversation, at no cost for qualifying researchers. Since the Academic Research track includes specialized, greater levels of access, it is reserved solely for non-commercial use".

The new "v2 endpoints" refer to the v2 API, introduced around the same time as the new Academic Research Product Track. Full details of the v2 endpoints are available [here](https://developer.twitter.com/en/docs/twitter-api/early-access).

In summary the Academic Research product track allows the authorized user:

1. Access to the full archive of (as-yet-undeleted) tweets published on Twitter
2. A higher monthly tweet cap (10m--or 20x what was previously possible with the standard v1.1 API)
3. Ability to access these data with more precise filters permitted by the v2 API


## Querying the Twitter API with `academictwitteR`

We begin by storing our access token with:


```r
bearer_token = "AAAAAAAAAAAAAAAAAAAAA_INSERT_YOUR_TOKEN_HERE"
```

The workhorse function of `academictwitteR` when it comes to collecting tweets containing a particular string or hashtag is `get_all_tweets()`.


```r
tweets <-
  get_all_tweets(
    "#BLM OR #BlackLivesMatter",
    "2020-01-01T00:00:00Z",
    "2020-01-05T00:00:00Z",
    bearer_token,
    file = "blmtweets"
  )
```

Here, we are collecting tweets containing one or both of two hashtags related to the Black Lives Matter movement over the period January 1, 2020 to January 5, 2020. 

## Storage conventions in `academictwitteR`

Given the sizeable increase in the volume of data potentially retrievable with the Academic Research Product Track, it is advisable that researchers establish clear storage conventions to mitigate data loss caused by e.g. the unplanned interruption of an API query.

We first draw your attention first to the `file` argument in the code for the API query above.

In the file path, the user can specify the name of a file to be stored with a ".rds" extension, which includes all of the tweet-level information collected for a given query.

Alternatively, the user can specify a `data_path` as follows:


```r
tweets <-
  get_all_tweets(
    "#BLM OR #BlackLivesMatter",
    "2020-01-01T00:00:00Z",
    "2020-01-05T00:00:00Z",
    bearer_token,
    data_path = "data/"
    bind_tweets = FALSE
  )
```

In the data path, the user can either specify a directory that already exists or name a new directory. In other words, if there is already a folder in your working directory called "data" then `get_all_tweets` will find it and store data there. If there is no such directory, then a directory named (here) "data" will be created in your working directory for the purposes of data storage.

The data is stored in this folder as a series of JSONs. Tweet-level data is stored as a series of JSONs beginning "data_"; User-level data is stored as a series of JSONs beginning "users_".

Note that the `get_all_tweets()` function always returns a data.frame object unless `data_path` is specified and `bind_tweets` is set to `FALSE`. When collecting large amounts of data, we recommend using the `data_path` option with `bind_tweets = FALSE`. This mitigates potential data loss in case the query is interrupted, and avoids system memory usage errors.

## Binding JSON files into data.frame objects

Users can then use the `bind_tweet_jsons` and `bind_user_jsons` convenience functions to bundle the JSONs into a data.frame object for analysis in R as such:


```r
tweets <- bind_tweet_jsons(data_path = "data/")
```


```r
users <- bind_user_jsons(data_path = "data/")
```

## Inspecting the data

Let's say, as an example, we queried the Twitter API with the following code:



```r
get_all_tweets(
  "#BLM OR #BlackLivesMatter",
  "2020-01-01T00:00:00Z",
  "2020-01-05T00:00:00Z",
  bearer_token,
  data_path = "data/academictwitteR_data",
  file = "data/blmtweets",
  bind_tweets = F
)
```

Will output JSON files like this:


```
##  [1] "data_1212161860600041475.json"  "data_1212202218138374144.json" 
##  [3] "data_1212454140183547909.json"  "data_1212541396005064704.json" 
##  [5] "data_1212602429998583808.json"  "data_1212745962499780610.json" 
##  [7] "data_1212848819668340737.json"  "data_1212931998357966848.json" 
##  [9] "data_1213102352925970433.json"  "data_1213244966530502656.json" 
## [11] "data_1213425494068285442.json"  "query"                         
## [13] "users_1212161860600041475.json" "users_1212202218138374144.json"
## [15] "users_1212454140183547909.json" "users_1212541396005064704.json"
## [17] "users_1212602429998583808.json" "users_1212745962499780610.json"
## [19] "users_1212848819668340737.json" "users_1212931998357966848.json"
## [21] "users_1213102352925970433.json" "users_1213244966530502656.json"
## [23] "users_1213425494068285442.json"
```

Binding the JSON files:


```r
blmtweets <- bind_tweet_jsons("data/academictwitteR_data")
```

OR:


```r
blmtweets <- readRDS("data/blmtweets.rds")
```

<table class="table table-striped table-hover table-condensed table-responsive" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> id </th>
   <th style="text-align:left;"> created_at </th>
   <th style="text-align:left;"> text </th>
   <th style="text-align:left;"> source </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 1212655100973461505 </td>
   <td style="text-align:left;"> 2020-01-02T08:41:20.000Z </td>
   <td style="text-align:left;"> RT @willallenactor: "All lives have mattered since creation. But black lives have mattered less since the birth of this nation."
 A clip fr… </td>
   <td style="text-align:left;"> Twitter for Android </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1212633229770948608 </td>
   <td style="text-align:left;"> 2020-01-02T07:14:26.000Z </td>
   <td style="text-align:left;"> @absurdistwords I would be dead if I had been born Black. 

That's the simple truth of it, looking back.

The fact that wytness probably saved my life is no reason why so many black lives full of unique talent should be sacrificed on the altar of #WhiteSupremacy 
#BlackLivesMatter
@UniteThePoor </td>
   <td style="text-align:left;"> Twitter for Android </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1212748345573117954 </td>
   <td style="text-align:left;"> 2020-01-02T14:51:52.000Z </td>
   <td style="text-align:left;"> .@JulianCastro is crucial in #Election2020. He proposed some of the boldest, progressive plans for immigration reform, police accountability &amp;amp; environmental justice. Whenever he had the chance to #SayHerName &amp;amp; affirm #BlackLivesMatter, he did. He made this race stronger, period. </td>
   <td style="text-align:left;"> Twitter for iPhone </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1213289492628434944 </td>
   <td style="text-align:left;"> 2020-01-04T02:42:11.000Z </td>
   <td style="text-align:left;"> RT @kateju9: #justicereform  for @WgarNews 

#BlackLivesMatter 
Our aboriginal communities are suffering. Lack of water 💧 their food recour… </td>
   <td style="text-align:left;"> Twitter for iPhone </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1213523690043760641 </td>
   <td style="text-align:left;"> 2020-01-04T18:12:48.000Z </td>
   <td style="text-align:left;"> EVERY #SmallBusiness #SmallBiz knows effects of losing #NetNeutrality  my team +I hope u consider helping #share this special project  #ForYourConsideration #Oscars2020 #Oscars  #eVeNgodThisFemaleIsNotYetRated™ #FYC #LGBTQ #BLM #ClimateEmergency #GLAAD  https://t.co/XGan48Tebs https://t.co/fvTBfWGWxw </td>
   <td style="text-align:left;"> Twitter for Android </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1212543103699050496 </td>
   <td style="text-align:left;"> 2020-01-02T01:16:18.000Z </td>
   <td style="text-align:left;"> RT @WildPalmsLtd: An NYPD officer got drunk, threatened the occupant, and used racial slurs. He was convicted of multiple counts but is som… </td>
   <td style="text-align:left;"> Twitter Web App </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1212858078128852992 </td>
   <td style="text-align:left;"> 2020-01-02T22:07:54.000Z </td>
   <td style="text-align:left;"> (2/2) Didn't think my heavy heart could feel better, but appreciate seeing what it means to fight - thx again @michaelharriot #BlackGirlsRock #BlackGirlMagic #BlackLivesMatter #BlackTwitter #BlackLoveMatters 🖤 #ENDWHITESUPREMACY #WPRTRP </td>
   <td style="text-align:left;"> Twitter Web App </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1213069645005148161 </td>
   <td style="text-align:left;"> 2020-01-03T12:08:35.000Z </td>
   <td style="text-align:left;"> RT @WildPalmsLtd: Some welcome news for a change. The racist NYPD cop who went coo-coo after a bachelor party calling the victims "f---ing… </td>
   <td style="text-align:left;"> Twitter for Android </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1212465486186926080 </td>
   <td style="text-align:left;"> 2020-01-01T20:07:53.000Z </td>
   <td style="text-align:left;"> The way I recall growing up before the wicked #socialist blacks with their hate and blame took over. #dem #blm @naacp and the likes now - #walkAway #redpill #BlackVoicesforTrump #maga - Excellent movie, enjoy. https://t.co/1FXtbutNg2 </td>
   <td style="text-align:left;"> Twitter Web App </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1213010362779537408 </td>
   <td style="text-align:left;"> 2020-01-03T08:13:01.000Z </td>
   <td style="text-align:left;"> RT @Anderhardt: #FridayThoughts #FridayMotivation #FridayVibes #fridaymorning #FridayFeeling #AHomeForEveryHorse #SaveOurHorses #horse #wil… </td>
   <td style="text-align:left;"> Twitter for iPhone </td>
  </tr>
</tbody>
</table>