---
layout: post
title: Blog Post 3 - Web Scraping IMDB for Film Recommendation
---

This is a web scraping project for PIC16B at UCLA. In this project, I wrote a web scraper using Scrapy to collect a list of movies on IMDB to show that which movies share at least one actor with our favorite movie.

In this blog post, I will be using the following movie for our web scraper: "Shang-Chi and the Legend of the Ten Rings". Here's the link to the IMDB website: https://www.imdb.com/title/tt9376612/

In other words, our main goal in this web scraping is to answer the following question:

> 
>
> What movie or TV shows share actors with my favorite movie?
>
> 

# 1. Setup

## §1.1. Locate the Starting IMDB Page

In this blog post, I pick my one of my favorite movie Shang-Chi and the Legend of the Ten Rings. It's IMDB page is at:

```python
https://www.imdb.com/title/tt9376612/
```

we will use this URL later.

## §1.2. Dry-Run Navigation

If we click "All Cast & Crew" on the web page above, we will be navigate to the Cast & Crew page. The URL will be like this:

```python
<original_url>fullcredits/
```

More Specifically, the Case & Crew URL for my movie will be:

```python
https://www.imdb.com/title/tt9376612/fullcredits
```

Then, we scroll down until we see the Series Cast section. We then click on the portrait of one of the actors, and the browser will take us to a page with different-looking URL, for example:

```python
https://www.imdb.com/name/nm4855517/
```

Eventually, let's scroll down until the actor's Filmography section. This section shows some titles of few movies.

The goal of our scraper is to replicate the process that we've done above. It will help us to find movies or TV shows that share actors with our favorite movie.

## §1.3 Initialize Our Project

1. First, we create a new Github Repository to house our scraper.
2. Then, we open a terminal in the location of our scraper repository on the laptop. Type the following code:

```shell
conda activate PIC16B
scrapy startproject IMDB_scraper
cd IMDB_scraper
```

This above code will initialize our web scraper, and it will create a lot of files. Don't be panic about those files! We don't really touch most of these files!

## §1.4 Tweak Settings

Before we start writing our scraper, we add the following line to the file `settings.py`:

```python
CLOSESPIDER_PAGECOUNT = 20
```

This line of code will prevent our scraper from downloading numerous data when we are still testing our scraper. This could help us save our time.



# §2. Write Your Scraper

Now, we begin to write our scraper!

First, we create a file called `imdb_spider.py` inside the spiders directory. And we add the following code in this file:

```python
# to run 
# scrapy crawl imdb_spider -o movies.csv

import scrapy

class ImdbSpider(scrapy.Spider):
    name = 'imdb_spider'
    
    start_urls = ['https://www.imdb.com/title/tt9376612/']
```

The start_urls in the above function is the URL corresponding to our favorite movie or TV shows.



## 1. Parse

Then, we implement our first parsing methods for the `ImdbSpider`class. In this method, we assume that we are on our favorite movie page, and we navigate to the Cast & Crew page. Once we navigate to this page, the next method `parse_full_credits(self, response)` will be called automatically.

```python
def parse(self, response):
    '''
    This method tells the spider what to do 
    when we get to the website
    '''
    # response is how scrapy stores the website
    cast_crew_url = response.url + "fullcredits/"

    # navigating to the case and crew page
    # using scrapy.Request() method
    # then, navigate to the next link and parse_full_credits()
    # will be call
    yield scrapy.Request(cast_crew_url, callback=self.parse_full_credits)
```



## 2. Parse Full Credits

After we complete our first `Parse()` method, we will jump to the second method `parse_full_credits()`. This function assume that we are currently on the page of Cast&Crew. The goal of this function is to locate the list of each actor on the page, and it will then call the next method `parse_actor_page()`. 

```python
def parse_full_credits(self, response):
    '''
    This function parse the series cast page for the movie,
    and it will then navigate to each actor's page
    '''
    # collect a list of actors links
    castlink=[actors.attrib["href"] for actors in response.css("td.primary_photo a")]

    # iterate over each actor's link
    # then yield the request to the next function
    for link in castlink:
        
        url = "https://www.imdb.com" + link
        
        yield scrapy.Request(url, callback = self.parse_actor_page)
```



## 3. Parse Actor Page

After we complete the previous `parse_full_credits` method, the scraper will automatically call the `parse_actor_page()` method. Now, we are going to write this `parse_actor_page()` function. For this function, we suppose that we are currently on the page of an actor. We will use the `css` selectors in this function to collect the name of each actor and the name of each movie or TV shows. Eventually, we yield these actors' and movies' names as a dictionary with two key-value pairs. The form of the dictionary should be like this: `{"actor" : actor_name, "movie_or_TV_name" : movie_or_TV_name}`.

```python
def parse_actor_page(self, response):
    '''
    This function parse the actors page,
    and it yield these actors' and movies' names
    as a dictionary with two key-value pairs.
    '''
    # use css selector to locate the actors' names
    actor_name = response.css("span.itemprop::text").get()
    # use css selector to locate the movies' names
    movie_list = response.css("div.filmo-row b a::text").getall()
    # pairs the actors names and movies names
    # as a key-value dictionary
    for movie in movie_list:
        yield {
            "actor" : actor_name,
            "Movie_or_TV_name" : movie
```



# §3. Testing Scraper & Make Recommendations

Once we've completed the above process, our spider is ready for testing!

Now, we can comment out the line of code that we just wrote in the `settings.py`:

```python
# CLOSESPIDER_PAGECOUNT = 20
```



Then, we use the following command to run our spider:

```python
scrapy crawl imdb_spider -o results.csv
```

After we run the above code, our results will be saved as a CSV file call `results.csv`, with a column of actor names and another column of names of movies or TV shows. 



Next, we can read the `results.csv` file into our Jupyter Notebook, and then we use packages like `matplotlib` or `plotly` to visualize our result. We can also show a pandas dataframe.



```python
import pandas as pd
from matplotlib import pyplot as plt
import numpy as np
df = pd.read_csv("results.csv")
df['Movie_or_TV_name'].value_counts().reset_index()[0:11]
```



| Rank |                   Movie                   | Number of Shared Actors |
| :--: | :---------------------------------------: | :---------------------: |
|  0   | Shang-Chi and the Legend of the Ten Rings |           59            |
|  1   |           Entertainment Tonight           |           13            |
|  2   |           I Don't Have a Phone            |           11            |
|  3   |                The Oscars                 |           10            |
|  4   |         Marvel Studios: Assembled         |           10            |
|  5   |  The Tonight Show Starring Jimmy Fallon   |           10            |
|  6   |              Celebrity Page               |           10            |
|  7   |             Made in Hollywood             |            7            |
|  8   |   The Mummy: Tomb of the Dragon Emperor   |            7            |
|  9   |            Jimmy Kimmel Live!             |            6            |
|  10  |       Entertainment Tonight Canada        |            6            |



From the above ranking, we can see the top rank of the movie is the Shang-Chi and the Legend of the Ten Rings, which is our original favorite movie. We can start looking from the 2nd movie: Entertainment Tonight. This movie has 13 shared actors with our favorite movies, which is the most recommended movie for me. 

