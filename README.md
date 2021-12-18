# FactoryTrace

[![CircleCI][1]][2]
[![Gem Version][3]][4]

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'n1_loader'
```

You can add integration with [ActiveRecord][5] by:
```ruby
require 'n1_loader/active_record'
```

You can add the integration with [ActiveRecord][5] and [ArLazyPreload][6] by:
```ruby
require 'n1_loader/ar_lazy_preload'
```

## Usage

```ruby
class Example
  include N1Loader::Loadable

  # with inline loader
  n1_loader :anything do |elements|
    # Has to return a hash that has keys as element from elements
    elements.group_by(&:itself)
  end

  # with custom loader
  n1_loader :something, MyLoader
end

# Custom loader that can be shared with many classes
class MyLoader < N1Loader::Loader
  # Has to return a hash that has keys as element from elements
  def perform(elements)
    elements.group_by(&:itself)
  end
end
```

### ActiveRecord

[ActiveRecord][5]

```ruby
class User < ActiveRecord::Base
  include N1Loader::Loadable
  
  n1_loader :orders_count do |users|
    hash = Order.where(user: users).group(:user_id).count
    
    # hash has to have keys as initial elements
    hash.transform_keys! { |key| users.find { |user| user.id == key } }
  end
end

# For single user
user = User.first
user.orders_count 

# For many users without N+1
User.limit(5).includes(:orders_count).map(&:orders_count)

# or with explicit preloader
users = User.limit(5).to_a
N1Loader::Preloader.new(users).preload(:orders_count)

# No N+1 here
users.map(&:orders_count)
```

### ArLazyPreload

[ArLazyPreload][6]

```ruby
class User < ActiveRecord::Base
  include N1Loader::Loadable
  
  n1_loader :orders_count do |users|
    hash = Order.where(user: users).group(:user_id).count
    
    # hash has to have keys as initial elements
    hash.transform_keys! { |key| users.find { |user| user.id == key } }
  end
end

# For single user
user = User.first
user.orders_count

# For many users without N+1
User.lazy_preload(:orders_count).all.map(&:orders_count)
# or 
User.preload_associations_lazily.all.map(&:orders_count)
# or 
ArLazyPreload.config.auto_preload = true
User.all.map(:orders_count)
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and the created tag, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/djezzzl/n1_loader. 
This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [code of conduct](CODE_OF_CONDUCT.md).

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the N1Loader project's codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](CODE_OF_CONDUCT.md).

## Changelog

*FactoryTrace*'s changelog is available [here](CHANGELOG.md).

## Copyright

Copyright (c) Evgeniy Demin. See [LICENSE.txt](LICENSE.txt) for further details.

[1]: https://circleci.com/gh/djezzzl/n1_loader/tree/master.svg?style=shield
[2]: https://circleci.com/gh/djezzzl/n1_loader/tree/master
[3]: https://badge.fury.io/rb/n1_loader.svg
[4]: https://badge.fury.io/rb/n1_loader
[5]: https://github.com/rails/rails/tree/main/activerecord
[6]: https://github.com/DmitryTsepelev/ar_lazy_preload
