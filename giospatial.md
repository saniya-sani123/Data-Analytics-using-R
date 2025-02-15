


# Introduction{-}

This tutorial introduces geospatial data visualization in R. 

```{r diff, echo=FALSE, out.width= "15%", out.extra='style="float:right; padding:10px"'}
knitr::include_graphics("https://slcladal.github.io/images/gy_chili.jpg")
```

This tutorial is aimed at beginners and intermediate users of R with the aim of showcasing how to visualize geospatial data, i.e. how to generate maps in R, and how to prepare data for geospatial visualizations using R. The aim is not to provide a fully-fledged analysis but rather to show and exemplify selected useful methods associated with generating maps. Very recommendable and detailed resources for geospatial data visualization using R can be found [here](https://keen-swartz-3146c4.netlify.app/), [here](https://bookdown.org/content/b298e479-b1ab-49fa-b83d-a57c2b034d49/),  or [here](https://geocompr.robinlovelace.net/index.html). If you are interested in cartography, [here](https://raw.githubusercontent.com/rstudio/cheatsheets/main/cartography.pdf) is a cheat sheet for cartography with R. 

<div class="warning" style='padding:0.1em; background-color:#f2f2f2; color:#51247a'>
<span>
<p style='margin-top:1em; text-align:center'>
The entire R Notebook for the tutorial can be downloaded [**here**](https://slcladal.github.io/content/gviz.Rmd).  If you want to render the R Notebook on your machine, i.e. knitting the document to html or a pdf, you need to make sure that you have R and RStudio installed and you also need to download the [**bibliography file**](https://slcladal.github.io/content/bibliography.bib) and store it in the same folder where you store the Rmd file. <br>
</p>
<p style='margin-left:1em;'>
</p></span>
</div>

<br>



**Preparation and session set up**

This tutorial is based on R. If you have not installed R or are new to it, you will find an introduction to and more information how to use R [here](https://slcladal.github.io/intror.html). For this tutorials, we need to install certain *packages* from an R *library* so that the scripts shown below are executed without errors. Before turning to the code below, please install the packages by running the code below this paragraph. If you have already installed the packages mentioned below, then you can skip ahead and ignore this section. To install the necessary packages, simply run the following code - it may take some time (between 1 and 5 minutes to install all of the libraries so you do not need to worry if it takes some time).



<div class="warning" style='padding:0.1em; background-color:#f2f2f2; color:#51247a'>
<span>
<p style='margin-top:1em; text-align:center'>
<b>NOTE</b><br><br>The installation of the packages will be relatively data intensive due to some of the required packages containing shp-files (shape files) - which renders these packages to be larger big in comparison to other R packages. It is thus recommendable to be logged into an institutional network that has a decent connectivity and download rate (e.g., a university network). </p>
<p style='margin-left:1em;'>
</p></span>
</div>



```{r prep1, eval = F, message=FALSE, warning=FALSE}
install.packages("sf")
install.packages("raster")
install.packages("dplyr")
install.packages("spData")
install.packages("tmap")  
install.packages("leaflet")
install.packages("ggplot2")
install.packages("spDataLarge",
                 repos = "https://nowosad.github.io/drat/", type = "source")
install.packages("ggspatial")
install.packages("rnaturalearth")
install.packages("rnaturalearthdata")
install.packages("ggmap")
install.packages("leaflet")
install.packages("maptools")
install.packages("rgdal")
install.packages("scales")
install.packages("maps")
install.packages("here")
install.packages("rgeos")
install.packages("oz")
# install klippy for copy-to-clipboard button in code chunks
install.packages("remotes")
remotes::install_github("rlesur/klippy")
```

Now that we have installed the packages, we activate them as shown below.


```{r pre2, message=F, warning=F}
library(sf)
library(raster)
library(dplyr)
library(spData)
library(spDataLarge)
library(tmap)  
library(ggplot2)
library(ggspatial)
library(rnaturalearth)
library(ggmap)
library(leaflet)
library(maptools)
library(rgdal)
library(scales)
library(rgeos)
library(oz)
# activate klippy for copy-to-clipboard button
#klippy::klippy()
```


Once you have installed R and RStudio and also initiated the session by executing the code shown above, you are good to go.


# Creating Basic Maps {-}

We will start by generating maps of the world using a in-build `world` data set that is part of the `spData` package and the `plot` function for generating the map.

```{r geo1, message=F, warning=F}
# Load the world data package
data(world)
plot(world)
```

We see that the  `world` data set contains information on various factors, such as information about regions (e.g., `continent` or `subregion`), country names (`name_long`),  population size (`pop`) or life expectancy (`lifeExp`). We can use this information to show a specific map as shown below.

```{r geo2, message=F, warning=F}
plot(world["lifeExp"])
```


We can use the `world` data set and filter for specific features, e.g., we can visualize only a single continent by filtering for the continent we are interested in. In addition, we define the x-axis and y-axis limits so that we zoom in on the region of interest.


```{r geo4, message=F, warning=F}
# extract europe (exclude russia and iceland)
world_eur <- world %>%
  dplyr::filter(continent == "Europe", 
                name_long != "Russian Federation", 
                name_long != "Iceland") %>%
  dplyr::select(name_long, geom)
# plot
plot(world_eur,
     xlim = c(5, 10),
     ylim = c(30, 70),
     main = "")
```


We can also overlay information such as population size over continents and countries as shown below.

```{r geo6, message=F, warning=F}
# plot world map
plot(world["continent"], reset = FALSE)
# define size bases on population size
cex <- sqrt(world$pop) / 10000
# center world map
world_cents <- sf::st_centroid(world, of_largest = TRUE)
# plot
plot(sf::st_geometry(world_cents), 
     add = TRUE, 
     cex = cex)
```

Overlaying is interesting because it allows us to highlight certain regions or countries as shown below.


```{r geo8, message=F, warning=F}
# extract map of europe
world_eur <- world %>%
  dplyr::filter(continent == "Europe")
# extract germany
ger <- world %>%
  dplyr::filter(name_long == "Germany")
# plot germany
plot(sf::st_geometry(ger), expandBB = c(.2, .2, .2, .2), col = "gray", lwd = 3)
# plot europe
plot(world_eur[0], add = TRUE)
```


We can also add information to the `world` data set and use the added information to generate customized plots.

```{r geo10, message=F, warning=F}
# countries I have been to
countries <- c("United States", "Norway", "France", "United Arab Emirates", 
             "Qatar", "Sweden", "Poland", "Austria", "Hungary", "Romania", 
             "Germany", "Bulgaria", "Greece", "Turkey", "Croatia", 
             "Switzerland", "Belgium", "Netherlands", "Spain", "Ireland", 
             "Australia", "China", "Italy", "Denmark", "United Kingdom", 
             "Slovenia", "Finland", "Slovakia", "Czech Republic", "Japan", 
             "Saudi Arabia", "Serbia","India")
# data frame with countries I have visited
visited <- world %>%
  dplyr::filter(name_long %in% countries)
# plot world
plot(world[0], col = "lightgray")
# overlay countries I have visited in orange 
plot(sf::st_geometry(visited), add = TRUE, col = "orange")
```

If you want to plot Australia, you can also simply use the `oz` package.


```{r oz, echo = F, eval = F}
# show map
oz::oz(states=TRUE, col="darkgreen")
```

# Creating Maps with ggplot2 {-}

So far, we have used the base `plot` function to generate maps. However, it is also possible to use `ggplot2` to generate maps and the easiest way is to use `borders` to draw a map. 

```{r geo11, message=FALSE, warning=FALSE}
# plot map
ggplot() +
  borders()
```


Another option is to add `geom_sf` to a `ggplot2` object as shown below. A nice feature is that we can add perspective and projection.


```{r geo12, message=FALSE, warning=FALSE}
# plot map
ggplot(data = world) +
  geom_sf(fill = "white") +
  coord_sf(crs = "+proj=laea +lat_1=-28 +lat_2=-36 +lat_0=-32 +lon_0=135 +x_0=1000000 +y_0=2000000 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs")
```

Or have a map that shows the world in a bit of an unusual perspective due to the Labert projection.


```{r geo13, message=FALSE, warning=FALSE}
# plot map
ggplot(data = world) +
  geom_sf(fill = "beige") +
  coord_sf(crs = "+proj=lcc +lat_1=-28 +lat_2=-36 +lat_0=-32 +lon_0=135 +x_0=1000000 +y_0=2000000 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs") +
  theme(panel.grid.major = element_line(color = "gray50", 
                                         size = 0.25),
        panel.background = element_rect(fill = "aliceblue"))
```





The nice thing about `ggplot2` is, of course, that it is very easy to add layers and create very pretty visualizations. 


```{r geo14, message=FALSE, warning=FALSE}
# plot map
ggplot(data = world) +
  geom_sf() + 
  theme_bw() +
  # adding axes title
  labs(x = "Longitude", y = "Latitude") +
  # adding title and subtitle
  ggtitle("A map of Australia") +
  # defining coordinates
  coord_sf(xlim = c(100.00, 160.00), 
           ylim = c(-45.00, -10.00), 
           expand = T) +
  # add distance measure
  annotation_scale(location = "bl", width_hint = 0.5) +
  # add compass 
  annotation_north_arrow(location = "br")
```


Again, we can customize the map according to what we want. In addition, we load a map with a higher resolution using the `ne_countries` function from the `rnaturalearth` package.

```{r map_gg3, message=FALSE, warning=FALSE}
# load data
world <- rnaturalearth::ne_countries(returnclass = "sf") 
# add to prevent errors
sf::sf_use_s2(FALSE)
# extract locations
world_points<- st_centroid(world)
# extract labels
world_points <- cbind(world, sf::st_coordinates(sf::st_centroid(world$geometry)))
# generate annotated world map
ggplot(data = world) +
  # land is gray
  geom_sf(fill= "gray90") +
  # axes labels
  labs(x = "Longitude", y = "Latitude") +
  # define zoom
  coord_sf(xlim = c(100.00, 180.00), 
           ylim = c(-45.00, -10.00), expand = T) +
  # add scale bar
  annotation_scale(location = "bl", width_hint = 0.5) +
  # add compass
  annotation_north_arrow(location = "br", which_north = "true", 
                         style = north_arrow_fancy_orienteering) +
  # define theme (add grid lines)
  theme(panel.grid.major = element_line(color = "gray60", 
                                         linetype = "dashed", 
                                         size = 0.25),
        # define background color
         panel.background = element_rect(fill = "aliceblue")) +
  # add text
  geom_text(data= world_points,aes(x=X, y=Y, label=name),
            color = "gray20", fontface = "italic", check_overlap = T, size = 3)
```

We can explore other designs and maps and show different regions of the world.

```{r geo16, fig.height=5, fig.width=6, message=FALSE, warning=FALSE}
# load data
europe <- ne_countries(scale = "medium", continent='europe', returnclass = "sf") 
# plot map
ggplot(data = europe) +
  # add map and define filling
  geom_sf(mapping = aes(fill = ifelse(name_long == "Germany", "0", "1"))) +
  # simply black and white background
  theme_bw() +
  # adding axes title
  labs(x = "Longitude", y = "Latitude") +
  # adding title and subtitle
  ggtitle("A map of central Europe") +
  # defining coordinates
  coord_sf(xlim = c(-10, 30), 
           ylim = c(40, 60)) +
  # add distance measure
  annotation_scale(location = "bl", width_hint = 0.5) +
  # add compass 
  annotation_north_arrow(location = "br",
                         # make compass fancy
                         style = north_arrow_fancy_orienteering) +
  theme(legend.position = "none",
        # add background color
        panel.background = element_rect(fill = "lightblue"),
        # remove grid lines
        panel.grid.major = element_blank(), 
        panel.grid.minor = element_blank()) +
  # define fill colors
  scale_fill_manual(name = "Country", values = c("darkgray", "beige")) +
  # add text
  geom_sf_text(aes(label = name_long), size=2.5, color = "gray20")
```

# Adding External Information {-}

While being very useful, displaying basic maps is usually less relevant because, typically, we want to add different layers to a map. In order to add layers to a map, we need to combine the existing data with some data that we would like to lay over the existing map. 

We will now lad external information which contains the locations of Australian airports.

```{r cus1, message=FALSE, warning=FALSE}
# load data
airports <- base::readRDS(url("https://slcladal.github.io/data/apd.rda", "rb")) %>%
  dplyr::mutate(ID = as.character(ID)) %>%
  dplyr::filter(Country == "Australia")
# inspect
head(airports)
```

Next, we load an additional data set about the route volume of Australian airports (how many routes go in and out of these airports).



```{r cus7, message=FALSE, warning=FALSE}
# read in routes data
routes <- base::readRDS(url("https://slcladal.github.io/data/ard.rda", "rb")) %>%
  dplyr::rename(ID = destinationAirportID) %>%
  dplyr::group_by(ID) %>%
  dplyr::summarise(flights = n())
# inspect
head(routes)
```


We can now merge the airports and the route volume data sets to combine the information about the location with the information about the number of routes that end at each airport.

```{r cus9, message=FALSE, warning=FALSE}
# combine tables
arrivals <- dplyr::left_join(airports, routes, by = "ID") %>%
  na.omit()
# inspect
head(arrivals)
```

Now that we have that data (which contains geolocations (longitudes and latitudes), we can visualize the location of the airports on a map and add information about the route volume of the airports in the form of, e.g., points that we plot over the airport - the bigger the point, the higher the route volume. In addition, we add the locations as texts and also make these labels correspond to the route volume.

```{r cus11, message=FALSE, warning=FALSE}
# create a layer of borders
ggplot(arrivals, aes(x=Longitude, y= Latitude)) +   
  borders("world", colour="gray20", fill="wheat1")  +
  geom_point(color="blue", alpha = .3, size = log(arrivals$flights)) +
  scale_x_continuous(name="Longitude", limits=c(110, 160)) +
  scale_y_continuous(name="Latitude", limits=c(-45, -10)) +
  theme(panel.background = element_rect(fill = "azure1", colour = "azure1")) +
  geom_text(aes(x=Longitude, y= Latitude, label=City),
            color = "gray20", check_overlap = T, size = log(arrivals$flights))
```





# Interactive Maps with leaflet and maptools {-}

The `leaflet` package offers very easy-to use options for generating interactive maps ([here](https://raw.githubusercontent.com/rstudio/cheatsheets/main/leaflet.pdf) is the link to the leaflet cheat sheet provided by RStudio). The interactivity is achieved by the `leaflet` function from the `leaflet` package which creates a leaflet-map with html-widgets which can be used, e.g., in html rendered R Notebooks or Shiny applications. The advantage of using this function lies in the fact that it offers very detailed maps which enable zooming in on  specific locations.

```{r map3, message=FALSE, warning=FALSE}
# generate basic leaflet map
m <- leaflet() %>% 
  leaflet::setView(lng = 153.05, lat = -27.45, zoom = 12)%>% 
  leaflet::addTiles()
# show map
m
```


Another option for interactive geospatial visualizations is provided by the `maptools` package which comes with a `SpatialPolygonsDataFrame` of the world and the population by country (in 2005). To make the visualization a bit more appealing, we will calculate the population density, add this variable to the data which underlies the visualization, and then display the information interactively. In this case, this means that you can use *mouse-over* or *hoover* effects so that you see the population density in each country if you put the cursor on that country (given the information is available for that country).

We start by loading the required package from the library, adding population density to the data, and removing data points without meaningful information (e.g. we set values like Inf to NA).

```{r int2, message=F, warning=F}
# load data
data(wrld_simpl)
# calculate population density and add it to the data 
wrld_simpl@data$PopulationDensity <- round(wrld_simpl@data$POP2005/wrld_simpl@data$AREA,2)
wrld_simpl@data$PopulationDensity <- ifelse(wrld_simpl@data$PopulationDensity == "Inf", NA, wrld_simpl@data$PopulationDensity)
wrld_simpl@data$PopulationDensity <- ifelse(wrld_simpl@data$PopulationDensity == "NaN", NA, wrld_simpl@data$PopulationDensity)
# inspect
head(wrld_simpl@data, 10)
```


We can now display the data and use color coding to indicate the different population densities.


```{r int3, message=F, warning=F}
# define colors
qpal <- colorQuantile(rev(viridis::viridis(10)),
                      wrld_simpl$PopulationDensity, n=10)
# generate visualization
l <- leaflet(wrld_simpl, options =
               leafletOptions(attributionControl = FALSE, minzoom=1.5)) %>%
  addPolygons(
    label=~stringr::str_c(
      NAME, ' ',
      formatC(PopulationDensity, big.mark = ',', format='d')),
    labelOptions= labelOptions(direction = 'auto'),
    weight=1, color='#333333', opacity=1,
    fillColor = ~qpal(PopulationDensity), fillOpacity = 1,
    highlightOptions = highlightOptions(
      color='#000000', weight = 2,
      bringToFront = TRUE, sendToBack = TRUE)
    ) %>%
  addLegend(
    "topright", pal = qpal, values = ~PopulationDensity,
    title = htmltools::HTML("Population density <br> (2005)"),
    opacity = 1 )
# display visualization
l
```







We will end this introduction here but if you want to want to learn more, check out the detailed resources for geospatial data visualization using R can be found [here](https://keen-swartz-3146c4.netlify.app/) or [here](https://geocompr.robinlovelace.net/index.html).

# Citation & Session Info {-}

Schweinberger, Martin. `r format(Sys.time(), '%Y')`. *Introduction to Geospatial Data Visualization with R*. Brisbane: The University of Queensland. url: https://slcladal.github.io/gviz.html  (Version 2023.02.09).

```
@manual{schweinberger2023gviz,
  author = {Schweinberger, Martin},
  title = {Introduction to Geospatial Data Visualization with R},
  note = {https://ladal.edu.au/gviz.html},
  year = {2023},
  organization = "The University of Queensland, Australia. School of Languages and Cultures},
  address = {Brisbane},
  edition = {2023.02.09}
}
```

```{r fin}
sessionInfo()
```

***

[Back to top](#introduction)

[Back to HOME](https://ladal.edu.au)

