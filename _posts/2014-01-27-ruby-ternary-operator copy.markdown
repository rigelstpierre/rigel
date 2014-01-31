---
layout: post
title: "Ruby Ternary Operator"
date:  2014-01-27
---

One of favorite things I've learned about in the last 6 months has been the Ternary Operator. Often time I'll find myself writing a simple `if statement` It would look something like this:

{% highlight ruby %}
def is_user_an_admin?
  if current_user.has_role?(:admin)
    @user = 'something awesome'
  else
    @user = 'something not so awesome'
  end
end
{% endhighlight%}

When it's a really simple method for checking if the `current_user`is an admin why not make your 7 line method much simpler using the ternary operator. Something like this:


{% highlight ruby %}
def is user_an_admin?
  @user = current_user.has_role?(:admin) ? 'something awesome' : 'something not so awesome'
end
{% endhighlight %}

It's simple and straight forward.
