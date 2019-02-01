---
layout: post
title:      "Manipulating Data With Jquery and Ajax"
date:       2019-02-01 17:21:36 +0000
permalink:  manipulating_data_with_jquery_and_ajax
---


**Rails application** has a great and convenient way to route and render methods for each action: **Get, Post, Patch, Delete**. New message is routed with `"/messages/new"`, handled by method `"new"` in **MessagesController** and then rendered with `"new.html.erb"` templates. These same routines go for other actions as well. If you wish to render different temples, or give a route different name... all you need to do is to specify those actions in **routes.rb** file on your Rails application

```ruby
get 'login' => 'sessions#new'
post 'login' => 'sessions#create'
get 'signup' => 'users#new'
post 'signup' => 'users#create'
```	
	
However, one thing not so convenient about using rails default way to handle requests is that you always need to `redirect to a different page` or `render the template again` if that request is successful or even failed loaded. That would be a nightmare if  the internet isn't as fast as when you are at a starbucks. You get the idea. 

How do we still use the boilerplate rails conventions but not having to redirect or let our users wait for redirecting after each click request. This is when **Jquery and Ajax** come in.


To get a better idea of what is **AJAX** and **JQUERY** and how to use them, please visit their websites or check out these articles:

* [AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)
* [JQUERY](https://jquery.com/)

Basically,** AJAX** (***Asynchronous JavaScript And XML***) is a set of **Web development techniques** that are able to make quick, incremental updates to the user interface without reloading the entire browser page, while **JQUERY** is a ***Javascript library*** widely use to target elements, handle events, animations... These two works very well together, **Jquery** provides several methods for **AJAX** functionalities to develop dynamic and powerful web applications. Oh hey, **AJAX** is now mainly deal with **JSON** so please look more into how to get the respond in **JSON** and all the workarounds with it.

Today subject is to talk about how we handle 4 main actions: **Get, Post, Patch, Delete** in any Rails app with the help of **Jquery and Ajax**


As you know, once you *click on a button*, *submit a form* in a Rails app, you are telling rails to go to a specific route, then it looks for an assinged actions, and then a view page with requested information is displayed to user. All these steps are maybe very fast that users won't notice that they are being redirected to another page but sometimes the wait time isn't that unnoticeable. 

Let's take an example of using rails routing the traditional way. User requests to ***view a person's profile***. He/she will click on `"View Profile"`, which behind the scene the route is redirected to `"GET: users/1"`, then the application will look for `show method` and find on database to see if user 1 exists or not, if it does, view template `"show.html.erb" is rendered` but if not, user will get an error and `is redirected back` to the page where he/she was before. 


Sounds fun! Now let's do this and more of this with the help of **Jquery and Ajax**. 

Let's talk about the easiest and simplest action: the **GET**. We will view the messages that belong to the current user

```javascript
$('td.open-message').on('click', function (event) {
// prevent default action after click is fired
  event.preventDefault();
// we need user id and message id to see that  specific message
  let user_id = parseInt(this.dataset.user); // depends on how you set these information
  let message_id = parseInt(this.dataset.message);// I set it to data-attribute to not mess up with css and id targets
  let url = '/users/' + user_id + '/messages/' + message_id;
// go to that route and handle JSON response;
  $.get(url, function (message) {
// Use template like Handlebars is a great way to advoid hard-coded html chunk of code.
    let template = Handlebars.compile(document.getElementById("open-message-template").innerHTML);
    let result = template(message);
    $('div.messagedivs').html(result);
  })
}
```

The first thing to do to **stop RAILS default redirect action** is by calling `event.preventDefault()`; This built-in function will stop whatever action that is assigned to a click event. We then notice the route of viewing a message (belongs to a user) is `"users/1/messages/3"`. This can easily be achieved buy calling `rake routes` in the terminal:

`user_message GET    /users/:user_id/messages/:message_id(.:format)                                messages#show`

In the same click function, we will access these ids information which we set it in the target element, I set it to data-attribute to not mess up other class, id selectors later on. After having those information, it is time to manipulate the request and the returned data. One thing to keep in mind is that we have to have a regular view page work fine and instead of rendering html, it have to render **JSON**. That's how we can work with **AJAX**.

To load message data from the server using a **HTTP GET** request., we can use Jquery [GET](https://api.jquery.com/jquery.get/) method:

`$.get("users/1/messages/3, function (message) { //handle returned data}`


The beauty of **AJAX** is that we can open a web page, get data and play around with it without actually opening any page at all. The **message variable** returned by **Jquery GET method** is exactly what will be render if we open that page.

With the returned data, we can parse information into a HTML template and append or replace the target area of the page with those information. 

![Open message without refreshing page](https://i.imgur.com/ctdmCqE.gif)


**Posting** is pretty similar to **GET**. We get the route first:

`posts GET    /posts(.:format)                                            posts#index`

```ruby
 //Handle CREATING A NEW POST on home page
  $('form#new_post').submit(function (event) {
      event.preventDefault();
      let values = $(this).serialize(); //Jsonize the data before sending it to POST method
      let template = Handlebars.compile(document.getElementById("post-template").innerHTML)
      let posting = $.post('/posts', values);
      posting.done(function (data) {
        let post = new Post(data);
        let result = template(post);
        $(".homepage_feeds").prepend(result);
      })
  });
```

With **GET**, you handle the **JSON** data return by server, with **POST** on the other hand , you send the **JSON** data to server to create a new item. In order to do that, you either need to have JSON data ready before sending to **POST** method `$.post(url, data(JSON type)`or you can JSONize your data by calling `.serialize()` method then pass it to POST.  Please read more about [Serialize](https://api.jquery.com/serialize/) here. 

After create a new post, you will have option to display that post on the posts index page or ignore it. To display it, using the `done` method. This method is saying after successfully create a new post,  it sends the **JSON** data return by the server to you. You can parse that data into HTML templates and display it on your show page: 

```ruby
$.post('/posts', values).done(function (data) {
        let post = new Post(data);
        let result = template(post);
        $(".homepage_feeds").prepend(result);
      })
```

![New Post](https://i.imgur.com/gMskqBQ.gif)

**PATCH** and **DELETE** is little more different. Since **JQuery** *doesn't support Patch and Delete*, we need to use low level **AJAX** method to handle these requests.

Similar concept is applied to **Patch** and **Delete**. With **Patch**, You need to provide a url and send new data (**JSON**) to the server to request updating an object, while with **Delete** you only need to provide an url in order to do so.

To **update a post**:

```ruby
function editpost() {
  let content = $('.postcontent').val();
  let id = $('.postcontent').data().id;
  $.ajax({
      url: "/posts/" + id,
      type: 'PATCH',
      data: {'content': content}, //send new data to update an existing post
      success: function (data) {
        $('.post-' + id).html(data.content);}
    })
  }
```

![Edit a post](https://i.imgur.com/8iYhV4z.gif)

To** delete a post**:

```ruby
//Handle DELETING A POST request 
  $('a.delete_post').click(function (event) {
    event.preventDefault();
    event.stopPropagation();
    let path = this;
    $.ajax({
          url: path.href,
          type: "DELETE",
          success: function () {
          $(path).parents(".post_parent").remove();
          }
			})
    })
	```
	
![Delete Post](https://i.imgur.com/dA6TxoZ.gif)
	
Pretty straightforward right? Please read more on how to use **Patch** and **Delete** with **Ajax** because it can be more complecated depends on each situation.
	
I use [Sweetalert](https://sweetalert2.github.io/) to override the default **data-confirm** option of a web page. It is a very nice touch to improve the user interface of a Rails Application. Please check out my github repository for this [WETEXT social application project](https://github.com/nhinhdao/wetext-friends-talk-project) to see how I implemented it. 

Thanks for reading.

I hope you all very productive days ahead :)
	
	
