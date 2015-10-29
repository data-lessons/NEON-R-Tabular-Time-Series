---
layout: post
title: "Lesson 03 Getting to Know Your Data"
date:   2015-10-23
authors: "Marisa Guarinello, Megan Jones, Courtney Soderberg"
dateCreated: 2015-10-22
lastModified: 2015-10-29
tags: [module-1]
description: "This lesson will teach individuals how to conduct basic data 
manipulation and create basic plots of time series data."
code1:
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: 

---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

##About
This activity will walk you through the fundamentals of data manipulation and 
basic plotting.

**R Skill Level:** Intermediate - you've got the basics of `R` down.

<div id="objectives">

<h3>Goals / Objectives</h3>
After completing this activity, you will know:
<ol>
<li>How to create basic time series plots in `R`.</li>
<li>How to manipulate data in `R`.</li>
</ol>

<h3>Things You'll Need To Complete This Lesson</h3>

<h3>R Libraries to Install:</h3>
<ul>
<li><code> install.packages("dplyr")</code></li>
<li><code> install.packages("lubridate")</code></li>
<li><code> install.packages("ggplot2")</code></li>

</ul>
<h4>Tools To Install</h4>

Please be sure you have the most current version of `R` and preferably
R studio to write your code.

#Plotting Time Series Data
One of the first things that it can often be useful to do once we've loaded our 
data and cleaned it up is to visualize our data. We can start to get a sense of 
general trends, and well as see possible outliers or non-sensical values. 

To do this, we're going to use `ggplot2` to plot the air temperature across our
2 year span time span for each 15 minute data point.


    #plot Some Air Temperature Data
    myPlot <- ggplot(yr.09.11,aes(datetime, airt)) +
               geom_point() +
               ggtitle("15 min Avg Air Temperature\nHarvard Forest") +
               theme(plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
               theme(text = element_text(size=20)) +
               xlab("Time") + ylab("Mean Air Temperature")

    ## Error in eval(expr, envir, enclos): could not find function "ggplot"

The dates on the x-axis are not particularly well formatted. We can reformat them 
so they are in the Month/Day/Year format we're used to.

    #format x axis with dates
    myPlot + scale_x_datetime(labels = date_format("%m/%d/%y"))

    ## Error in eval(expr, envir, enclos): object 'myPlot' not found

#Challenge: Using the daily precipitation data you imported earlier, create a 
#plot with the x-axis in a European Format (Day/Month/Year)

#Manipulating Data
This plot is interesting, and we can see that there is a clear seasonal trend, unsurprisingly. But the air temperature data at 15 minute intervals might be 
more granular than we want. Instead, we may want to look at how daily average 
temperature changes over time. To do this we first need to learn a bit about how 
to manipulate data in R. We'll use the `dplyr` package and a "split-apply-combine"
technique.

We use the `group-by()` function to determine how we split up our data. For 
example, if we wanted to count up the number of observations per Julian dat, 
we would do:


    yr.09.11 %>%
      group_by(julian) %>%
      tally()

    ## Error in eval(expr, envir, enclos): could not find function "%>%"
The `%>%` at the end of the lines are 'pipes'. They allow you to take the output
of one function and send it to the next function. This means you don't have to 
save the intermediate steps between functions.

We're more interested though in calculting single values by julian day. We can 
use the `summarize()` function for this. To get the mean air temperature for 
each julian day, we would write:

    yr.09.11 %>%
      group_by(julian) %>%
      summarize(mean_airt = mean(airt, na.rm = TRUE))

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

Julian days repeat though, so if we want to summarize our air temperature data 
for each day, we actually need to group our data by two different values at once,
year and julian day. To do this, we're first going to need to create a new year
variable. For this we'll use the `lubridate` package.


    yr.09.11$year <- year(as.Date(yr.09.11$datetime, "%y-%b-%d",
          tz = "America/New_York"))

    ## Error in as.Date(yr.09.11$datetime, "%y-%b-%d", tz = "America/New_York"): object 'yr.09.11' not found

This code will work, but we previously already set up datetime as a time variable,
so we can be a bit more efficient an just put:


    yr.09.11$year <- year(yr.09.11$datetime)

    ## Error in year(yr.09.11$datetime): object 'yr.09.11' not found

Now we have our two variables. So, to get the mean air temperature for each day
for each year, we would write:


    yr.09.11 %>%
      group_by(year, julian) %>%
      summarize(mean_airt = mean(airt, na.rm = TRUE))

    ## Error in eval(expr, envir, enclos): could not find function "%>%"


Could we create the year variable within our `dplyr` function call? Yep. We can
use the `mutate()` function, and include our `lubridate` function within the
`mutate()` call.


    yr.09.11 %>%
      mutate(year = year(datetime)) %>%
      group_by(year, julian) %>%
      summarize(mean_airt = mean(airt, na.rm = TRUE))

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

#Challenge: Can you get the average air temperate for each month?

We also want to save this information so that we can plot the daily air
temperature as well. So, we save the output of our dplyr function as a new
dataframe. Since we're going to plot this, we also want to retain the datetime
information so that we can have our nicely formatted x-axis. But, we only want
one datetime entry per air temperature measurement. So, we ask R to give us the
first datetime entry for each day for each year in the `summarize()` function.


    temp.daily <- yr.09.11 %>%
      mutate(year = year(datetime)) %>%
      group_by(year, julian) %>%
      summarize(mean_airt = mean(airt, na.rm = TRUE), datetime = first(datetime))

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

#Challenge: Using the daily preceptitation datafile, what is the precipitation
for each year? Does it make more sense to get the total precepitation for each
year, or the average percipitation for each year?

Now that we have our new data file, we want to plot the daily averages.


    dailyPlot <- ggplot(temp.daily,aes(datetime, mean_airt)) +
               geom_point() +
               ggtitle("Daily Avg Air Temperature\nHarvard Forest") +
               theme(plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
              theme(text = element_text(size=20)) +
               xlab("Time") + ylab("Mean Air Temperature")

    ## Error in eval(expr, envir, enclos): could not find function "ggplot"

    #format x axis with dates
    dailyPlot + scale_x_datetime(labels = date_format("%m/%d/%y"))

    ## Error in eval(expr, envir, enclos): object 'dailyPlot' not found