# Facade pattern

The [facade pattern](http://c2.com/cgi/wiki?FacadePattern) is used to provide
a unified interface from multiple different interfaces.

In Rails, we can use the facade pattern to collect data from multiple sources
to display inside views, as well as format the data for display.

# Examples

A good example for using the facade pattern is some kind of dashboard, on which
we need to show data from multiple different sources.

``` ruby
# app/facades/dashboard_facade.rb
class DashboardFacade
  attr_reader :user

  def initialize(user)
    @user = user
  end

  def certificates
    @certificates ||= current_user.certificates
  end

  def nearly_expired_certificates
    @nearly_expired_certificates ||=
      certificates.nearly_expiring.order(expiry_date: :asc)
  end

  def suppliers
    @suppliers ||= user.connected_suppliers
  end
end
```

Controller:

``` ruby
class DashboardsController < ApplicationController
  def show
    @dashboard = DashboardFacade.new(current_user)
  end
end
```

View:

``` ruby
= render 'certificates', certificates: @dashboard.certificates
= render 'certificates', certificates: @dashboard.nearly_expired_certificates
= render 'suppliers', suppliers: @dashboard.suppliers
```

# Further reading
[Facade pattern](http://c2.com/cgi/wiki?FacadePattern)
[Sandi Metz Rules For Developers](https://robots.thoughtbot.com/sandi-metz-rules-for-developers)
