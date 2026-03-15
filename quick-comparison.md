# 01 - Quick Comparison: Ruby vs Crystal

They Look Like Twins Until One Punches You

I know what you're thinking. You saw some Crystal code and went "wait, that's just Ruby with types?" Yeah. That's what I thought too. Then I spent two weeks fighting a compiler and questioning my entire existence as a developer.

Let me save you from that.

The "This Looks Familiar" Trap

Here's Ruby code you've written a million times:

```ruby
class User
  def initialize(name)
    @name = name
  end

  def greet
    "Hello, #{@name}"
  end
end

user = User.new("John")
puts user.greet
```

Now look at this Crystal code:

```crystal
class User
  def initialize(@name : String)
  end

  def greet
    "Hello, #{@name}"
  end
end

user = User.new("John")
puts user.greet
```

See? Almost identical. That's the bait.

Where Rubyists Get Stabbed

1. Types Are Like Vegetables

You know you need them. You know they're good for you. But you really wish you could just skip them.

```ruby
# Ruby - this is fine
def add(a, b)
  a + b
end

add(5, 3)      # 8
add("5", "3")  # "53" (works but... why?)
add(5, "3")    # TypeError at 3am in production
```

```crystal
# Crystal - eat your vegetables
def add(a : Int32, b : Int32) : Int32
  a + b
end

add(5, 3)      # 8
add("5", "3")  # Compile error (good catch)
add(5, "3")    # Also compile error
```

Ruby trusts you. Crystal does not. Crystal has been burned before.

2. The &.( ) Lie

Ruby developers love this thing:

```ruby
user&.profile&.settings&.theme  # nil if anything missing
```

It's beautiful. It's elegant. It works everywhere.

Crystal has this too. Sort of. Not really.

```crystal
user&.profile&.settings&.theme  # 50% chance of compiler error
```

The compiler will look at you and say "I need to know FOR SURE that .profile exists." Not "probably." Not "it should." FOR SURE.

So you end up writing this instead:

```crystal
if user = user
  if profile = user.profile
    if settings = profile.settings
      theme = settings.theme
    end
  end
end
```

It's ugly. It's verbose. It compiles. Pick your battles.

3. Quotes Are Serious Business

In Ruby, you use single or double quotes based on vibes:

```ruby
name = 'John'        # fine
message = "Hello"    # also fine
```

In Crystal, single quotes mean ONE CHARACTER. That's it.

```crystal
name = 'John'        # Error: multiple characters in char literal
name = "John"        # Correct

letter = 'a'         # This is fine (Char type, not String)
letter = "a"         # Also fine (String with one char)
```

If you take one thing from this guide: use double quotes. Always. Your future self will thank you.

4. Hashes Need Therapy

Ruby lets you do whatever you want with hashes:

```ruby
data = {}
data["name"] = "John"
data[:age] = 25
data[42] = "answer"
```

Crystal looks at this and has an anxiety attack.

```crystal
# You have to tell Crystal what goes in your hash
data = {} of String => JSON::Any
data["name"] = "John".to_json
data["age"] = 25.to_json
# No symbol keys unless you explicitly allow them
```

That JSON::Any thing? It's Crystal's way of saying "I don't know what type this is yet, but I'll allow it." Use it. Love it. It'll save you.

5. The Curly Brace Identity Crisis

This looks like a hash in Ruby:

```ruby
options = {name: "John", age: 25}
options[:name]  # "John"
```

In Crystal, this is NOT a hash. It's a NamedTuple. Fixed keys. Fast. Immutable. You can't add anything to it.

```crystal
options = {name: "John", age: 25}
options[:name]        # "John"
options[:city] = "Lagos"  # Error! Can't modify NamedTuple
```

If you want a real hash, you have to be explicit:

```crystal
hash = {"name" => "John", "age" => 25}
hash["city"] = "Lagos"  # Works
```

6. Constants Are Everywhere

In Ruby, constants live in classes and modules:

```ruby
module MyApp
  VERSION = "1.0"
end
```

In Crystal, constants are like weeds. They grow everywhere.

```crystal
module MyApp
  VERSION = "1.0"
end

class User
  MAX_AGE = 150
end

def process
  TIMEOUT = 30  # Yes, even here
  puts TIMEOUT
end
```

Any uppercase identifier is a constant. Anywhere. Get used to it.

7. No More Method Missing Magic

Ruby developers love method_missing. It's how half the gems work.

```ruby
class Magic
  def method_missing(name, *args)
    "You called #{name}"
  end
end

obj = Magic.new
obj.whatever  # "You called whatever"
```

Crystal doesn't have this. At all. The compiler needs to know every method exists before you run anything.

There are macros that do similar things at compile time, but it's not the same. If you rely on method_missing, Crystal will be painful.

8. Order Actually Matters

In Ruby, you can require files in any order and it's probably fine:

```ruby
require "./core/bot"
require "./api/client"  # Works even if bot needs client
```

In Crystal, the order matters. A lot.

```crystal
require "./api/client"  # Must come first
require "./core/bot"    # Because bot uses Api::Client

# Wrong order:
require "./core/bot"    # Error: undefined constant Api::Client
require "./api/client"
```

I learned this on pipeline 23. You don't have to.

9. Self Is Sometimes Required

Ruby lets you skip self most of the time:

```ruby
def from
  message&.from || callback_query&.from
end
```

Crystal sometimes needs you to be explicit:

```crystal
def from
  self.message&.from || self.callback_query&.from
end
```

When in doubt, add self.. The compiler will tell you if it's wrong. This is one of those "add it until you forget why" things.

The Trade-Off You're Making

Ruby gives you freedom. Crystal gives you safety.

Ruby trusts you. Crystal does not trust you.

Ruby lets you figure it out at 3am when prod breaks. Crystal makes you figure it out at compile time so prod never breaks.

After 64 failed pipelines, I stopped fighting it. I realized the compiler wasn't being mean. It was saving me from myself.

The Golden Rule

In Ruby, if your tests pass, it might work.

In Crystal, if it compiles, it works.

That's the trade. That's why the compiler screams at you about every little thing. It's not personal. It's just doing its job.

Next up: 02 - The &. Problem and Other Syntax Surprises
