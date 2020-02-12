---
layout: post
title:      "My js and rails project -- connect frontend and backend"
date:       2020-02-12 21:36:31 +0000
permalink:  my_js_and_rails_project_--_connect_frontend_and_backend
---

I am always the big fan of Studio Ghibli so this time I want to build a app about its movies.  This is a single page application where you can read information of Ghibli movies, create movies and delete movies.  However, instead of the traditional way where you get redirected to different pages for different purposes, it is done by using AJAX calls, where I pass an URL and an object as two arguments to`fetch()` method to send data to the local server.  This article is about how do I connect frontend and backend to inplement `create movies` and `delete movies` features. 


**set up :  frontend mirrors backend **

It is the first time I have to create a frontend website and a backend server and connect them all by myself. To connect frontend with backend, I apply object-oriented programming.  Thanks to OO programmming, I am able to encapsulate the related data and behavior in one class, which is more organized and easier to maintain codes. I set up same classes in the frontend JS as in the backend rails so that all data and their own behaviors(/functions/methods) are mirrored each other and make handling data more convienient.  

In both the frontend and backend, I have these classes: `Movie`, `Character`, `Director`. 
The constructors in JS look like: 
```
class Movie{
  constructor(movie){
    this.id = movie.id
    this.title = movie.title
    this.description = movie.description
    this.rt_score = movie.rt_score
    this.image = movie.image
    this.release_year = movie.release_year
    this.director = new Director(movie.director)
    this.characters = movie.characters.map(c=>new Character(c))
  }
	...
}
```

```
class Director{
  constructor(director){
    this.name = director.name
    this.introduction = director.introduction
    this.image = director.image
    this.id = director.id
    this.addDirectorTotheForm()
  }
	...
}
```

```
class Character{
  constructor(character){
    this.name = character.name
    this.image = character.image
    this.introduction = character.introduction
  }
...
}
```

The schema in the backend looks like this: 
```
ActiveRecord::Schema.define(version: 2020_02_02_012434) do

  enable_extension "plpgsql"

  create_table "characters", force: :cascade do |t|
    t.string "name"
    t.string "introduction"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.integer "movie_id"
    t.string "image"
  end

  create_table "directors", force: :cascade do |t|
    t.string "name"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.text "introduction"
    t.string "image"
  end

  create_table "movies", force: :cascade do |t|
    t.string "title"
    t.string "description"
    t.integer "director_id"
    t.integer "rt_score"
    t.string "image"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.integer "release_year"
  end

end
```

For example, since properties of movie class in the frontend corresponds to those in the backend movie class, when I try to fetch all the movie instances in the database to JS, I just need to assign all matching attributes to movie instances in JS.  And then it is like I have a  "copy" of that movie instance in the frontend. What's more, I have a "copy" of the relationship too by initiating a director and characters instances when initiating a movie instance:  

```
class Movie{
  constructor(movie){
    ...
    this.director = new Director(movie.director)
    this.characters = movie.characters.map(c=>new Character(c))
  }
	...
}
```

Except OO programming, I am not sure what else I can do to preserve the associations: maybe set up some global variable and add a few functions, that would be a lot of headache. 


**create new movies**

Ok, so once the classes correspondence is set up, we can start to build up the create and delete function, which again, requires collaboration between frontend and backend. It is easy to build a form that asks for information about a movie, its director and its characters, but how to submit them with AJAX calls and in the back, how to link them? 

Since directors are limited in Studio Ghibli, I input all directors in the database already. Therefore, to link director, I use a select tag. While options' innertext are set to be directors'names for users to understand, options'values are set to directors'ids. In that way, the option value can be assigned to the backend instance's "director_id" attribute, and link to that instance with the corresponding director right away with no problems at all. 

For the characters, however, it is a bit complicated because we have to do two things: to create characters and to link that with movies. I don't want to make two forms and ask users to fill up and submit twice so I just use one form, one submit button.  In the back, I put  "accepts_nested_attributes_for :characters" in the "movie.rb" file. That enables me to create associated records through the parent. As a result, when I submit the parameters in this form: 

```
params = {
     ..., "characters_attributes"=>{"0"=>{"name"=>"juju", "image"=>"", "introduction"=>""}, "1"=>{"name"=>"howl", "image"=>"", "introduction"=>""
		 }
   }
```
One movie as well as two characters will be created at the same time with only one statement in the back: `movie = Movie.create(movie_params)`. 

Therefore, what is important is, I have to submit parameters in the above format so that it is recognized and used to generate movies and characters. Fortunately, I am using `fetch()` method to do this, so I can set up the format of the submitted data: 

```
         let characters_attributes = {};
         let inputs = inputArray[0]
         let inputsForNames = inputArray[1]
         let inputsForImages = inputArray[2]
  
         for (let i = 0; i< character_counter +1; i++){
          
             let name,image,introduction
             name = inputsForNames[i].value
             image = inputsForImages[i].value
             introduction = textareas[i+1].value
  
            if (!name==""){
            let object = {
              name: name,
              image: image,
              introduction: introduction 
            }
            characters_attributes[i] = object
            }
        }
				
          let formData = {
          title: inputs[0].value,
          director_id: select.value,
          description: textareas[0].value,
          image: inputs[1].value,
          rt_score: inputs[2].value,
          release_year: inputs[3].value,
          characters_attributes: characters_attributes
        }
				
				  let configObj = {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json"
          },
          body: JSON.stringify(formData)
        };

```

So what I am doing is I will loop over fields for characters in the form, and collect their information.  If the character name is not empty, then the character will be saved in `characters_attributes` in this format:  `"<number>"=><object>`. Later on, the form data will be passed to `configObj` and then through the `fetch()` to the `create` method in `movies controller`. 


**delete existed movies**

To delete a movie is much easier because instead of providing sufficient information to the server as in the `create` method, only the movie's id is required to identify and thus delete movies. Basically I pass the target movie to my `deleteMovie` function, and then pass the movie's id to the body of the second arguement `configObj` of `fetch()` so that it can be delivered to the `destroy` method in `movies controller`: 

```
     static deleteMovie(){
      let result = confirm("Do you want to delete this movie?")
      if (result){
      let configObj = {
        method: "DELETE",
        headers: {
          "Content-Type": "application/json",
          "Accept": "application/json"
        },
        body: JSON.stringify({
          id: this.id
        })
      };
  
       fetch(`${BACKEND_URL}/movies/${this.id}`, configObj)
       .then(resp=>resp.text())
       .then(resp=>alert(resp))
       .then(()=>document.getElementById(this.title).remove())
       .catch(error=>console.log(error))
    }    
    }
```

Once it is successfully deleted from the database in the backend, I add a `.then(()=>document.getElementById(this.title).remove())` statement to make sure it is removed from the DOM in the frontend too. 

**END**

Thanks for reading all. If you are interested in the whole application, please refer to the [Repo](https://github.com/chanwkkk/ghibli-movies-js-api.git). 
 




