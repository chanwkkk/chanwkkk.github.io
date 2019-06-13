---
layout: post
title:      "How to use ActiveRecord to map classes to database tables"
date:       2019-06-13 00:13:49 +0000
permalink:  how_to_use_activerecord_to_map_classes_to_database_tables
---

Recently I have been learning `ActiveRecord` and to be honest, I can't help wowing because I really love it! It is a great tool when working with both database and models.  So I decided to organize what I have learn so far and write it down. I believe that will help solidfy my  knowledge about AR and hopefully it will help you too. 

In the beginning, I want to briefly talk about ORM(Object Relational Mapping ), which is  a way for us to *"to manage database data by "mapping" database tables to classes and instances of classes to rows in those tables"* (From Flatiron School tutorial). In my opinion, tables in database and classes in OO language are two parallel worlds where they have same data and structure when we set them up in the beginning. However, after establishment, data will change and ORM is an efficient way to connect them so they can reflect each other's change. When using ORM, we usually build methods in the following pattern.

Set up a database connection in the environment document:

> require 'sqlite3'
>  
> DB = {:conn => SQLite3::Database.new("db/music.db")}
> 

`DB` can be referred to the database all the time since then and to build the method: 
> 
> Class Song
> 
> def save
> 
>     sql = <<-SQL
> 		
>       INSERT INTO songs (name, album) 
> 			
>       VALUES (?, ?)
> 			
>     SQL
>  
>     DB[:conn].execute(sql, self.name, self.album)
>  
>   end
>  end 

As you can see, we write some raw SQL statements and then tell the connection to implement those statements with passed arguments. In that way, when a new song is created in the class, we can call `save`on it so it can be saved into the database right away. 

In the process of learning ORM, I wrote a lot of methods like this, what we can use to create, read, update and delete(CRUD) data in the database,like `update`,`find_by`, `create`... And active record is  basically a combination of all these methods! So instead of writing every single method by ourelves, we can use all of those methods as we like as long as we tell classes to inherit from `ActiveRecord::Base`, . And here is one more reason I love about AR, we don't have to worry about  what kind of grammar the SQL software has . AR  will do everything for you! 

Great, now let's see how can we use AR.

***Set up the connection***

Do you remember we set up a connection to database when we work on ORM? Same here when using `ActiveRecord`.

> ActiveRecord::Base.establish_connection(
> 
>   :adapter => "sqlite3",
>   
>   :database => "db/students.sqlite"
>   
> )

`adapter` tells `ActiveRecord` which sql software is used and then it will take care of the SQL language for us.  `database` tells `ActiveRecord` where the database is. 


***Set up tables ***

As we mention before,  what we are trying to do is mapping database tables to classes. So we will have to create table and assigning the specific class's attributes to that table as a column. Like if we have a `Cat` class with `name`,`breed`,`age` attributes then we will have a table called `cats` with `name`,`breed`,`age` method.  To create a table, we will need to use `migration`. 

Usually I will create a `db` directory on the top level, and then create a new directory called `migrate` under `db`, to store all the migrations we have.  To name a migration file, we name it by putting time (including seconds) or if our application is very small, simply putting numbers like `01` (to represent it is the first migration) before the name of your file. Because when it runs, AR will run the file in alpha-numerical order. Like `02xxxxx. rb` will be ran earlier than `03xxxxxx.rb`, `add_column.rb` will be ran faster than `delete_column`, so to control the order of files being ran , we better name file with a prefix number.  After that, we can start to create tables or change tables.  

Within the table we just create, define a class after our file name but without the number. For example, if our file is called `01_create_songs_table.rb`, then name our class as `CreateSongsTable`, and make that class inherit from `ActiveRecord::Migration[4.2]`. `[4.2]` is the version of `ActiveRecord::Migration`, we have to point out which version we use. 

Now, we can start to write what we want to do inside the class. Define a method called `change` first: 

> def change
> 
>   create_table :artists do |t|
>   
>   end
>   
> end

Pass a method called `create_table` in, put the name as symbol, and then define the column information like the type and name as following example: 

> class CreateSongsTable < ActiveRecord::Migration[4.2]
> 
>   def change
>   
>     create_table :songs do |t|
>     
>       t.string :name
>       
>       t.string :genre
>       
>       t.integer :year
>       
>     end
>     
>   end
>   
> end
> 

After all this, we can run `rake db:migrate` in the terminal to tell AR to implement this migration. Easy right?
Migration is basically for tables, so if we want to remove table, remove column, or add column, we can create new migration files and pass methods to `change` method. 

To know more, here is a very comprehensive reference:
[https://edgeguides.rubyonrails.org/active_record_migrations.html](http://)

Now we have a table that is called `songs`,  how do we relate that with the class `Song`? Simple, we only have to tell `Song` class to inherit from AR, like this:

> class Song < ActiveRecord: Base
> 
> end

Then AR will know to look for a table called `songs` and relate both of them. And that's why we name the table as `songs`--to lowercase and pluralize the class name so that AR can find it.   


***Simulate relations between classes ***

Now we successfully relate one table to one class. What if we have a few classes that are related to each other. Can we simulate relations between classes too? Yes!

First, make sure we make every class inherit from `ActiveRecord: Base`, and then, we have to decide which of the following association we should use to describe the relationship between classes.  There are four associations in AR: `belongs_to` and `has_one` to describe one-to-one relationship, `has_many` to describe one-to-many relatioship and `has_and_belongs_to_many` to describe many-to-many relationship. 

To use them, you can write statement right after the class definition, for example:

> class Song < ActiveRecord: Base
> 
>   belongs_to :artist
> 	
> end

On the contrary, 

> class Artist < ActiveRecord: Base
> 
>   has_many :songs
> 	
> end

And then, AR will do the rest and  help we connect two classes. AR will automatically add new methods to class once the connection is done. Like if we make a new song by :`s=Song.new` and make a new artist by `a=Artist.new`, it is ok for us to do this: `s.artist=a`. When we ask s : who is your artist by putting: `s.artist`, it will tell us it is the `Artist` object a.  And we can also do like `s.build_artist` to build an artist for s. To know more handy methods you can use, you can go to this following link: [ https://guides.rubyonrails.org/association_basics.html#belongs-to-association-reference
](http://). It is really worth to have a look, they are easy to use and save us work. 

However, just one thing to notice, we have to put the singular form of the class name after `belongs_to`, while the pluralized form after `has_many`, that is a must  because that is how AR accordingly find tables, and set up foreign ids and so on.  

***Perform CRUD actions ***

After we set up all the relationships among tables and classes, we should be able to use it. I am not going to cover all of that here, but you can go to this link to find out how to make queries: [https://guides.rubyonrails.org/active_record_querying.html](http://). Instead of writing a long query statement, AR offers a lot of methods for us to manage data in database through models, which makes queries neater and easier!

Thanks for finishing the blog and again, if you find any inappropriation in this article, please contact me at chanwingkeihaha@gmail.com. I will be appreciated!

