---
layout: post
title:      "How to use yelp api for my ruby program"
date:       2019-05-31 00:52:42 -0400
permalink:  how_to_use_yelp_api_for_my_ruby_program
---



I never heard of API until recently when I work on my CLI program. Actually, what I am trying to do through the program is to get information to help myself and my other parent friends to find the best day care center in the neighborhood. Without too many thinking, I decide to use Yelp to look for information since it has a lot of users around America and even the whole world and therefore, should provide more opinions with great variety.

At first, I got information from Yelp by scraping from the website. “Why don’t you use API?” However, after I talked to my tutor Beth, she asked me that and I decided to challenge myself and use API even I never heard of it before she metioned that. While to a lot of great developers out there, API may be very easy things, but to me, it took me a long time to finally get to understand a bit of its concept and usage. So here, I am not trying to make a tutorial talk about API, but actually just trying to write down what I know or understand about API, like making notes for the future me. The rest of the article talks about just my basic understanding of API after I have done some research. Therefore, they might not be super accurate. Please e-mail me when you find something wrong, thanks!

***1.  My understanding: what is API?***

Literally, API means Application Programming Interface. But I don’t really know what that means exactly. After I have been doing some research, I got this impression that, API is a way for a software to communicate with another, or to many other softwares. Usually when we, as human being users, surf the Internet using web browser, we send a request and get some vivid images or attractive layout more than just plain text information we want. In the API case, since it is software to software, compelling presentation is not necessary, so it will return with only the “meat”, the information we want. The format it has is usually very simple and easy to utilize.

What's more, compared to scraping, which requires you to go through all the HTML document and find the nodes and tags and everything, API is easier as you can usually get the result back by using the link it gives you and passing some specific parameters in query strings. Also I feel like compared to scraping, a website or a software should be happier to offer API instead of letting  people secretly doing things on data and maybe cause unknown problems or dangers to the program itself. By the specific `API keys` or `Client ID`s, a program that gives out those is more possible to track data like who is using this API and what are clients using those for..stuff like that. I believe those are of great business values. As for what are `API keys` or `Client ID`s, I am going to illustrate now. 

***2.   My understanding: some concepts you need to know before using API***
* Authentication

As I know, some APIs you can use without any kind of authentication, but most of them do require one because as I mention  above, API providers probably want to track a lot of information from whoever is using their API, therefore, a pair of key and value are required to identify clients and thus track them. There are `client_id` and `API_keys` pair or `client_id` and `client_secret` pair. Make sure you read the document about how to use the key and value pair you have. 
 
* Tools to send request and get response

There are a lot when I look it up online: httparty, faraday, http, etc, tools which  you will pass parameters and the link that you get from the base software to and get response from. Then you might get some JSONs, which you will have to parse before finding and using datas. 

***3. An example: How to use Yelp API for my ruby program?***

Ok, now I will explain how I use Yelp API to get information I want. 

* ***Authentication***

Go to [https://www.yelp.com/developers/documentation/v3/authentication](http://) and then follow the instruction to get  an API key first. 

* ***Endpoints documents***

Go to [https://www.yelp.com/developers/documentation/v3/get_started](http://) and read documents in detail. Under the "Endpoints" colum are all features that Yelp API can implement. The table underneath explains very well. You will find "Name","Path" and "Description" columns in the table. "Name" and "Description" of the endpoints are easy to understand, basically just illustrating what it does. As for the "Path", you will need to combine that with the base url that is provided right under "Endpoints " column--"https://api.yelp.com/v3". For example, if you want to do some business search, the url you will have to pass to the http, or faraday, or httparty, or any other tools you will use  is : "https://api.yelp.com/v3/businesses/search". 

According to the description of different endpoints, you can find the one you think match your app the most and then click into the path to see details like the query strings. You can pass the parameters in query string like this: add "?" after the url, and then add parameters like `paras=value` and connect different parameters queries with `&`. Let's say, if you want to query day care centers in a place where zip code is 80120, you want to check out the table on this page first: [https://www.yelp.com/developers/documentation/v3/business_search ](http://). Then you will know the keyword query string for "day care centers" should be "term", and for the zip code "80120" is " location", so the whole url will become: [https://api.yelp.com/v3/businesses/search?location=80120&term=day care center](http://)

* ***Install and require `http` and `JSON`***

Add dependencies in the `gemspec` file first like this : 

> spec.add_development_dependency "http"
> 
> spec.add_development_dependency "json"

Let's say, I use `http` in my `Scraper.class` , so I will have to put the following in the beginning of my `Scraper.rb` or whichever document you use as environment document. 

> require "json"
> 
> require "http"
> 


* ***Hide your API key***

As I mention above, API key is used to identify your app so you don't want to give out your API key to anybody. Here is a way to hide your key . It is from my best tutor I have: beth and here is the link where she demonstrate how to hide it in a finding- london-best-spas app: [https://github.com/Gingertonic/london-spas-cli/blob/master/lib/london_spas/api.rb ](http://). Later on if you want to access it, just put `ENV[API_KEY]`.



* ***Use `HTTP` class method in my `Scraper.class`***


> def self.search(term, location)
> 
>     url = "#{API_HOST}#{SEARCH_PATH}"
>     
>     params = {
>     
>     term: term,
>     
>     location: location,
>     
>     limit: SEARCH_LIMIT,
>     
>     sort_by: "rating"
> 
>     }
> 
>     response = HTTP.auth("Bearer #{ENV['API_KEY']}").get(url, params: params)# where you can JSON back
>     
>     
>     response.parse# where you parse JSON to hash in ruby
>     
>   end
>   

Now you should be able to use that API through `Scraper.search` method ! API is a very powerful tool and I know that I am going to learn more in the recent future. I hope this helps you too! Attached is my CLI program link just in case you want to take that as an example and see how I use it: [https://github.com/chanwkkk/top10_local_day_care_centers.git
](http://). Thanks for reading this!













