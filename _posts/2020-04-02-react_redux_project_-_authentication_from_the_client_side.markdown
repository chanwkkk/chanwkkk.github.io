---
layout: post
title:      " React Redux Project - Authentication from the client side"
date:       2020-04-02 19:12:49 -0400
permalink:  react_redux_project_-_authentication_from_the_client_side
---


My final project is a food order app, where people can sign up, log in and order food. It is a combination of a react-based frontend and a rails API backend. You can refer to the [repo ](http://https://github.com/chanwkkk/food-order-app)to know more about this.  In this article, I am going to explain how I create both the frontend and backend, but most importantly, I will talk about the authentication system from the client side, in which I used gem JWT. 

## Rails API Backend

I use Rails to create my API from the scratch. Rails is very handy where I can just generate a new Rails api app from typing this in terminal: 

```
 rails new my_api --api
```

In this way, according to the documents of Rails, would do these three things: 

1. > Configure your application to start with a more limited set of middleware than normal. Specifically, it will not include any middleware primarily useful for browser applications (like cookies support) by default.
2. > Make ApplicationController inherit from ActionController::API instead of ActionController::Base. As with middleware, this will leave out any Action Controller modules that provide functionalities primarily used by browser applications.
3. > Configure the generators to skip generating views, helpers, and assets when you generate a new resource.

As you can see, it saves a lot of unnecessary work to produce unneeded templates. After that, I start to build up the backend in MVC(model, view, controller) pattern. However, view part is not needed in an API app. 

The structure looks like this: 
```

Models: 
   -Order
	 -User
	 -Dish
Controllers: 
  -orders_controller
	-dishes_controller
	-users_controller
	-session_controller
	
```

And with the help of gem `active_model_serializers`, I am able to render the appropriate JSON response to include the attributes and associations I want to the frontend. 

## React Frontend

To build the frontend, I use `React`, so just like how I can build a Rails app within one command, same when I use `React`! I can use `create-react-app my-frontend` to create a basic SPA (single page application). After that, I: 

1. install `Redux` so that I can monitor `state` in different components better
2. use the middleware `thunk` so I can make asynchrounous request to the backend
3. build different reducers and then connect root reducer to store in `redux`,  so redux will know  how to handle actions 

Now, the configurated `index.js` looks like this: 

```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import {createStore, applyMiddleware,compose} from 'redux';
import {Provider} from 'react-redux';
import thunk from 'redux-thunk';
import {combineReducers} from 'redux'
import 'bootstrap/dist/css/bootstrap.min.css';
import orderReducer from './reducers/orderReducer'
import dishReducer from './reducers/dishReducer.js'
import userReducer from './reducers/userReducer'
import loadingReducer from './reducers/loadingReducer';



const rootReducer = combineReducers({
    dishes: dishReducer,
    user: userReducer,
    loading: loadingReducer,
    order: orderReducer
  })

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;

let store = createStore(rootReducer, composeEnhancers(applyMiddleware(thunk)))



ReactDOM.render(
<Provider store={store}>
    <App />
</Provider>,
    document.getElementById('root')
);

```


## My Authentication System Using JWT token

Ok, enough configuration. Now I am going to explain how I implement my authentication system. So the basic idea is that: 
```
1. A user is going to log in in the frontend
2. Once logged in, the backend will pass a JWT token with an encrypted `user_id` by gem JWT to the frontend, 
3. The token will be then stored in the `localStorage` in the browser
4.  Next time, when a user is using the same browser to connect the backend, it will pass that token to the backend
5.  the backend will decode the `user_id` and then find the right user and render it to the backend. 
```

#### Backend

In the backend, the critical part is to encode and decode `user_id`, and communicate it with the frontend. I basically implement this in the `session_controller.rb`: 

```
class SessionController < ApplicationController

    def login
        user = User.find_by :email=>params[:email]
        if user && user.authenticate(params[:password])
            payload = {user_id: user.id}
            token = encode_token(payload)
            user_json = user.to_json(:include => [
                :orders=>{:include=> :dishes}])
            render json: {
                user: user_json,               
                jwt: token}
        else
            render json: {status: "error", message: "We don't find such an user according to your information,please try again."}
        end
    end
                                

    
    def auto_login
        if session_user
            render json: session_user, include: ['order','orders.dishes']
        else
            render json: {errors: "No User Logged In."}
        end     
    end
end

```

As you can see,  once a user is found and authenticated, it will generate a `token` including the information of the `user_id`,  which is encoded by the method `encode_token` that I wrote in `application_controller`, from which my `session_controller` is inherited. After, the `token` will be passed to the frontend with the user information as well to log in the user in the frontend. Here is how the `encode_token` looks like: 

```
    def encode_token(payload)
        JWT.encode(payload, 'secret')
    end
```

That is very easy! Just call `JWT`'s `encode` method, and pass the `payload`, which is the thing you want to hide, and the string `secret`, which stands for your password and something you need later to decode the `token`.

But one thing I need to point out specifically is that, you should encode a hash instead of only the `user_id`, in another word, the `payload` should be:  `{user_id: user.id}` instead of only the integer `user_id`. Because JWT has a lot of verifications when decoding a token. When verifying, it checks it as an hash, therefore, assuming a key and value pair there. If you only encode an integer, everything will break because it doesn't like this way. 

What does the `auto_login` method mean here then? Its basic function, as you can see, is to find a user, which is the task of method `session_user`, sitting in the `application_controller`. Here is how it looks like: 

```
    def auth_header_token
        request.headers['Authorization'].split(' ')[1]
    end

    def session_user
      decoded_hash = decoded_token
      if !decoded_hash.empty?
        user_id = decoded_hash[0]["user_id"]
        user = User.find_by :id=>user_id
      end
    end


    def decoded_token
        if auth_header_token
          begin
            JWT.decode(auth_header_token, 'secret',true, algorithm: 'HS256')
          rescue JWT::DecodeError
            []
          end
        end
    end
```

As you can see, the method `decoded_token` is trying to call `JWT.decode`, if you pass the token(which is grabbed by `auth_header_token`), "password"(the word that you pass when encoding), a true value and an algorithm corretly,  the token will be returned, and thus in method `session_user`, I can find the `user` instance with the `user_id` that is decoded. After this step, the `session_user` is sending either a user or `nil` back to `auto_login` method, then the correct user or error message will be sent to the frontend. 

#### Frontend

First of all, in both signing up and logging in a user, we need to store a JWT token in localStorage. How do I do that? The logic is basically sitting in my `userAction` file: 

```
export function loginUserFetch(userInfo){
    return dispatch=>fetch('http://localhost:3001/login', {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(userInfo)
        })
        .then(r=>r.json())
        .then(data=>{
            if(data.error){
                alert(data.error)
            }else{
             let user_json = JSON.parse(data.user) 
             localStorage.setItem("token", data.jwt)
             dispatch(loginUser(user_json))
            }
         })
 }
 



export function createUser(userinfo){
    return dispatch=>fetch('http://localhost:3001/signup', {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(userinfo)
        }).then(r=>r.json())
           .then(data=>{
           if(data.error){
               alert(data.error)
           }else{
            localStorage.setItem("token", data.jwt)
            dispatch(loginUser(data.user))
           }
        })
}
```

As you can see, in both `createUser` and `loginUserFetch` function, when we deal with the information that is passed from backend, once we successfully get a user, we will  store the value of `data.jwt` into a valuable called `token`. That would be used by our function to auto login later. And once we got to this point, we can dispatch a `login` action, which looks like a normal action and send an object to the reducer to store the user information in a globally-reachable store in `redux`. It looks like this: 

```
const loginUser = userObj => {
 return{
    type: 'LOGIN_USER',
    payload: userObj
}
}
```

As for how to auto login a user, here is how I implement: 

First of all, since it is a SPA, and everything is just sub components under `App` Component, so I decide to fetch that logged in user in `App` component. The best place to do it is to fire this action in `componentDidMount` function because it will fetch the user once the component is done mounting. 

```
class App extends React.Component {
   
  componentDidMount(){
    this.fetchEverything()   
  }

  fetchEverything = () =>{
    this.props.fetchLoggedInUser()
    ...
  }
	
	......
	
	end
```
 And the `fetchLoggedInUser` is a function in `userAction` and look like: 
 
 ```
 export function fetchLoggedInUser(){
    return dispatch=>{
        const token = localStorage.token
        if (token) {
            return fetch("http://localhost:3001/auto-login", {
              method: "GET",
              headers: {
                'Content-Type': 'application/json',
                Accept: 'application/json',
                'Authorization': `Bearer ${token}`
              }
            })
              .then(resp => resp.json())
              .then(data => {
                if (data.error) {
                    alert(data.error)
                  localStorage.removeItem("token")
                } else {
                   dispatch(loginUser(data))               
                }
              })
          }
    }
}

 ```
 
 So to start, it will check if there is a "token" stored in `localStorage`, if not, then will skip the rest of the codes, if yes, it will fire a `GET` request to the backend and embed the `token` in the Header, which, if you remember, will be grabbed by method `auth_header_token` in `application_controller` in the backend and be decoded by method `decoded_token` in the same place. If the backend authenticate this and send back a user object, then we can dispatch the `loginUser`action and log in the user everytime the SPA is rendered!
 
 
 ## END
 
 Thanks. That's all for today about my final react-redux project. If you have any question, you are super welcomed to DM me!








