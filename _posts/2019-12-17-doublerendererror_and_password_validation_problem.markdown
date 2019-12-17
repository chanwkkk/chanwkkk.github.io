---
layout: post
title:      "DoubleRenderError and password validation problem "
date:       2019-12-17 22:50:05 +0000
permalink:  doublerendererror_and_password_validation_problem
---


For my rails project, I decided to make a website about events. It is a website mainly between organizations, who hold an event and volunteers, who join the event. On one way, for organizations, they can post an event there for volunteers to join, comment on an volunteer based on the experience they work with him/her, and thus provide a reference to other organizations when picking volunteers; on the other way, for volunteers, they can always look up the events they are interested in and then reach them, also, they can see their reviews from the  website and know their performances better and react to those reviews.
It has been a big big challenge for me since it is the first time I build a rail project from the scratch. During the process, stackoverflow.com has always been a great help. Among all the problems I encountered, I want to share two that bother me the longest and my solutions for them. 

1. DoubleRenderError


I got this error when I tried to add some helpers method for the `show` route of `User`. My `application_controller.rb` looks like this: 

```
class ApplicationController < ActionController::Base

    def current_user
      User.find_by :id=>session[:user_id]
    end

    def log_in?
      !!session[:user_id]
    end

    def log_in_first
        if !log_in?
         flash[:error]="You have to log in first to continue your operation"
          redirect_to "/login"
        end
      end

      def correct_user?
        if !(current_user.id==@user.id)
         flash[:error]="You have no right to do this operation."
          redirect_to "/"
        end
      end

end
```

And here's my `show` action in `users_controller.rb`:

```
    def show
        log_in_first
        @user = User.find_by id: params[:id]
        correct_user?
        if @user
            render 'show'
        else
				     redirect_to '/login'
        end
    end
				    
```

As you can see, I tried to verify that the user is logged in and it is the correct user before a user can browse on someone's user page. So let's say if someone's not logged in and he/she wants to go to the `show` route, he/she will be redirect to the `login` page and see the error message: "You have to log in first to continue your operation".

However, when I tested it as a not-logged-in user, I got this error:

```
AbstractController::DoubleRenderError (Render and/or redirect were called multiple times in this action. Please note that you may only call render OR redirect, and at most once per action. Also note that neither redirect nor render terminate execution of the action, so if you want to exit an action after redirecting, you need to do something like "redirect_to(...) and return".)
```

Actually, the error message is very clear for telling me what I have done wrong: I expected that in method `log_in_first`, once I got redirected to `/login`, the `show` action terminates right away, which means, it will not move on.

However, the message told me: " `Also note that neither redirect nor render terminate execution of the action`". Here we are!  So `log_in_first` will redirect the `show` action, but after that, the `show` action will continue! After that, for sure it will stuck on the method `correct_user?` too since there is not even an user here! That is a second `redirect`, and then, in the `show` action, whether the `@user` is persisted or not, `render` or `redirect_to` will be called. There would be three `redirect`s!

But according to the name of the error, "DoubleRenderError" or the error message: "Please note that you may only call render OR redirect, and at most once per action", we know that more than one `render` or `redirect` is not acceptable. That is where the problem comes from. 

To solve this, instead of using the recommendation way——"...so if you want to exit an action after redirecting, you need to do something like "redirect_to(...) and return", I take out the helper methods and use `before_action`, so that way the helpers will do whatever they need to and let the `show` action call either the `render` or `redirect` only one time within the action and thus solve the problem. 

So now, the whole `application_controller.rb` looks like this: 

```
class ApplicationController < ActionController::Base



    def current_user
      User.find_by :id=>session[:user_id]
    end

    def log_in?
      !!session[:user_id]
    end

    def authenticate_user
      unless log_in?
        flash[:error] = "You have to log in first."
        redirect_to '/login'
      end
    end

    def authorized_user?
      unless @user.id == current_user.id
        flash[:error] = "You have no right to do this operation."
        redirect_to '/'
      end
    end

end

```

And the `users_controller.rb` looks like this: 

```
class UsersController < ApplicationController
  before_action :authenticate_user, only: [:show]
	
	......
	

    def show
      @user = User.find_by id: params[:id]
      if @user
          render 'show' 
      else
        flash[:error] = "That user doesn't exist."
        redirect_to '/'
      end
    end
		
	......
	
		end
```

Now it works!

2. Injecting the password when signing up an user through third party account

There are two ways to sign up a new user in my app: the traditional one that you provide password and email, and the one that you use the third party account, which is google account here in my app. 

To make sure a user provides password and email, I put validations for `User` model. Therefore, if an attempt to sign up a user without providing both of those two items, it will fail. 

It is not too difficult when a form is provided to capture a password and an email. However, it is hard when I use Omniauth. Omniauth is a gem in rails that offers another way to authenticate a user's identity. When you click the button that says  "Log in with Google", Google would usually ask if you want to authorize this operation for the website to access your information to create a user in their database(if you haven't already logged in your google account, just log in normally first). Once you agree, a user is created with the personal information sent by Google and you will usually be redirected to the page you tried to access before. 

The good thing is, it is just one "agree" button away before you can sign up, very fast because it saves you a lot of troubles of filling out a form. On the other side, the bad thing is, how can a user tell the database what the password is if there is not even a form? 

At first, I was looking for a way to validate two ways of signing up differently. For example, ask for password only in the traditional route to sign up. But after I talk to Howard, he told me better not to do that because password is very important and therefore, we should never give a roundabout route to skip setting up a password. He gave me an suggestion on changing the corresponding action to create a user through third party accounts, which is, `new_from_google` action in `Sessions Controller` here. 

It looks like this now: 
```
    def new_from_google
       auth = request.env['omniauth.auth']['info']
       @user = User.find_or_create_by(email: auth['email'])
       if @user.persisted?
        session[:user_id] = @user.id
        redirect_to user_path(@user)
       else
         @user.name = auth['name']
       end
    end
```
The key lies in the assignment statement: `@user = User.find_or_create_by(email: auth['email'])`. The method `find_or_create_by` accepts a block and would passed the params to `create`. So I can change it to the following way. When a user with that specific email cannot be find, the block including a password will be passed to create a new user and the problem is solved!  And next time, when the same user wants to log in, a google account's presence is enough. 

```
    def new_from_google
       auth = request.env['omniauth.auth']['info']
       @user = User.find_or_create_by(email: auth['email']) do |u|
         u.name = auth['name']
         u.password = "superhardpassword"
       end
       if @user.persisted?
        session[:user_id] = @user.id
        redirect_to user_path(@user)
       else
         @user.name = auth['name']
       end
    end
```

I think in real life situation, we can also provide a way for the user to change their password later, just in case they want to log in using the email/password combination. 


