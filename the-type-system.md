# 03 - The Type System: Friend or Foe?

What Rubyists Need to Know 

Types Are Not Optional

In Ruby, types are suggestions. In Crystal, they're laws.

```ruby
# Ruby - this runs until it crashes
def add(a, b)
  a + b
end

add(5, 3)      # 8
add("5", "3")  # "53" (maybe not what you wanted)
add(5, "3")    # TypeError at runtime
```

```crystal
# Crystal - this gets checked BEFORE it runs
def add(a : Int32, b : Int32) : Int32
  a + b
end

add(5, 3)      # 8
add("5", "3")  # Compile error
add(5, "3")    # Compile error
```

The compiler stops you before anything bad happens. It's annoying at first. Then you realize it's saving your sleep schedule.

Type Inference Is Real (And Pretty Smart)

You don't always have to write types. Crystal figures a lot out.

```crystal
name = "John"        # Crystal knows this is a String
age = 25             # Crystal knows this is an Int32
items = []           # ❌ Wait, this one fails
```

Empty arrays and hashes need help:

```crystal
items = [] of String  # Tell Crystal what goes in
data = {} of String => Int32  # Keys are strings, values are integers
```

The rule: if Crystal can see what's inside, it can guess the type. If it's empty, you have to tell it.

Union Types (Variables That Can Be Multiple Things)

This is where Rubyists get confused:

```crystal
value = rand > 0.5 ? "hello" : 42
# value is now String | Int32 (a union type)
```

You can call methods that exist on BOTH types:

```crystal
value.to_s  # Works (both String and Int32 have to_s)
value.length  # Error - Int32 doesn't have length
```

To use a union type safely, you check what it is:

```crystal
if value.is_a?(String)
  puts value.length  # Now Crystal knows it's a String
else
  puts value + 10    # Now Crystal knows it's an Int32
end
```

This is annoying at first. Then you realize you'd have written the same check in Ruby anyway.

Nil Is A Type (Not Just Nothing)

In Ruby, nil is an object. In Crystal, nil is a type.

```crystal
name = find_user_name  # Might return String or nil
# name is String | Nil
```

You can't call methods on something that might be nil:

```crystal
name.upcase  # Error - name could be nil
```

You have to check:

```crystal
if name = find_user_name
  name.upcase  # Crystal knows it's not nil here
else
  "No name found"
end
```

Or use the safe navigation we talked about:

```crystal
name&.upcase  # Returns nil if name is nil
```

This seems strict. Then you remember all those "undefined method for nil" errors in production and it starts to make sense.

Type Annotations (When You Need Them)

You'll see stuff like this:

```crystal
@age : Int32?
@property name : String
def process(id : Int64) : Hash(String, JSON::Any)
```

The ? means "can be nil". So Int32? means "Int32 or nil".

My Problems With Types (30% of This)

The "I Know What I'm Doing" Compiler Fight

This was me for about 20 pipelines:

```crystal
data = {}  # ❌ Error: can't infer type
```

Me: "Just let me use the hash, I'll fill it later!"
Crystal: "Tell me what goes in it first."
Me: "I don't know yet!"
Crystal: "Then I can't compile it."

The fix was always the same:

```crystal
data = {} of String => JSON::Any  # Happy now?
```

The JSON::Any Lifesaver

When you don't know what type something will be, JSON::Any is your escape hatch:

```crystal
thing = JSON::Any.new("hello")  # String
thing = JSON::Any.new(42)       # Int32
thing = JSON::Any.new(true)      # Bool
thing = JSON::Any.new([1,2,3])   # Array
```

It's like Ruby's "whatever works" but with rules. I use this constantly when dealing with API responses.

The Union Type Explosion

Sometimes your types get out of hand:

```crystal
result = some_complex_operation
# result is String | Int32 | Nil | Array(String) | Hash(String, Int32)
```

At this point, you need to rethink your life choices. Usually means you need better design, not more complex types.

The Proc Type Syntax

This one still hurts my brain:

```crystal
def run(ctx : Context, &block : Context -> String)
```

That Context -> String means "a block that takes a Context and returns a String". After 64 pipelines, I can read it. I still can't write it without checking docs.

What Rubyists Actually Need To Remember

1. Types Are Not Optional, But They're Not Everywhere

You don't have to annotate everything. Crystal is smart:

```crystal
def add(a, b)  # Crystal can figure out types from usage
  a + b
end
```

Add types when:

· The inference isn't clear
· You want documentation
· The compiler complains

2. Nil Checking Is Required

Every time something can be nil, you check it. Every time.

```crystal
# Ruby - might crash
user.name.upcase

# Crystal - you're forced to think
if user = get_user
  user.name.upcase
end
```

3. Empty Collections Need Types

```crystal
items = []           # ❌ Error
items = [] of String # ✅ Correct
```

4. JSON::Any Is Your Friend

When dealing with unknown data:

```crystal
data = {} of String => JSON::Any
data["count"] = 5.to_json
data["name"] = "John".to_json
data["active"] = true.to_json
```

5. The Compiler Is Not Your Enemy

This took me 64 pipelines to learn. The compiler isn't being mean. It's preventing mistakes. Every time it complains, it's saving you from a potential production crash.

In Ruby, "it works on my machine" is a meme.
In Crystal, "it compiles" means it will work everywhere.

The Trade-Off

Ruby trusts you. Crystal does not trust you.

Ruby lets you figure it out later. Crystal makes you figure it out now.

Ruby is fun until 3 AM debugging. Crystal is annoying until you realize you don't have 3 AM debugging.

After 64 pipelines, I stopped fighting and started listening. The compiler wasn't being mean. It was teaching me to write better code.

Next: 04 - Hashes, NamedTuples, and Other Collections
