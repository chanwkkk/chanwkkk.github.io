---
layout: post
title:      "Using Active Record with Sinatra"
date:       2019-07-03 03:30:40 -0400
permalink:  using_active_record_with_sinatra
---


In the previous blog, I talked about Active Record and how to use it to make a connection between database and models( or classes as we always call). After I learn Sinatra, I want to organize my understanding about combining AR and Sinatra here, so as to improve my master of both. 


***Set up the environment***


* set up the database

We have to tell the program which database to connect, so we can put the following statement in `environment.rb`: 

        
         configure :development do
         	set :database, 'sqlite3:db/database.db'
          end
     
	 
or this:

			 ActiveRecord::Base.establish_connection(

			 :adapter => “sqlite3”,

			 :database => “db/students.sqlite”

			)
	 
* create a class to inherit from `Sinatra:: Base`

After the database connection, we have to create under the root directory a file `app.rb`, in which we create a class, usually called `Application` and make it inherit from `Sinatra::Base`, so that all instances of `Application` can use all Sinatra features. 

* create a file `config.ru`

A `config.ru` file is basically like an entry point for the app. So we will have to put the following code in that file: 

```
require 'sinatra'
 
require_relative './app.rb'
 
run Application
```

Here is some explanation from Flatiron School tutorial explaining why we need `config.ru`: 

> The purpose of config.ru is to detail to Rack the environment requirements of the application and start the application....In the first line of config.ru we load the Sinatra library. The second line requires our application file, defined in app.rb. The last line of the file uses run to start the application represented by the ruby class Application, which is defined in app.rb.
> 

When `shotgun` or `rackup` command is run, this file is where it looks for as an entry point.

***Work flow***

So now we have get everything set up and let me explain the flow of the application first. You might need some basic knowledge for MVC though. MVC is short for `Models-Views-Controller`. That is like a useful pattern for us to structure our web application according to different purpose. Here is the explanation from Flatiron School tutorial: 

> Models: The 'logic' of a web application. This is where data is manipulated and/or saved.
> 
> Views: The 'front-end', user-facing part of a web application - this is the only part of the app that the user interacts with directly. Views generally consist of HTML, CSS, and Javascript.
> 
> Controllers: The go-between for models and views. The controller relays data from the browser to the application, and from the application to the browser.
> 

In my understanding, `Models` are the core part of the pattern(others might have different opinion). When we make a web application, we want to solve a problem right? And `Models` is the key to the problem — it makes an abstract concept become specific. For example, if we want to deal with a `Tic-Tac-Toe` problem, we will make a class of `Tictactoe` and then it might have attributes of `board` and `players`, and also some class methods or instance methods like `win?` or `validate_move?`. All of them are the logic of the problem, reflect what the problem is and how it works,  and thus help to solve a problem. 

`View` and `Controller`, however, I think are the auxiliary part in the pattern. `View` talks to the user by collecting data or displaying data to the user in a certain way. And the data, is  the result of operating on models' data.  `Controller`,  a bridge between `Views` and `Models`,  recognizes the request a user makes to models, finds the right route and then passes the information to the `Model` so it can work internally before responsing the user. 

Ok, now we already know generally what MVC is and we will understand easier about the work flow here: first, the user will make a request and then that request will be passed from `View` to `Controller`; secondly, `Controller` will distinguish different request and fire a specific block of code where it will use the logic of `Models`; after all these operations `Controller` will pass back the data to `Views`, where passed data is showed to the user.


***use ActiveRecord***

So why would I explain this much to you about the work flow of the app? Does that have anything to deal with using AR in Sinatra? Yes! The next problem will be, which part in MVC should we apply AR on? 

The answer for me is: `Controller`. Remember how we said `Controller` is a bridge? So `Controller` is responsible for passing the data `Models` need so `Models` can focused on its own business rather than doing the transmission from raw data to handy data. AR is a good help in this stage. It gives us a lot of handy methods. 

Let's illustrate it in detail with the example I used in last blog post: let's say I have a class called `Song`, firstly, I will create a class: 

```
class Song < ActiveRecord:: Base

end
```

Later I am going to create a corresponding table in a newly made file called:` 01_create_songs_table.rb`:

class CreateSongsTable < ActiveRecord::Migration[4.2]

```
def change

  create_table :songs do |t|

      t.string :name
  
      t.string :genre
  
      t.integer :year
  
   end
 end

end
```

Then we can run `rake db:migrate`  to get the migration done. As for more details about the above set up, please refer to my last blog post. 

But anyways, now we are able to use AR, and the following codes will become possible:

```
song1=song.new("song1", "pop",1992)
song1.save
Song.all
…
```

***Sinatra ActiveRecord CRUD ***

Now, let's see how we can use AR to CRUD(create, read, update, delete) in Sinatra. As we discussed before, AR statement should be written in `Controller`, which in this case, is the `app.rb`. 
In Sinatra, there are routes in `Controller`, basically comprised of  one HTTP method such as `get` , `post`,etc and a URL. When a user request comes in,  only the code block within the right route(the route with same method and URL, also called as controller action) will be executed. Since URL is basically what you want to name, so I will just specify the HTTP method here.  As for the `erb`, I don't want to explain that part too much here since it is not the point. But just remember that is a `View` where the generated data displays. 

* Create: 

The "create" feature corresponds to `post` method, and whatever URL you like to name. So to create a new instance of `Song`, we need to get some data from user first, and then use the hash-like `params` to `initialize` and `save` to the database. With the help of AR, we can use the handy  `create` method, a combination of both methods: 
```

post '/songs/new' do 

    @song=Song.create(params[:name], params[:genre], params[:year])
		
		erb:  index
		
	end
```

Now, when a user gives the app some data through route '/songs/post', a `Song` instance will be made and persist to the database. 

* Read: 

There are basically two kinds of "read" need: one is to 'read' all of the songs, another one is to "read" a certain song's info. We can retrieve both with the help of AR.  They should both respond to a `get` request because the user only wants to know infomation about `Song` class. 

In either case, we should set up an instance variable within the route, that is, the `Controller` part. 

For reading all songs from `Song` class, we will need to use the `all` method that AR has to get information from our database of `songs`. So the route will look like this: 

```
get “/songs“ do

	  @songs = Song.all
	
	  erb :index
end
```

For getting a certain song among all the songs, we will want to set up the route specially: the URL should be set up to something like `/songs/:id`. That means, whatever you input after `/songs/` in a URL will be captured as a `id` argument used to search database. The route will generally look like this: 

```
get "/songs/:id" do

   @song = Song.find(params[:id])
   
   erb :show
 
end
```

For example, if you type in a URL like `/songs/1`, the route will use the `find` method of AR to search in the database and return a `Song` object whose id is 1.  

* Update:

This is a bit complicated. First of all, we will need to add the following code in `config.ru`. 

```
use Rack::MethodOverride
run ApplicationController
```

The reason for this is explained by Flatiron school tutorial like this: 

> The MethodOverride middleware will intercept every request sent and received by our application. If it finds a request with name="_method", it will set the request type based on what is set in the value attribute.
> 


So now you might have guessed that we are going to make a request with name="_method". Exactly! Actually, we will need a form so that user can edit the information of any songs and submit it. When we set up the `input` tag, we can name it "_method", and then set up the `value`: 

```
<form action="songs/<%= song.id %>" method="post">
    <input id="hidden" type="hidden" name="_method" value="patch">
    <input type="text" ...>
</form>
```

So what happens here is: when the name file is detected as `_method`, the request type will be set to the value, which is `patch` request here. and the right `patch` route will be searched among all controller actions. And the following codes are what that corresponding controller action may look like:


```
patch '/songs/:id' do

	 @song = song.find(params[:id])
	
	 @song.update(params[:name], params[:genre], params[:year])
	
	 @song.save
	
	 redirect '/song/'+params[:id]
	
end 
```

A `Song` instance will be made,  be updated with the new information the user fills in the form and submits and `save`d to the database.  After all these, the user should be redirect to "read" the updated version of info of the song. 

* delete: 

To implement the `delete` function, we can introduce a form that might look like this: 

```
<form action="songs/<%= song.id %>" method="post">
    <input id="hidden" type="hidden" name="_method" value="DELETE">
    <input type="submit" value="delete">
</form>
```

The first line of this code block defines the route as "post '/songs/:id' ", the second line, however, change the request type to `DELETE`, so the route becomes "delete '/songs/:id' " now. The third line, is literally creatiing a button called that says "delete".  

Or, instead of complicatedly making the logic behind the delete button a form, we can simply make it a link like this: 

`<a href="/songs/<%= .id %>"  name="_method"  value="delete">Delete</a>`

It will work the same and bring you to the `delete` route. 

And the controller action for both in app.rb will use the `destroy` method in AR, and it should look like this: 

```
delete '/songs/:id' do

	@song = Song.find(params[:id])
	
	@song.destroy
	
	redirect '/songs'
	
end
```

A `Song` object is found and deleted, and a user should see the root page again or whatever page we want to be seen. 


Thanks for finishing the blog and again, I feel much more clearer about the concepts and hope that helps you too! if you find any inappropriation in this article, please contact me at chanwingkeihaha@gmail.com. I will be appreciated!






	

	
	

    

   







