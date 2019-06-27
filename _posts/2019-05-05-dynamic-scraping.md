---
title: "Dynamic Web Scraping"
date: 2019-05-05
tags: [web-scraping, maps]
excerpt: "Web Scraping an interactive map."
toc_label: "Interactive Map Scraping"
toc_icon: "map-marked"  #  Font Awesome icon name (without fa prefix)
header:
  overlay_image:"../assets/images/halong_bay.jpg"
---

# Interactive Map Scraping 

Web-scraping can throw up lot's of different data sets, but sometimes it is not as simple as selecting a div tag. This post aims to provide an example workflow of a simple scraping project and how to create neat, alternative datasets from the web. In this example we are able to access the data held in an interactive map with a little bit of searching round the HTML script. 

The [map used](https://www.bpf.org.uk/what-we-do/bpf-build-rent-map-uk)  is created by the British Property Federation and shows build to rent statistics in the UK. Initial thoughts are to scrape the full HTML and then parse through the relevant data to find the relevant information. The preferred tool of choice is [selenium using a chrome diver](http://chromedriver.chromium.org/getting-started).

## First look
![image-20190129213726123](../assets/images/map_scrape/image-20190129213726123.png)

As the screenshot shows, the web client will show more information about a build to rent property when clicked on. This is a great visual tool showing the geographic distribution of these properties, but hard to do any further analysis. First select a property then find the name of the property development in the HTML, fro example:  *property_1*. Then select another property marker and search for *property_1*, this is to try and get a better sense of the structure of the HTML. No luck, it seems that the information is loaded on request.

## Second look

Using [this](https://onlinejournalismblog.com/2017/05/10/how-to-find-data-behind-chart-map-using-inspector/) very helpful website to utilise the network tab in the inspect module as shown below. ![image-20190129214436938](../assets/images/map_scrape/image-20190129214436938.png)

As the top right of the inspect pane shows, the page loads a file when the marker is clicked. This element contains a  JSON-type file, opening the link directly confirms this is intact a JSON element. 

```JSON
/**/ typeof _cdbi_layer_attributes_0_4 === 'function' && _cdbi_layer_attributes_0_4({"title":"Surrey House","prs_units":322,"deliverer_contact":"Salmon Harvester Properties","buyer_funder_contact":"Salmon Harvester Properties","manager":"-","planning_status":"Detailed Application","prs_type":"Build to rent","owner":"Salmon Harvester Properties"});
```

Repeat this a couple of times and find consistent results and take note of the addresses:

```
https://cartocdn-gusc-b.global.ssl.fastly.net/savills/api/v1/map/savills@465f18a4@abe58d5bc799578ceeba1b9ab6e7945f:1539185524180/1/attributes/265?callback=_cdbi_layer_attributes_0_20
https://cartocdn-gusc-b.global.ssl.fastly.net/savills/api/v1/map/savills@465f18a4@abe58d5bc799578ceeba1b9ab6e7945f:1539185524180/1/attributes/48?callback=_cdbi_layer_attributes_0_16
https://cartocdn-gusc-b.global.ssl.fastly.net/savills/api/v1/map/savills@465f18a4@abe58d5bc799578ceeba1b9ab6e7945f:1539185524180/1/attributes/285?callback=_cdbi_layer_attributes_0_22
https://cartocdn-gusc-b.global.ssl.fastly.net/savills/api/v1/map/savills@465f18a4@abe58d5bc799578ceeba1b9ab6e7945f:1539185524180/1/attributes/528?callback=_cdbi_layer_attributes_0_21
https://cartocdn-gusc-b.global.ssl.fastly.net/savills/api/v1/map/savills@465f18a4@abe58d5bc799578ceeba1b9ab6e7945f:1539185524180/1/attributes/506?callback=_cdbi_layer_attributes_0_4
```

It may be a bit difficult to see in this view, but the web addresses are identical bar a number in-between the layer and call back attribute. Test out this hypothesis by only changing this number, there are no changes to the web page. This makes life a little easier as we only need to change one element of the web address. 

# Selenium to scrape the data

Using python we can set up an empty file to hold the scraped elements and define our browser element. 

```python
from selenium import webdriver
import webbrowser
import string
from io import open

text_file = open("scraped_data.txt", "w")

'''Scrapes data from interactive map.'''
#Open the website using Chrome
chromedriver = "pythoncode/chromedriver"
browser =  webdriver.Chrome(chromedriver)

```

Loop through webpages using formatted string to alter the web address and store output in a text file on a new line. At this moment we are unsure how many properties are not his map, a speculative loop from 0 to 1000 with a try/except block should find all properties. 

```python
#Itterate through the webpages
for property_number in range(0,1000):
    prop_no = property_number
    #Store results in a text file
    with open("scraped_data.txt", "a") as f:
        url =f"https://cartocdn-gusc-b.global.ssl.fastly.net/savills/api/v1/map/savills@465f18a4@abe58d5bc799578ceeba1b9ab6e7945f:1539185524180/1/attributes/{prop_no}?callback=_cdbi_layer_attributes_0_22"
    try:
        browser.get(url)
        results = browser.page_source
        #Only want relevant JSON, strip away the HTML
        results = results[215:-22].encode('utf-8')
        f.write(f"{results}\n")
    except:
        print(prop_no)
        break
         
```



**Result**: The script maxed out somewhere around 568, we now we have  a text file with all the data.

# Cleaning the JSON data

Maybe a parser whilst the data is being scraped would have provided a more robust solution, however for now the excel split tool will suffice. Less than a minute using the import external data tool and we have a clean looking dataset. We also have more features then shown on the website so that is nice.![image-20190129233250146](../assets/images/map_scrape/image-20190129233250146.png)



# Adding Longitude and Latitude

In this proves we have lost the longitude and latitude element of the data. An extremely useful tool provided by google is the google maps api. The set up can be a bit fiddly but the documentation is really clear if you have never used it before. The [geocode google api](https://developers.google.com/maps/documentation/geocoding/start) is exactly what we need. Create a function to take the addresses from the csv file as input, pass that as an argument to the google api and return the longitude and latitude. 

```python
def get_long_lat(address):
    try:
        # Geocoding an address
        geocode_result = gmaps.geocode(address)
        #returns a JSON type value
        result = (geocode_result[0])
        location_dict = result.get('geometry')      #get part of the return object
        location = location_dict.get('location')
        return(location)
    except:
        return("Error")
        print(f"{address} not found")
```

Create a function to iterate through the original csv file and append the relevant lonagiufde and latitude values. We now have a clean data set that can be used with other location specific variables or distance related calculations. 

I hope this has been a useful run through of an example web-scraping project. Any thoughts or comments please let me know!


