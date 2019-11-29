---
title:  "XPATH Addresses for Website Elements Made Easy"
date: 2019-11-20
tags: [Webscraping, Data Science]
excerpt:  #"XPATH Addresses for Website Elements Made Easy"
---

Webscrape tutorials and most subject matter articles will tell you
that scraping is easy.  They will introduce you HTML tags and show you
that simply by identifying tags you can readily snatch the element you want to
scrape.  Easy as that?  Not exactly.

The reality is that webscraping is fraught with complexities.  Two that
readily come to mind are javascript links and dynamic CSS elements.
This article will deal with the later.

## Dynamic CSS elements

Dynamic CSS elements are webpage elements that change with each record.  So,
if you are scraping a set of 5 pages representing 5 different records and
you use a dynamic element in your HTML tag identification or XPATH address, you're
going to get undesired results.  Let's have a look.

### Dynamic Element example

I recently had a need to scrape company information from a website.  The site had
one page per company and I needed to scrape the name and company location.  The
challenge I found was that the name and location values were associated with
website elements that shared identical except for a single attribute.  The differing
attribute was dynamic and changed with for each company.  
