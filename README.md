# Reproduction Rails 4.2 app for exit 1 RSpec 3.x bug with test_help

## Installation
`$ bundle install`

## Bug Reproduction
`$ bundle exec rspec --color spec/test_spec.rb; echo $?;`

Will output `1` when the test_spec.rb is a no-op that succeeds and should 
therefore return a `0` exit code. 

If you remove the `--color` flag, the suite exits with `0`:

`$ bundle exec rspec spec/test_spec.rb; echo $?;`

Alternatively, if you stop requiring `rails/test_help.rb` from within 
`spec/rails_helper.rb` and replace it instead with 
`ActiveRecord::Migration.maintain_test_schema!`, the suite also exists with 
`0` properly as well.

## Context

The default in rspec-rails' `rails_helper.rb` uses:

```ruby
ActiveRecord::Migration.maintain_test_schema!
```

But this appears to have been replaced in Rails 4.2 and 5 in `test_helper.rb`
with: 

```ruby
require 'rails/test_help'
```

This is a drop-in replacement, except this file requires a bunch of additional 
test_unit/minitest specific files. 

Check [test_helper.rb](test/test_helper.rb) to see this same `rails/test_help` 
required. It requires a few additional files and most notably runs 
`ActiveRecord::Migration.maintain_test_schema!` as just like the 
rspec-rails' `rails_helper.rb` does. 

I can only surmise that since most of the code in `rails/test_help.rb` is meant 
for test_unit/minitest, it simply doesn't mesh well with RSpec at some level.
