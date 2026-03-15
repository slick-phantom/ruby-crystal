# 07 - Common Errors Decoded

What Those Scary Messages Actually Mean

Crystal's error messages look terrifying at first. They're long, they're full of type jargon, and they make you feel like you're not smart enough to use this language.

I felt that way for about 50 pipelines. Then I learned to read them.

Here's every common error decoded in plain English.

Error 1: undefined method 'foo' for Nil

What You See

```crystal
Error: undefined method 'upcase' for Nil
```

What It Means

You tried to call a method on something that might be nil. Crystal caught it before your users did.

What You Wrote

```crystal
user.name.upcase
```

What Crystal Thinks

"user.name could be nil. nil doesn't have an upcase method. This will crash at runtime."

How To Fix

```crystal
if name = user.name
  name.upcase
else
  "No name"
end

# Or using safe navigation
user.name?.try(&.upcase)
# Or
user.name&.upcase  # if the compiler is feeling generous
```

The Rubyist Translation

Remember all those "undefined method for nil:NilClass" errors in production? Crystal shows them to you BEFORE you deploy.

Error 2: type must be ..., not ...

What You See

```crystal
Error: instance variable '@age' of User must be Int32, not Nil
```

What It Means

You declared a variable as one type, then tried to put a different type in it.

What You Wrote

```crystal
class User
  @age : Int32
  
  def initialize
    @age = nil  # Int32 can't be nil
  end
end
```

How To Fix

```crystal
class User
  @age : Int32?  # Add ? to allow nil
  
  def initialize
    @age = nil  # Now works
  end
end
```

The Rubyist Translation

Ruby lets you put anything anywhere. Crystal needs to know: "can this be nil or not?"

Error 3: can't infer the type of ...

What You See

```crystal
Error: can't infer the type of instance variable '@data'
```

What It Means

Crystal looked at your code and couldn't figure out what type @data should be. It needs help.

What You Wrote

```crystal
class Processor
  def initialize
    @data = {}  # Empty hash with no type info
  end
end
```

How To Fix

```crystal
class Processor
  def initialize
    @data = {} of String => JSON::Any
  end
  # Or
  @data : Hash(String, JSON::Any) = {} of String => JSON::Any
end
```

The Rubyist Translation

Ruby doesn't care what's in your hash. Crystal needs to know: "strings as keys? symbols? what about the values?"

Error 4: expected argument #1 to be X, not Y

What You See

```crystal
Error: expected argument #1 to 'String#+' to be String, not Int32
```

What It Means

You tried to add a string and a number. Crystal doesn't do that.

What You Wrote

```crystal
"Count: " + 5
```

How To Fix

```crystal
"Count: " + 5.to_s
# Or using interpolation
"Count: #{5}"
```

The Rubyist Translation

Ruby quietly converts numbers to strings. Crystal makes you do it explicitly. It's not being mean, it's being predictable.

Error 5: undefined constant ...

What You See

```crystal
Error: undefined constant Api::Client
```

What It Means

You tried to use Api::Client but Crystal can't find it. Usually a require order problem.

What You Wrote

```crystal
# In bot.cr
class Bot
  getter client : Api::Client  # Where is Api::Client?
end
```

How To Fix

```crystal
# In your main file, make sure api/client.cr is required BEFORE bot.cr
require "./api/client"
require "./core/bot"
```

The Rubyist Translation

Ruby loads everything and hopes for the best. Crystal needs things in order. It's like building a house - you need the foundation before the walls.

Error 6: method ... is undefined

What You See

```crystal
Error: method 'process' is undefined
```

What It Means

You're calling a method that doesn't exist. Crystal checked EVERY path and couldn't find it.

What You Wrote

```crystal
def start
  proccess  # Typo!
end
```

How To Fix

Check your spelling. Crystal doesn't guess.

The Rubyist Translation

Ruby would wait until runtime to tell you about the typo. Crystal tells you immediately.

Error 7: can't use ... as the type of ...

What You See

```crystal
Error: can't use Object as the type of instance variable '@shrine'
```

What It Means

You tried to use Object as a type. That's too vague. Crystal needs something more specific.

What You Wrote

```crystal
@shrine : Object
```

How To Fix

```crystal
# Define an interface
module Uploadable
  abstract def upload(io, path)
end

@shrine : Uploadable
```

The Rubyist Translation

"Object" means "anything" in Ruby. In Crystal, "anything" isn't specific enough. You need to say WHAT methods the object must have.

Error 8: no overload matches ...

What You See

```crystal
Error: no overload matches 'Hash(String, JSON::Any)#[]=' with types String, Int32
```

What It Means

You're trying to put an Int32 into a hash that expects JSON::Any.

What You Wrote

```crystal
data = {} of String => JSON::Any
data["count"] = 5  # 5 is Int32, not JSON::Any
```

How To Fix

```crystal
data["count"] = 5.to_json  # Convert to JSON::Any
```

The Rubyist Translation

Ruby doesn't care about types. Crystal cares deeply. Convert your values before storing them.

Error 9: recursive dependency ...

What You See

```crystal
Error: recursive dependency: Bot -> Context -> Bot
```

What It Means

You have a circular dependency. File A requires B, B requires A. Crystal can't resolve it.

What You Wrote

```crystal
# bot.cr
require "./context"
class Bot
  def process(ctx : Context)
  end
end

# context.cr
require "./bot"
class Context
  def initialize(@bot : Bot)
  end
end
```

How To Fix

```crystal
# Forward declare or restructure
class Bot  # Don't require context here
  def process(ctx : Context)
  end
end

# context.cr
require "./bot"  # Require bot here
class Context
  def initialize(@bot : Bot)
  end
end
```

The Rubyist Translation

Ruby lets you have circular dependencies. They're messy but they work. Crystal says "clean up your mess before I compile it."

Error 10: must be a constant, not ...

What You See

```crystal
Error: must be a constant, not 42
```

What It Means

You used a number where Crystal expected a constant. Usually in macro land.

What You Wrote

```crystal
macro define_method(name)
  def {{name}}
  end
end

define_method(42)  # 42 isn't a valid method name
```

How To Fix

Use a symbol or string instead of a number.

The Rubyist Translation

Some things in Crystal happen at compile time. Numbers don't make good method names. Crystal is just being logical.

Error 11: can't find file ...

What You See

```crystal
Error: can't find file './api/client'
```

What It Means

The file you're trying to require doesn't exist at that path.

What You Wrote

```crystal
require "./api/client"  # But you're in src/core/bot.cr
```

How To Fix

```crystal
require "../api/client"  # Go up one level first
```

The Rubyist Translation

Paths are relative to the current file, not the project root. This is actually the same as Ruby, but Ruby is more forgiving if you're wrong.

Error 12: expected argument ## to be ...

What You See

```crystal
Error: expected argument #2 to 'Hash#[]=' to be JSON::Any, not String
```

What It Means

You're passing the wrong type to a method.

What You Wrote

```crystal
hash = {} of String => JSON::Any
hash["name"] = "John"  # "John" is String, not JSON::Any
```

How To Fix

```crystal
hash["name"] = JSON::Any.new("John")
# or
hash["name"] = "John".to_json
```

The Rubyist Translation

Ruby doesn't check types. Crystal does. It's not personal.

My Error Log (What I Actually Screamed)

- Pipeline 8: "WHY CAN'T YOU FIND API::CLIENT IT'S RIGHT THERE" (wrong require order)
- Pipeline 17: "JUST LET ME USE THE HASH" (needed type annotation)
- Pipeline 23: "BUT & WORKS IN RUBY" (needed self. or if statements)
- Pipeline 29: "I SWEAR THIS FILE EXISTS" (wrong relative path)
- Pipeline 37: "IT'S NOT NIL I JUST CHECKED" (forgot ? in type)
- Pipeline 41: "BUT I WANT IT TO BE ANYTHING" (use JSON::Any)
- Pipeline 52: "THIS WORKED YESTERDAY" (alphabetical order in CI)
- Pipeline 58: "JUST LET ME COMPILE" (finally fixed everything)

What Rubyists Actually Need To Remember

1. Errors Are Gifts

Every error is a crash that won't happen in production. Crystal is showing you where your code would fail. Thank it.

2. Read The Whole Error

The first line tells you what. The last line tells you where. The middle tells you why.

3. "Undefined constant" Usually Means Require Order

Check your main file. Are you loading things in the right order?

4. "Type mismatch" Usually Means Missing Conversion

Strings aren't JSON::Any. Int32 isn't String. Convert explicitly.

5. "Can't infer type" Means You Need To Be Explicit

Empty arrays and hashes need type annotations. Crystal can't read your mind.

6. "No overload matches" Means Wrong Types

You're calling a method with the wrong argument types. Check what the method expects.

7. The Error Message Is Trying To Help

It looks scary, but it's literally telling you what's wrong. Read it slowly. It usually says exactly what to fix.

The Trade-Off

Ruby errors happen when your users run your code. Crystal errors happen when you run the compiler.

Ruby errors wake you up at 3 AM. Crystal errors wake you up at 3 PM.

After 64 pipelines, I prefer 3 PM errors. They don't interrupt my sleep.

Next: 08 - My 64 Pipeline Failures
