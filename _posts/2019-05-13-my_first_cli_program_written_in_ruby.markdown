---
layout: post
title:      "My first CLI program written in Ruby "
date:       2019-05-13 04:09:43 -0400
permalink:  my_first_cli_program_written_in_ruby
---


I have been working on my first CLI program these few days and to be honest, it is a big challenge to me. It is the first time for me to build something totally from scratch--it intimidated me when I saw the task but turned out fun in the end, luckily. And it occured to me that what I can do with programming is not only fancy grand things which is so far beyond my reach, but also some little and practical things like what I did this time. My CLI program is called `top 10 local day care centers`, what it tries to do is returning users with the top 10 local day care centers using the zip-code that user provides. Even to be a more delighted program, I believe it still needs some improvement like pretty interfaces but it is already helping me and I feel a sense of achievement from implementing what I want to do already. 

Here are the codes on github: https://github.com/chanwkkk/top10_local_day_care_centers.git 

***How do I start ?***

For all these people who are as scared as me in the beginning, I definitely recommend to finish the walkthrough videos to build CLI gems because those will give you  some kinds of clues. And I find myself benefit a lot from the outside-in mode that is presented in the video. That means, imagining a user sitting in front of the program, what kind of interface you want to present , like greeting the user first, and then what choices you will offer him and how the program is going to react, etc. So I made an outline of work flow: 


1. Welcome a user and explain what does this CLI do
2. Ask for a user’s input for zip codes, decided if that is a valid input
3. Search through yelp and get data
4. Store data with Center object(Using data to make Center objects)
5. Show the user a list of center names
6. Ask the user which one they want to know more
7. Quit or go back to step 2

You can always improve it later because you want to change the work flow sometimes so to add a feature to the program or you want to just add more details so the outline becomes a better guidance. I add another function to it later, to ask users if they want to know more infomation about other day care centers, if they say yes, then will bring them back to the list again instead of going back to the root menu and re-entering the zip code, saving one step therefore saving their time. That makes work much easier!

Even I had a good start, I was lost in the middle because I felt like step3 is the most important and I wanted to work on that part first. I tried to use yelp's API instead of plainly scraping data from it. However, I am totally new to API, so I spent tons of time doing research and asking professionals, which showed me a lot of ways to do it, yet I was not smart enough to understand it. But luckily I was smart enough to listen to my tutor Beth and turned to work on CLI first. That is really helpful because CLI.class is like a main bone to my program, once I get it done, all I need to do is just adding some flesh to it. Like when I reached the step 2, I was thinking if the user provided a good zip code, i would give it a list, so I put a method  called `#list_centers` there right away, as if I already had that method on hand. Then I made some notes there to remind me of implementing that method later before my program finds out the name means nothing and breaks. 

By doing that, soon I finished my `#call` method, which is a main method in my main bone and then I started to fill up the flesh in it, like filling up codes for `#list_centers` for example. And then, it will reveals to you that how you will need other classes like `Center.class` and `Scraper.class`  to implement what you want to achieve. 


***How do I perfect the project?***

**1. don't make too much assumptions**

As I mentioned in my last post, it is very important to not have too much assumption in the program because it will break your program easily. Let's say if in step2, if user gives an input of strings instead of numbers, or give out a random 4-digits or 3-digits number, it will break. Also, very critically, when I try to make an object, attributes returned from yelp API can be quite inconsistent: like some places might have attribute `address 1`, others not; some might have attribute `phone_number` others might not. Therefore, I made sure all attributes are existed, so the program will not break easily, hopefully. 

> def specific_center_page(index)
>    
>      if @center_array[index.to_i-1].class==Center
>     center=@center_array[index.to_i-1]
>     end
> 
>      puts "#{center.name}'s more infomation:"
>      puts "Rating: #{center.rating}"
> 
>       if center.phone_number
>       puts "Phone number: #{center.phone_number}"
>       end
> 
>       if center.url
>       puts "Yelp page: #{center.url}"
>       end
>       ending_specific_page
>  end
>  

**2. DRY (Don't repeat yourself) and encapsulation**

I tried really hard to avoid repeating myself and use encapsulation more. They are kind of the same thing--encapsulation helps with avoiding repetition. If you encapsulate something you want to do in one method, next time you use it, instead of copying a whole bunch of codes, you can just call it--that is easier and codes look cleaner, in my opinion. What's more, when you have to change code, you don't have to bother changing everywhere but only the orignial method, which will change wherever reference to this method too. 

Also, I love encapsulation when I need ending! Look at my codes and you will find two methods for ending here. Here is one example: 

> >
> >   def ending
> 
>     puts "Do you want to re-enter a zip code? [Y/N]"
>     input=gets.strip
>    
>     if input.upcase=="Y"
>          call
>       elsif input.upcase=="N"
>       else
>      puts "Wrong input!"
>      puts "Give me only 'Y' or 'N' please."
>      ending
>     end
>    end

You see, I need those because when I check the validation of user input, if users give me a bad input, I should be able to restart the whole process. How can I do that? I just call that method within that method! That is much simpler than `#while` or other roundabouts.  

**3. Saving  work by using instance variable**

The last but not least thing I learn from the project is to make a variable instance variable if you have to keep using it. I do that when I have to deal with the method `#list_centers`.  If you remember, that method is made for showing a list according to a zip code that user inputs within `#call`method. In this step, I can easily pass a zip code(the input) to `#list_centers` since it is in `#call` too. But I expect after users choose a center from the list and know more about it, they get another chance to see the list and pick another center from it. That means, I have to do the `list_centers` again. But how am I supposed to re-get a zip code as its argumet?Should I ask the user again? 

No. I don't have to. All I need is storing the input as an instance variable called `@zip`. Anytime I need it, I just use it, and don't have to worry about asking for zip codes again and again. Therefore, it works but an argument need not to be passed. 

**Extra notes**

Last but not least, I want to list some of my notes here. It might look very small, but that does cause my headache, and I hope that I won't make those mistakes again!
*    When using `require_relative`, you have to use a relative path instead of an absolute one. It is absolute to  to the current file you are writing codes on, not to which directory you are in right now. To require an document in another directory, if I want to going up one directory, I need to use `../`, for example, if I am writing a`bin/exec.rb` and require a `lib/hello.rb`, I should put it like this : `require_relative '../lib/`. But if I want to go one more up, I should write `../../documents` instead of `.../`. For requiring documents in the same directory, using `./`. 
*    When setting up the models that I need, I have two choices to define classes: 
>  
> class Top10LocalDayCareCenters::Cli
>
> ...
> 	end
	
Or

	
> 	  module Top10LocalDayCareCenters
> 	       class Cli
> 			      ...
> 			   end
> 	   end
  
Either one is fine, but I just found that if I choose the first one, when I refer to other classes in the same directory and under the same module, I need to do this: `Top10LocalDayCareCenters::Scraper`, while in the second one, I can call on class `Scraper` directly. 


***The end***

Ok, so here is all I want to talk about so far. I am still working on my project, so maybe later on when I learn more, I will post it here! Again, if you have any advice for me, please don't hesitate to email me:  chanwingkeihaha@gmail.com. Thanks for reading all of this! 




















