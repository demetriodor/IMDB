# IMDB
Interactive plots of IMDB ratings data with R

# Data
If you have a list on your IMDB profile with your movie ratings, you can directly export the list as a `csv` file.
If you don't have your own list, you can play with mine, which is available
[here](https://raw.githubusercontent.com/demetriodor/IMDB/master/data/ratings.csv).

# Libraries
```
library(tidyr)
library(dplyr)
library(utf8)
library(devtools)
install_github("nachocab/clickme")
library(clickme)
```

# Load and transform data
```
d<-read.csv("ratings 2019.csv") %>% 
  filter(Title.Type=='movie') %>%
  mutate (Directors = recode (Directors, 'Joel Coen, Ethan Coen' = 'Coen Brothers', 
                              'Luc Dardenne, Jean-Pierre Dardenne'='Dardenne Brothers',
                              'Lilly Wachowski, Lana Wachowski'='Wachowski Sisters',
                              'Vittorio Taviani, Paolo Taviani'='Taviani Brothers',
                              'Bobby Farrelly, Peter Farrelly'='Farelly Borthers')) %>%
  separate(Genres, sep=', ', into = paste("Genre", 1:7, sep = ".")) %>%
  separate(Directors, sep=', ', into = paste("Director", 1:5, sep = ".")) %>%
  mutate(title =  utf8_encode(as.character(Title))) %>%
  mutate(mine.j = jitter(Your.Rating, amount=0.1)) %>%
  mutate (imdb.j = jitter(IMDb.Rating, amount=0.05))

#Recode the years
d$period <- cut(d$Year, breaks=c(-Inf, 1940,1950, 1955, 1960, 1965, 1970, 1975, 
                                 1980, 1985, 1990, 1995, 2000, 2003, 2007, 2010, 
                                 2013, 2014, 2015, 2016, 2017,2020), 
                        labels=c('1920-1940','1941-1950','1951-1955','1956-1960','1961-1965','1966-1970','1971-1975',
                                 '1976-1980','1981-1985','1986-1990','1991-1995','1996-2000','2001-2003','2004-2007','
                                 2008-2010','2011-2013', '2014','2015','2016','2017', '2018' ))

d$decade <- cut(d$Year, breaks=c(-Inf, 1940,1950, 1960, 1970, 1980, 1990, 2000, 2010, 2020), 
                        labels=c('<1940','1940s','1950s','1960s','1970s', '1980s', '1990s', '2000s', '2010s'))              
```

# Summaries 
Let's summarize the number of movies per genre first. Each movies can have up to five genres, so we want to collect all before counting.
```
genres<- d %>%
  select(starts_with("Genre")) %>%
  gather(key, value, na.rm = TRUE) %>%
  count(value) %>%
  arrange(desc(n))
genres
```
And now a summary of total and average rating per director. If need to rearrange the data first, because the director names are distributed in several columns.

```
d2<-d %>% 
  gather(key = Director.N, value = Director, Director.1:Director.4 ) %>%
  filter(!is.na(Director))

d2 %>%
  group_by(Director) %>%
  summarize(total.rating = sum(Your.Rating, na.rm=TRUE),
            mean.rating = mean(Your.Rating, na.rm=TRUE),
            films = n()) %>%
  arrange(desc(total.rating))
d2
```
# Interactive plot
And now for the interactive plot of the personal ratings vs. the IMDB ratings. 
The plot is done with the `clickme` library, which seems not to be maintained or at least updated any more. 
The code is adapted from an old version that worked. But some of the parameters are not passsed successfully anymore.
```
clickme(points,
        x = d$imdb.j, y = d$mine.j, 
        title = "Scatterplot of general and personal movie ratings", 
        subtitle = "(Hover over a dot to reveal the title. Click on the menu to the right to select.)",
        formats = list(x = ".1f", y = ".0f"),
        opacity = .8, jitter = 0.2, radius = 5,
        height = 600, width = 900,
        x_title = "Internet Movie Database (IMDb) rating", 
        y_title = "My rating",
        names = d$title,
        color_groups = d$period,
        color_title = "Time period",
        file = "F:/imdb_dots_2019.html")
```
The code above works and produces an interactive plot in the browser. 
But the `height`, `weight`, `jitter`, and `file` parameters are specified yet do not affect the output. 
Maybe the syntax has changed, but I cannot find the documentation. 
In any case, we can adjust these parameters directly in the output html file by editing the file in any text processor. 
The output html file is in a folder called 'output' within the folder of the 'clickme' package in the folder of your `R` installation.

The resulting html with the plot is here. The interactive plot itself is here. And below is a picture of the plot.

# The End (for now)




