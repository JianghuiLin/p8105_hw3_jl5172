# p8105_hw3_jl5172

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE)
```

Load package and data cleaning
```{r}
library(tidyverse)
library(p8105.datasets)
data("brfss_smart2010")
brfss<-janitor::clean_names(brfss_smart2010) #clean names
brfss<-rename(brfss,state=locationabbr,location=locationdesc) # use appropriate variable names
brfss<-filter(brfss,topic == "Overall Health") # fiocusing on Overall Health
brfss<-filter(brfss,response=="Excellent"|response=="Very good"|response=="Good"|response=="Fair"|response=="Poor") #Keep responses required
mutate(brfss,response=as.factor(brfss$response)) 
brfss$response<-fct_relevel(brfss$response,"Excellent","Very good","Good","Fair","Poor") # organize response as facctors in required ordering

```

In 2002, which states were observed at 7 locations?
```{r}
brfss %>% 
  filter(year=="2002") %>%   #filter by year 2002
  distinct(state,location) %>%  # distinct by state and location
  count(state) %>%  # count the number of observations for each states
  filter(n==7)  # looking for states which have been observed for 7 locations
```
We can see that CT,FL and NC were observed at 7 location in 2002.

Make a “spaghetti plot” that shows the number of locations in each state from 2002 to 2010.
```{r}
brfss %>% 
  group_by(state,year) %>%  # group by state and year
  distinct(location) %>%   #distinct by locations
  count(state) %>%  #count how many locations in each state were observed in each year
  ggplot(aes(x = year, y = n)) +  #use ggplot to plot spaghetti
    geom_line(aes(color = state))+  # color each line by state 
  labs(
    title = "Number Of Locations in Each States 2002-2010",
    x = "Year",
    y = "Number of Locations",
    caption = "Data from the p8105.datasets package"
  ) +
  viridis::scale_color_viridis(
    name = "", 
    discrete = TRUE)+
  theme(legend.position = "right")
```



Make a table showing, for the years 2002, 2006, and 2010, the mean and standard deviation of the proportion of “Excellent” responses across locations in NY State.
```{r}
brfss %>% 
  filter(year=="2002"|year=="2006"|year=="2010",state=="NY",response=="Excellent") %>% 
  group_by(year) %>% 
  summarize(
          mean_proportion_excellent_NY =round(mean(data_value)/100,3),
          sd_excellent_response=round(sd(data_value),3)) %>% 
  View()
```
We can see from the table that the mean propoortion of excellent response is highest in 2002. There is not much difference between 2006 and 2010. For the variance,2002 is also the highest one.



For each year and state, compute the average proportion in each response category (taking the average across locations in a state). Make a five-panel plot that shows, for each response category separately, the distribution of these state-level averages over time.
```{r}
brfss %>% 
  spread(key= response, value = data_value) %>% 
  janitor::clean_names() %>% 
  group_by(state,year) %>% 
  summarize(excellent_mean=mean(excellent,na.rm = T),
            verygood_mean=mean(very_good,na.rm = T),
            good_mean=mean(good,na.rm=T),
            fair_mean=mean(fair,na.rm=T),
            poor_mean=mean(poor,na.rm=T)
            ) %>% 
 gather(key=response_variable,value=mean_value,excellent_mean:poor_mean) %>% 
  ggplot(aes(x=year,y=mean_value,color=state))+
  geom_line()+
  facet_grid(~response_variable)+
  labs(
    title = "Average Proportion In Each Response Category",
    x = "Mean Proportion",
    y = "Year",
    caption = "Data from the p8105.datasets package"
  )
```



#Problem 2
Clean data Use suitable variable names.
```{r}
cart<-janitor::clean_names(instacart)

```
write a short description of the dataset, noting the size and structure of the data, describing some key variables, and giving illstrative examples of observations.

Exploration of the dataset. 
```{r}
cart %>% 
  distinct(product_id) %>% 
  nrow()  # There are 39123 products

cart %>% 
  distinct(order_id) %>% 
  nrow()    #There are 131209 orders.

cart %>% 
  distinct(department) %>% 
  nrow()  # There are 21 departmets.

```
According to my basic analysis, there are 39123 distinct products being ordered, 131209 orders have been placed and 21 departments.The column 'order_dow' has been changed to order_day(which day of the week), 0 represent Sunday ,1 represents Monday,etc...In the column reordered,the corresponding value is 1 if this product has been ordered by this user in the past,0 otherwise.The column add_to_cart_order means order in which each product was added to cart.Key variables are order_id, product_id, reordered, user_id, order_number as they are some essential information for us to observe the shopping behaviors of the users in Instacart.

How many aisles are there? 
```{r}
cart %>% 
  distinct(aisle_id) %>% 
  nrow()
```
We can see that their are 134 distinct id in the column aisle_id thus there are 134 aiseles there.




which aisles are the most items ordered from?
```{r}
aisles_by_norder<-cart %>% 
  group_by(aisle) %>% 
  summarise(most_order_aisle=n()) %>% # use n() to summarize how many  times does each aisle_id appear.
  arrange(desc(most_order_aisle)) %>%  #arrange them in descending order
  as.tibble() 
  
famous_aisles<-aisles_by_norder$aisle[1:5] # famous aisle would appear on top of most_order_aisle.
famous_aisles
```
We can see that the most famous aisle is "fresh vegetables" with 150609 items ordered from that aisle.The top 5 famous aisles are"fresh fruits","packaged vegetables fruits","yogurt" and "packaged cheese".


Make a plot that shows the number of items ordered in each aisle. Order aisles sensibly, and organize your plot so others can read it.
```{r}
cart %>% 
  group_by(aisle) %>% 
  summarize(number_items_aisle = n()) %>% 
  arrange(desc(number_items_aisle)) %>% 
  ggplot(aes(x = reorder(aisle, -number_items_aisle), y = number_items_aisle, fill = aisle)) +
  geom_bar(stat="identity")+
  coord_flip()+
  theme(text = element_text(size = 6), plot.title = element_text(hjust = 0.5), legend.position = "hide", title = element_text(size = 10,face='bold'))+
  labs(title="Number of Items Ordered in Each Aisle",x="Aisle",y="Number of Items",caption = "Data from p8105.datasets package")
```

Make a table showing the most popular item in each of the aisles “baking ingredients”, “dog food care”, and “packaged vegetables fruits”.
```{r}
cart %>% 
  filter(aisle=="baking ingredients"|aisle=="dog food care"|aisle=="packaged vegetables fruits") %>%  #filter only these three aisles out
  group_by(aisle,product_name) %>% # pre-group based on aisle and product name
  summarise(n=n()) %>% # summarize number of order per product
  top_n(1,n) %>%  #return 1 row per group, order = 'n' column
  as.tibble() # form a table
```


Make a table showing the mean hour of the day at which Pink Lady Apples and Coffee Ice Cream are ordered on each day of the week; format this table for human readers (i.e. produce a 2 x 7 table).
```{r}
apple_coffee<-cart %>% 
  filter(product_name=="Pink Lady Apples"|product_name=="Coffee Ice Cream") %>% 
  select(product_name,order_hour_of_day,order_dow) %>% 
  group_by(product_name,order_dow) %>% 
  summarise(mean_time_order=mean(order_hour_of_day)) %>% 
  spread(key=order_dow,value=mean_time_order) %>% 
  as.tibble()

colnames(apple_coffee)<-c("Product Name","Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday")
  apple_coffee
  
```



#Problem 3
The goal is to do some exploration of this dataset. To that end, write a short description of the dataset, noting the size and structure of the data, describing some key variables, and indicating the extent to which missing data is an issue. Then, do or answer the following (commenting on the results of each):
```{r}
ny<-janitor::clean_names(ny_noaa)
nrow(ny)
ncol(ny)
```
This is a dataset which contains `r ncol(ny)` variables and `r nrow(ny)` rows with weather information from 1981-2010. Each row contain information from distinct New York weather station with weather station ID,data of obeservation, precipitation(tenths of mm),snowfall(mm),snow depth(mm),maximum temperature(tenths degrees C),minimum temperature(tenths of degrees C).


Do some data cleaning. Create separate variables for year, month, and day. Ensure observations for temperature, precipitation, and snowfall are given in reasonable units. 
```{r}
ny<-ny %>% 
separate(date,c("year", "month", "day"), sep = "-") %>% 
mutate(prcp=as.numeric(prcp)/10,
tmax=as.numeric(tmax)/10,
tmin=as.numeric(tmin)/10) %>% 
rename(precipitation_mm=prcp) %>% 
rename(tmin_C=tmin) %>% 
rename(tmax_C=tmax) %>% 
rename(snowfall_mm=snow) %>% 
rename(snow_depth_mm=snwd)

```

For snowfall, what are the most commonly observed values? Why?
```{r}
ny %>% 
  group_by(snowfall_mm) %>% 
  summarize(n_snowfall=n()) %>% 
  arrange(desc(n_snowfall)) %>% 
  top_n(5)
  
```
For snowfall,the most commonly observed value is 0mm because for most of the day, there is no snowfall in New York. 25mm(2nd),13mm(3rd) and 51mm(4th) are also observed frequently.


Make a two-panel plot showing the average max temperature in January and in July in each station across years. Is there any observable / interpretable structure? Any outliers?
```{r}
ny %>% 
  filter(month=="01"|month=="07",!is.na(tmax_C)) %>% 
  mutate(month=recode(month,"01"="Jan")) %>% 
  mutate(month=recode(month,"07"="July")) %>% 
  group_by(id,year,month) %>% 
  summarize(mean_tmax_C=mean(tmax_C)) %>% 
  ggplot(aes(x=year,y=mean_tmax_C),fill=month)+
  geom_boxplot()+
  facet_grid(~month)+
  scale_x_discrete(breaks = c(1981, 1985, 1989, 1993, 1997, 2001,2005,2010))+
  labs(title="Boxplots For Average Max Temperation In Jan & July Across 1981-2010",x="Year",y="Average Maximum Temperature/C",caption = "Data from p8105.datasets package")
  
```
Generally Speaking,the avarage maximum temperatures for July across years are much higher than that of January.There are outliers in almost each year/month.


Make a two-panel plot showing
(i) tmax vs tmin for the full dataset (note that a scatterplot may not be the best option); 
```{r}
library(ggridges)
library(ggplot2)
library(hexbin)
ny %>% 
  filter(!is.na(tmax_C),!is.na(tmin_C)) %>% 
  ggplot(aes(x=tmin_C,y=tmax_C))+
  geom_hex()+labs(title="Hexplot of Maximum Temperature",x="Minimum Temperature/°C",y="Maximum Temperature/°C",caption = "Data from the ny_noaa package")
  
```
We can see from the hexplot that maximum temperature of a day are generally proprotional with minimum temperatures.The most frequent range of minimum temperatures lied between -15°C and 20°C. The most frequent range of maximum temperatures lied between -5°C and 30°C.

Make a plot showing the distribution of snowfall values greater than 0 and less than 100 separately by year.
```{r}
ny %>% 
  filter(!is.na(snowfall_mm),snowfall_mm>0,snowfall_mm<100) %>% 
  ggplot(aes(x = snowfall_mm,y=year,color=year)) +
  geom_density_ridges(alpha = .5, scale = 0.85) +
  labs(title="Density Plot of Snowfall by Year",x="Snowfall/mm",y="Year",
       caption = "Data from the ny_noaa package")
  
```
In this density plot, each line represent the snowfall distribution for that year.We can see that the most frequent snowfall(mm) is about 0-30mm for each year.


