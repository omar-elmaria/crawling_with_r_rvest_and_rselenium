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

The website is simple to crawl as it does not use Javascript to render its content and does not employ sophisticated anti-bot mechanisms. Therefore, The “rvest” package in R is sufficient to crawl the content of the website. In addition to the “rvest” package, we use the “dplyr” and “stringr” libraries to wrangle and clean the crawled data.

# 2. Questions?
If you have any questions or wish to build a scraper for a particular use case (e.g., Competitive Intelligence or price comparison), feel free to contact me on [LinkedIn](https://www.linkedin.com/in/omar-elmaria/)
