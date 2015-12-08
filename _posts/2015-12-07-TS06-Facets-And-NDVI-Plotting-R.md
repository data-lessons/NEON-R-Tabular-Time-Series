---
layout: post
title: "Lesson 06: Plot using Facets and Plot Time Series with NDVI data"
date:   2015-10-19
authors: [Megan A. Jones, Marisa Guarinello, Courtney Soderberg]
contributors: [Leah Wasser]
dateCreated: 2015-10-22
lastModified: 2015-12-08
category:  
tags: [module-1]
mainTag: 
description: "This lesson teaches how to plot subsetted time series data using
the `facets()` function (e.g., plot by season) and to plot time series data with
NDVI."
code1:
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Time-Series-Plot-Facets-NDVI
comments: false
---

{% include _toc.html %}

##About
This lesson teaches how to plot subsetted time series data using
the `facets()` function (e.g., plot by season) and to plot time series data with
NDVI.

**R Skill Level:** Intermediate - you've got the basics of `R` down.

<div id="objectives" markdown="1">

###Goals / Objectives
After completing this activity, you will:

 * Know how to use `facets()` in ggplot2.
 * Be able to combine different types of data into one plot.

###Things You'll Need To Complete This Lesson
Please be sure you have the most current version of `R` and, preferably,
RStudio to write your code.

####R Libraries to Install
<li><strong>ggplot:</strong> <code> install.packages("ggplot2")</code></li>
<li><strong>scales:</strong> <code> install.packages("scales")</code></li>
<li><strong>gridExtra:</strong> <code> install.packages("gridExtra")</code></li>

####Data to Download
<a href="http://files.figshare.com/2437700/AtmosData.zip" class="btn btn-success">
Download Atmospheric Data</a>

The data used in this lesson were collected at Harvard Forest which is
an National Ecological Observatory Network  
<a href="http://www.neoninc.org/science-design/field-sites/harvard-forest" target="_blank"> field site</a>. 
These data are proxy data for what will be available for 30 years
on the [NEON data portal](http://data.neoninc.org/ "NEON data")
for both Harvard Forest and other field sites located across the United States.

####Setting the Working Directory
The code in this lesson assumes that you have set your working directory to the
location of the unzipped file of data downloaded above.  If you would like a
refresher on setting the working directory, please view the [Setting A Working Directory In R]({{site.baseurl}}/R/Set-Working-Directory/ "R Working Directory Lesson") lesson prior to beginning
this lesson.

####Skills Needed
This lessons assumes familiarity with both the `dplyr` package and `ggplot()` in
the `ggplot2` package.  If you are not comfortable with either of these we
recommend starting with the [Subset & Manipulate Time Series Data with `dplyr` lesson]({{site.baseurl}}/R/Time-Series-Subset-dplyr/ "Learn dplyr") 
and the [Plotting Time Series with ggplot in R lesson]({{site.baseurl}}/R/Time-Series-Plot-ggplot/ "Learn ggplot")  
respectively, to gain familiarity.

</div>

#The Data Approach
What we want to do with ecological time series data often extends beyond
plotting a single variable as it changes across time. For example, when studying
phenology it is likely that a combination of precipitation, temperature, and
solar radiation (PAR) plays a role in the annual greening and browning of 
plants.  

In this lesson we will learn how to plot several multiple variables variables
together across the seasons, prior to adding in NDVI data.  Using the NDVI data 
we can finally directly compare the observed plant phenology with patterns we've
already been exploring.  

###Load the Data
We will need the daily micro-meteorology data for 2009-2011 from the Harvard
Forest. If you do not have these data sets as `R` objects, please load them from
the .csv files in the downloaded data and convert date-time variables to 
POSIXct date-time class. 


    #Remember it is good coding technique to add additional libraries to the top of
      #your script 
    library(lubridate) #for working with dates
    library(ggplot2)  #for creating graphs
    library(scales)   #to access breaks/formatting functions
    library(gridExtra) #for arranging plots
    library(dplyr)  #for subsetting by season
    
    #set working directory to ensure R can find the file we wish to import
    #setwd("working-dir-path-here")
    
    #daily HARV met data, 2009-2011
    harMetDaily.09.11 <- read.csv(file="AtmosData/HARV/Met_HARV_Daily_2009_2011.csv",
                         stringsAsFactors = FALSE)
    
    #covert date to POSIXct date-time class
    harMetDaily.09.11$date <- as.POSIXct(harMetDaily.09.11$date)
    
    #monthly HARV temperature data, 2009-2011
    harTemp.monthly.09.11<-read.csv(file="AtmosData/HARV/Temp_HARV_Monthly_09_11.csv",
                                    stringsAsFactors=FALSE)
    
    #convert datetime from chr to datetime class & rename date for clarification
    harTemp.monthly.09.11$date <- as.POSIXct(harTemp.monthly.09.11$datetime)

##Graph Two Variables on One Plot
Is there a relationship between PAR, a measure of solar radiation, and
precipiation.  It makes sense there would be as solar radiation is lower on 
cloudy days and precipitation is also more likely when clouds are present. 

We will use `ggplot()` to graph PAR and precipitation from the daily Harvard
Meteorological data.  We can simply do this by in putting both variables into the
aesthetics (`aes`) argument of `ggplot()`.  Unless we specify, the first 
variable will be the x-axis and the second the y-axis.  


    #PAR v precip 
    par.precip <- ggplot(harMetDaily.09.11,aes(prec, part)) +
               geom_point(na.rm=TRUE) +    #removing the NA values
               ggtitle("Daily Precipitation and PAR at Harvard Forest") +
               theme(plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
               theme(text = element_text(size=20)) +
               xlab("Total Precipitation (mm)") + ylab("Mean Total PAR")
    par.precip

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-R/PAR-v-precip-1.png) 

While there is a lot of noise in the data, there does seem to be a trend of 
lower PAR when precipitation is high. Yet in this data we don't tease apart any 
of annual patterns as date isn't part of this figure.  


##Subset by Season
One way to add a time component to this figure is to subset the data by season 
and then to plot a facetted graph showing the data by season.  The initial step to do this is to subset by season.  

First, we have to switch 
away from coding and data analysis and think about ecology.  What is the best
way to break a year apart into seasons?  

The divisions will change depending on the geographic location and climate where
the data were collected.  We are using data from Harvard Forest in Massachusetts
in the northeastern United States.  Given the strong seasonal affects in this 
area dividing the year into 4 seasons is ecologically relevant.  Based on
general knowledge of the area it is also plausible to group the months as 

 * Winter: December - February
 * Spring: March - May 
 * Summer: June - August
 * Fall: September - November 
 
In order to subset the data by season we will again use the `dplyr` package.  


    #create a column of only the month
    harMetDaily.09.11 <- harMetDaily.09.11  %>% 
      mutate(month=format(date,"%m"))
    
    #check structure of this variable
    str(harMetDaily.09.11$month)

    ##  chr [1:1095] "01" "01" "01" "01" "01" "01" "01" "01" ...

Notice that the new column `month` is recognized as character class.  

We can now use `mutate()` and an `ifelse` statement to create a new variable
called `season` by grouping three months together. Because month is a character
variable we need to put the month number in quotes. 

Within `dplyr` `%in%` is short-hand for "or"; the 3rd line of code essentially
says "If the month variable is equal to 12 or  01 or  02, set the season
variable to winter".  The "Error" at the end is what will be printed in the 
column if the month does not equal any of the month dates specified.  


    harMetDaily.09.11 <- harMetDaily.09.11 %>% 
      mutate(season = 
               ifelse(month %in% c("12", "01", "02"), "winter",
               ifelse(month %in% c("03", "04", "05"), "spring",
               ifelse(month %in% c("06", "07", "08"), "summer",
               ifelse(month %in% c("09", "10", "11"), "fall", "Error")))))
    
    #check to see if this worked
    head(harMetDaily.09.11$month)

    ## [1] "01" "01" "01" "01" "01" "01"

    head(harMetDaily.09.11$season)

    ## [1] "winter" "winter" "winter" "winter" "winter" "winter"

    tail(harMetDaily.09.11$month)

    ## [1] "12" "12" "12" "12" "12" "12"

    tail(harMetDaily.09.11$season)

    ## [1] "winter" "winter" "winter" "winter" "winter" "winter"

##Use Facets in ggplot
Facets allow us to plot subsets of data together.  Using `facet_grid()` we can
create the same plot of PAR and precipiation for each season and display them in
a grid.


    #run this code to plot the same plot as before but with one plot per season
    par.precip + facet_grid(. ~ season)

    ## Error in layout_base(data, cols, drop = drop): At least one layer must contain all variables used for facetting

It didn't work!  Why? 

We added the season variable to `harMetDaily.09.11` after we created the
original `par.precip` plot. We need to re-run the `par.precip` code again so 
that `season` is now included in the data (go back and re-run that code). 



Now that we've done that we can again try the facet code. 


    par.precip + facet_grid(. ~ season)

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-R/plot-by-season2-1.png) 

    # for a landscape orientation of the plots we change the order of arguments in facet_grid():
    par.precip + facet_grid(season ~ .)

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-R/plot-by-season2-2.png) 

    #and another arrangement of plots:
    par.precip + facet_wrap(~season, ncol = 2)

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-R/plot-by-season2-3.png) 

How does our ability to see patterns in the data vary depending on which
arrangement the graphs are in? 

###Defining the Order of Facetted Plots
Notice how our facetted plots are currently show up as "fall", "spring",
"summer", "winter".  As no level is assigned the data are class character, `R`
simply arranges them in alphabetical order.  However, "fall", "spring", 
"summer", "winter" do have a specified order to them.  It doesn't mater which
season starts the cycle but they should be in order to ease our understanding
of annual cycles. 

In order to create a logical order we need to assign levels to the seasons. We
can do this by assigning the seasons as a factor with a defined order (or 
level).  


    harMetDaily.09.11$season<- factor(harMetDaily.09.11$season, level=c("spring",
                                                        "summer","fall","winter")) 
    
    #check to make sure it worked
    str(harMetDaily.09.11$season)

    ##  Factor w/ 4 levels "spring","summer",..: 4 4 4 4 4 4 4 4 4 4 ...

    #rerun original par.precip code to incorporate the levels. 
    par.precip <- ggplot(harMetDaily.09.11,aes(prec, part)) +
             geom_point(na.rm=TRUE) +    #removing the NA values
            ggtitle("Daily Precipitation and PAR at Harvard Forest") +
             theme(plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
             theme(text = element_text(size=20)) +
             xlab("Total Precipitation (mm)") + ylab("Mean Total PAR")
               
    #new facetted plots
    par.precip + facet_grid(. ~ season)

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-R/assigning-level-to-season-1.png) 

We now how a plot with the seasons in a logical order. 

#Challenge: Compare Precipitation and Air Temperature Across Seasons
1) Using the same data create a faceted plot showing the relationship between 
precipitation and air temperature across the seasons.  Create the figure with
"winter" as the first facetted plot.   
2) Compare how different orientations of the plots highlight different patterns.

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-R/challenge-code-prec.airtemp-1.png) ![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-R/challenge-code-prec.airtemp-2.png) 

#Graph variables and NDVI data together


    #first read in the NDVI CSV data
    NDVI.2009 <- read.csv(file="Landsat_NDVI/Harv2009NDVI.csv", stringsAsFactors = FALSE)

    ## Warning in file(file, "rt"): cannot open file 'Landsat_NDVI/
    ## Harv2009NDVI.csv': No such file or directory

    ## Error in file(file, "rt"): cannot open the connection

    #check out the data
    str(NDVI.2009)

    ## Error in str(NDVI.2009): object 'NDVI.2009' not found

    View(NDVI.2009)

    ## Error in View : object 'NDVI.2009' not found


#plot of NDVI vs. PAR using daily data
First lets get just 2009 from the `harmet.Daily` data since that is the only
year for which we have NDVI data.


    harMet.daily2009 <- harMet.daily %>% 
      mutate(year = year(date)) %>%   #need to create a year only column first
      filter(year == "2009")

#ggplot does not provide for two y-axes and the scale for these two variables
are vastly different.
So we will create a plot for each variable using the same time variable (julian
day) as our x-axis.
Then we will plot the two plots in the same viewer so we can compare


    #create plot of julian day vs. PAR
    par.2009 <- ggplot(harMet.daily2009, aes(jd,part))+
      geom_point(na.rm=TRUE)+
      ggtitle("Daily PAR at Harvard Forest, 2009")+
      theme(legend.position = "none", plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
               theme(text = element_text(size=20)) 
    
    #create plot of julian day vs. NDVI
    NDVI.2009 <- ggplot(NDVI.2009,aes(julianDays, meanNDVI))+
      geom_point(aes(color = "green", size = 4)) +
      ggtitle("Daily NDVI at Harvard Forest, 2009")+
      theme(legend.position = "none", plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
               theme(text = element_text(size=20)) 

    ## Error in ggplot(NDVI.2009, aes(julianDays, meanNDVI)): object 'NDVI.2009' not found

    #display the plots together
    grid.arrange(par.2009, NDVI.2009) 

    ## Error in arrangeGrob(...): object 'NDVI.2009' not found




    #Let's take a look at air temperature too
    airt.2009 <- ggplot(harMet.daily2009, aes(jd,airt))+
      geom_point(na.rm=TRUE)+
      ggtitle("Daily Air Temperature at Harvard Forest, 2009")+
      theme(legend.position = "none", plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
               theme(text = element_text(size=20)) 
    
    grid.arrange(airt.2009, NDVI.2009)

    ## Error in arrangeGrob(...): object 'NDVI.2009' not found


    #all 3 together
    grid.arrange(par.2009, airt.2009, NDVI.2009)

    ## Error in arrangeGrob(...): object 'NDVI.2009' not found
