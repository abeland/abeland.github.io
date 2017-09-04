Rails is pretty neat for getting stuff up and running quickly and will scale decently later on for most uses.
However, there are a bunch of things I tend to forget regarding how to set up autoloading, rspec, etc.
I like to keep things simple so here are a few things I do to move fast as I start new projects:

## 1. Set up autoloading of `lib/`

You throw a bunch of junk into `lib/` only find Rails complaining about `uninitialized constant`(s) and such.
I just tell Rails to autoload everything in `lib/`:

```
# config/application.rb
config.autoload_paths += Dir["#{config.root}/lib/**/"]
config.enable_dependency_loading = true
```

## 2. Set up autoloading in all your rspecs

This one took me a while to realize, but when you first set up RSpec, you have to require everything you want to test.
For example, if you have a module called `Sorting`, and you go to write a test:

```
RSpec.describe Sorting do
  ...
end
```

You will get a `NameError` because there is an `uninitialized constant`.
And it's true, there's nothing that `require`d your `Sorting` module.
So I like to change my .rspec to be:

```
# .rspec
--color
--require rails_helper
```

(`spec/rails_helper.rb` should be have been created for you when you followed the instructions for the rspec-rails gem).

`rails_helper.rb` itself requires `spec_helper.rb` so you can remove the `--require spec_helper` line from your `.rspec` file.
Yes, requiring `rails_helper` in your `.rspec` file will make running tests slower because of the longer set up time, but
the convenience factor of not having to explicitly `require` every module in `lib/` that I need is well worth it.

## 3. Alias everything

If you are constantly typing stuff like `bundle exec rails console` instead of something much shorter like `rc` then you are wasting your life.
By my unscientific measurement, it takes me about 3-4 seconds to type the former and only 1 the latter. Let's be generous and
say it takes you 3 seconds to type out the former, but 1 second for the latter. So, you can save 2 seconds each time you fire up
the rails console. Let's say you do this about 20 times a day (I think I do it much more, but whatever). That's 40 seconds
saved per day. Times 6 days a week (you only work 5 days a week huh?), that's 240 seconds per week. Times 50 weeks worked per year
(again, you only work like 40 huh?), that's 12000 seconds per year, which is about 3 hours and 20 minutes. And that's just one example.
I also alias firing up my rails server, running rspec tests, logging in to remote machines, etc. My basic rule of thumb is:

> If you type the same command more than once per day, it should be aliased.

Feel free to check [my bash profile](https://gist.github.com/abeland/04529fcc239d1e5308695a97897510cb)

NOTE: This version of my bash profile may change often so just browse [my gists](https://gist.github.com/abeland) for the most recent iteration.
