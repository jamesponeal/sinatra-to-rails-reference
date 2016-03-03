
#Sinatra to Rails Routes Reference Sheet

###Introduction:

Rails is an MVC web framework created to give Ruby a way to establish a web presence easily and efficiently.  For someone like me who learned Sinatra first, one of the the things that stood out to me the most about Rails is the different syntax and procedure involved in creating routes.

This document is intended to provide a basic comparison between creating and managing routes in Rails compared to Sinatra using a application with two tables: categories and listings, where categories have many listings, and listings belongs to category.

###Getting Started with Rails:

This reference sheet assumes that you have already successfully set up a rails skeleton with all required gems installed, and modules and helpers included where they need to be.

One of the major differences in handling routing in a Rails application compared to Sinatra is the use of a routes.rb file.  In the routes.rb file you will set your application home page (root), and also use a 'resources' command, which makes the standard REST paths available to you.  To get started, create the home page as shown below:

```
root 'categories#index'
```

In the routes.rb file you would also create a resources command for the table names.  For our 'categories and listings' example, where a category has many listings and our listings belongs to a category, the resources command would look like this:

```
resources: categories do
  resources: listings
end
```

Now that this is set up, if you type the command 'rake routes' from your command line, you will get a summary of the routes that have been created for you.  It will look similar to this:


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

The basic formula for creating a Rails route is by using the command shown in the Prefix column followed by the word 'path'.  For example, if you want to create a path to create a new category, your link should use 'new_category_path', which then triggers the 'new' method to be called in your categories controller.

Rails greatly streamlines the standard CRUD paths into simple single line path names and controller methods.





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
```
get '/categories/new' do
  @category = Category.new
  erb :"category/new"
end
```

#####Rails:
```
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
```
post '/categories/new' do
  @category = Category.new(name: params[:name])
  if @category.save
    redirect '/categories/#{@category.id}'
  else
    @errors = @post.errors.full_messages
    erb :"/category/new"
  end
end
```

#####Rails:
```
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

#### Show
Clicking on the category name makes a get request to the server which finds the following route in categories_controller.rb:

#####Sinatra:
```
get '/categories/:id' do
  @category = Category.find(params[:id])
  @listings = @category.listings
  erb :"category/show"
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
```
def show
  @category = Category.find(params[:id])
  @listings = @category.listings
end
```






#### Edit













def create
  @category = Category.new(category_params)
    if @category.save
      redirect_to @category
    else
      render 'new'
    end
end

def edit
  @category = Category.find(params[:id])
end

def update
  @category = Category.find(params[:id])
  if @category.update(category_params)
    redirect_to @category
  else
    render 'edit'
  end
end

def destroy
  @category = Category.find(params[:id])
  @category.destroy
  redirect_to categories_path(@category)
end






