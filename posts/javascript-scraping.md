
---
title: Web scraping with Node and Cheerio
tags: Javascript, Node, Scraping, Cheerio
---

In this blog post I will provide an overview of how to scrape websites in a node environment.  First, you must require ‘request’ and ‘cheerio’ in your file.  If you haven’t already, install these files by typing the following command into your terminal: npm install request cheerio –save.  We will use node’s request module to make HTTP requests to other servers (websites).  Cheerio allows us to select DOM elements receiving in our HTTP responses using jQuery-like syntax.

For the sake of this example, I will scrape basic information from BusinessInsider.  Request takes a url and then a callback function.  Conditional upon there not being an error, we set $ equal to the html from the response loaded into .  Once this variable has been instantiated, we can access it in the same way we would typically traverse DOM elements, using jQuery.


```
var request = require('request');
var cheerio = require('cheerio');

url = 'http://www.businessinsider.com';

request(url, function(error, response, html) {
  if (!error) {
    var $ = cheerio.load(html);
    $('.title').each(function(link) {
      console.log($(this).text());
    });
  }
});
```