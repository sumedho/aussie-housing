Banks - New Housing Loans Approvals
================

Creating the plot
-----------------

This is a short tutorial on creating the plot shown. It makes use of the subtitle and caption features that have recently been added to ggplot2. Until these changes are available in CRAN they need to be installed with the following:

`devtools::install_github("hadley/ggplot2")`

If the devtools aren't installed they can be installed using the command:

`install.packages("devtools")`

#### Loading the libraries

``` r
library(ggplot2)
library(dplyr)
library(reshape2)
```

#### Reading and preparing the data

The data is read from the csv file. It has a number of columns, but the ones used in this tutorial are date, owner and investor. These are the quartely date, the number of loans to owner occupiers and number of loans to investors respectively.

``` r
ams <- read.csv("data/adi_monthly_stats.csv", stringsAsFactors=FALSE)
str(ams)
```

    ## 'data.frame':    32 obs. of  12 variables:
    ##  $ date           : chr  "31/03/08" "30/06/08" "30/09/08" "31/12/08" ...
    ##  $ outside_service: num  1567 1487 1230 1124 863 ...
    ##  $ thirdparty     : num  23436 25595 22331 23189 22295 ...
    ##  $ owner          : num  36444 36123 34970 37674 41663 ...
    ##  $ investor       : num  17377 17653 15977 16044 15203 ...
    ##  $ lvr1           : num  16511 15972 14603 15204 15638 ...
    ##  $ lvr2           : num  17503 18014 17133 18422 19688 ...
    ##  $ lvr3           : num  9930 10582 9456 8520 8879 ...
    ##  $ lvr4           : num  9877 9208 9756 11571 12662 ...
    ##  $ io_loans       : num  19017 20654 16534 16393 15881 ...
    ##  $ low_doc        : num  6413 6120 6569 5589 5066 ...
    ##  $ non_stan       : int  449 501 484 457 368 278 103 91 98 147 ...

##### Melt data and change dates

To plot the data, it needs to be converted from short format to long. The melt command takes the owner and investor columns and **melts** them together based on the date. The date is converted from a character column to a POSIXlt date format. Columns in the dataframe are then renamed.

``` r
df <- melt(ams,id.vars="date",measure.vars = c("owner","investor"))
df$date <- strptime(df$date,format="%d/%m/%y")
colnames(df) <- c("date","type","value")
str(df)
```

    ## 'data.frame':    64 obs. of  3 variables:
    ##  $ date : POSIXlt, format: "2008-03-31" "2008-06-30" ...
    ##  $ type : Factor w/ 2 levels "owner","investor": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ value: num  36444 36123 34970 37674 41663 ...

##### Convert millions to billions

The data is supplied in millions of dollars but is easier to understand if converted to billions. This is done by multiplying the number by 1e6 (Millions) and dividing by 1e9 (Billions)....or more simply dividing millions by billions which results in multiplying the value by 0.001.

``` r
df$value <- df$value*0.001
```

#### Creating information for plot

The data for annotating the plot is stored in variables to make managing the data easier. This makes the final ggplot commands easier to read and all changes can be done in one spot.

``` r
change <- strptime("22/5/2015",format="%d/%m/%Y")
caption <-"Source: APRA (www.apra.gov.au) from the Quarterly Authorised Deposit-taking 
Institution Property Exposures. The black vertical line indicates when changes to curb investor loans 
started being implemented (approx middle of may 2015)"
title <- "Banks - New Housing Loans Approvals"
subtitle = "($ billion, domestic books)"
```

#### The final plot

The final plot has a number of elements.

-   As the data is quarterly a smoothing is applied which makes the trends easier to see. This was created using `geom_smooth(span=0.2,se=FALSE)`. The span adjusts the level of smoothing and `se=FALSE` turns off the shaded confidence region.
-   Title, subtitle and caption (subtitle and caption are new features in ggplot2) are added with the next line.
-   The vertical line indicating approximate date for the APRA captial requirement changes is created using `geom_vline(aes(xintercept=as.numeric(change)))`
-   The theme and colour palette were changed to a lighter, more subtle look using `theme_light()` and `scale_color_brewer(palette="Paired")`
-   The last three theme lines change various aspects of the text. The subtitle and caption text type and family is adjusted along with the final margins of the plot.
-   The plot is stored in the variable p and the final plot is displayed by calling this variable.

``` r
p <- ggplot(df,aes(date,value,col=type))+
  geom_smooth(span=0.2,se=FALSE)+
  labs(title=title,
       subtitle=subtitle,
       caption=caption)+
  ylab("Value in billions")+
  xlab("Date")+
  geom_vline(aes(xintercept=as.numeric(change)))+
  theme_light()+
  scale_color_brewer(palette="Paired")+
  theme(plot.caption=element_text(size=8, hjust=0, margin=margin(t=15)))+
  theme(plot.subtitle=element_text(family="serif",face="italic"))+
  theme(plot.margin=unit(rep(0.5, 4), "cm"))
p
```

![](owner_vs_investor_files/figure-markdown_github/final%20plot-1.png)
