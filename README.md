# Rails Dynamic Request

## Review

You already know how to create a static request, which is where you create a page that doesn't take any parameters and simply renders a view, an example would be: `localhost:3000/about`. For Rails to process this request, the `route.rb` file contains a route such as:

```ruby
get 'about', to: "static#about"
```

This is mapped to the `static` controller and `about` action, which renders the `about.html.erb` view template.

## Dynamic Requests

If you are comfortable with static requests you will pick up on dynamic requests quickly. If we,  according to REST, wanted to get the post with the id of `42` we would go to `/posts/42`. So you could create a new line in your routes file for each post... but that would get ridiculous and you would have to modify your web server every time someone posts. So! we have dynamic routes. A breakdown of what is happening is below:

1. The `routes.rb` file takes in the request and processes it like normal, except this time it also parses the `42` as a parameter and passes it to the posts' controller.

2. From that point the controller that you write will parse the `42` parameter and run a query on the Post model, storing the result in an instance variable.

3. Lastly the controller passes the instance variable to the associated view which renders that specific post record details to the client.

In review, what's the difference between static and dynamic routes?

* Static routes render pages that have a hard coded path connected to them, for example the `/welcome` path will always show the `welcome` page.

* Dynamic routes will render different data based on the parameters passed to the route. For example the `/posts/42` route will render the `post` data for `post`: `42`, whereas `/posts/222` will render the `post` data for record with id `222`.


## Code Implementation

In order to setup a dynamic request feature we will start by writing a test to verify that the page exists:

```ruby
# spec/features/post_spec.rb

require 'rails_helper'

describe 'navigate' do
  before do
    @post = Post.create(title: "My Post", description: "My post desc")
  end

  it 'to post pages' do
    visit "/posts/#{@post.id}"
    expect(page.status_code).to eq(200)
  end
end
```

Running `rspec` gives us an expected error of: `ActionController::RoutingError: No route matches [GET] "/posts/1"`. To correct this error let's draw a route that maps to a show action in the posts' controller:

```ruby
get 'posts/:id', to: 'posts#show'
```

Here you will notice something that's different from the static route, the `/:id` tells the routing system that this route can receive a parameter, and that parameter will be passed to the controller's show action. With this route in place let's run our tests again, you will see we get a different failure this time that says: `AbstractController::ActionNotFound: The action 'show' could not be found for PostsController`. This means that we need to create a corresponding `show` action in the posts' controller, let's get this failure fixed with the code below:

```ruby
# app/controllers/posts_controller.rb

class PostsController < ApplicationController
  def show
  end
end
```

Running the tests now you will get a failure saying that that we have a missing view template, let's fix that by creating a `posts` folder in the `view` folder and create a `show.html.erb` file in the new `views/posts` directory.

Running the tests now and we're all green, which means that the request will be properly routes through the controller and view and returns a HTTP status code of `200`.

If you start the Rails server and navigate to `/posts/1` or anything `post` record the router will know what you're talking about, however the controller needs to be told what to do with the `id`.

Now that we have the routing configured, let's build a test (this version currently has the test already written in)

to see if the post content is rendered on the show page with the title being in an `h1` tag and the description in a `p` tag, below are the scenarios:

```ruby
# spec/features/post_spec.rb

require 'rails_helper'

describe 'navigate' do
  before do
    @post = Post.create(title: "My Post", description: "My post desc")
  end

  it 'shows the title on the show page in a h1 tag' do
    visit "/posts/#{@post.id}"
    expect(page).to have_css("h1", text: "My Post")
  end

  it 'to post pages' do
    visit "/posts/#{@post.id}"
    expect(page.status_code).to eq(200)
  end
end
```

This gives us a failure that says: `expected to find css "h1" with text "My Great Post" but there were no matches`. We first need to get the id sent by the user through the dynamic URL. This variable is passed into the controller in a hash called `params`. Since we named the route `:id`, the `id` will be the value for the `:id` key. In this case that means it will be in `params[:id]`, let's set that up here:

```ruby
# app/controllers/posts_controller.rb

def show
  @post = Post.find(params[:id])
end
```

In this line, our show action is running a database query on the Post model that will return a post that has the id that matches the route parameters, and it will store this record in the ```@post``` instance variable and make it available to the ```show.html.erb``` file. Let's get our spec passing by placing the post's title on the show view template:

```ERB
<% # app/views/posts/show.html.erb %>
<h1><%= @post.title %></h1>
```

We're back to all green! Now let's implement the description spec:

```ruby
it 'shows the description on the show page in a p tag' do
  visit "/posts/#{@post.id}"
  expect(page).to have_css("p", text: "My post desc")
end
```
This will give us a failure since there are no matches on the template yet, to implement this fix, update the view:

```ERB
<% # app/views/posts/show.html.erb %>
<h1><%= @post.title %></h1>
<p><%= @post.description %></p>
```

Now we're all passing, you now know how to create dynamic routes in Rails! However we would be remiss if we didn't follow the full "Red, Green, Refactor" TDD workflow. There are a few elements of the application that can be refactored. Instead of doing that long drawn out `get` route in our `routes.rb` file, we can use ruby's RESTful defaults and use the resources method. Problem is, we only have one of the seven RESTful routes. Thankfully, if we pass in another option, the only option we can choose which of the seven RESTful routes we care about. In this case we only care about the `show` action.

Remove:

```ruby
get 'posts/:id', to: 'posts#show'
```

Replace with:

```ruby
resources :posts, only: :show
```

We will go into detail on what the `resources` method does in a future lesson, for right now just know that it entails the seven key RESTful routes, and in this case we are only wanting it to make the `show` action available.

Running the tests for a final time you can see that they're all still passing, nice work!

<a href='https://learn.co/lessons/rails-dynamic-request-readme' data-visibility='hidden'>View this lesson on Learn.co</a>

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/rails-dynamic-request-readme'>Rails Dynamic Request</a> on Learn.co and start learning to code for free.</p>

<p class='util--hide'>View <a href='https://learn.co/lessons/rails-dynamic-request-readme'>Rails Dynamic Request</a> on Learn.co and start learning to code for free.</p>
