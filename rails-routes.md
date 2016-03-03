
#Routes Reference - Rails vs Sinatra

#####Resources:
http://guides.rubyonrails.org/getting_started.html <br />
http://guides.rubyonrails.org/routing.html


##Introduction:

Rails is an MVC web framework created to give Ruby a way to establish a web presence easily and efficiently.  For someone like me who learned Sinatra first, one of the the things that stood out to me the most about Rails is the different syntax and procedure involved in creating routes.

This document is intended to provide a basic comparison between creating and managing routes in Rails compared to Sinatra using a application with two tables: categories and listings, where categories have many listings, and listings belongs to category.

This reference sheet assumes that you have already successfully set up a rails skeleton with all required gems installed, and with modules and helpers included where they need to be.

##Getting Started with Rails:

One of the major differences in handling routing in a Rails application compared to Sinatra is the use of a routes.rb file.  In the routes.rb file you will set your application home page (root), and also use a 'resources' command, which makes the standard REST paths available to you.  To get started, create the home page as shown below:

``` ruby
root 'categories#index'
```

In the routes.rb file you would also create a resources command for the table names.  For our 'categories and listings' example, where a category has many listings and our listings belongs to a category, the resources command would look like this:

``` ruby
resources: categories do
  resources: listings
end
```

###Rake Routes

Now that this is set up, it's time to learn the most valuable command for planning routes in Rails.  If you type the command ```rake routes``` from your command line, you will get a summary of the routes that have been created for you.  If you retain nothing else from this... remember ```rake routes```.  The print-out in your terminal will look something like this:


```
               Prefix Verb   URI Pattern                                          Controller#Action

    category_listings GET    /categories/:category_id/listings(.:format)          listings#index
                      POST   /categories/:category_id/listings(.:format)          listings#create
 new_category_listing GET    /categories/:category_id/listings/new(.:format)      listings#new
edit_category_listing GET    /categories/:category_id/listings/:id/edit(.:format) listings#edit
     category_listing GET    /categories/:category_id/listings/:id(.:format)      listings#show
                      PATCH  /categories/:category_id/listings/:id(.:format)      listings#update
                      PUT    /categories/:category_id/listings/:id(.:format)      listings#update
                      DELETE /categories/:category_id/listings/:id(.:format)      listings#destroy
           categories GET    /categories(.:format)                                categories#index
                      POST   /categories(.:format)                                categories#create
         new_category GET    /categories/new(.:format)                            categories#new
        edit_category GET    /categories/:id/edit(.:format)                       categories#edit
             category GET    /categories/:id(.:format)                            categories#show
                      PATCH  /categories/:id(.:format)                            categories#update
                      PUT    /categories/:id(.:format)                            categories#update
                      DELETE /categories/:id(.:format)                            categories#destroy
```

The 'rake routes' command in the terminal is the key to knowing what routes are available to you in Rails, how to call them, and how they relate to the RESTful Sinatra routes you're used to.

Rails greatly streamlines the standard CRUD paths into easy-to-read path names and simple controller methods.  The basic formula for creating a Rails route is by using the command shown in the Prefix column followed by the word 'path'.  For example, if you want to create a path to create a new category, your link should use ```new_category_path``` which then triggers the 'new' method to be called in your categories controller.

Any path that includes any form of ```:id``` in the URL also needs some data to be passed along with it.  For example, if you want to utilize a route that will show the details of a specific category, you will need to pass the category along with the path as shown:
```category_path(category, listing)```

###Nested Routes

Nested routes can be tricky in both Sinatra and Rails, but the rules are very similar.  Follow the paths laid out by the 'rake routes' command, and pass in the objects that are needed.  One important thing to remember is that any path dealing with a nested path will utilize the controller of the nested table.

For example, to edit a listing that belongs to a category, both the category and listing must be passed along with the route, and the method call will happen on the listing controller.

```<%= link_to 'Edit Listing', edit_category_listing_path(@category, @listing) %>```

Clicking on this link will make a get request to the server, and call an 'edit' method in a file called 'listings_controller.rb', which will also render a view called 'edit.html.erb' in the 'views/listings' folder.  Rails magic... it's a beautiful thing.


##Example Routes for Categories and Listings:

###Route to Category Index Page:

Route to generate the categories index page (categories_controller.rb):

#####Sinatra:
``` ruby
get '/categories' do
  @categories = Category.all
  erb :index
end
```

#####Rails:
``` ruby
def index
  @categories = Category.all
end
```


###Route to Create a New Category:

Clicking the 'Create New Category' link from the index page makes a get request to the server which will find the following route in categories_controller.rb:

#####Sinatra:
``` ruby
get '/categories/new' do
  @category = Category.new
  erb :"categories/new"
end
```

#####Rails:
``` ruby
def new
  @category = Category.new
end
```

This sends back the '/category/new' file to be rendered by the browser where a form should be rendered to take in user inputs to create a new category.  Example forms could be something as follows:

#####Sinatra:
```
<form method="post" action="/categories/new">
  A bunch of cool html formsy stuff with a fancy-pants submit button goes here.
</form>
```

#####Rails:
```
<%= form_for @category do |f| %>
  <p>
    <%= f.label :name %><br>
    <%= f.text_field :name %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
```

After the user enters the required information and clicks the submit button, this makes a post request to create the category object and save it to the database.  In Sinatra, the post action must be explicitly called out.  In Rails, the path from the submit of the form to the create method in the categories controller is 'automatically' connected due to the resource command in the routes.rb file.

#####Sinatra:
``` ruby
post '/categories/new' do
  @category = Category.new(name: params[:name])
  if @category.save
    redirect '/categories/#{@category.id}'
  else
    @errors = @post.errors.full_messages
    erb :"/categories/new"
  end
end
```

#####Rails:
``` ruby
def create
  @category = Category.new(category_params)
  if @category.save
    redirect_to category_path
  else
    render 'new'
  end
end
```


###Routes to Show, Edit, and Delete a Category:

Back on the index page, there are a list of categories.  The category name is a link to the 'show' route for categories.  The view file for this index page could look something like this:

#####Sinatra:
```
<% @categories.each do |category| %>
    <a href="<%= "/categories/#{category.id}" %>"><%= category.name %></a>
    <a href="<%= "/categories/#{category.id}/edit" %>">Edit</a>
    <form method="post" action="/categories/<%=category.id%>">
      <input type="hidden" name="_method" value="delete">
      <input type="submit" value="Delete" class="button">
    </form>
<% end %>
```

#####Rails:
```
<% @categories.each do |category| %>
    <%= link_to category.name, category_path(category) %>
    <%= link_to 'Edit', edit_category_path(category) %>
    <%= link_to 'Destroy', category_path(category), method: :delete, data: { confirm: 'Are you sure?'} %>
<% end %>
```

#### _*Show*_
Clicking on the category name makes a get request to the server which finds the following route in categories_controller.rb:

#####Sinatra:
``` ruby
get '/categories/:id' do
  @category = Category.find(params[:id])
  @listings = @category.listings
  erb :"categories/show"
end
```

Which renders a view containing the listings, that looks something like this:
```
<% @listings.each do |listing| %>
  <%= listing.price %>
  <a href="/categories/<%=@category.id%>/listings/<%= listing.id %>"><%= listing.name %></a>
  <a href="/categories/<%=@category.id%>/listings/<%= listing.id %>/edit"
  <form method="post" action="/categories/<%=@category.id%>/listings/<%= listing.id %>">
    <input type="hidden" name="_method" value="delete">
    <input type="submit" value="Delete Entry" class="button">
  </form>
<% end %>
```

#####Rails:
``` ruby
def show
  @category = Category.find(params[:id])
  @listings = @category.listings
end
```

Which renders the categories/show view, that might look something like this:

```
<%= @category.name %>
<%= link_to "Create A New Listing", new_category_listing_path(@category) %><br>
<table>
  <tr>
    <td><u>Item</u></td>
    <td><u>Price</u></td>
  </tr>
  <% @category.listings.each do |listing| %>
    <tr>
      <td><%= link_to listing.title, category_listing_path(@category, listing) %></td>
      <td><%= listing.price %></td>
    </tr>
  <% end %>
</table>
```

#### _*Edit*_

From our index page, clicking on the 'Edit' link will make a get request to the server which follows the following route in the categories controller:

#####Sinatra:

``` ruby
get '/categories/:id/edit' do
  @category = Category.find(params[:id])
  erb :'categories/edit'
end
```

Which renders a view for editing the category name along with any additional information the user should be editing.  Clicking the 'submit' button will send a PUT request to the server to update the category via the following path:

``` ruby
put '/categories/:id' do
  @category = Category.find(params[:id])
  @category.assign_attributes(params[:category])
  if @category.save
    redirect '/categories'
  else
    erb :errors
  end
end
```

#####Rails:

In Rails, clicking the edit link will trigger the edit method in the controller file to be called.

``` ruby
def edit
  @category = Category.find(params[:id])
end
```

Through Rails magic, the controller knows to then render the 'edit' view in the view/categories folder. Clicking the 'submit' button will send a PUT request to the server to update the category via the following controller method:

``` ruby
def update
  @category = Category.find(params[:id])
  if @category.update(category_params)
    redirect_to category_path(@category)
  else
    render 'edit'
  end
end
```

#### _*Delete*_

Once again starting from our index page, clicking on the 'Delete' link will make a delete request to the server which follows the following route in the categories controller:

#####Sinatra:

``` ruby
delete '/categories/:id' do
  @category = Category.find(params[:id])
  @category.destroy
  redirect '/categories'
end
```

#####Rails:

``` ruby
def destroy
  @category = Category.find(params[:id])
  @category.destroy
  redirect_to categories_path
end
```

##Conclusion

If you've made it this far, thanks for reading and I hope you learned something.  If you want to read more, check out the resources at the top of this document, or refer to the [Ruby on Rails Guides](http://guides.rubyonrails.org/).
