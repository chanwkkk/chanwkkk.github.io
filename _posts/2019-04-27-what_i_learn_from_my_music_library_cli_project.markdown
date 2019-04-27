---
layout: post
title:      "What I learn from my Music Library CLI project"
date:       2019-04-27 03:18:58 -0400
permalink:  what_i_learn_from_my_music_library_cli_project
---


Finally, after almost five days, I finished the Music Library CLI project. I might  not  be smart, but that is not the main reason why I spend so much time on this. 

Music Library CLI project is a complicated lab to me, testing all I know about Ruby, and thus give me a chance to scrutinize what I have learned in these few days. Considering the process to finish the lab to be a review, I decide to go slowly. And I am happy to say that, yes, it helps me a lot, now I am more aware of *avoiding to take things for granted* and *abstract things away*.  I am going to write down what I have learned exactly  through examples, and hopefully consolidate my understanding on Ruby. 

**1. should I require relevant documents in the beginning of a .rb document?**

It depends. 

It confused me after I finish the required module `Findable`--what should I do to "tell" my other classes that "Hey, look for the module called `Findable`?" I went back to check the tutorial and two labs I completed before and finally figured out. 

In the lab that describes a "story" that a kid tries to learn dancing moves from a dancer, I encapsulated the dancing move into a module so that both the `Dancer` class and `Kid` class can use the dancing move there, avoding to copy and paste same code (for the same dacing movements). Besides to `extend`  or  `include` the module, the classes also need to  "find" the module. To do this, I put the following codes in the beginning of both class documents : 

> require_relative './class_methods_module.rb'
> 
> require_relative './dance_module.rb'
> 
> require_relative './fancy_dance.rb'
> 

While in another lab, where I dealt with the relationship between a `Song` class and an `Aritst` class and three other modules, I didn't  put any of `require_relative` statements but things still work. 

The secret lies on an document that sets up the whole environment. Actually in the second example, there is an `environment.rb` which requires all the documents already. The `spec_helper` `require_relative` the `environment.rbon.rb`. That means, once the code starts to run, `spec_helper` will load the `environment.rb` right away and therefore set the whole "environment" up for the rest of the code running. 

That is, in my understanding, actually another way to show an important property of coding : abstract.  Because instead of copying and pasting the same "require_relative" statements on every class document, we encapsulate that process in the `environment.rb` document and never have to worry about that no more. Let's say even I add another class without requiring the module, it will still run ok because the environment is set up from the beginning when `spec_helper` is loaded, having nothing to do with other documents. 

**2. Just do it? Wrong!**

"Just do it" maybe a good slogan for life or for sports, but from what I learned from this lab, it is probably wrong. When I came across a problem, I used to write codes down right away without thinking too much. But I got in trouble this time. 
After I finished the basic work of setting up `Song`,`Artist`,`Genre` classes, I was on my way to fill up more codes to make sure that upon initialization of a new song, Artist `#add_song` is invoked to add itself to the artist's collection of songs. Without thinking too much of it, I put the following codes: 

> def initialize(name, artist=nil,genre=nil)
> 
> @name=name
> 
> self.artist=(artist)
>  
> self.genre=(genre)
> 
> end

When I run the codes, however, it broke down! Here was the error message I got: 

> Failures:
> 
>   1) Song #initialize accepts a name for the new song
>   
>      Failure/Error: artist.add_song(self)
>      
>      NoMethodError:
>      
>        undefined method `add_song' for nil:NilClass
>        
>      ./lib/Song.rb:37:in `artist='
>      
>      ./lib/Song.rb:12:in `initialize'
>      
>       ./spec/001_song_basics_spec.rb:9:in `new'
>       
>      ./spec/001_song_basics_spec.rb:9:in `block (3 levels) in <top (required)>'

From the error message, it was obvious that the method broke down because artist=nil, and you cannot call that `add_song` on nil class. To prevent that, I should've make sure to only assign a value to `@artist` when the value is an instance of `Artist`.class. So I modified my codes:

> def initialize(name, artist=nil,genre=nil)
> 
>     @name=name
>     
>     self.artist=(artist) if artist.class==Artist
>     
>     self.genre=(genre) if genre.class==Genre
>     
>   end
> 

Now it works, finally. Because if the value we assigned is  an object from nil class, it will not even get to be assigned to `@artist`. Another example of my not thinking carefully before finalizing my code was about the `save`method. Basically, that is a method to adds the Song instance to the `@@all` class variable. It sounds very easy, so I wrote the following code within 30 seconds:

>   def save
> 	
>     self.class.all<<self 
> 		
>   end

It ran ok until it hit the  `#list_songs` method , which was designed to print all songs in the music library in a numbered list (alphabetized by song name). I thought I could get it done by `sort` ing the names of the songs in @@songs, the array that stores all the songs that have been made. But it fails. After I used "pry" to dig into it for quite a bit, I found that I have many identical song objects pushed in `@@songs`.  Since there are so many chain methods woven together that I cannot really find out which one contributes to which identical songs exactly, I decided to go back to `save` and wrote the following codes:

> def save
> 
>     self.class.all<<self unless self.class.all.include?(self)
> 		
>   end

So I can make sure a song will not be saved if there is already a copy of it in the @@songs array. 

After these two examples, I learn that I should broaden my mind more and also practice more so that I will have more experiences, and thus be able to think of more possibilities when I design my codes more comprehensively. 


Here are the links of the programs I just mentioned:
1. Kid and dancer: [https://github.com/chanwkkk/modules-reading-online-web-sp-000.git](http://)
2. Artist and song: [https://github.com/chanwkkk/artist-song-modules-online-web-sp-000.git](http://)
3. My music library cli program: [https://github.com/chanwkkk/ruby-music-library-cli-online-web-sp-000.git](http://)
