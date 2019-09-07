---
layout: post
title:      "Validation, error message and helper method"
date:       2019-09-07 16:23:50 -0400
permalink:  validation_error_message_and_helper_method
---


Sinatra project is my second portfolio project and it not only helps me clarify some concepts about sinatra but also motivates me to learn new stuffs when I need to implement  a new feature in my app.  I appreciate such a chance to organize and review my knowledge and I would love to share what I learn from my project to you. 

## How do I use validation in my app?

Validation is very useful when we persist data to database. For example, in my `User.rb`, where I define class `User`, I have codes like this:

```
class User < ActiveRecord::Base
  has_secure_password
  has_many :reviews
  has_many :centers, through: :reviews

  validates :username, presence: true
  validates :password_digest, :email, presence: true
  validates :email, uniqueness: true
end
```
Those sentences starting with the word `validates` are the tradition you use validation. Following `validates`, you put the attributes name as a target, and then after, you use a validation helpers. In another word, in the first part of the validation, it tells the model which attribute needs to be validated, and the second part tells the model how will that attribute get validated. 

For instance, look at `validates :email, uniqueness: true`. This statement tells the class `User`: "Hey, I want the email address to be unique on creation of every `User` instance." In this way, when you use a same email address to create a new user (or, in this case, sign up), the user instance is invalid and thus cannot be saved into the database, which guarantees the `uniqueness` of the email address in the database. 

`Uniqueness` is not the only key word I have used in this app. I used `presence`, `validates_associated` too. `Presence` is very common. When I say `  validates :username, presence: true`, that means I require that `username` attribute is provided when I create an instance from that model. In another word, if that attribute is missing, I won't be able to create an object sucessfully. So if I do this statement: 

```
User.create(:email=>"hi@hi.com").valid?
```

I get  `false` back, which means it is not valid and will not be saved into database. 

 `Validates_associated` helper is very helpful too. I use it to validate the user instance associated with the review when creating the review. It makes sure that when the review is created, it has a valid user, or itself won't be considered as valid either and thus won't be saved. That makes sense right? If the producer (the user, in this case) does not existed in the database, the production (the review, in this case ) won't either.
 
 
To implement these function, since validation is  Active record's feature. the model that is using that has to inherit from `ActiveRecord::Base`.Also, one thing I want to mention specially here is, be careful about the `User` class, or any other class that will have `password` attribute and use `has_secure_password` method. Here is my story:  at first, I want to make sure that a `password` is presented when people sign up. So I put this statement under `User` class: 
```
validates :password, presence: true
```

And then, I tried to use this following statement to create a user instance: 

```
User.create(:username=>"John", :password=>"12345", :email=>"hello@hi.com")
```

However, out of my expectation, I fail. I keep trying and trying but still fail even a username, a password and an email are all provided as they are needed. Why is that? 

Finally, I figure out the reason. As I explain before, the validation will look for `password` attribute everytime a user instance is instantiated. If the attribute can't be found, the instance is not valid. 

"But you did provide the `password`!" You might be thinking. Yes, I did. However, because I use gem `bcrypt` and `has_secure_password` method in Active Record, in the user table, I have to set the column name, which is supposed to be `password`, to `password_digest`.  Because `bcrypt` will access the attribute of `password` and store a salted, hashed version of password into a column called `password_digest` to better protect users' password information. Therefore, a `password` is required but not found because only `password_digest` is there. 

After I change the validation to: 
```
 validates :password_digest, presence: true
 ```
It works!
So do be careful when you use gem `bcrypt` and `password_digest` together with validation in Active Record. Plus, if you want to know more about validations and helpers, check this out: [https://guides.rubyonrails.org/active_record_validations.html#validation-helpers](http://)


## Error message

Sometimes the app might break due to users'  inappropriate operations. Of course, there are ways you can stop them. But, once you stop them, how do you tell them "Hey, what you just did is wrong. Now, you should do this or that "? That is where error messages become useful. For my app, I use the gem `sinatra-flash`. Basically, after setting up, you only have to put the message in the controller file and then leave a statement in the view file to make sure the message will show up if there exists any. The set up and how to use this are on this website: [https://github.com/SFEley/sinatra-flash](http://). 

It is not difficult. However, I just want to share one thing with you when using this gem: The flash message only works on a next request. What does that mean? I will show you in my story.
At first, to display the error, I write this following codes: 
```
...#users_controller.rb
flash[:error]="You have no right to check this user's file."
erb :"users/error"
...
```
It is an error message I left in my `/users/:id` route, to make sure that a user cannot check on other users' profiles. And if it does, it will load an error page and display the error message. It does navigate me to the `error.erb`, but it doesn't show the error message I expect. Instead, I see totally blank message. I don't know why, so I test another error message to see if there are more clues: 
```
flash[:error]="The user doesn't exist."
erb :"users/error"
```
This message is supposed to show up when I try to view a user's file when that user doesn't even exist. Guess what? I am navigated to the error page and get ` "You have no right to check this user's file."`, I get the error message from last request! I try many times and it still does the same thing--giving me last request. Finally, after reading the document about that gem,  and with the help of Jennifer, I find out that that is because the `flash[:error]` message only get assigned at the end of the request. In another word, the first error message only get assigned after the `erb  :"users/error"` statement when this request is done and can only be assessed in the next request. 

Accordingly, I change the codes from `erb :"/users/error"` to `redirect to "..."`. Since `redirect to` is actually another request, my error message will get assigned before that statement and successfully show up on the page whichever I redirect to!


## Helper method

Last but not least, I want to talk about helper methods. Sometimes we need to wrap some repetitive codes into a single method in order to save some work and also make the codes look more semanctically clear but at the same time, these methoes don't actually belong to any models we already have. So, we will have to wrap them into a helper method. 

In my app, I need two helpers methods: `current_user` and `log_in?`. I need to use them quite often to  verify the user or if they already log in before a review is created or edited. I try two different ways to use helper methods this time in my app and they have their own pros and cons.


*** put it in `helper.rb` in `models` directory***

One of the choices is to give the methods their own home! Since they don't belong to any models, I create a new file called `helper.rb`, and then write both methods there within class `Helper`. Since we will not instantiate any Helper instances because we only want to use methods there, we will write all the methods as class methods instead of instance methods. Here are my codes:

```
class Helper

  def self.current_user(session)
    User.find_by :id=>session[:user_id]
  end
	
	
  def self.log_in?(session)
    !!session[:user_id]
  end


end
```

Next time when I try to use these methods, I will just do `Helper.current_user(session)` or `Helper.log_in?(session)`, which also makes reading codes easier.
  
*** put it in the `applicationcontroller.rb` under `controllers` directory * **

The second way is leave it in the controller document. Within class `ApplicationController`, on the very bottom, we can add these following codes:

```
  helpers do

      def current_user
        User.find_by :id=>session[:user_id]
      end

      def log_in?
        !!session[:user_id]
      end

  end
	```
And when we want to use these methods, I just put `current_user` or `log_in?`, then they will be called right away. 

From the two versions above, maybe you already see their differences. the second example is simpler: no need to pass a `session` argument because when methods are within `ApplicationController`, `session` is enabled and can be reached everywhere in the controller; no need to build another class; no need to call it as a class method. That is the reason I choose the second one. However, I believe when there are more and more helpers methods, the first one will be easier to maintain and be taken care of. 

Ok, thanks for reading! Here is a link to my project if you are interested in it: [https://github.com/chanwkkk/my-daycare-center.git](http://)
