# Flatiron Picture-gram

## Your Task
Your task is to create a Rails app with full CRUD functionality.  The domain will be a picture sharing application in which you will be asked to deliver the following features:
 - A user can sign-in
 - A user can upload pictures by providing a URL
 - In addition to the URL, pictures can be given one or more tags and a title
 - A user can browse all of another user's pictures or browse pictures from across all the users by tag name
 - A user can leave comments on individual pictures

This reading will provide details on how to best proceed with this task. Following the order of the steps as listed here will make it easier to go from `rails new` to a functioning application. There are many options and possibilities to extend and customize this project.  The best way to get to that point is to get a solidly functioning app up and running as soon as possible.

Note that there are **no tests**.  Your goal is to use Behavior Driven Development to confirm that your code is doing what it should.  This means instead of running `rspec` or `learn` you should be running your code frequently:  Open up the `rails console` and confirm that your methods and associations work, run a local server with `rails server` and see that your app has the desired behavior.

## Overview

1. - [ ] [Create a Repository](#create-a-repository)
2. - [ ] [Set up the Domain](#domain-modeling)
3. - [ ] [Add seed data](#seeding-your-database)
4. - [ ] [Add Methods on the Models](#model-methods)
5. - [ ] [User Authentication](#user-sign-up)
6. - [ ] [User Show Page](#user-show-page)
7. - [ ] [Picture Show and Index Pages](#picture-show)
8. - [ ] [Tag Show and Index Pages](#tag-index-and-show)
9. - [ ] [Extensions and Bonus Features](#extensions)


## Create a Repository

Create a new repository on Github, if you have a partner only one of you should create the repo. The partner who does not own the repo should not make their own fork, but can clone down from the same remote repository. If the owner adds the partner as a collaborator (through the settings for the repository on Github) both partners can push changes to the same repository.

Don't forget to commit often and provide descriptive commit messages!

[🖇️ Github: Add A Collaborator](https://help.github.com/articles/inviting-collaborators-to-a-personal-repository/)

## Domain Modeling

The first step is to set up the domain. It is ok to take some time to thoroughly understand the domain before proceeding as it provides the structure on top of which everything else is built.

- #### `users` table

 You'll definitely need a `users` table, no surprise there.  
 - A user will need an email, username and a password
 - The email and username should be unique.  Use a validation for this.
 - You should use the `bcrypt` gem to handle user authentication. What should the column name be that stores encrypted passwords when using `bcrypt`?

[🖇️ Rails & Bcrypt](http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password)

- #### `pictures` table

 The `pictures` table should have columns for an image_url (a String) and a title.  A picture `belongs_to` a user so you'll also need a column storing a reference, a foreign key, to the user.

- #### `tags` table

 There won't be too much on the `tags` table. The only thing you'll need to add is a `name` column referring to the name of the tag, something like 'Nature', 'Friends', 'Summer', or 'Travel'

- #### `picture_tags` table
 The specs for the application are that a picture should be able to have more than one tag. This means that the same picture could be categorized as 'Collage', 'Art', and 'Museum'. For any individual tag, there may be many pictures, i.e. multiple pictures may have the tag 'NYC'.  The relationship between a picture and a tag, then, is a many-to-many relationship.

 This indicates we will need a join table which we will call `picture_tags`.  What columns need to be on this table?

- #### `comments` table
 A comment should be able to be left on any picture by any user, even their own photo. It should have a column called content that will store the text of the comment. A comment `belongs_to` a picture and `belongs_to` the user who left the comment. Again, what columns does this indicate will be required?

The next step is to look at each of the models after you have ran the migrations. The `belongs_to` relationships were described above, it's up to you to add in the correct `has_many` and `has_many :through`s

## Seeding your Database

When working on labs the rspec tests go through the steps of creating some data used to test out the functionality.  For example, there would be no way of checking if the pictures index page displayed the appropriate information if there were no pictures in the database.

Since there are no rspec tests here, it will be extra important to add some seed data. You could open up Rails Console and type in `Category.create`, `User.create`, etc. and this would in fact persist the newly created instances. The problem with doing this through Rails Console is that you will have no way to repeat or reproduce the steps you went through to add the records to the db.

What if someone else wanted to clone down your repository and have some data to work with, or what if at some point you wanted to reset your database (clear all data from the tables, and start from scratch)?  All the data you created would have to be manually re-entered, not too fun.  Wouldn't it be great if there was a single command that would create all the seed data we needed.

Well there is! The place to add the code that will get run with the `rake db:seed` command is the seed file (found at `db/seeds.rb`).

The seed file is just a regular ruby file, there's nothing too special about the code we'll write in it. In it you will write the code that will add records to the database, something like:

```ruby
[1,2,3].each do |num|
  User.create(
    username:"user#{num}",
    email: "user#{num}@example.com",
    password: "test123"
  )
end

["Nature", "NYC", "Art", "Humor"].each do |tag_name|
  Tag.create(name: tag_name)
end

url = "http://www.defenders.org/sites/default/files/styles/large/public/dolphin-kristian-sekulic-isp.jpg"

Picture.create(
  image_url: url,
  title: "Saw a dolphin!",
  user_id: User.first.id
)

```

Create a few users and several tags. You may want to create some comments and associate pictures with tags. Having concrete seed data *will* help you build your application.

#### 🔎 Common Error Note:
> If you try to run `rake db:seed` and you get an error saying that 'users' has no such column 'password', there is a single line you need to add to you `User` model. Take a look at the readings and documentation for `bcrypt` to see how to take care of this. What does adding this line do?

[🖇️ Has Secure Password reading](https://github.com/learn-co-curriculum/has_secure_password_readme)

#### 🔎 Common Error Note:
> When you type `rails new` into your terminal and create a new rails project the version of Rails you are using will be Rails 5, you can check the version on your gemfile.  Most of the labs are using Rails 4. There are few minor differences, and in one in particular that may affect your ability to persist instances to the database. Read more here:

[🖇️ Rails 5 Belongs_to](http://blog.bigbinary.com/2016/02/15/rails-5-makes-belong-to-association-required-by-default.html)

## Testing Associations in Rails Console

Before running a local server and looking at the app in the browser, you want to be sure it works correctly in the console.

Test every association, if you want to see if a picture is properly associated to tags and comments try something like:

```ruby
picture = Picture.first
picture.tags
picture.comments
```

As long as calling those methods return something and do not throw an error, they are likely correctly implemented, depending on your seed data they may return a full or empty collection. Double check that the results you are seeing make sense given your seed data.

Adding macro's such as `has_many` gives your class many different methods. Check out the AR documentation here:

[🖇️ AR association macro auto-generated methods](http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html)

## Model Methods

Next, we need to add some methods on the models that we will use later to display data in the browser.  Again, use the console to test these methods.

#### `Tag.most_popular`

In your application you might want to recommend a user check out some popular tags to see what pictures other user's are posting. We can define popular tags as the tags *that are associated with the most pictures*. The goal is to define a class method `most_popular` that will return the 3 most popular tags

To begin, let's pull all the tags out of the database with `Tag.all`.  In the console try this out

```ruby
Tag.all.map do |tag|
  tag.pictures.length
end
```
Notice that you will see many SQL statements in the console. One SQL statement to get all the tags (`SELCT tags.* from tags`) and one SQL statement for calling the `pictures` method on each tag instance (`SELECT "pictures".* FROM "pictures" INNER JOIN "picture_tags" ON "pictures"."id" = "picture_tags"."picture_id" WHERE "picture_tags"."tag_id" = ?`).

Read the documentation for Ruby's sort_by and use it to return the three tag's with the most pictures.

[🖇️ Ruby Enumerable: sort_by Documentation](https://apidock.com/ruby/Enumerable/sort_by)

#### 🔎 Refactor Note:
 >This section provides details on how to optimize and refactor this method. First, get your implementation of the method working and move forward!  This would be something to revisit only if you have more time. For now, it is enough to understand why your implementation could stand to use some refactoring

 >Let's take a moment to assess the performance of this method. You make a SQL query to pull all the rows from the tags table out the db, your ORM (ActiveRecord) converts them into Ruby Objects, then you use a Ruby enumerable method to iterate through each object and for each one make an additional SQL query to find the associated records, which the ORM loads into memory and converts into objects.

 > With only a few tags in the database, this isn't too big of a load on your computer. But imagine an application like twitter or instagram with literally millions of hashtags.  That's a ton of SQL being run.  It would be much more efficient to make a *single* SQL query that gave us back the appropriate Tags.

 > Think back to all of the complex queries you used when first learning SQL like `JOINS`, `GROUP BY` and `HAVING`, or `COUNT`.  You can use ActiveRecord to make these type of queries! This will be tricky. Check some of these resources and ask an instructor for help if you have the time to tackle this refactoring.

>[🖇️ Order by Count of a has_many association](http://stackoverflow.com/questions/16996618/rails-order-by-results-count-of-has-many-association)

#### `Tag.trending`

The method above, `Tag.most_popular`,  will return the most popular tags of all time for your application. Your next task is to add a method `Tag.trending` that will return a trending tag.

This will be defined as the Tag *from the last 10 pictures* with the most comments.

Just like `.most_popular`, begin with a Ruby iterative approach and if you have time think about refactoring this method.

#### `User#received_comments`

A user instance will already have a method called `comments`, it's one of the methods that you get "for free" from ActiveRecord, when you write `has_many :comments`.

The `#comments` method runs some SQL that finds all the comment records that have the user instance's id as the foreign key `user_id`.  This returns a collection of **all the comments the user has made**.  Which is great information to know, as a user I should be able to know all the comments I have written.

Though, what if as a user I wanted to know **all the comments other users have made on my pictures.** There is not currently the functionality to do that. So, let's add it! Later, we'll use this method to show a logged in user what other users have been saying about their pictures.

Define an instance method for a user called `received_comments` that shows all the comments made on that user's pictures.

If you looked at each comment, you could check if the picture that comment belongs_to is one of the user's pictures.

#### 🔎 Refactor Note:
>Like before we can try to minimize the number of SQL queries this method makes, when you have some more time check out these resources.

>The way to approach this is to think about what do you want to get back from this method call-- a collection of Comment objects. That means you should be querying the comments table.

>Please do not let trying to refactor these methods halt your forward progress, keep pressing to get your MVP up and running.

> [🖇️ ActiveRecord where](https://apidock.com/rails/ActiveRecord/QueryMethods/where)

> [🖇️ ActiveRecord IN clause](http://stackoverflow.com/questions/10181864/ruby-activerecord-in-clause)

## One feature at a time

If everything works up to this point you have a functioning program... as long as your users don't mind opening up Rails Console to use your application :/

Let's get this working in the browser! The approach should be to tackle one feature at a time. If tasked to deliver 5 features, providing 3 totally working and 2 un-started is much better than 5 broken but half-completed features.

A Rails project has a ton of files and directories. The following may seem like silly advice to give, but as you navigate around to the different files be sure to keep the project tidy. Close folders when you have finished working on the files inside. Always knowing where to look and not being overwhelmed by the file structure will go a long way to streamlining the development process.

This reading will provide pretty in-depth instructions on the first few features, and leave things more up to you as you move forward.

## User Sign-up

Since all of the views will depend on if a user is signed in or not, this seems like a logical place to start.

Use a process called Error Driven Development to implement the user sign-up page. The goal will be to see an error then solve that error, which will lead to the seeing a new error, solve and repeat.

#### Step 1: The Router

Run a local server and visit *localhost:3000/signup*. You'll get an error that says **No route matches [GET] "/signup"**. Rails is actually super nice here and even tells you what needs to be done:

>You don't have any routes defined!

>Please add some routes in config/routes.rb.

The router's job is to match up a request to a controller action, it says, "If the request url matches this specific pattern go to this controller action and execute the code found there, if the request's route matches this other pattern go to this other controller action and do whatever is found there". When a GET request to 'signup' is received, send the request to an appropriate controller and action.

If you think about what the intended outcome is for signing **up**, it's another way of saying you want to create a new user. That's a straight-forward CRUD action on the `users` resource. So a request to '/signup' may get routed to the users controller. What is the controller action that generally indicates you intend to render a form to create a *new* resource?

#### Step 2: The Controller
If you properly set up the route, and revisit *localhost:3000/signup* you may see something like:
`uninitialized constant UsersController`. Solve that error and the next error will be:
`The action 'new' could not be found for UsersController`. Resolve that error and before refreshing the page, take a guess what the next error will be.  What does that indicate Rails is *implicitly* trying to do here?

[🖇️ Rendering by Default](http://guides.rubyonrails.org/layouts_and_rendering.html#rendering-by-default-convention-over-configuration-in-action)

#### Step 3: The View
The current error should say something about a *missing template*. Controller actions will always do one of two things, *render* something or *redirect* to another controller action that will then render something. If you do not tell a Rails controller action to do anything, it will, by default, try to render a file. Rails is very *opinionated* on what that file is looking for should be called and where it should be located. Since you are inside of the `Users` controller in an action called `new`, it will look for `views/users/new.html.erb`. Create that `views/users/new.html.erb` file, and then put something, (anything really, an `<h1>Hello</h1>`, for example) in it so we can confirm it works. If you see what you wrote, you're good to move on.

#### Step 4: The form

Since we are performing a CRUD action here, creating a user, `form_for` is the appropriate Rails form helper method to use.

If necessary, review how `form_for` works. What arguments does it take and how do we pass it the argument?

[🖇️ Rails Forms reading](https://github.com/learn-co-curriculum/rails-forms-readme)

[🖇️ Form_for lab](https://github.com/learn-co-curriculum/rails-form_for-lab)

[🖇️ Validations with form_for](https://github.com/learn-co-curriculum/validations-with-form_for-rails)

#### 🔎 Common Error Note:
> You may get an error here, something like: `undefined method 'users_path'`. This is a little bit of a cryptic error.  Where were you trying to call the method `users_path`?

> Remember that `form_for` is just a Rails helper method that writes a bunch of HTML for you. In Sinatra, for example, you had to write out the html for a form yourself. Recall that you would give the `<form>` element two properties 'method' and 'action'. 'method' referring to the HTTP verb (GET, POST) and action referring to the route or path the form will be submitted to.

> So, rails is using the `form_for` method to build some HTML and is trying fill out the 'action' property of the form with a certain path using another Rails helper method `users_path`.  The problem is that method doesn't exist because you don't have the route-- and that you know how to solve!  Add in the RESTful route for the POST request that should create a user

#### Step 5: Back in the Controller

The `new` controller action shows the user a form.  The `create` controller action is where that form will be submitted to and will process that request by persisting a new user to the database.

There's a few steps to this process. Start with:

```ruby
# app/controllers/users_controller.rb
def create
  binding.pry
end
```

Make sure you do, in fact, hit the pry, make sure the params have the correct fields.

Things to look out for when creating a user:
 - [🖇️ Strong Param Basics](https://github.com/learn-co-curriculum/strong-params-basics)
 - [🖇️ AR Validations](https://github.com/learn-co-curriculum/activerecord-validations-readme)
 - [🖇️ Handling Validations in Controllers](https://github.com/learn-co-curriculum/validations-in-controller-actions-rails)
 - Be sure that in addition to creating the new record, you set the set the user's id in the **session**.
 - [🖇️ Rails Sessions](http://guides.rubyonrails.org/action_controller_overview.html#accessing-the-session)


Upon successfully creating a user you should redirect to the user's show page. Add the appropriate route and controller action. For now, let's just display the user's username on the show page.

The application should acknowledge that the user has signed in.  At the top of the page, there should now be a link to sign out something indicating the currently signed in user. This brings us to the next step.

#### Step 6: Application Layout, `current_user`, `sessions#destroy`

Navigate to `app/views/layouts/application.html.erb`. Have you noticed how your view files may contain just a few lines of html, but when you inspect the page in the browser you see much more code, the head and body tags and all that? What you see here in the application layout file is **being rendered every time rails renders any view**. This file is *always* rendered and the html for the current view is inserted in where it says `yield`.  Try it out, put something above the yield. You will see it on every page of the app.

Your task is to create a navbar that is present on every view. Depending on if there is a current user signed or not in it should display different text and links. You can use conditionals to structure what the nav should look like.

Here is some pseudo code

```
<% if a user is signed in %>
  <%= Welcome, username %>
  <%= A link to sign out %>
<% else %>
  <%= A link to sign up %>
  <%= A link to sign in %>
<% end %>
```

This task may be easier by adding a helper method like `current_user` to the ApplicationController that will be accessible in all views and all controllers.

[🖇️ Rails Controller Helper Methods](https://apidock.com/rails/AbstractController/Helpers/ClassMethods/helper_method)

You'll want to implement the ability to destroy a session so you can test out that your conditional is working properly.

#### 🔎 Common Error Note:
> The rails `link_to` method returns the html that creates an `<a>` tag.  The default behavior of an `<a>` tag, when clicked, is to make a GET request to the route specified by the 'href' property.

> Here, we want a link that when clicked goes to the destroy action in the sessions controller. The route to destroy a session should be a DELETE request to '/sessions'. How can you make an `<a>` tag send a delete request?  Try Googling something like, "rails link_to delete request"  

[🖇️ Sessions Controller Reading](https://github.com/learn-co-curriculum/sessions_controller_readme)

You will need to add several routes (and controllers) to get this functioning properly. You may also want to add links to the pictures index page or tags index page that are always present.

It's ok to do the minimum amount of work necessary here to get the links on the page.  Make sure the links don't break the app and you have the corresponding routes and actions and you'll get to fleshing out the functionality later.

## User Sign In

The process detailed above of creating a `new` action which will show the user a form and a `create` action which will process the form submission will be very similar to what we do next for the process of a previously signed-up user signing **in**.

The key differences:
 - `form_for` is designed for Creating and Updating resources (the C & U of CRUD). A user signing in, while creating a session, isn't exactly a CRUD action.  In this case it will be better to use `form_tag`.

 [🖇️ form_tag Documentation](https://apidock.com/rails/ActionView/Helpers/FormTagHelper/form_tag)

 - Since CRUD actions are not being performed on the user resource, the routes for signing-in will be handled by the SessionsController not the UsersController

 - Setting a session will be done similarly to above, but first the information the user entered will have to be *authenticated*.  Basically, what's stored in the database in the `password_digest` column isn't the literal or plain-text password that a user entered when signing up, but the result of running that password through an encryption function. When a user re-enters their password to sign in, twhat they entered must be run through that same encryption function to see if the encrypted result matches the encrypted result stored in the `password_digest` column.  That's what bcrypt's `authenticate` method does for you.

 [🖇️ Bcrypt Reading](https://github.com/learn-co-curriculum/has_secure_password_readme)

## User Show Page

The User Show page is where a user should be redirected after signing in or up. This will be the most robust page of the application, with a lot of features.

It should include:

- All the user's pictures

- A form to create a new picture. The form should allow a user to associate the picture with one or several tags. A user should see a  list of checkboxes for tags that already exist in the database as well as an input field to create a new tag if desired. Creating a nested form is difficult, this may take some time.

#### 🔎 Refactor Note:
> The expression `user.pictures` should return a collection of that user-instance's pictures. If we wrote `user.pictures.class` what might you think ruby would tell us the class of that collection of picture objects is?

> 'Array' is a decent guess, that object certainly behaves like an Array, but it is actually an instance of the  ActiveRecord_Associations_CollectionProxy class.

> When objects are instances of a certain class, they get all the methods that class defines. The Collection_Proxy class is like a super-charged Array with a bunch of special methods.  You will actually be able to create new picture objects directly off of it with ActiveRecord's association builder methods. Read the documentation for these methods and see if you can clean up the code in your controller's that creates associated objects

> [🖇️ AR Association build](https://apidock.com/rails/ActiveRecord/Associations/CollectionProxy/build)

  > [🖇️ AR Association create](https://apidock.com/rails/ActiveRecord/Associations/CollectionProxy/create)

Using your model methods, display the following:

- A section that shows what other users have said about the current user's pictures.

-  The list of your application's most popular tags. Each item of this list should be a link to the Tag show page.

- The trending tag. This should also link to the Tag show page.


## Picture show

The picture show page should display the picture, the picture's tags as links to the tag show page, as well any comments associated with that picture.

It should also have a form to add a new comment.  It is totally fine to submit a form to the Comments Controller from the Pictures Show page.

## Picture Index

The Pictures Index page should display all of the pictures from all users with the most recent at the top.

## Tag Index and Show

The Tag index should display all the tag's in the database as links to the tag show page.

The tag show page should display all the pictures associated with that tag.

## Extensions

#### ❤️ Likes ❤️

Wouldn't it be awesome if users could *like* other photos. Conceptually, a "like" belongs to the user who added the like and to the photo it was left on.  What columns does this indicate should be on a `likes table`? When a user clicks a button to like a photo, what's the CRUD action they are performing?

<p class='util--hide'>View <a href='https://learn.co/lessons/rails-guided-project-immersive-module-two'>rails-guided-project-immersive-module-two</a> on Learn.co and start learning to code for free.</p>
