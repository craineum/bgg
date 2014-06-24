bgg gem [![Build Status](https://travis-ci.org/jemiahlee/bgg.svg)](https://travis-ci.org/jemiahlee/bgg) [![Code Climate](https://codeclimate.com/github/jemiahlee/bgg.png)](https://codeclimate.com/github/jemiahlee/bgg)
===========

An object-oriented API for interacting with the [boardgamegeek](http://boardgamegeek.com) [XML Version 2 API](http://boardgamegeek.com/wiki/page/BGG_XML_API2).

This is based on a fork of earlier work I had done on
[bgg-api](http://github.com/bhardin/bgg-api), along with Brett and a few
other contributors.

## Versions supported

Please note that this code uses Ruby 1.9 hash syntax and thus does not
support versions earlier than that.

## Installing the Gem

Do the following to install the  [bgg gem](http://rubygems.org/gems/bgg) Ruby Gem.

Add the following to your `Gemfile`:

```ruby
gem "bgg"
```

Then run `bundle install` from the command line:

    bundle install

## Using the Gem

Require the gem at the top of any file you want to use.

    require 'bgg'

You can either use the low level basic api and then parse the XML document in a way that suits your needs,
or you can use the specialized methods

There are object types and subtypes for many of the APIs documented at
[BGG_XML_API2](http://boardgamegeek.com/wiki/page/BGG_XML_API2), but not all of them are implemented.
Everything around a user and their game collection should work, as well
as generalized searching for board games.

### Requesting Data

There are a few ways to request data from the api.

#### Simple requests

BggApi is the root entry point to the different api calls.

```ruby
BggApi.collection 'username' # Default call 
BggApi.collection('username', { brief: 1 }) # Adding params based on api documentation
```

#### Predefined requests

These are here to make it so you do not need to know the bgg xml api. To
get the same as above.

```ruby
Bgg::Request::Collection.board_games('username').brief.get
Bgg::Request::Collection.board_games('username', { brief: 1 }).get # You can still pass params here if you want
```

#### The long way

If you would like to pass around the request objects, this is a longer
method to do so.

```ruby
my_request = Bgg::Request::Collection.new('username')
my_request.add_params({ brief: 1 })
my_collection = my_request.get
```

### Working with Results

Each api method has it's own data structure, although there is some
common themes.

There is a possibilty that there may be data for
one item and not another if the original bgg record is missing it.  For
instance a user has not rated an item in their collection.  In these
cases we return nil.

Another possible reason to get nil is if you pull a data item from the
result set when there is a needed request param.  For example, to get
the user_rating for a collection item you need to specify stats: 1, or
all_fields.

#### Enumerated Objects

Most results return an enumerated root object (Although not all).
The child object will always inherit from Bgg::Result::Item.

```ruby
my_collection.count
my_collection.first.user_rating  # Returns a collection item object
```

#### Custom methods

Weather or not they are enumerated all have their own attributes and/or methods.

```ruby
my_collection.played # Returns array of played items
```

#### XML always available

Sometimes it is easier to pull out what you want specifically then
to try and get only what the objects provide.  So if you need it
the XML is always available at any level.  Since we are using Nokogiri
this will enable it's methods and return values to you.

```ruby
my_collection.xml.xpath('items/item/stats/@minplayers') # All items minimum number of players
my_collection.first.xml.xpath('stats/ranks/rank/@value') # All rankings for an item
```

### Collection

Objects for collection

#### Bgg::Request::Collection

```ruby
request = Bgg::Request::Collection.board_games('username', { params: hash })
request = Bgg::Request::Collection.board_game_expansions('username', { params: hash })
request = Bgg::Request::Collection.rpgs('username', { params: hash })
request = Bgg::Request::Collection.video_games('username', { params: hash })
request.all_fields # Adds params to the bgg call that will request all data possible, returns self
request.brief # Adds params to the bgg call that will request a smaller subset of data, returns self
request.add_params( { params: to_add } ) # Adds params to the bgg call
result = request.get # Execute bgg call and return result
```

#### Bgg::Result::Collection

These might return nil if missing data or request params.  See [Working with Results](#working-with-results)

* Methods
  * owned, items marked as owned
  * played, items that have at least one play
  * user_rated, items that have a user_rating
  * user_rated 5, items that have a value of 5 (with decimal places dropped) 
  * user_rated 3..5, items that are between 3 and 5 (with decimal places dropped) 

#### Bgg::Result::Collection::Item

These might return nil if missing data or request params.  See [Working with Results](#working-with-results)

* Attributes
  * Array: theme_ranks (array of Rank objects)
  * Floats: average_rating, bgg_rating, user_rating
  * Integers: collection_id, id, own_count, play_count, play_time, type_rank, year_published
  * Range: players (minimum..maximum players)
  * Strings: comment, image, name, thumbnail, type
  * Time: last_modified
* Methods
  * Booleans: for_trade?, owned?, played?, preordered?, published?,
    wanted? want_to_buy?, want_to_play?

#### Bgg::Result::Collection::Item::Rank

* Attributes
  * Float: rating
  * Integers: id (ID of theme), rank
  * String: title (Title of theme)

Contributing to bgg
-----------------------

* Fork the project
* Start a feature/bugfix branch
* Test whatever you are committing. Ensure this test is a specification
  of the behavior of the functionality, not just an error case or
  success case.
* Commit and push until you are happy with your contribution
* Submit a pull request. Squash commits that should be squashed (it is
  not necessary for you to have just one commit to your pull request,
  but have each commit be a logical piece of work.)

Copyright
---------

Copyright (c) 2014 [Jeremiah Lee](https://github.com/jemiahlee), [Brett Hardin](http://bretthard.in), and [Marcello Missiroli](https://github.com/piffy). See LICENSE.txt for further details.

