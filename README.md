# `acts_as_favoritor` - Add Favorites to your Rails app

<img src="https://travis-ci.org/slooob/acts_as_favoritor.svg?branch=master" /> [![Gem Version](https://badge.fury.io/rb/acts_as_favoritor.svg)](https://badge.fury.io/rb/acts_as_favoritor)

`acts_as_favoritor` is a Rubygem to allow any ActiveRecord model to associate any other model including the option for multiple relationships per association with scopes.

You are able to differentiate followers, favorites, watchers, votes and whatever else you can imagine through a single relationship. This is accomplished by a double polymorphic relationship on the Favorite model. There is also built in support for blocking/un-blocking favorite records as well as caching.

---

## Table of Contents

* [Installation](#installation)
* [Usage](#usage)
    * [Setup](#setup)
    * [`acts_as_favoritor` methods](#acts_as_favoritor-methods)
    * [`acts_as_favoritable` methods](#acts_as_favoritable-methods)
    * [`Favorite` model](#favorite-model)
    * [Scopes](#scopes)
    * [Caching](#caching)
* [Configuration](#configuration)
* [Testing](#testing)
    * [Test Coverage](#test-coverage)
* [To Do](#to-do)
* [Contributing](#contributing)
    * [Contributors](#contributors)
* [License](#license)

---

## Installation

`acts_as_favoritor` works with Rails 4.0 onwards. You can add it to your `Gemfile` with:

```ruby
gem 'acts_as_favoritor'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install acts_as_favoritor

If you always want to be up to date fetch the latest from GitHub in your `Gemfile`:

```ruby
gem 'acts_as_favoritor', github: 'slooob/acts_as_favoritor'
```

Now run the generator:

    $ rails g acts_as_favoritor

You can set your default scope by passing `--scope custom_scope`. Learn more about scopes [here](#scopes).

If you specify `--cache true`, `acts_as_favoritor` will be using cache. Learn more about caching [here](#caching).

You can skip the creation of a configuration file by passing `--skip_configuration`. Learn more about configuring `acts_as_favoritor` [here](#configuration).

To wrap things up, migrate the changes into your database:

    $ rails db:migrate

**Note:** Use `rake db:migrate` instead if you run Rails < 5.

This will create a Favorite model as well as a migration file.

## Usage

### Setup

Add `acts_as_favoritable` to the models you want to be able to get favorited:

```ruby
class User < ActiveRecord::Base
    acts_as_favoritable
end

class Book < ActiveRecord::Base
    acts_as_favoritable
end
```

Specify which models can favorite other models by adding `acts_as_favoritor`:

```ruby
class User < ActiveRecord::Base
    acts_as_favoritor
end
```

### `acts_as_favoritor` methods

```ruby
book = Book.find 1
user = User.find 1

# `user` favorites `book`.
user.favorite book

# `user` removes `book` from favorites.
user.remove_favorite book

# Whether `user` has marked `book` as his favorite. Returns `true` or `false`.
user.favorited? book

# Total number of favorites by `user`.
user.favorites_count

# Returnes `user`'s favorites that have not been blocked as an array of `Favorite` records.
user.all_favorites

# Returns all favorited objects of `user` as an array (unblocked). This can be a collection of different object types, e.g.: `User`, `Book`.
user.all_favorited

# Returns an array of `Favorite` records where the `favoritable_type` is `Book`.
user.favorites_by_type 'Book'

# Returns an array of all favorited objects of `user` where `favoritable_type` is 'Book', this can be a collection of different object types, e.g.: `User`, `Book`.
user.favorited_by_type 'Book'

# Returns the exact same result as `user.favorited_by_type 'User'`.
user.favorited_users

# Total number of favorited books by `user`.
user.favorited_by_type_count 'Book'

# Returns the exact same result as `user.favorited_by_type_count 'Book'`.
user.favorited_books_count

# Returns the Arel scope for favorites.
# This does not return the actual favorites, just the scope of favorited including the favoritables, essentially: `book.favorites.unblocked.includes(:favoritable)`.
book.favorites_scoped

# Block a favoritable
user.block book

# Unblock a favoritable
user.unblock book

# Whether `user` has blocked `book`. Returns `true` or `false`.
user.blocked? book

# Returns an array including all blocked Favoritable records.
user.blocks

# Total number of `user`'s favorites blocked.
user.blocked_favoritables_count

# Whether `user` has favorited, not favorited or blocked `book`. Returns `true`, `nil` or `false`.
user.favoritable_type book
```

These methods take an optional hash parameter of ActiveRecord options (`:limit`, `:order`, etc...)

    favorites_by_type, all_favorites, all_favorited, favorited_by_type

### `acts_as_favoritable` methods

```ruby
# Returns all favoritors of a model that `acts_as_favoritable`
book.favoritors

# Returns the Arel scope for favoritors. This does not return the actual favoritors, just the scope of favorited records including the favoritors, essentially:  `book.favorited.includes(:favoritors)`.
book.favoritors_scoped

# Total number of favoritors.
book.favoritors_count

# Returns an array of records with type `User` following `book`.
book.favoritors_by_type 'User'

# Returns the exact same as `book.favoritors_by_type 'User'`.
book.user_favoritors

# Total number of favoritors with type `User`.
book.favoritors_by_type_count 'User'

# Returns the exact same as `book.favoritors_by_type_count 'User'`.
book.count_user_favoritors

# Whether `book` has been favorited by `user`. Returns `true` or `false`.
book.favorited_by? user

# Block a favoritor
book.block user

# Unblock a favoritor
book.unblock user

# Whether `book` has blocked `user` as favoritor. Returns `true` or `false`.
book.blocked? user

# Returns an array including all blocked Favoritor records.
book.blocks

# Total number of `book`'s favoritors blocked.
book.blocked_favoritors_count

# Whether `user` has favorited, not favorited or blocked `book`. Returns `true`, `nil` or `false`.
book.favoritor_type user
```

These methods take an optional hash parameter of ActiveRecord options (`:limit`, `:order`, etc...)

    favoritors_by_type, favoritors, blocks

### `Favorite` model

```ruby
# Scopes
## Returns all `Favorite` records where `blocked` is `false`.
Favorite.unblocked
## Returns all `Favorite` records where `blocked` is `true`.
Favorite.blocked
## Returns an ordered array of the latest create `Favorite` records.
Favorite.descending

# Returns all `Favorite` records in an array, which have been created in a specified timeframe. Default is 2 weeks.
Favorite.recent
Favorite.recent 1.month.ago

# Returns all favorites of `user`, including those who were blocked.
Favorite.for_favoritor user

# Returns all favoritors of `book`, including those who were blocked.
Favorite.for_favoritable book
```

### Scopes

Using scopes with `acts_as_favoritor` enables you to Follow, Watch, Favorite, [...] between any of your models. This way you can separate distinct functionalities in your app between user states. For example: A user sees all his favorited books in a dashboard (`'favorite'`), but he only receives notifications for those, he is watching (`'watch'`). Just like YouTube or GitHub do it. Options are endless. You could also integrate a voting / star system similar to YouTube or GitHub

By default all of your favorites are scoped to `'favorite'`.

You can create new scopes on the fly. Every single method takes `scope` as an option which expexts an array containing your scopes as strings.

So lets see how this works:

```ruby
user.favorite book, scope: [:favorite, :watching]
user.remove_favorite book, scope: [:watching]
second_user = User.find 2
user.favorite second_user, scope: [:follow]
book.block user, scope: [:all] # applies to all scopes
```

That's simple!

When you call a method which returns something while specifying multiple scopes, the method returns the results in a hash with the scopes as keys:

```ruby
user.favorited? book, scope: [:favorite, :watching] # => { favorite: true, watching: false }
user.favorited? book, scope: [:all] # => true
```

`acts_as_favoritor` also provides some handy scopes for you to call on the `Favorite` model:

```ruby
# Returns all `Favorite` records where `scope` is `my_scope`
Favorite.send(my_scope + '_list')

## Examples
### Returns all `Favorite` records where `scope` is `favorites`
Favorite.favorite_list
### Returns all `Favorite` records where `scope` is `watching`
Favorite.watching_list
### Very unnecessary, but `all_list` returns literally all `Favorite` records
Favorite.all_list
```

### Caching

When you set the option `cache` in `config/acts_as_favoritor` to true, you are able to cache the amount of favorites/favoritables an instance has regarding a scope.

For that you need to add some database columns:

*acts_as_favoritor*

```ruby
add_column :users, :favoritor_score, :text
add_column :users, :favoritor_total, :text
```

*acts_as_favoritable*

```ruby
add_column :users, :favoritable_score, :text
add_column :users, :favoritable_total, :text
add_column :books, :favoritable_score, :text
add_column :books, :favoritable_total, :text
```

Caches are stored as hashes with scopes as keys:

```ruby
user.favoritor_score # => { favorite: 1 }
user.favoritor_total # => { favorite: 1, watching: 1 }
second_user.favoritable_score # => { follow: 1 }
book.favoritable_score # => { favorite: 1 }
```

**Note:** Only scopes who have favorites are included.

---

## Configuration

The installer creates a yaml config file for you (`config/acts_as_favoritor.yml`). Inside of that file you are able to adopt this gem to your specific needs.

When you delete this file, all settings will rollback to their defaults. You can prevent the creation of this configuration file when running the generator by passing the option `--skip_configuration`.

Currently supported Settings:

**default_scope:** Specify your default scope. Learn more about scopes [here](#scopes).

**cache:** Whether `acts_as_favoritor` uses cache or not. Learn more about caching [here](#caching).

If you have an idea for a new setting, propose it by creating a new [issue](https://github.com/slooob/acts_as_favoritor/issues).

---

## Testing

Tests are written with Shoulda on top of `Test::Unit` with Factory Girl being used instead of fixtures. Tests are run using rake.

1. Fork this repository
2. Clone your forked git locally
3. Install dependencies

    `$ bundle install`

4. Run tests

    `$ rake test`

### Test Coverage

Test coverage can be calculated using SimpleCov. Make sure you have the [simplecov gem](https://github.com/colszowka/simplecov) installed.

1. Uncomment SimpleCov in the Gemfile
2. Uncomment the relevant section in `test/test_helper.rb`
3. Run tests

    `$ rake test`

---

## To Do

* Adding magic methods for scopes, e.g.: `user.follow second_user` instead of `user.favorite second_user, scope: [:follow]`

---

## Contributing

We hope that you will consider contributing to `acts_as_favoritor`. Please read this short overview for some information about how to get started:

[Learn more about contributing to this repository](https://github.com/slooob/acts_as_favoritor/blob/master/CONTRIBUTING.md), [Code of Conduct](https://github.com/slooob/acts_as_favoritor/blob/master/CODE_OF_CONDUCT.md)

### Contributors

Give the people some :heart: who are working on this project. See them all at:

https://github.com/slooob/acts_as_favoritor/graphs/contributors

## License

MIT License

Copyright (c) 2017 Jonas Hübotter

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
