# Many-to-Many Relationships in Rails

## Learning Objectives

* Differentiate between a one-to-many and a many-to-many relationship
* Describe the role of a join table in a many-to-many relationship
* Create a Model to represent a join table
* Use the `has_many :through` association to connect two models via a join model in Rails
* Use a many-to-many relationship to implement a feature in a Rails application

## Framing

When we think back on what we have learned so far in this unit, we now have the ability to model real world entities and their relationships, and we have built web applications that have persisted data within these models.

**Q**. Reviewing what we have learned about relational databases and ActiveRecord, what has been the predominate relationship we have used so far?

---

Up to this point, we have focused on domains that only have two models, i.e. `Artists`, and `Songs`, which in turn have a strict one-to-many relationship. At its core, we expressed these relationships with ActiveRecord methods, and linked the tables via a foreign key on the child table.

This type of relationship is probably the most common, but today we will be looking at another widely used and very useful relationship that will help build out additional features within our Rails apps.

## Many-to-Many Relationships (10 minutes)

Put simply, Many-to-Many relationships arise when one or more records in a table, has a relationship with one or more records in another table.

Many-to-many relationships are fairly common in applications. Some examples include:

* `Posts` can be sorted into multiple `Categories`, `Categories` contain many `Posts`.
* `Theaters` can show many `Movies`, and `Movies` may appear in many `Theaters`.
* `Playlists` contain many `Songs`, `Songs` can be on multiple `Playlists`.
* `Doctors` can have many `Patients`, a `Patient` can have more than one `Doctor`

Unlike one-to-many relationships, we can't just add a foreign key to one of the
two tables (the `belongs_to` table) to store these associations. We'd run into a
problem where the column would need to store multiple ids, rather than just the
one id in a one-to-many relationship.

Instead, we must create a new table, a ***join table*** to store these associations.

### Join Tables

A join table is a separate intermediate table in our database whose job is to store information
about the relationship between our two models of the many-to-many. For each
many-to-many relationship, we'll need one join table.

> Why are they called "join tables"? On a database level, join tables are created using SQL methods like `INNER JOIN` and `OUTER JOIN`. Learn more about them [here](http://www.sql-join.com/).

Each join table should have, at minimum, **two foreign_key columns**. Each foreign key will represent one of the tables it's joining. In the example of `Doctors` and `Patients`, we would create a **new** join table that has a `doctor_id` column and a `patient_id` column.

We can also add columns as needed to store additional information about the relationship. For example, we may choose to add a `date_of_visit` column, which stores a `datetime` value representing when the appointment is, and could be different for each doctor + patient visit.

## Join Models & Tables

In order to do many-to-many relationships in Rails, convention says to create a new model to represent our join table. The name can technically be anything we want, but the model name should be as descriptive as possible, and indicate that it represents an *association*.

### You Do: Naming Join Tables (10 minutes)

In pairs, spend **5 minutes** answering the following questions for the below pairs of models...  
  1. Should the relationship between these two models be represented using a many-to-many relationship?
  2. What would be a descriptive name for their resulting join table?
  3. What would be a useful additional column to include in the join table (e.g., `order`)?

Models  
  1. Authors and Books
  2. Students and Courses  
  3. Users and Groups  
  4. Blog Posts and Categories  
  5. Reddit Posts and Votes  

### I Do: Generating the Model / Migration (10 minutes)

> **Note**: You are encouraged to **not** code along during this section -- just sit back and enjoy the ride! You will have the chance to implement this during in-class exercises with Tunr.  

In order to see how to implement an example of a common many-to-many relationship in Rails, I'm going to build an event tracking application. For this application, I am only going to focus on the ability for a user to attend an event.

<details>
<summary><strong>Q.</strong> What should the three models in our application be?</summary>

Let's call them: `User`, `Event`, and `Attendance`

</details>

---

For our domain's purposes, let's create a new model `Attendance` to represent the many-to-many relationship between our other two models: `User` and `Event`.

```bash
$ rails new attendance-tracker -d postgresql
$ rails g model User username:string age:integer
$ rails g model Event title:string location:string
```

We generate the model just like any other. If we specify the attributes (i.e.,
columns on the command line) Rails will automatically generate the correct
migration for us.

Onto the model files...

```bash
$ touch app/models/attendance.rb
```

```rb
# models/attendance.rb

class Attendance < ApplicationRecord
  # Associations to come later...
end
```

Now the migration...  

```bash
$ rails g migration create_attendances
```

```rb
# db/migrate/*****_create_attendances.rb

class CreateAttendances < ActiveRecord::Migration[5.0]
  def change
    create_table :attendances do |t|
      t.integer :num_guests, null: false
      t.references :user, index: true, foreign_key: true, null: false
      t.references :event, index: true, foreign_key: true, null: false

      t.timestamps
    end
  end
end
```

> **What is `t.references`?** It creates a column for referencing rows in another table (foreign key).

This will generate an Attendance table with `user_id`, `event_id` and `num_guest` columns. Take a look at it using `psql` in the Terminal.

### You Do: Create the Favorite Model in Tunr (10 minutes / 0:40)

> 5 minutes exercise. 5 minutes review.

For the in-class exercises you will be adding a "favoriting" feature to Tunr. In this version of Tunr, a user should be able to favorite a song.

To get started:

1. Clone down [this repo](https://github.com/ga-wdi-exercises/tunr_rails_many_to_many/tree/favorites-starter)
2. Checkout to the `favorites-starter` branch
2. Run `$ bundle install`
3. Run `$ rails db:drop db:create db:migrate db:seed`

> **Note**: Make sure to work off the `favorites-starter` branch.

Then:

- Create a model and migration for `Favorite`.
  - It should have `song_id` and `user_id` columns.

> A `User` model and user authentication functionality has already been provided for you. Because of this, you may see some code in here -- particularly in `models/user.rb` and `routes.rb` that was added by the gem Devise

### Adding the ActiveRecord Relationships (10 minutes / 0:50)

Once we create our join model, we need to update our other models to indicate the associations between them. Let's visualize these associations with an ERD.

**Board**: Diagram Attendance Tracker ERD

For example, in our Users/Events example, we should have this...

```ruby
# models/attendance.rb
class Attendance < ApplicationRecord
  belongs_to :event
  belongs_to :user
end

# models/event.rb
class Event < ApplicationRecord
  has_many :attendances
  has_many :users, through: :attendances
end

# models/user.rb
class User < ApplicationRecord
  has_many :attendances
  has_many :events, through: :attendances
end
```

We're essentially defining `Attendance` as an intermediary model/table between `Event` and `User`. An event has many users through `Attendance` and vice versa.

## Break (10 minutes / 1:00)

### You Do: Update Tunr Models (10 minutes / 1:10)

Take **5 minutes** to update the Song, User and Favorite models to ensure we have the
correct associations.

> If you finish early, go ahead and start testing out these new associations using the Rails console.

### Testing Our Associations (10 minutes / 1:20)

It's a good idea to use the `rails console` to test creating our associations.

Here's an example of using the association of users / events...

```ruby
george = User.create({username: "George", age: 18})
lorraine = User.create({username: "Lorraine", age: 17})

prom = Event.create({title: "Enchantment Under The Sea: 1955 Prom", location: "Hill Valley High School"})
after_party = Event.create({title: "Betty's Awesome After-party", location: "Super Secret!" })
brunch = Event.create({title: "Brunch!", location: "Lou's Cafe" })

# We can create the association directly
george_going_to_the_prom = Attendance.create(user: george, event: prom, num_guests: 1)

# Or using helper functionality
george.attendances.create(event: after_party, num_guests: 0)

# Or the other way
brunch.attendances.create(user: lorraine, num_guests: 10)
prom.attendances.create(user: lorraine, num_guests: 1)

# To see who's going to an event
prom.users
after_party.users
brunch.users

# To see a user's events
george.events
lorraine.events

```

### We Do: Add Web Interface to Tunr (15 minutes / 1:35)

So we've been able to generate associations between our models via the rails console. But what about our end users? How would somebody go about creating/removing a favorite on Tunr?

<details>
<summary>At a high level, what type of code do we need to add to support our new favoriting feature for Tunr?</summary>

We need to add functionality by **modifying our controller, view and routes**.

</details>

---

Before we add anything new, let's do a quick recap of the code we are starting from...

#### Controller

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
    @song = @artist.songs.new
  end

  # create
  def create
    @artist = Artist.find(params[:artist_id])
    @song = @artist.songs.create(song_params)
    redirect_to artist_song_path(@artist, @song)
  end

  #show
  def show
    @song = Song.find(params[:id])
  end

  # edit
  def edit
    @artist = Artist.find(params[:artist_id])
    @song = Song.find(params[:id])
  end

  # update
  def update
    @song = Song.find(params[:id])
    @song.update(song_params)
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

#### Routes

There's more to this than just updating the `Songs` controller, we also need to make sure that our application has routes to support "favoriting" and "unfavoriting". For the sake of convenience, we have already defined the desired routes for you...

```rb
# config/routes.rb

Rails.application.routes.draw do
  devise_for :users
  root to: 'artists#index'

  resources :artists do
    resources :songs, except: [:index, :show]
  end

  resources :songs, only: [:index, :show] do
  # This creates two custom routes for songs that correspond with controller actions of the same name.
    member do
      post 'add_favorite'
      delete 'remove_favorite'
    end
  end
end
```
> Read more about `member` routes here: [Rails Routing - Adding More RESTful Actions](http://guides.rubyonrails.org/routing.html#adding-more-restful-actions)
>
> **Hint**: check out the output of `rails routes` to see what those lines generated!

#### View

Great, now that we know we have the necessary routes defined, we need a way for the user to actually interact with our Web app so they can favorite a song.

We've gone ahead a provided some starter code in `app/views/artists/show.html.erb`, so let's look at the interface to how the user will favorite a song...

```erb
<h3>Songs <%= link_to "(+)", new_artist_song_path(@artist) %></h3>
<ul>
  <% @artist.songs.each do |song| %>
    <li>
      <%= link_to "#{song.title} (#{song.album})", song_path(song) %>

      # If this song has already been favorited, set the link to remove favorite.
      <% if song.users.include? current_user %>
        <%= link_to "&hearts;".html_safe, remove_favorite_song_path(song), method: :delete, class: "fav" %>
      # If the song has not been favorited, set the link to add favorite.
      <% else %>
        <%= link_to "&hearts;".html_safe, add_favorite_song_path(song), method: :post, class: "no-fav" %>
      <% end %>

    </li>
  <% end %>
</ul>
```

## Break (10 minutes / 1:45)

### You Do: Update Songs Controller (20 minutes / 2:05)

Take **15 minutes** to create the `add_favorite` and `remove_favorite` actions in the **songs controller**. Look at the `artists/show.html.erb` view to see how we route to these actions.

Below are some line-by-line instructions on how to implement `add_favorite` and `remove_favorite`. We encourage you not to look at the solution unless you are stuck!  

#### Instructions

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

Because we are using Devise to handle user authentication, it gives us access to a `current_user` method that, when called, returns the user who is currently logged in. Conceptually, think of it as running something like `User.find_by(logged_in: true)`.  

This means that in your controller you can write code like `Favorite.create(user: current_user)`.

#### If You Need the Solution...

[...you can take a peek at it here.](https://github.com/ga-wdi-exercises/tunr_rails_many_to_many/tree/favorites-solution)

## Closing Q&A (10 minutes / 2:15)

1. Why do we need to have Many-to-Many relationships? Give examples.

2. What extra feature(s) do we need to add to our schema/model in order to implement Many-to-Many?

3. How do we add non-standard routes inside our `resources` directive?

## Bonus

###  Additional Exercise: Many-to-Many Scribble

If there's time left, spend the remainder of class working on Scribble. If you have completed the required steps, try implementing a many-to-many relationship between `Posts` and `Categories` using a `Tags` join table. This will require creating some new classes.

[More information is available on the Scribble repo.](https://github.com/ga-wdi-exercises/scribble/blob/master/readme.md#bonus)

## Screencasts

* [Screencast on Youtube](https://www.youtube.com/watch?v=JxW8lJzLhxI)
* [Andy 1](https://youtu.be/PXNrk6m4WRg)
* [Andy 2](https://youtu.be/Gr3GV8dUkDE)

## References

* [Rails Guides - Has Many Through](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)
* [Tunr Favorites Starter](https://github.com/ga-wdi-exercises/tunr_rails_many_to_many/tree/favorites-starter)
* [Tunr Favorites Solution](https://github.com/ga-wdi-exercises/tunr_rails_many_to_many/tree/favorites-solution)
