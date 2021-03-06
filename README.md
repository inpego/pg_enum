# PgActiveRecordEnum

Integration of PostgreSQL native enums with ActiveRecord enums.

## Motivation

1.	All the activerecord enum's abilities out of the box.
2.	All schema definitions are placed inside migrations, not inside models. That makes code cleaner and eliminates the possibility to accidentally change model's enum definition even if it's defined via hash.
3.	All the enums changes may be followed by migrations timestamps. If enum is defined inside model, it's changes may be investigated only through source control, which is far less convenient.
4.	Performance degrade could hardly be noticed, 'cause PostgreSQL uses four-byte integers to provide enums internally.
5.	Data validation is carried out by database itself. That makes it impossible to write inconsistent values via `update_column` or `update_all` method calls, which is an issue when using original activerecord enums, or even working with the database directly.
6.	The last but not least, all the enum's values are displayed with their original values when working with the database via console or GUI client. No need to look into the code base to find out which number stands for which value.


## Installation

Add this line to your application's Gemfile:

```ruby
gem 'pg_activerecord_enum'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install pg_activerecord_enum

## Usage

Migration:

```ruby
class CreateFruits < ActiveRecord::Migration[5.1]
  def change
    create_table :fruits do |t|
      t.enum :fruit_type, values: %i[banana orange grape], allow_blank: true, null: false, default: 'banana', index: true
      t.timestamps
    end
  end
end
```

Or for existing table:

```ruby
class CreateFruits < ActiveRecord::Migration[5.1]
  def change
    add_enum :fruits, :color, values: %i[red green yellow orange], allow_blank: true
  end
end
```

**allow_blank** adds empty value to enum, other options are default options for column.

**Note:** migration rollback will remove enums only if other tables do not depend on them.

Model:

```ruby
class Fruit < ApplicationRecord
  pg_enum :fruit_type
  pg_enum :color, _prefix: true
end
```

```ruby
Fruit.banana.count # 0
banana = Fruit.create
banana.banana? # true because default
banana.orange! #
banana.orange? # true
Fruit.orange.count # 1
banana.color_orange!
banana.color # "orange"
```

In PostgreSQL DB:
```
# select * from fruits;
 id | fruit_type |         created_at         |         updated_at         | color  
----+------------+----------------------------+----------------------------+--------
  3 | orange     | 2018-02-28 20:45:13.724395 | 2018-02-28 20:45:13.731656 | orange
(1 row)

# \d fruits
                                     Table "public.fruits"
   Column   |            Type             |                      Modifiers                      
------------+-----------------------------+-----------------------------------------------------
 id         | bigint                      | not null default nextval('fruits_id_seq'::regclass)
 fruit_type | fruit_type                  | not null default 'banana'::fruit_type
 created_at | timestamp without time zone | not null
 updated_at | timestamp without time zone | not null
 color      | color                       | 
Indexes:
    "fruits_pkey" PRIMARY KEY, btree (id)
    "index_fruits_on_fruit_type" btree (fruit_type)
    
# \dT+ fruit_type;
                                         List of data types
 Schema |    Name    | Internal name | Size | Elements |  Owner   | Access privileges | Description 
--------+------------+---------------+------+----------+----------+-------------------+-------------
 public | fruit_type | fruit_type    | 4    | banana  +| postgres |                   | 
        |            |               |      | orange  +|          |                   | 
        |            |               |      | grape   +|          |                   | 
        |            |               |      |          |          |                   | 
(1 row)

# \dT+ color;
                                      List of data types
 Schema | Name  | Internal name | Size | Elements |  Owner   | Access privileges | Description 
--------+-------+---------------+------+----------+----------+-------------------+-------------
 public | color | color         | 4    | red     +| postgres |                   | 
        |       |               |      | green   +|          |                   | 
        |       |               |      | yellow  +|          |                   | 
        |       |               |      | orange  +|          |                   | 
        |       |               |      |          |          |                   | 
(1 row)
```

In `schema.rb`:
```ruby
create_table "fruits", force: :cascade do |t|
  t.enum "fruit_type", default: "banana", null: false, values: ["banana", "orange", "grape", ""]
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.enum "color", values: ["red", "green", "yellow", "orange", ""]
  t.index ["fruit_type"], name: "index_fruits_on_fruit_type"
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/inpego/pg_activerecord_enum.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
