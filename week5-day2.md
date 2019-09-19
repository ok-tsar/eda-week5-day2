
title: "Week 5: Rankings and other patterns"
author: "Cassy Dorff"
date: "5/7/2019"

---

# Overview
Today we will work through a short demonstration on how to illustrate flows using alluvial diagrams. Alluvial diagrams were originally developed to visualize structural change in large complex networks. In essence, they are particularly good at revealing how changes in network structures have developed over time. An excellent illustration is to imagine air traffic flows from one airport to another, which we will do today. I also recommend reading the paper (Mapping Change in Large Networks)[https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0008694] to better understand the connection between networks and alluvial diagrams, as well as see a nice example using citation networks. 

# Recall: `dplyr`
`dplyr` does not accept tables or vectors, just data frames (similar to `ggplot2`)! `dplyr` uses a strategy called "Split - Apply - Combine". Some of the key functions include:

* `select()`: Subset columns.
* `filter()`: Subset rows.
* `arrange()`: Reorders rows.
* `mutate()`: Add columns to existing data.
* `summarise()`: Summarizing data set.
* joins: Combine two data frames together

First, lets library the package.
```{r, message = F, warning=F}
# install.packages("dplyr")
library(dplyr)
```

Today, we will be working with a data set from the `hflights` package. The data set contains all flights from the Houston IAH and HOU airports in 2011. Install the package `hflights`, load it into the library, extract the data frame into a new object called `raw` and inspect the data frame.

**NOTE:** The `::` operator specifies that we want to use the *object* `hflights` from the *package* `hflights`. In the case below, this explicit programming is not necessary. However, it is useful when functions or  objects are contained in multiple packages to avoid confusion. A classic example is the `select()` function that is contained in a number of packages besides `dplyr`.
```{r}
# install.packages("hflights")
library(hflights)
raw <- hflights::hflights
str(raw)
```

## Using `select()` and the Piping Operator `%>%`

Using the **piping operator** will make the `R` code faster and more legible, because we are not saving every output in a separate data frame, but passing it on to a new function. First, let's use only a subsample of variables in the data frame, specifically the year of the flight, the airline, as well as the origin airport, the destination, and the distance between the airports.

Notice a couple of things in the code below:

* We  assign the output to a new data set.
* We use the piping operator to connect commands and create a single flow of operations.
* We use the select function to rename variables.
* Instead of typing each variable, we can select sequences of variables.
* Note that the `everything()` command inside `select()` will select all variables.

```{r}
data <-  raw %>%
  dplyr::select(Month,
                DayOfWeek,
                DepTime,
                ArrTime,
                ArrDelay,
                TailNum,
                Airline = UniqueCarrier, #Renaming the variable
                Time = ActualElapsedTime, #Renaming the variable
                Origin:Cancelled) #Selecting a number of columns. 
names(data)
```

Suppose we didn't really want to select the `Cancelled` variable. We can also use `select()` to drop variables.
```{r}
data <- data %>%
  dplyr::select(-Cancelled)
```

## Alluvial diagrams

Now we have the data ready to explore using alluvial diagrams. Alluvial diagrams show how the same set of items regroup according to different dimensions in the data. Some people refer to this as representing data 'flows' between nodes (groups).

Houston is the fourth most populous city in the nation, with an estimated July 2018 population of 2,325,502 (trailing only New York, Los Angeles and Chicago).  Here is our motivating question: *There are two airports in Houston, which is one of the largest cities in the United States. What are Houston airports' top ten most frequent destinations?* 

We can visualize the combination of origin airport (IAH and HOU) and destination airports using alluvial diagrams.  First, we create a frequency table for all observed combinations of origin and destination airport for the ten most common destinations using `group_by()` and `slice()`.

```{r}
#A great way to get started is to check out the vignette 
vignette(topic = "ggalluvial", package = "ggalluvial")
```

```{r, warning = F, message = F, fig.height = 4, fig.width = 6}
dest_top10 <- data %>% #new data object
  group_by(Dest) %>% #group by destination
  summarise(count = n()) %>% #count up total # of flights
  arrange(desc(count)) %>% #order these from low to high, i.e. descending order
  slice(1:10) #take the top ten values 

flows <- data %>% #new data object
  filter(Dest %in% dest_top10$Dest) %>% #filter data$Dest by top 10
  group_by(Origin, #group by origin, destination, and airline summarize using counts
           Dest,
           Airline) %>%
  summarise(count = n())
```

Now that we have our data ready, we can try our hand at a basic alluvial plot. Below, we use the `ggalluvial` package, which contains the `geom_alluvium()` geometric object.

```{r}
# install.packages("ggalluvial")
library(ggalluvial)



```

We can easily use `fill` to make the graph more interesting. Note, because we want to add `fill` to the alluvium geom, we will map `Origin` onto `fill` within the aesthetic mappings function below. An axis in this case is a dimension (variable) along which the data are vertically grouped at a fixed horizontal position

```{r, warning = F, message = F, fig.height = 4, fig.width = 6}



```

We should also add labels to illustrate the destination airport. To add a bit more clarity to the graphic, we will also want to be able to understand the different groupings shown by the strata in the plot. Here we can use the `geom_stratum()` geometric object to clarify the groupings.

```{r, warning = F, message = F, fig.height = 4, fig.width = 6}



```

The plot above looks nice, and to some extent we have visualized our original question, which asked us to show the top ten most frequent destinations.  But the distinction by fill is not necessarily useful here, and the plot is not that interesting. Perhaps we want to know a bit more about flight patterns, particularly as they relate to the airlines that operate out of Houston. Let's  display an additional variable to represent how these common flight paths vary across airlines.

Note (from vignette): An important feature of these diagrams is the meaningfulness of the vertical axis: No gaps are inserted between the strata, so the total height of the diagram reflects the cumulative weight of the observations

```{r, warning = F, message = F, fig.height = 4, fig.width = 6}



```

The plot above is hardly legible because there are too many airlines displayed! 

A reasonable solution is to only display well-known airlines. First, let's create a quick barplot to check which airlines are the most common carriers on the top ten routes. Then, we will create a new variable coding only the most common, i.e. "Continental" (CO), "Southwest" (WN), and "Other" using `case_when()`.

Note: recall that if you want the heights of the bars in a barplot to represent values in the data, use `stat="identity"` and map a value to the y aesthetic (the barplot will represent the Y values). Also `case_when` allows you to vectorise multiple `if_else` statements, `TRUE` here is equivalent to `ELSE` statement. The help file has a very useful little toy example! 

```{r, warning = F, message = F, fig.height = 2, fig.width = 4}




```

Now, we can re-plot the alluvial diagram. Note: Given a dataset with alluvial structure, stat_flow calculates the centroids (x and y) and weights (heights; ymin and ymax) of alluvial flows between each pair of adjacent axes.

```{r fig.height=4, fig.width=, message=FALSE, warning=FALSE}
ggplot(flows,aes(y = count,axis1 = Origin, axis2 = Dest)) +
  geom_alluvium(aes(fill = Airline_reduced)) +
  geom_stratum(width = 1/12, fill = "black", color = "grey") +
  geom_label(stat = "stratum", label.strata = TRUE) +
  scale_fill_manual(name = "Airline",
                    values = c("Continental" = "blue",
                               "Southwest" = "darkorange",
                               "Other" = "grey")) +
  theme_light() +
  labs(title = "Flights from Houston 2011",
       x = "",
       y = "Number of flights") +
  theme(axis.text.x = element_blank())
```

## References
- super helpful alluvial vignette: https://cran.r-project.org/web/packages/ggalluvial/vignettes/ggalluvial.html
- toy example from: https://github.com/thereseanders/workshop-dataviz-fsu
- alluvial diagrams: https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0008694
- flights data: https://github.com/hadley/hflights
- ggalluvial: http://corybrunson.github.io/ggalluvial/
- https://towardsdatascience.com/alluvial-diagrams-783bbbbe0195
- https://github.com/corybrunson/ggalluvial/issues/18
- http://corybrunson.github.io/ggalluvial/index.html