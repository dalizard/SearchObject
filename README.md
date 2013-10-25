# SearchObject

Provides DSL for creating search objects

## Installation

Add this line to your application's Gemfile:

    gem 'search_object'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install search_object

## Usage

Just include the ```SearchObject.module``` and define your search options:

```ruby
class MessageSearch
  include SearchObject.module

  scope { Message }

  option :name          { |scope, value| scope.where :name => value }
  option :created_at    { |scope, dates| scope.created_between(dates)}
  option :opened, false { |scope, value| value ? scope.unopened : scope.opened }
end
```

Then you can just search the messages scope:

```ruby
search = MessageSearch.new(params[:filters])

# accessing option values
search.name                        # => name option
search.created_at                  # => created at option

# accessing results
search.count                       # => number of results found
search.results?                    # => is there any results found
search.results                     # => results found

# params for url generations
search.params                     # => returns the option values
search.params(:opened => false)   # => overwrites the 'opened' option
```

## Plugins

```SearchObject``` support plugins, which are passed to the ```SearchObject.module``` method.

Plugins can be also added as ```include SearchObject::Plugin::PluginName```. If you wan't your own plugins just add them under ```SearchObject::Plugin``` module.

### Paginate plugin

Really simple paginte plugin, which uses ```.limit``` and ```.offset``` methods.

```ruby
class ProductSearch
  include SearchObject.module(:paging)

  scope { Product }

  option :name
  option :category_name

  # you are required to overwrite this method
  def per_page
    10
  end
end

search = ProductSearch.new(params[:filters], params[:page]) # page number is required
search.page                                                 # => page number
search.results                                              # => paginated page results
```

### Model plugin

Extends your search object with ActiveModel, so you can use it in rails forms

```ruby
class ProductSearch
  include SearchObject.module(:model)

  scope { Product }

  option :name
  option :category_name
end
```

```erb
# in some view:
<%= form_for ProductSearch.new do |form| %>
  <% form.label :name %>
  <% form.text_field :name %>
  <% form.label :category_name %>
  <% form.text_field :category_name %>
<% end %>
```


## Tips & Tricks

### Passing scope as argument

``` ruby
class ProductSearch
  include SearchObject.module

  scope :name
end

# first arguments is treated as scope (if no scope option is provided)
search = ProductSearch.new(Product.visible, params[:f])
search.results #=> products
```


### Default search option

```ruby
class MessageSearch
  include SearchObject.module

  scope { Message }

  scope :name # automaticly applies => { |scope, value| scope.where name: value }
end
```

### Handling nil options

```ruby
class MessageSearch
  include SearchObject.module

  scope { Message }

  # nil values returned from option blocks are ignored
  scope :started { |scope, value| scope.started if value }
end
```

### Using instance method in options

```ruby
class MessageSearch
  include SearchObject.module

  scope { Message }

  option :date { |scope, value| scope.by_date parse_dates(value) }

  private

  def parse_dates(date_string)
    # some 'magic' method to parse dates
  end
end
```

### Overwriting methods

We can have fine grained scope, by overwriting ```initialize``` method:

```ruby
class MessageSearch
  include SearchObject.module

  scope :subject
  scope :category

  def initialize(user, filters)
    super Message.for_user(user)
  end
end
```

Or we can add simple pagination by overwriting both ```initialize``` and ```fetch_results``` (used for fetching results):

```ruby
class MessageSearch
  include SearchObject.module

  scope { Message }

  scope :subject
  scope :category

  attr_reader :page

  def initialize(filters = {}, page = 0)
    super filters
    @page = page
  end

  def fetch_results
    super.paginate @page
  end
end
```

## Code Status

[![Code Climate](https://codeclimate.com/github/RStankov/SearchObject.png)](https://codeclimate.com/github/RStankov/SearchObject)

[![Build Status](https://secure.travis-ci.org/RStankov/SearchObject.png)](http://travis-ci.org/RStankov/SearchObject)

[![Code coverage](https://coveralls.io/repos/RStankov/SearchObject/badge.png?branch=master)](https://coveralls.io/r/RStankov/SearchObject)

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
