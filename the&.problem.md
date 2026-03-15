# 02 - The &. Problem and Other Syntax Surprises

The Operator That Looks Great But Isn't

Every Rubyist sees &. and falls in love. It's clean. It's concise. It solves a real problem.

```ruby
# Ruby - beautiful
user&.profile&.settings&.theme
```

Then you try it in Crystal and... nothing works the way you expect.

How & Actually Works in Crystal

Here's the thing: Crystal DOES have &.. It works. But it has conditions.

```crystal
# This works fine
user = get_user
user&.name  # Returns nil if user is nil, otherwise user.name
```

The problem comes when you try to chain multiple calls:

```crystal
# This might work, might not - depends on the compiler's mood
user&.profile&.settings&.theme
```

The compiler needs to know that profile exists, that settings exists, that theme exists. ALL AT COMPILE TIME. If any of those methods might be missing, the compiler panics.

When & Works vs When It Doesn't

It works when:

```crystal
class User
  def profile : Profile?
    @profile
  end
end

class Profile
  def settings : Settings?
    @settings
  end
end

# Crystal can trace the types through each call
user&.profile&.settings  # Works
```

It doesn't work when:

```crystal
# If Crystal can't guarantee the intermediate types
data&.dig("user", "profile", "settings")  # Might fail
```

The Practical Solution

After 64 pipelines, here's what I actually do:

```crystal
# Instead of chaining &. a dozen times
if user = user
  if profile = user.profile
    if settings = profile.settings
      theme = settings.theme
    end
  end
end
```

Is it verbose? Yes.
Do I still have to think about it? No.
Does it compile every single time? Also yes.

There's also the try method:

```crystal
user.try(&.profile).try(&.settings).try(&.theme)
```

But honestly? The nested if statements are easier to debug.

The Self Situation

This cost me three pipelines:

```crystal
def from
  message&.from  # Randomly fails to compile
end
```

The fix was absurd:

```crystal
def from
  self.message&.from  # Now it works
end
```

Adding self. tells the compiler "this method exists in THIS class, stop overthinking it."

Quotes Will Waste Your Time

In Ruby, quotes are style. Use whichever you want.

```ruby
name = 'John'     # fine
name = "John"     # also fine
```

In Crystal, they are NOT the same.

```crystal
name = 'John'     # Error: multiple characters in char literal
name = "John"     # Correct
```

Single quotes are for ONE CHARACTER. That's it. If you come from Ruby, you will make this mistake at least once. Probably more.

```crystal
char = 'a'        # This is a Char (like a single letter)
string = "a"      # This is a String (text)
```

They are different types. You can't mix them.

The Hash Situation

Ruby hashes are flexible:

```ruby
data = {}
data["name"] = "John"
data[:age] = 25
data[42] = "answer"
```

Crystal needs to know what it's dealing with:

```crystal
# String keys, any values
data = {} of String => JSON::Any
data["name"] = "John".to_json
data["age"] = 25.to_json

# Symbol keys, any values
symbol_data = {} of Symbol => JSON::Any
symbol_data[:name] = "John".to_json
```

That JSON::Any type can hold strings, numbers, booleans, arrays, even other hashes. Use it when you're not sure what you'll get.

The Curly Brace Confusion

In Ruby, {} means hash. Always.

In Crystal, it depends:

```crystal
# This is a NamedTuple (fixed keys, fast, immutable)
person = {name: "John", age: 25}
person[:name]        # "John"
person[:city] = "Lagos"  # Error - can't modify

# This is a Hash (flexible, mutable)
person = {"name" => "John", "age" => 25}
person["city"] = "Lagos"  # Works
```

They look almost identical. Your brain will mix them up constantly.

The Return Trap

In Ruby, you can return whenever:

```ruby
def find_user(id)
  return nil if id.nil?
  User.find(id)
end
```

In Crystal, the last expression is the return. Using return exits early, which is fine, but all return paths must return the SAME TYPE.

```crystal
def find_user(id : Int32?) : User?
  return nil if id.nil?           # Returns Nil
  User.find(id)                    # Returns User?
end
# This works because User? includes Nil
```

But this fails:

```crystal
def process
  return "error" if bad?           # String
  return 42 if weird?               # Int32
  true                               # Bool
end
# Error: mismatched return types
```

Every path must return the same type. This takes time to get used to.

The For Loop That Vanished

Ruby has for loops (even if you don't use them).

Crystal doesn't. Use each.

```crystal
# Ruby
for i in 1..10
  puts i
end

# Crystal
(1..10).each do |i|
  puts i
end
```

Not a big deal, but if you're porting old code, it'll trip you up.

What Actually Matters

After all those pipeline failures, here's what I actually remember:

1. Double quotes always - Single quotes are for characters only
2. Add self. when the compiler complains - It usually fixes it
3. Write out conditionals instead of chaining &. - More code, fewer headaches
4. Be explicit with hash types - {} of String => JSON::Any is your friend
5. NamedTuples {key: value} are not Hashes - You can't add to them

The syntax isn't trying to annoy you. It's trying to prevent you from making mistakes at 3 AM. It just has a weird way of showing it.

Next: 03 - The Type System
