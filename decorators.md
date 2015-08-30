# Decorator pattern

The [decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern) is
used to wrap an object and extend its functionality without modifying the
object itself. It's similar to the presenter and adapter patterns, which also
wrap an object (or multiple objects), but provides a different functionality
compared to those two patterns.

In Rails, the decorator pattern is generally used to format
view-specific data, as well as handling simple view logic. The decorator pattern
is preferred to using helpers for the following reasons:

1. It's more object oriented - you can use inheritance and encapsulation
2. The helper methods are globally available in views, and not tied to any object
3. Instead of forwarding an object to the helper, you call the method on the object itself

# Examples
### 1. Using SimpleDelegator

[SimpleDelegator](http://ruby-doc.org/stdlib-2.2.3/libdoc/delegate/rdoc/SimpleDelegator.html)
is a Ruby class that provides means to easily delegate all method calls to an object passed
to the constructor. A simple implementation of a decorator using SimpleDelegator would look 
something like this:

``` ruby
class UserDecorator < SimpleDelegator
  def full_name
    first_name + " " + last_name
  end

  def birthday
    return "Private" if birthday_private?
    birthday.strftime("%d %b %Y")
  end
end
```

Controller:

``` ruby
class UserController < ApplicationController
  def show
    user = User.find(params[:id])
    @user = UserDecorator.new(user)
  end
end
```

View:

``` ruby
= @user.full_name
= @user.birthday
```

### 2. Using Draper

[Draper](https://github.com/drapergem/draper/) is a gem that simplifies creating 
decorators and adds some additional sugar on top of the SimpleDelegator decorators.

One of the benefits of using draper is that it provides the view context inside of the
decorator, so you can easily use view-specific methods in your decorator. This isn't
something too desireable, so make sure to only use it for simple conditional renders.

``` ruby
class UserDecorator < Draper::Decorator
  # Using decorates_associaton always returns a decorated object or a collection
  # when calling the association on the already decorated object, e.g. user.comments
  decorates_association :comments

  # You can delegate either specific methods to the underlying object, or use delegate_all
  # to delegate all methods sent to the decorator to the underliying object
  delegate :first_name, :last_name

  def view_profile
    return "Private" if profile_private?
    h.link_to "View", user_profile_path(object)
  end
end
```

Controller:

``` ruby
class UserController < ApplicationController
  # decorates_assigned creates a helper method with the decorated object
  # which you can use instead of the plain object stored in the instance variable
  decorates_assigned :user

  def show
    @user = User.find(params[:id])
  end
end
```

View:

``` ruby
# Calling a method on a helper method that contains the decorated object,
# created by decorates_assigned
= user.view_profile

# Calling a method on the underocated object
= @user.first_name

```

Be sure to read the documentation, since Draper offers a lot more than what's
been shown here.

# Further reading

* [7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)
* [Refactoring Fat Models with Patterns by Bryan Helmkamp](https://www.youtube.com/watch?v=5yX6ADjyqyE) - A talk going through all the patterns from the 7 Patterns to Refactor Fat ActiveRecord Models blog post
* [Evaluating Alternative Decorator Implementations](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in) - Some other ways to implement the decorator pattern
* [What I dislike about Draper](http://thepugautomatic.com/2014/03/draper/) - A critique of Draper
