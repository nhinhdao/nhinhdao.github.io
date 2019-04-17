---
layout: post
title:      "The Message Book"
date:       2018-11-20 18:12:47 -0500
permalink:  the_message_book
---


**The Message Book** is an upgraded version of the [sinatra-messages-transfer-project](https://nhinhdao.github.io/sinatra_social_web_application). It is a web application which allows **users** to **make friends**, **send messages** to each other. This version, however, I added **posts** and was built with **Ruby on Rails**.

For major details and logics behind this app, please visit my github project page [rails-messages-transfer-project](rails-messages-transfer-project) or review my [sinatra-messages-transfer-project](https://nhinhdao.github.io/sinatra_social_web_application) post. 

I want to highlight some of the new features and the useful gems I've used in this project.

First of all, **user** can now choose to log in through a third website. Because it is a simple web application, I limit  user to only using **Facebook**. Basically, together with signing up as a new **user** which means more accounts to maintain, one can just log in through **facebook** and **facebook** will send the **token** uniquely assigned to each account back to the web application. This token is then stored in the web application database and is used to set retrive user's informations.

For the steps to install needed gems and intergrate **facebook omniauth** to a rails project, please visit [omniauth-facebook](https://github.com/mkdynamic/omniauth-facebook).

When a user provide his/her informations to log in through **facebook**, behind the scene **facebook** will send the **token** back to the web application by mapping the action to the route provided in `routes.rb` file: `get '/auth/facebook/callback' => 'sessions#create'`

```ruby
def create
    if params[:username].present? && params[:password].present? #user logged in through form provided in the home page
      @user = User.find_by(:username => params[:username])
      if @user && @user.authenticate(params[:password])
        session[:current_user_id] = @user.id
        flash[:message] = "Welcome back, #{@user.username}"
        redirect_to '/'
      else
        flash[:notice] = "Hmm, We can't find you. Sorry, please try again!"
        render :new
      end
    elsif request.env['omniauth.auth'].present? #user logged in through facebook
      @user = User.find_or_create_by(uid: auth['uid']) do |u|
        u.username = auth['info']['name']
        u.email = auth['info']['email']
        u.image = auth['info']['image']
      end
      session[:current_user_id] = @user.id
      flash[:message] = "Thank you for checking in, #{@user.username}"
      redirect_to '/'
    else #missing or wrong log in identities
      flash[:notice] = "Hmm, We can't find you. Sorry, please try again!"
      render :new
    end
  end
```

With **regular log in**, the controller will check if the provided credentials matches with the user's stored data to log them in or throw an error if not. **Facebook login**, on the other hand, the application won't do anything until **user** logs in successfully through **facebook**. Application only deals with the **token** that **facebook** sends back, not how **user** log into **facebook**. If they fail to log in, bring them back to the log in page so they can try again or sign up for a new account. If **user** log in with **facebook** successfully, their profile's information will be pulls from **facebook** to replace the missing credentials requires: **username**, **image**, **email**,...

In order to do that, a **method** provided  by the omiauth provider is added to **sessions_controller.rb** file:
```
def auth
    request.env['omniauth.auth']
  end
```

Then getting user's profile is pretty straightforward:

```
@user = User.find_or_create_by(uid: auth['uid']) do |u|
        u.username = auth['info']['name']
        u.email = auth['info']['email']
        u.image = auth['info']['image']
      end
```

Next one, I want to mention is the use of **Faker** gem to generate seed files. For a local web application, there won't by any **users**, **posts**, **messages** or **activities** until one is created so it is necessary to create some data so a new user will have better experience.

**Faker** gem is a very handy one to do this job. Please check out [stympy/faker](https://github.com/stympy/faker) to install it into your project.

Here is how my **seeds.rb** file looks like:

```ruby
10.times do |account|
  username = Faker::Internet.unique.username
  email = Faker::Internet.unique.email(username)
  image = Gravatar.new("#{email}").image_url + '?d=wavatar'
  password = 'nhinh12345'
  User.create(username: username, email: email, image: image, password: password, password_confirmation: password)
end


User.all.each do |user|
  pcontent = Faker::GameOfThrones.quote
  user.posts.create(content: pcontent)
end


friends = User.all[1..5]
for fr in friends do
  mcontent1 = Faker::Community.quotes
  mcontent2 = Faker::SiliconValley.quote
end
```

**Faker** gem helps you generate **username**, **email**, **quote**, **address**, **telephone**... mostly everything you need to identify an object. It will save you tons of time on typing, thinking what unique names and emails to use...

Beside **Faker** gem, I added **Gravatar** to generate **avatar** for **user**, which make your site looks more lively and closer to a social application. Install [gravatar](https://github.com/sinisterchipmunk/gravatar) is simple.

You can choose **type of avatar** by added  `+ '?d=<type of icon>'` at the end of the line of code. There are a few types including: wavata, identicon, monsterid, and default> For example:

```ruby
user.image = Gravatar.new("#{email}").image_url + '?d=identicon'

or 

<img src="<%= Gravatar.new("#{email}").image_url + '?d=wavatar' %>" style="width: 80px">
```


* identicon ![identicon](https://i.imgur.com/N5sTOMS.png?1) 
* wavata ![wavatar](https://www.gravatar.com/avatar/ee4d1b570eff6ce63c7d97043980a98c?default=wavatar&forcedefault=1)        
* monsterid ![monsterid](https://www.gravatar.com/avatar/ee4d1b570eff6ce63c7d97043980a98c?default=monsterid&forcedefault=1)    
* default ![default](https://www.gravatar.com/avatar/ee4d1b570eff6ce63c7d97043980a98c?forcedefault=1)

One more useful gem for those who wants to make their pages readable if they have long content pages is [will_paginate](https://github.com/mislav/will_paginate). 

Just need to add **gem 'will-paginate'** to your **gemfile**, then decide how many item you want to display in one page then simply paste it into your code:

```ruby
@feeds = Post.paginate(page: params[:page], per_page: 4)
```

and add <%= will_paginate @feeds %> to your erb view file. Easy peasy.

![will-paginate](https://i.imgur.com/CzaHCD1.png)

Lastly, a very impotant gem I want to add here if you are heavily work with **sql** and **model**. Please please please add **gem 'hirb'** to your **gemfile**. Then in the console, type in **'Hirb.enable'**. Now look how much different it makes

> before

![before Hirb](https://i.imgur.com/G9Vh0N5.png)

> after

![after Hirb](https://i.imgur.com/9qqNjow.png)

Wow, how much more clear, readable and understandable form 1 extra line of code. Thanks [Gabriel Horner](https://rubygems.org/profiles/cldwalker) for this awesome gem. I always got a hard time reading those tangling lines to see all the **emails**, **usernames**...  but with **Hirb** everything is clearly displayed in one table. It is so much faster than writing extra codes to grab them down the line. 

Hmm, I think that's good for now. All highlights of my project are explained briefly here to you. Thank you for going this far. I will constantly looking for **helpful gems** and **meaningful ideas** to build more and more projects. Exploring the **Ruby on Rails** world is fun and challenging. I hope you find fun with this language too.

Have a productive day, friends!

Stay tuned.

