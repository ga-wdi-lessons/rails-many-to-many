# Many-to-Many Relationships (using has_many :through) in Rails

## Learning Objectives

* Differentiate between a one-to-many and a many-to-many relationship
* Describe the role of a join table in a many-to-many relationship
* Create a Model to represent a join table
* Use `has_many through` to connect two models via a join model in Rails
* Use a many-to-many relationship to implement a feature in a Rails app

### Screencast of this lesson

[Screencast on Youtube](https://www.youtube.com/watch?v=JxW8lJzLhxI)
[Andy 1](https://youtu.be/PXNrk6m4WRg)
[Andy 2](https://youtu.be/Gr3GV8dUkDE)

### References

* [Rails Guides - Has Many Through](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)
* [Tunr Favorites Starter](https://github.com/ga-dc/tunr_rails_many_to_many/tree/favorites-starter)
* [Tunr Favorites Solution](https://github.com/ga-dc/tunr_rails_many_to_many/tree/favorites-solution)

## Many-to-Many Relationships (10 minutes / 0:10)

Many-to-many relationships are fairly common in apps. Examples might include:

* Posts can be sorted into multiple Categories, Categories contain many Posts.
* Groups can host many Photos, Photos may appear in many Groups.
* Playlists contain many songs, Songs will appear on multiple Playlists.

Unlike one-to-many relationships, we can't just add a foreign key to one of the
two tables (the `belongs_to` table) to store these associations. We'd run into a
problem where the column would need to store multiple ids, rather than just the
one id in a one-to-many relationship.

Instead, we must create a new table, a *join table* to store these associations.

### Join Tables

A join table is a separate intermediate table in our database whose job is to store information
about the relationship between our two models of the many-to-many. For each
many-to-many relationship, we'll need one join table.

> Why are they called "join tables"? On a database level, join tables are created using SQL methods like `INNER JOIN` and `OUTER JOIN`. Learn more about them [here](http://www.sql-join.com/).

Each join table should have, at minimum, two foreign_key columns. Each foreign key will represent one of the tables it's joining. In the example of Songs and Playlists, we would have a `song_id` column and a `playlist_id` column.

We can also add columns as needed to store additional information about the relationship. For example, we may choose to add an `order` column which stores an integer representing what order that song appears on the playlist. A song may be first on one playlist, but 10th on another - that info would be stored on the join table.

## Join Models & Tables

In rails, we should always create a model to represent our join table. The name can technically be anything we want, but really the model name should be as descriptive as possible, and indicate that it represents an *association*.

### You Do: Naming Join Tables (10 minutes / 0:20)

In pairs, spend **5 minutes** answering the following questions for the below pairs of models...  
  1. Should the relationship between these two models be represented using a many-to-many relationship?
  2. What would be a descriptive name for their resulting join table?
  3. What would be a useful additional column to include in the join table (e.g., `num_guests`)?

Models  
  1. Authors and Books
  2. Students and Courses  
  3. Doctors and Patients  
  4. Blog Posts and Categories  
  5. Reddit Posts and Votes  

### Generating the Model / Migration (10 minutes / 0:30)

> We will be using Attendances as the in-class example. We encourage you **not** to code along -- just watch. You will have the chance to implement this during in-class exercises with Tunr.  

Attendances represent the many-to-many relationship between two models: Users and Events. Let's set them up in a brand new Rails application...

```bash
$ rails new attendance-tracker -d postgresql
$ rails g model User username:string age:integer
$ rails g model Event title:string location:string
```

We generate the model just like any other. If we specify the attributes (i.e.,
columns on the command line) Rails will automatically generate the correct
migration for us.

Onto the model files...

```rb
# models/attendance.rb

class Attendance < ActiveRecord::Base
  # Associations to come later...
end
```

Now the migration...  

```bash
$ rails g migration create_attendances
```

```rb
# db/migrate/*****_create_attendances.rb

class CreateAttendances < ActiveRecord::Migration
  def change
    create_table :attendances do |t|
      t.integer :num_guests, null: false
      t.references :user, index: true, foreign_key: true
      t.references :event, index: true, foreign_key: true

      t.timestamps null: false
    end
  end
end
```

> **What is `t.references`?** It does the same thing as writing out `belongs_to :model`.

This will generate an Attendance table with `user_id`, `event_id` and `num_guest` columns. Take a look at it using `psql` in the Terminal.

### You Do: Create the Favorite Model in Tunr (10 minutes / 0:40)

> 5 minutes exercise. 5 minutes review.

For the in-class exercises you will be adding a "favoriting" feature to Tunr. In this version of Tunr, a user should be able to favorite a song.

[Here's some starter code](https://github.com/ga-dc/tunr_rails_many_to_many/tree/favorites-starter). Make sure to work off the `favorites-starter` branch.

Create a model and migration for `Favorite`. It should have `song_id` and `user_id` columns.

> A `User` model and user authentication functionality has already been provided for you. Because of this, you may see some code in here -- particularly in `models/user.rb` and `routes.rb` that you have not seen before. We will explain what it does in a later mini-lesson.

### Adding the ActiveRecord Relationships (10 minutes / 0:50)

Once we create our join model, we need to update our other models to indicate the associations between them. Let's visualize these associations with an ERD.

For example, in our Users/Events example, we should have this...

```ruby
# models/attendance.rb
class Attendance < ActiveRecord::Base
  belongs_to :event
  belongs_to :user
end

# models/event.rb
class Event < ActiveRecord::Base
  has_many :attendances
  has_many :users, through: :attendances
end

# models/user.rb
class User < ActiveRecord::Base
  has_many :attendances
  has_many :events, through: :attendances
end
```

We're essentially defining `Attendance` as an intermediary model/table between `Event` and `User`. An event has many users through `Attendance` and vice versa.

### You Do: Update Tunr Models (10 minutes / 1:00)

Take **5 minutes** to update the Song, User and Favorite models to ensure we have the
correct associations.

> If you finish early, go ahead and start testing out these new associations using the Rails console.

## Break (10 minutes / 1:10)

### Testing Our Associations (10 minutes / 1:20)

It's a good idea to use the `rails console` to test creating our associations.

Here's an example of using the association of users / events...

```ruby
bob = User.create({username: "Bob", age: 25})
carly = User.create({username: "Carly", age: 28})

prom = Event.create({title: "Under the Sea: 2015 Prom", location: "Greenville High School"})
after_party = Event.create({title: "Eve's Awesome After-party", location: "Super Secret!" })
brunch = Event.create({title: "BRUNCH!", location: "IHOP" })

# We can create the association directly
bob_going_to_the_prom = Attendance.create(user: bob, event: prom, num_guests: 1)

# Or using helper functionality
bob.attendances.create(event: after_party, num_guests: 0)

# Or the other way
brunch.attendances.create(user: carly, num_guests: 10)
prom.attendances.create(user: carly, num_guests: 1)

# To see who's going to an event
prom.users
after_party.users
brunch.users

# To see a user's events
bob.events
carly.events

# To delete an association
Attendance.find_by(user: bob, event: prom).destroy # will only destroy the first one that matches

Attendance.where(user: bob, event: prom).destroy_all # will destroy all that match
prom.attendances.where(user: bob).destroy_all
```
## Break (10 minutes / 1:30)

### Updating The Controller (15 minutes / 1:45)

So we've been able to generate associations between our models via Pry. But what about our end users? How would somebody go about creating/removing a favorite on Tunr?
* We need to add that functionality by modifying our controller, view and routes.

Let's take a look at `songs_controller.rb`...
* What do we currently have in here?
* Can we use any of these actions to handle adding/removing songs? Or do we need to add something new?

```rb
class SongsController < ApplicationController
  # index
  def index
    @songs = Song.all
  end

  # new
  def new
    @artist = Artist.find(params[:artist_id])
    @song = Song.new
  end

  # create
  def create
    @artist = Artist.find(params[:artist_id])
    @song = Song.create!(song_params.merge(artist: @artist))
    redirect_to artist_song_path(@artist, @song)
  end

  #show
  def show
    @song = Song.find(params[:id])
  end

  # edit
  def edit
    @song = Song.find(params[:id])
  end

  # update
  def update
    @song = Song.find(params[:id])
    @artist = Artist.find(params[:artist_id])
    @song.update(song_params.merge(artist: @artist))
    redirect_to artist_song_path(@song.artist, @song)
  end

  # destroy
  def destroy
    @song = Song.find(params[:id])
    @song.destroy
    redirect_to songs_path
  end

  private
  def song_params
    params.require(:song).permit(:title, :album, :preview_url, :artist_id)
  end
end
```

We have CRUD functionality for the songs themselves, but that's about it.
* We need to add some actions to our controller that handle this additional functionality. You'll do that for Tunr in the next exercise.
* These will not correspond to RESTful routes.

There's more to this than just updating the Songs controller, but we've done some of the work for you...

```rb
# config/routes.rb

Rails.application.routes.draw do
  devise_for :users
  root to: 'artists#index'
  get '/songs', to: 'songs#index'
  resources :artists do
    resources :songs
    resources :genres
  end

  # This creates two custom routes for songs that correspond with controller actions of the same name.
  resources :songs do
    member do
      post 'add_favorite'
      delete 'remove_favorite'
    end
  end
end
```

> This application uses a gem called Devise to handle User authentication. All you need to know about that for now is that it generates a User model.
>
> Also, see how we've indented resources statements? That's called nested resources. It enables us to visit URLs like `http://localhost:3000/artists/1/songs/2`. You'll learn more about those later.

<!-- AM: Run rake routes. -->

```erb
# app/views/artists/show.html.erb

<h3>Songs <%= link_to "(+)", new_artist_song_path(@artist) %></h3>
<ul>
  <% @artist.songs.each do |song| %>
    <li>
      <%= link_to "#{song.title} (#{song.album})", artist_song_path(@artist, song) %>

      # If this song has already been favorited, set the link to remove favorite.
      <% if song.favorites.length > 0 %>
        <%= link_to "&hearts;".html_safe, remove_favorite_song_path(song), method: :delete, class: "fav" %>
      # If the song has not been favorited, set the link to add favorite.
      <% else %>
        <%= link_to "&hearts;".html_safe, add_favorite_song_path(song), method: :post, class: "no-fav" %>
      <% end %>

    </li>
  <% end %>
</ul>
```


### You Do: Update Songs Controller (20 minutes / 2:05)

Take **15 minutes** to update the `add_favorite` and `remove_favorite` actions in the playlists controller to
add and remove songs from the playlist. Look at the `artists/show.html.erb` view to see how we route to these actions.

Below are some line-by-line instructions on how to implement `add_favorite` and `remove_favorite`. I encourage you not to look unless you are stuck!  

Start out by logging into the application using the "Sign Up" feature. It should be visible in the top-right corner on the home page. Once you've done that, tackle the controller actions.

`add_favorite` should...  
  1. Save the song which you will be favoriting in an instance variable.  
  2. Create a new Favorite instance that...  
    a. Belongs to the song.  
    b. Belongs to the user who is creating the favorite.  
  3. Redirect to the show page for the artist once the song is added.

`remove_favorite` should...  
  1. Save the song you will be un-favoriting in an instance variable.  
  2. Delete the Favorite instance that references the song that is being un-favorited.  
  3. Redirect to the show page for the artist once the song is added.

#### How Do We Get the Logged-In User?

Because we are using Devise to handle user authentication, it gives us access to a `current_user` method that, when called, returns the person who is currently logged in. At a high level, think of it as running something like `User.find_by(logged_in: true)`.  

This means that in your controller you can write code like `Favorite.create(user: current_user)`.

#### If You Need the Solution...

[...you can take a peek at it here.](https://github.com/ga-dc/tunr_rails_many_to_many/tree/favorites-solution)

## Closing Q&A (10 minutes / 2:15)

## Homework: Scribble

If there's time left, spend the remainder of class working on Scribble. If you have completed the required steps, try implementing a many-to-many relationship between `Posts` and `Categories` using a `Tags` join table. This will require creating some new classes.

[More information is available on the Scribble repo.](https://github.com/ga-dc/scribble/blob/master/readme.md#many-to-many-bonus)

Example: [Deployed Scribble](https://wdi-scribble.herokuapp.com/)
