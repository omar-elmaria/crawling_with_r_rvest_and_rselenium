# crawling_with_r_rvest_and_rselenium
This repo contains two Rmd files. The first file scrapes wine listings under the brand name "mövenpick" using the rvest package. The second scrapes Javascript-rendered apartment listings on the Swiss real estate website (homegate.ch) using RSelenium

# 1. Objective of the First Part of the Project (Crawling Mövenpick Wine)
The goal of the first part of the project is to crawl this [website](https://www.moevenpick-wein.com/de/rotweine?p=1) and extract the data points shown below
- product_title
- product_name
- product_url
- rating_score (out of 100 or 20)
- reviewer (could be a person, a magazine, or simple a displayed "Score")
- country
- city
- price (in CHF)
- image_url

![image](https://user-images.githubusercontent.com/98691360/202925388-aac7ba63-2bca-44fd-9d47-00add9d2aefd.png)

## 1.2. Analysis of the Website
The website is simple to crawl as it does **not** use Javascript to render its content and does not employ sophisticated anti-bot mechanisms. Therefore, The “rvest” package in R is sufficient to crawl the content of the website. In addition to the ```rvest``` package, we use the ```dplyr``` and ```stringr``` libraries to **wrangle** and **clean** the crawled data.

The URL ```https://www.moevenpick-wein.com/de/rotweine``` returns **2177** results. There are **24 results** on each page, which means there are ~ **91 pages** to scrape. The website is **paginated** https://www.moevenpick-wein.com/de/rotweine**?p=1**, meaning we can create a **for** loop to scrape the results from every page by changing the numeric parameter at the end of the URL.

## 1.3. Crawling Process
We start by defining the **CSS/XPath** selectors of **each data point** we want to crawl. The website is well-structured, so every item is stored in a "list" with a class "item" as shown below.

![image](https://user-images.githubusercontent.com/98691360/202925458-e9d5dc30-0e42-4738-a042-08119412f16e.png)

The CSS/XPath selectors of the nine data points are as follows:
- **product_title**
  - `span.product-name-1` --> `span` with a class `product-name-1`
- **product_name**
  - The product name is composed of two parts, so we extract them separately and combine them into one variable afterward
  ![image](https://user-images.githubusercontent.com/98691360/202925544-4a7bcc9f-c2ff-47d6-bd82-190f6acffdbf.png)
  - `p.product-name > span.product-name-part:first-child` --> The CSS selector of the first part of the product_name. `p` with a class `product-name` AND a first child of `span` with class `product-name`
  - `p.product-name > span.product-name-part:nth-child(2)` --> The CSS selector of the second part of the product_name. `p` with a class `product-name` AND a second child of `span` with class `product-name`
  - After extracting both parts separately, we combine them into one variable using the `paste0` function --> `paste0(product_name_p1, " ", product_name_p2)`
- **product_url**
  - `h2.product-name > a %>% html_attr("href")` --> `h2` with a class `product-name` AND a child `a`. Since this is a URL, we extract the `href` attribute using the `html_attr` method in R
- **rating_score and reviewer**
  - The rating score is a composite string that consists of **two parts**
  - **Part 1** is the person/magazine that reviewed the wine bottle
  - **Part 2** is the score out of 100 or 20
  - In the example below, the person who reviewed the wine is "Tim Atkin". He gave the bottle a score of 98/100
  - Since the score can be out of 100 or 20, we crawl the raw score in **string format** and also the individual components to calculate a percentage (e.g., 98/100 = 0.98 or 15/20 = 0.75)
  
  ![image](https://user-images.githubusercontent.com/98691360/202926510-db9154a0-1f40-4d71-b3a8-47bcee6f003f.png)
  - The CSS selector of the **composite review (reviewer + score)** is `p.rating-score` --> `p` with class `rating-score`
  - **Part one** can be extracted using this regex --> `[a-zA-Z]+`. This regex extracts any characters from A-Z in the string (lowercase or uppercase)
  - **Part two** as a whole can be extracted using this regex --> `\\d+\\/\\d+`. This regex extracts any part of the string that matches this pattern **number/number**
  - The **left part of the raw score** (before the division sign) can be extracted using this regex --> `"\\d+(?=\\/\\d+)`. This regex extracts any digit **before** a division sign that is followed by numbers
  - The **right part of the raw score** (after the division sign) can be extracted using this regex --> `(?<=\\/)\\d+`. This regex extracts any digit **after** the division sign
  - To check how these regular expressions work, one can use this [website](https://regex101.com/). Please note that in R, a double backslash is required. On this website, only one backslash is used
- **country & city**
  - The country and city are displayed together and separated by pipe “|”
  - `p.cellar-name` is the CSS selector of this composite string --> `p` of class `cellar-name`
  - To extract the **country**, one can use this regex --> `\\w+(?=\\s\\|)`. It extracts any word characters **before** " |"
  - To extract the **city**, one can use this regex --> `(?<=\\|\\s)\\w+`. It extracts any word characters **after** "| "
- **price**
  - The XPATH selector of price is `//span[@data-price-type = 'finalPrice']/span` --> `span` with attribute `data-price-type = 'finalPrice'` AND a `span` child
price is displayed as a composite string with the currency symbol (e.g., CHF 950.00)
  - In addition, wines that have 4-digit prices are displayed with an **apostrophe** (e.g., CHF 1,150.00)
  - To handle the **first case**, we can use this regex --> `(?<=CHF\\s).*`. It extracts any alphanumeric character **after** (CHF )
  - To handle the **second case**, we can use the `str_replace` method to replace the apostrophe with a blank character
- **image_url**
  - The CSS selector of image_url is `img.product-image-photo` --> `img` with class `product-image-photo`
  - The URL of the image is stored under an attribute called `src`. We can extract it with the `html_attr` method

After extracting these data points, we can place them in a data frame using the `data.frame` method in R. Since we loop over each page, we can define an **empty data frame** before the "for" loop. This data frame will grow over time because we will append the results of each crawling iteration to it. We can use a **temporary data frame within the “for” loop** to store the results of iteration "i’s", and then bind the results to the data frame we created before the “for” loop. In code form, this looks something like this...

![image](https://user-images.githubusercontent.com/98691360/202926210-ac8de620-e70d-4100-b293-74abcfec09dc.png)

## 1.4. Throttling Our GET Requests
To **stay polite** to the server, we need to throttle our GET requests to prevent our scraper from getting blocked. We can create a function to produce **random time values** and use the `Sys.sleep()` function to **slow down the scraper** in a random fashion. In code form, this looks something like this…

![image](https://user-images.githubusercontent.com/98691360/202926270-3193fe62-7328-4c2b-8674-3543caaf2dbf.png)

## 1.5. Further Automating the Scraper
To fully automate the scraper, we can extract the **last page number** from the first page and set it as the end of the "for" loop’s range. The first page does not display the last page number, but rather the **total number of products**, as shown below.

![image](https://user-images.githubusercontent.com/98691360/202926338-e2a583a1-a1dd-4937-843e-ac4c9054fb19.png)

Since there are 24 results on each page, we can calculate the total number of pages using this formula --> `ceiling(last_page “2177” / 24) = 91`

The CSS selector of this number is `div.filter-results-count > strong` --> `div` with class `filter-results-count` AND child `strong`

## 1.6. Bonus Part
The dataset contains the **"price"** and **"country"** fields, we can use these two fields to construct a price histogram per country, as shown below. The code is included in the last section of the .Rmd file

![image](https://user-images.githubusercontent.com/98691360/202926405-cd3603c3-42e7-464a-baa5-bf78be9c90d1.png)

# 2. Questions?
If you have any questions or wish to build a scraper for a particular use case (e.g., Competitive Intelligence or price comparison), feel free to contact me on [LinkedIn](https://www.linkedin.com/in/omar-elmaria/)
