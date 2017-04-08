---
title: "Hawker Centres in Singapore"
author: "Vivek Kalyan"
date: "05/04/2017"
---



An exploration of various attributes of hawker centres in Singapore using R.

Packages required:

* data.table
* sp
* rgdal
* spatstat
* ggmap
* maptools
* ggplot2



### Data
Data is taken from [NEA website](http://www.nea.gov.sg/public-health/hawker-centres/tender-notice) and converted to csv using Tabula.

{% highlight r %}
tenders <- fread("tabula-tender-bids-from-mar-2012-to-jan-2017.csv", 
  header = F, fill= T)
# add table headers
colnames(tenders) <- c("centre", "stall", "area", "trade", "bid", "month")
{% endhighlight %}

### Cleaning up the data table
The data and conversion using Tabula is not perfect, so we need to clean up the table. Inspecting the table, we notice that there are empty rows. These empty rows are removed.

We also notice that the hawker centres actually have 3 different types, namely Cooked stalls, Lock-up stalls and Market stalls. They are categorised using label rows. We remove these label rows from the table and attach the appropriate labels to the corresponding rows.


{% highlight r %}
# remove empty rows
tenders <- tenders[centre != "",]
# remove label rows
tenders <- tenders[!1135,]
tenders <- tenders[!1749,]
# set type to corresponding type
tenders[1:1134, type:="cooked",]
tenders[1135:1748, type:="lockup",]
tenders[1749:nrow(tenders), type:="market",]
{% endhighlight %}

We also notice that there is a group of rows that are misaligned. We align the columns correctly and extract the data from the column in which the columns were combined using regex.


{% highlight r %}
tofix <- tenders[month=="",]
tofix[,c("month","bid","trade","area"):=list(bid,trade,area,stall),]
tofix[,stall:=stringi::stri_extract_last(tofix[,centre,], 
  regex="[0-9]{2}-[0-9]{2,3}"),]
tofix[,centre:=stringi::stri_replace_last(tofix[,centre,], 
  replacement = "",regex=" [0-9]{2}-[0-9]{2,3}"),]
tenders[month=="",] <- tofix
{% endhighlight %}

Looking at the list of centres we notice one of the names clearly being named wrongly, so we change it.


{% highlight r %}
tenders[centre=="BLK 51 OLD AIRPORT ROAD OLD AIRPORT ROA", 
  centre:="BLK 51 OLD AIRPORT ROAD",]
{% endhighlight %}

We then format the string data into manipulatable data for bid, data and area.

Some of the area data is badly formated, with out of position decimal places or extra spaces in between the numbers. We strip the string of all decimal places and spaces, convert numerically and divide by 100 to get the accurate numeric value.


{% highlight r %}
tenders[,bidNum:=as.numeric(gsub(bid,pattern="\\$|,", replacement="")),]
tenders[,date:= as.Date(paste0("01-", month),"%d-%b-%Y"),]
tenders[,areaNum:=as.numeric(gsub(area,pattern=" |\\.", 
  replacement=""))/100,]
tenders[,priceM2:=bidNum/areaNum,]
{% endhighlight %}

Now, we can begin taking a closer look at the data.

### Initial Analysis

Let's begin by asking some simple questions.

#### Is there a relation between area and price/m<sup>2</sup>?

{% highlight r %}
ggplot(tenders[,list(price=mean(priceM2)), by=areaNum], aes(areaNum, price)) + 
  geom_point(alpha=0.5,color="#DD8888") + 
  xlab("Area (m2)") + 
  ylab("Price ($)") + 
  ggtitle("Bids for Hawker Centres 2012-2017")
{% endhighlight %}

![plot of chunk unnamed-chunk-7](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-7-1.png)

We see that the highest bid stalls are those with around 5m<sup>2</sup> in area. This is not surprising that these stalls have the highest demand as most stall owners would not require a large stall to prepare their food. 

#### Is there a relation between type of stall and price/m<sup>2</sup>?

{% highlight r %}
ggplot(tenders[,list(price=mean(priceM2)), by=type], aes(type, price)) +
geom_bar(color="black", fill= "#DD8888",stat="identity") + 
xlab("Type of stall") + 
ylab("Price ($)") + 
ggtitle("Average bid for Hawker Centres 2012-2017")
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-8-1.png)

Cooked food stalls have the highest prices, while market stalls have the lowest prices.

#### Does the price/m<sup>2</sup> of stalls change over time?

{% highlight r %}
ggplot(tenders[,list(price=mean(priceM2)), by=date], aes(date,price)) + 
geom_point(color="#7F2626") + 
geom_line(color="#DD8888") + 
geom_smooth(method = "lm", color="#FF5555", alpha=0.2) + 
xlab("Time") + 
ylab("Price ($)") + 
ggtitle("Average bid for Hawker Centres 2012-2017") + 
scale_x_date(date_breaks = "1 year", date_labels = "%Y")
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-9-1.png)

The trendline shows that the price/m<sup>2</sup> is increasing over time. The increase is pretty gradual, so a major reason for this could be inflation.

### Spatial Data
Spatial data is taken from [here](https://data.gov.sg/dataset/hawker-centres).

To get geodata for our hawker centres, we make use of google's geocoding service that returns the coordinates of our hawker centres. To make it easier for the service to parse our locations, we add ", Singapore" to provide some context to google's service. We then merge this data together.

To prevent the need to make ~3k requests every time we create this document, we save the coordinates data into a file that can be loaded.

{% highlight r %}
if (file.exists("centres-geocoded.csv")) {
  centres <- read.csv("centres-geocoded.csv")
} else {
  centres <- tenders[,list(count=.N), by=centre]
  centres[,location:=paste0(centre, ", Singapore"),]
  g <- geocode(centres[,location,], output="latlon",  source="google", sensor = F)
  centres <- cbind(centres, g)
  write.csv(centres, "centres-geocoded.csv")
}
tenders.sp <- merge(tenders, centres, by.x = "centre", 
  by.y = "centre", all.x = T)
tenders.sp[,c("count","location"):=NULL]
{% endhighlight %}

### Spatial Analysis

Is there a specific spatial distribution with regard to number of bids per type?


{% highlight r %}
ggplot(tenders.sp, aes(x=lon, y=lat)) + 
geom_point() + 
geom_density2d() + 
coord_fixed() + 
facet_wrap(~trade, ncol = 5) + 
theme(axis.text = element_blank(), axis.ticks = element_blank()) + 
xlab("") + 
ylab("") + 
ggtitle("Density of bids for Hawker Centres 2012-2017")
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-11-1.png)

We notice that certain types of food stalls are closely packed such as mutton. However, most types of food do not have a specific region at where they are located.

### Spatial Point Patterns

We create a table for the centres, with the location, average price and total number of bids. We convert this data into 'ppp' format which allows us to use spatstat on.


{% highlight r %}
centres.sp <- tenders.sp[lat > 0, list(lon=lon[1], lat=lat[1], 
  price=mean(priceM2),count=.N), by=centre]
coordinates(centres.sp) <- c('lon', 'lat')
centres.ppp <- unmark(as.ppp(centres.sp))
{% endhighlight %}

We load a map of Singapore and define the window to be the Singapore map instead of the centres data. This allows us to visualize the data in our entire study area.


{% highlight r %}
sg <- readOGR(".","sg-all")
{% endhighlight %}

{% highlight r %}
sg.window <- as.owin(sg)
centres.ppp <- centres.ppp[sg.window]
plot(centres.ppp, main="Hawker Centres 2012-2017")
{% endhighlight %}

![plot of chunk unnamed-chunk-14](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-14-1.png)

Our preliminary plot suggests that the spatial distribution of the hawker centres is clustered. We plot the Ripley's reduced second moment function to check our hypothesis.


{% highlight r %}
plot(Kest(centres.ppp), 
  main="Spatial Distribution of Hawker Centres 2012-207")
{% endhighlight %}

![plot of chunk unnamed-chunk-15](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-15-1.png)

The graph shows that at all values of r, the points show clustering behavior (the lines are above the blue line, which is the line we expect if the points were completely random). This means that no matter how we define our areas in the map, the points exhibit clustering.

We plot the density and contour maps to locate the area that they are clustering towards.


{% highlight r %}
plot(density(centres.ppp,0.02), 
  main="Spatial Density of Hawker Centres 2012-2017")
{% endhighlight %}

![plot of chunk unnamed-chunk-16](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-16-1.png)

{% highlight r %}
contour(density(centres.ppp, 0.02), 
  main="Spatial Contour of Hawker Centres 2012-2017")
{% endhighlight %}

![plot of chunk unnamed-chunk-17](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-17-1.png)

The clustering seems to be towards the centre of Singapore. It is possible that the locations of the hawker centres is related to the population density. We load a population raster file for Singapore and plot the estimate of intensity of the hawker centre point pattern as a function of the population. 


{% highlight r %}
pop <- as.im(readGDAL("sg-pop.tif"))
{% endhighlight %}

{% highlight r %}
plot(rhohat(centres.ppp, pop), 
  main="Intensity of hawker centres vs population", 
  xlab="Population", 
  ylab="Intensity")
{% endhighlight %}

![plot of chunk unnamed-chunk-19](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-19-1.png)

We can also weigh each centre by its average tender price.


{% highlight r %}
plot(rhohat(centres.ppp, pop, weights = centres.sp$price), 
  main="Intensity of hawker centres (weighted by price) vs population", 
  xlab="Population", 
  ylab="Intensity")
{% endhighlight %}

![plot of chunk unnamed-chunk-20](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-20-1.png)

Generally, the intensity of the hawker centres is related to the population in the area. However, there are some outliers. In areas with very high population, the intensity of hawker centres is much lower than what we would expect based on the rest of the pattern. We want to visualize where these outliers are located, thus we plot a Singapore population density map with the locations of the centres. 


{% highlight r %}
plot(pop, 
  main="Hawker Centres 2012-2017 on Singapore population density map")
plot(centres.ppp, col="white", add=T)
{% endhighlight %}

![plot of chunk unnamed-chunk-21](/img/rfigures/rsource/hawker-centres/2017-04-05-hawker-centres-singapore/unnamed-chunk-21-1.png)

We see that the areas with high population and low hawker centres are places such as Woodlands, Chua Chu Kang, Hougang. Considering that our data is created from tender notices from 2012-2017, it is possible that hawker centres at these locations did not have tenders for their stalls.

This data exploration could be useful in determining where Singapore Government can open hawker centres and how much a prospective vendor should bid to apply for a tender.
