# 04 - Hashes, NamedTuples, and Other Collections

What Rubyists Need to Know (70% of This)

The Three Ways To Store Things

Ruby has one way to store key-value data: a hash.

```ruby
# Ruby - one tool for everything
data = {}
data[:name] = "John"
data["age"] = 25
data[42] = "answer"
```

Crystal has THREE different tools. Each has a job. Mixing them up will cost you pipelines.

Tool 1: Hash (The Workhorse)

This is what you'll use 90% of the time.

```crystal
# You have to tell Crystal what goes in
settings = {} of String => String
settings["theme"] = "dark"
settings["language"] = "en"

# For mixed types, use JSON::Any
data = {} of String => JSON::Any
data["name"] = "John".to_json
data["age"] = 25.to_json
data["active"] = true.to_json
```

Use hashes when:

· You need to add/remove keys
· Keys come from user input or variables
· You don't know all the keys upfront

Tool 2: NamedTuple (The Fast One)

This looks like a hash but isn't.

```crystal
# This is a NamedTuple, NOT a hash
user = {name: "John", age: 25}
user[:name]  # "John"
user[:city] = "Lagos"  # ❌ Error - can't modify
```

NamedTuples are:

· Fast (faster than hashes)
· Immutable (can't change once created)
· Fixed keys (you know all keys at compile time)

Use NamedTuples when:

· You know all the keys upfront
· The data won't change
· You want better performance
· You're passing options to a method

Tool 3: Struct (The Organized One)

```crystal
# Define a real structure
struct User
  property name : String
  property age : Int32
  
  def initialize(@name, @age)
  end
end

user = User.new("John", 25)
user.name  # "John"
user.age = 26  # Can modify if you use property
```

Use structs when:

· You have a fixed set of fields
· You want method behavior attached to data
· You're modeling real things (User, Order, etc.)

The Hash Creation Trap

Ruby lets you create hashes however you want:

```ruby
# Ruby - all valid
h1 = {}
h2 = Hash.new
h3 = {name: "John", age: 25}
```

Crystal needs more information:

```crystal
# Empty hash - must specify type
h1 = {} of String => Int32

# Hash with initial values - type inferred
h2 = {"name" => "John", "age" => 25}  # Hash(String, String | Int32)

# Wait, that's not right - age is Int32 but now it's String | Int32
# Better to be explicit:
h3 = {"name" => "John", "age" => "25"}  # All strings
```

The lesson: be consistent with your types. Mixing strings and numbers in a hash gives you a union type, which is annoying to work with.

The Symbol Key Confusion

Ruby loves symbol keys:

```ruby
user = {name: "John", age: 25}
user[:name]  # "John"
```

Crystal supports this too, but there's a catch:

```crystal
# This is a NamedTuple, not a hash
user = {name: "John", age: 25}
user[:name]  # "John"

# This is a hash with symbol keys
user = {:name => "John", :age => 25}
user[:name]  # "John"
```

They look almost identical. The first is a NamedTuple (immutable). The second is a Hash (mutable). Your brain will mix them up constantly.

Accessing Values Safely

Ruby has dig for nested data:

```ruby
data = {user: {profile: {name: "John"}}}
data.dig(:user, :profile, :name)  # "John"
```

Crystal has a few options:

```crystal
data = {"user" => {"profile" => {"name" => "John"}}}

# Option 1: Chain with safe navigation
data["user"]?.try(&.["profile"]?).try(&.["name"]?)

# Option 2: Use dig? if available (some types have it)
data.dig?("user", "profile", "name")

# Option 3: The verbose but clear way
if user = data["user"]?
  if profile = user["profile"]?
    name = profile["name"]?
  end
end
```

Option 3 is more code but easier to debug. I use it when I care about knowing exactly what failed.

Iterating Over Collections

Ruby has dozens of ways to iterate. Crystal has each and friends.

```crystal
# Same as Ruby
data.each do |key, value|
  puts "#{key}: #{value}"
end

data.each_key do |key|
  puts key
end

data.each_value do |value|
  puts value
end
```

The difference: in Crystal, each_key and each_value are often faster but less common. I stick with each most of the time.

The Map Method

map works the same, but the return type is different:

```crystal
numbers = [1, 2, 3]
strings = numbers.map(&.to_s)  # Returns Array(String)
```

Crystal infers the return type from the block. It's smart enough to know that to_s returns String, so the result is Array(String).

The Compact Method

Ruby has compact to remove nil values:

```ruby
[1, nil, 2, nil].compact  # [1, 2]
```

Crystal has this too, but only for arrays:

```crystal
[1, nil, 2, nil].compact  # [1, 2]
```

For hashes, you need .reject:

```crystal
{"a" => 1, "b" => nil, "c" => 2}.reject { |_, v| v.nil? }
```

I learned this on pipeline 37.

My Problems With Collections (30% of This)

The Empty Hash Curse

This haunted me for days:

```crystal
def process
  result = {}  # ❌ Error: can't infer type
  # ... fill result ...
  result
end
```

The fix was always the same:

```crystal
def process
  result = {} of String => JSON::Any
  # ... fill result ...
  result
end
```

Now I just start every method with the type spelled out. Saves time.

The NamedTuple vs Hash Identity Crisis

This cost me pipelines:

```crystal
def create_user
  {name: "John", age: 25}  # Returns NamedTuple
end

user = create_user
user[:city] = "Lagos"  # ❌ Error (it's a NamedTuple)
```

I'd spend 20 minutes debugging before realizing I returned the wrong thing. Now I'm explicit:

```crystal
def create_user : Hash(String, JSON::Any)
  {"name" => "John", "age" => 25}
end
```

The JSON::Any Serialization Mess

When you use JSON::Any, you have to convert values:

```crystal
data = {} of String => JSON::Any
data["count"] = 5  # ❌ Error - 5 is Int32, not JSON::Any
data["count"] = 5.to_json  # ✅ Correct
```

But then when you read it back:

```crystal
count = data["count"].as_i  # Need .as_i to get Int32 back
```

This gets tedious. I made helper methods:

```crystal
def put_int(hash, key, value)
  hash[key] = value.to_json
end

def get_int(hash, key)
  hash[key]?.try(&.as_i)
end
```

The Nested Hash Nightmare

Accessing nested data is painful:

```crystal
data = {"user" => {"profile" => {"name" => "John"}}}

# Ruby: data["user"]["profile"]["name"]
# Crystal: 
if user = data["user"]?
  if profile = user["profile"]?
    name = profile["name"]?
  end
end
```

After 64 pipelines, I wrote a helper:

```crystal
def dig(hash, *keys)
  result = hash
  keys.each do |key|
    if result.is_a?(Hash)
      result = result[key]?
    else
      return nil
    end
  end
  result
end
```

Now I can do dig(data, "user", "profile", "name").

The Merge Confusion

Ruby has merge that works great:

```ruby
h1 = {a: 1}
h2 = {b: 2}
h1.merge(h2)  # {a: 1, b: 2}
```

Crystal has merge too, but it returns a NEW hash:

```crystal
h1 = {"a" => 1}
h2 = {"b" => 2}
h3 = h1.merge(h2)  # h1 is unchanged
```

If you want to modify in place, use merge!:

```crystal
h1.merge!(h2)  # h1 now has both keys
```

I learned this when my data wasn't changing and I spent an hour debugging.

What Rubyists Actually Need To Remember

1. Three Tools, Three Jobs

| Tool |  Use When |  Example |
 | ---- | ---- | ---- |
| Hash | Dynamic keys, mutable | User input, API responses|
| NamedTuple|  Fixed keys, immutable | Options, configs|
| Struct | Real objects with behavior | Models, entities|

2. Empty Collections Need Types

```crystal
items = [] of String
data = {} of String => JSON::Any
```

3. NamedTuples Are Not Hashes

```crystal
named = {name: "John"}  # Can't add keys
hash = {"name" => "John"}  # Can add keys
```

4. JSON::Any Requires Conversion

```crystal
data["key"] = value.to_json
value = data["key"].as_i
```

5. Access Nested Data Safely

```crystal
# Verbose but safe
if level1 = data["level1"]?
  if level2 = level1["level2"]?
    value = level2["level3"]?
  end
end
```

6. Use Helper Methods For Common Patterns

After a few pipelines, you'll develop your own helpers. That's normal. That's growth.

The Trade-Off

Ruby collections are simple. One tool, one way, figure it out at runtime.

Crystal collections are precise. Three tools, each with a job, figure it out at compile time.

Ruby is easier to write. Crystal is easier to debug.

After 64 pipelines, I prefer knowing that if my collections compile, they won't blow up at 3 AM. The initial headache is worth the sleep.

Next: 05 - Modules, Classes, and Namespaces
