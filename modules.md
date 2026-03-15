# 05 - Modules, Classes, and Namespaces

What Rubyists Need to Know (70% of This)

They Look The Same (Until They Don't)

Ruby classes and Crystal classes look almost identical:

```ruby
# Ruby
class User
  attr_reader :name
  
  def initialize(name)
    @name = name
  end
  
  def greet
    "Hello, #{@name}"
  end
end
```

```crystal
# Crystal
class User
  getter name : String
  
  def initialize(@name : String)
  end
  
  def greet
    "Hello, #{@name}"
  end
end
```

The differences are small but important.

Instance Variables Need Types

In Ruby, you just use @name whenever.

In Crystal, every instance variable needs a type. Either you specify it:

```crystal
class User
  @name : String
  
  def initialize(@name : String)
  end
end
```

Or Crystal infers it from initialize:

```crystal
class User
  def initialize(@name : String)
  end
  # @name is automatically String
end
```

If you set an instance variable outside initialize, you MUST give it a type:

```crystal
class User
  @name : String?
  
  def set_name(name)
    @name = name  # Works because @name can be nil or String
  end
end
```

The ? means "can be nil". You'll see this a lot.

Property Shortcuts

Ruby has attr_reader, attr_writer, attr_accessor.

Crystal has getter, setter, property:

```crystal
class User
  getter name : String      # read only
  setter age : Int32         # write only
  property email : String    # read + write
  
  def initialize(@name, @age, @email)
  end
end
```

These create the methods automatically. Use them. Your fingers will thank you.

Initialize Shorthand

Ruby lets you do this:

```ruby
def initialize(name, age)
  @name = name
  @age = age
end
```

Crystal has a shorthand:

```crystal
def initialize(@name : String, @age : Int32)
end
```

The @name : String in the parameter list means "assign this parameter to @name and also make @name a String". It's two lines in one. I use this constantly.

Inheritance Works (Mostly)

Ruby inheritance works one way. Crystal works the same:

```crystal
class Animal
  def speak
    "Some sound"
  end
end

class Dog < Animal
  def speak
    "Woof!"
  end
end
```

The difference: Crystal needs to know types. If you override a method, the return type should match.

```crystal
class Animal
  def speak : String
    "Some sound"
  end
end

class Dog < Animal
  def speak : String  # Must return String
    "Woof!"
  end
end
```

Abstract Classes and Methods

Crystal has real abstract classes:

```crystal
abstract class Animal
  # Every animal must implement this
  abstract def speak : String
  
  # Regular method with implementation
  def description
    "I'm an animal"
  end
end

class Dog < Animal
  def speak : String  # Must implement this
    "Woof!"
  end
end
```

If you forget to implement speak in Dog, the compiler yells at you. In Ruby, you'd find out at runtime. In Crystal, you find out when you try to compile.

This saved me from at least 5 bugs in telecr.

Modules Are Mixins (Same As Ruby)

Modules work exactly like you expect:

```crystal
module Loggable
  def log(message)
    puts "[LOG] #{message}"
  end
end

class Bot
  include Loggable
end

bot = Bot.new
bot.log("Started")  # Works
```

You can also extend for class methods:

```crystal
module ClassMethods
  def create_default
    new
  end
end

class Bot
  extend ClassMethods
end

Bot.create_default  # Works
```

The Constant Problem

In Ruby, constants are in classes and modules.

In Crystal, constants are EVERYWHERE:

```crystal
MAX_SIZE = 100  # File-level constant

class Processor
  TIMEOUT = 30  # Class constant
  
  def process
    RETRY_COUNT = 3  # Method constant (yes, really)
    puts RETRY_COUNT
  end
end
```

Any uppercase identifier is a constant. You can't reassign them. This tripped me up when I used MaxSize as a variable name by accident.

The Self Situation

Ruby has self. Crystal has self too. But Crystal needs it more often:

```crystal
class Bot
  def start
    # Calling another method in same class
    configure  # Works
    self.configure  # Also works
  end
  
  def configure
  end
end
```

Sometimes Crystal can't find the method without self:

```crystal
def from
  # Without self, Crystal might look in wrong place
  self.message&.from
end
```

When in doubt, add self.. The compiler will tell you if it's wrong.

Class Variables (Use With Caution)

Ruby has @@class_var. Crystal has them too, but they're tricky with inheritance:

```crystal
class Parent
  @@count = 0
  
  def self.count
    @@count
  end
end

class Child < Parent
  @@count = 5  # This MODIFIES Parent's @@count
end
```

Class variables are shared across the entire hierarchy. Most Crystal devs avoid them and use class instance variables instead:

```crystal
class Parent
  @count : Int32 = 0
  
  def self.count
    @count
  end
  
  def self.count=(value)
    @count = value
  end
end
```

This took me a while to understand. Now I just avoid @@ entirely.

The New Method

Ruby uses new to create instances. So does Crystal.

But Crystal also lets you define your own constructors:

```crystal
class User
  getter name : String
  
  def initialize(@name)
  end
  
  def self.from_json(json : String)
    data = JSON.parse(json)
    new(data["name"].as_s)
  end
end

user = User.from_json('{"name": "John"}')
```

This is the same as Ruby. Nothing surprising here.

My Problems With Classes (30% of This)

The Uninitialized Instance Variable

This error haunted me:

```crystal
class Bot
  @logger : Logger  # Declared but not initialized
  
  def initialize
    # Oops, forgot to set @logger
  end
end
```

Crystal requires every instance variable to be initialized. Either in initialize or with a default value:

```crystal
class Bot
  @logger : Logger = Logger.new  # Default value
  @name : String  # Must be set in initialize
  
  def initialize(@name)
  end
end
```

If you forget, the compiler screams. I forgot a lot.

The Nilable Instance Variable

Sometimes you want a variable that starts as nil:

```crystal
class Bot
  @webhook : Webhook?  # Can be nil or Webhook
  
  def initialize
    @webhook = nil
  end
end
```

The ? is required. Without it, Crystal thinks @webhook must always be a Webhook. You'll get errors when you try to set it to nil.

The Type Inference Lie

Crystal is smart about inferring types from initialize:

```crystal
class Bot
  def initialize(@token)
    # @token type inferred from argument
  end
end
```

But if you have multiple initialize methods (overloading), Crystal gets confused:

```crystal
class Bot
  def initialize(@token : String)
  end
  
  def initialize(@token : String, @name : String)
  end
  # Error: can't infer type for @token
```

You need to be explicit when you have multiple constructors.

The Generic Class Confusion

Crystal has generics (like Java or C#). This still hurts my brain:

```crystal
class Box(T)
  def initialize(@value : T)
  end
  
  def value : T
    @value
  end
end

int_box = Box(Int32).new(42)
string_box = Box(String).new("hello")
```

I don't use these often. When I do, I copy-paste from examples.

The Include vs Extend vs Inherit Decision

In Ruby, it's simple: include for instance methods, extend for class methods, inherit for "is-a" relationships.

In Crystal, it's the same. The only difference is that include can also bring in macros and other compile-time stuff. I still use the Ruby rules and it works.

What Rubyists Actually Need To Remember

1. Instance Variables Need Types

```crystal
class User
  @name : String          # Explicit type
  # or
  def initialize(@name)   # Inferred from parameter
  end
end
```

2. Use getter/setter/property

```crystal
property name : String    # read + write
getter age : Int32        # read only
setter email : String     # write only
```

3. @param in initialize is shorthand

```crystal
def initialize(@name, @age)  # Assigns and types in one line
end
```

4. Abstract methods must be implemented

```crystal
abstract def speak : String  # All subclasses MUST implement
```

5. Constants are everywhere, not just classes

```crystal
TIMEOUT = 30  # File level is fine
```

6. Add self. when the compiler complains

```crystal
self.message&.text  # Explicit is better than debug
```

7. Use ? for nilable variables

```crystal
@webhook : Webhook?  # Can be nil or Webhook
```

The Trade-Off

Ruby classes are flexible. You can change them at runtime, add methods, remove methods, do whatever.

Crystal classes are rigid. Everything must be known at compile time.

Ruby is fun for prototyping. Crystal is safe for production.

After building telecr, I've learned to love the rigidity. It means when my bot runs at 3 AM, it won't crash because I forgot to initialize something.

Next: 06 - Require Order and Project Structure
