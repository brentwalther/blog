---
layout: default
title: Advanced Constraint Techniques for Rails Routing
category: code
---
Rails routes constraints are extremely useful for setting up many different routing configurations based on the state of specific requests or the state of your application. You could, for example, constraint a set of routes to a particular subdomain or restrain users from accessing your app's standard routes if your application is in "beta mode" because it's not yet ready for the general public.

For more advanced constraints, it's most convenient to use the last form of rails constraints where you create a class and give an instance of it as a parameter. A simple example would be to check if the user is an administrator (assuming you've defined this attribute on your User model):
{% highlight ruby %}
class IsAdministratorConstraint
  def matches?(request)
  request.env['warden'].user.admin? rescue false
  end
end

Rails.application.routes.draw do
  get '/admin(/*other)', to: 'admin#index',
    constraints: IsAdministratorConstraint.new
end
{% endhighlight %}

This would work for a single route but these advanced constraints can also be used with ruby block syntax to specify more than one route that should be constrained:
{% highlight ruby %}
Rails.application.routes.draw do
  constraints(IsAdministratorConstraint.new) do
    get '/admin', to: 'admin#index'
    get '/foo',   to: 'foo#bar'
  end
end
{% endhighlight %}

Great! This becomes very useful because it means we can constrain any arbitary set of routes with a single constraint. Let's say, for example, you'd like your app to have a landing page mode where users are directed to a landing page if the mode is set to `true` but administrators can still access the app. We can define a new constraint that takes a parameter for whether the routes should be constrained to beta mode or non beta mode:

{% highlight ruby %}
class IsBetaModeRoute
  def initialize(is_beta_mode_route)
    @is_beta_mode_route = is_beta_mode_route
  end

  def matches?(request)
    # always let administrators through
    return true if IsAdministrator.new.matches?(request)

    if @is_beta_mode_route
      # the following line could be any mechanism to test wether your app's beta mode is set
      request.env['site'].beta_mode_enabled?
    else
      !request.env['site'].beta_mode_enabled?
    end
  end
end
{% endhighlight %}

and then we could use it like this:

{% highlight ruby %}
Rails.application.routes.draw do
  constraints(IsBetaModeRoute.new(true)) do
    get '/', to: 'pages#splash_page'
  end
  
  constraints(IsBetaModeRoute.new(false)) do
    get '/foo',   to: 'foo#bar'
    
    resources :model
  end
end
{% endhighlight %}

Cool! We can constrain different sets of routes for certain parameters. Here's a big problem I ran into however: "How can I use multiple constraint classes to constrain a set of routes?"

Well, if you've done research I'm sure you've figured out that you can only specify a single constraint for each route. That's a problem if you have multiple constraints that you'd like to apply. I, for example, wanted to ensure that two different modes were disabled on my application for a set of routes.

##Using multiple constraints in rails routing
I fumbled with this idea longer than I'd like to admit. In the end however, the solution to this was simple. Let's say you have multiple constraint classes that all respond to `matches?`. Assuming you'd like to use more than one, you can write some simple "logical operator" constraints to take a list of other constraints and then apply the logical operation. In my case, I wanted all constraints to be satisfied so I created an `AndConstraint` that took a list of constraints and "ANDed" them all. It looked like this:

{% highlight ruby %}
class AndConstraint
  def initialize(constraints = [])
    @constraints = constraints
  end

  def matches?(request)
    matches = true
    @constraints.each do |constraint|
      matches &&= constraint.matches?(request)
    end
    matches
  end
end
{% endhighlight %}

and this worked very well. In my routing, I simple initialized this constraint with a list of other constraints that all should be met. Here's an example of what it looked like:

{% highlight ruby %}
Rails.application.routes.draw do
  constraints(
    AndConstraint.new([
      FooConstraint.new(true),
      BarConstraint.new])) do
      
    get '/', to: 'foo#bar'
  end
end
{% endhighlight %}

and this served it's purposes well. With this strategy, I can use multiple advanced constraint classes to constraing a single (or set of) rails routes. While I only created an `AndConstraint`, you could also create an `OrConstraint` with the same logic.

Some may argue that it would be better to build a single constraint class that performed all the logic internally but in my case I found that to be counter productive. Many of my constraints took parameters and I wanted to be able to mix and match these constraints with various parameters. This solution did the trick. Multiple constraints!

Have a different way of doing it? [Email me.](mailto: brentwalther@gmail.com)
