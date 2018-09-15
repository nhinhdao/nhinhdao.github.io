---
layout: post
title:      "Sinatra Social Web Application"
date:       2018-09-13 21:41:15 -0400
permalink:  sinatra_social_web_application
---


![Home Page](https://i.imgur.com/A6WoZas.png?2)

Building a simple **sinatra web application** isn't hard but it sure will take time and effort to end up having a site that is interactive and informative.

I want to build a website that mimics the connection between **users** (as **user** and **friends**) and their messages (I hope to make *real-time conversation* soon). 

From the few first lines of code, I already bumped into some headache. Think about relationship between them:
**User** `has_many` **Friends**, sounds good. But **Friend** `has_many` **Users**? or `belongs_to` a **User**? doesn't sound right. After all **Friend** is just another **User**. 

Since **User** and **Friend** both belong to **User** class, they can't have a `has_many` or `belongs_to` relationship as usual.  This situation requires me to create a medium class: **Friendship** that both belongs to **User** and **Friend** and also doesn't violate the rule : **Friend** is an **User**. Luckily, thanks to **ActiveRecord**, there is an attribute call `:class_name` to assign `User class` for **Friend** class:

```ruby
class User < ActiveRecord::Base
  has_many :messages
  has_many :friendships
  has_many :friends, through: :friendships
end

class Message < ActiveRecord::Base
  belongs_to :friend, class_name: 'User'
  belongs_to :user
end

class Friendship < ActiveRecord::Base
  belongs_to :user
  belongs_to :friend, class_name: 'User'
end
```

Things are getting easier. I can just work on it like how I work with regular Ruby **ActiveRecord::Relation**. 
But as things go along, more and more problems come along as well. 

```ruby
>> message_test = Message.all.first
+----+----------------------------------------+---------+-----------+
| id | content                                | user_id | friend_id |
+----+----------------------------------------+---------+-----------+
| 1  | Make sure to bring some food for lunch | 1       | 2         |
+----+----------------------------------------+---------+-----------+
```


Message belongs to a **User** and a **Friend**, hence it has a `user_id` and a `friend_id`.  `user_id` is who composed the message, `friend_id` is who the message was sent to. But `friend_id` is just another `user_id`. How can a message has **2 user_ids**. This causes a problem that only `message_test.user` has this message in record :

```ruby
>> message_test.user
+----+-------------+-----------------------+--------------------------------------------------------------+
| id | username    | email                 | password_digest                                              |
+----+-------------+-----------------------+--------------------------------------------------------------+
| 1  | Hannah Lala | hannah.lala@gmail.com | $2a$10$oKR.B5GJGSDGr6r6WutSauyho01ssoHTDXJCdiuPdeXXXPwbdIeZK |
+----+-------------+-----------------------+--------------------------------------------------------------+


>> message_test.user.messages
+----+----------------------------------------+---------+-----------+
| id | content                                | user_id | friend_id |
+----+----------------------------------------+---------+-----------+
| 1  | Make sure to bring some food for lunch | 1       | 2         |
+----+----------------------------------------+---------+-----------+
```

but the friend, with id = 2 **won't have it **

```ruby
>> message_test.friend
+----+-------------+-----------------------+--------------------------------------------------------------+
| id | username    | email                 | password_digest                                              |
+----+-------------+-----------------------+--------------------------------------------------------------+
| 2  | Allan Candy | allan.candy@gmail.com | $2a$10$8.g0TXsDPp.QwNxVAuPgYOG01FTqk3.FARtFlrPXGYO.vlhHwZXyW |
+----+-------------+-----------------------+--------------------------------------------------------------+

>> message_test.friend.messages
+----+--------------------------------------------------------------------------------------------+---------+-----------+
| id | content                                                                                    | user_id | friend_id |
+----+--------------------------------------------------------------------------------------------+---------+-----------+
| 4  | Damn it. It is raining tomorrow. It ruins my plan man                                      | 2       | 7         |
| 5  | Wanna go biking this Sunday? The weather is mad nice tho. We better go before it gets cold | 2       | 8         |
+----+--------------------------------------------------------------------------------------------+---------+-----------+

>> message_test.friend.messages.find_by_id(message_test.id)
=> nil
```

User's message.all array only contains the messages were composed by them, not the messages were sent to them.

This is what I have to deal with as I want to use only one mother class to build more than 1 other classes. 

But it is not the end of the world.

With a few thoughts, I can just manipulate it as any other ruby arrays/objects. Remember, all messages are stored in Message.all array.

To get the messages that were composed by current user

```ruby 
my_messages = current_user.messages
```

or 
```ruby 
my_messages = Message.all.select {|m| m.user == current_user}
```

To get the messages that were sent to current user

```ruby 
messages_from_friends = Message.all.select {|m| m.friend == current_user}
```

![](https://i.imgur.com/MfwFp6t.png?1)

Similarly, **Friendship** belongs to a **User** and a **Friend** (another **User**), a `Friendship.new` can only have 1 **user_id** and 1 **friend_id**, not **2 user_ids**. 

Same problem arises, when I make friend with another user:

```ruby
 params[:friends].each do |i|
				a = User.find_by_id(i)
				Friendship.create(user_id: current_user.id, friend_id: a.id)
end
```
				
only my friend_list has him, his friend_list doesn't has me

For 2 people both become friend with each other at the same time, I need to make 2 instances of **Friendship** class:

```ruby
post '/users/create_friends' do
    params[:friends].each do |i|
				a = User.find_by_id(i)
				Friendship.create(user_id: current_user.id, friend_id: a.id)
				Friendship.create(user_id: a.id, friend_id: current_user.id)
				flash[:notice] = "Successfully connected to #{a.username}."
    end
    redirect '/users/friends'
end
```

![Friend_list](https://i.imgur.com/2hITe8a.png?1)

Those big tangles were combed but there are also lots of minor problems as you all face when you program. 

I think all we need to do is give it some times and do some researchs, they will become fixable.

If you are interested in how my first web application works, feel free to clone my github repository at: [Sinatra Messages Transfer Project](https://github.com/nhinhdao/sinatra-messages-transfer-project)

Remember to do as instruction in README.md. 

I would love to hear feedback any suggestion from all of you on my first sinatra site.

Thanks all
