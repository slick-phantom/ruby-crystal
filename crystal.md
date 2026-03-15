
# 08 - Why Crystal is Actually Amazing (And You Should Stick With It)

## What Ruby Got Right

Ruby is beautiful. Let's be honest about that.

```ruby
5.times { puts "Hello" }  # Reads like English
```

Ruby made programming fun again. It gave us Rails, which gave us a million jobs. Ruby is why many of us are developers today.

I love Ruby. I still maintain telegem (https://gitlab.com/ruby-telegem/telegem). I'm not here to bury Ruby.

I'm here to tell you why I'm staying with Crystal.

What Crystal Does Better

1. Performance That Actually Matters

Ruby is slow. We all know it. We accept it. We throw more servers at it.

```ruby
# Ruby - 1000 requests/second
```

```crystal
# Crystal - 10,000+ requests/second
```

That's not a small difference. That's the difference between one server and ten servers. Between affordable hosting and expensive scaling.

Learn more about Crystal performance: https://crystal-lang.org/guides/performance.html

2. No Runtime Crashes (Seriously)

When was the last time you deployed a Rails app and felt 100% confident?

```ruby
# Ruby - might crash at 3am
user.name.upcase  # if user is nil, goodbye
```

```crystal
# Crystal - won't compile if it can crash
if user = user
  user.name.upcase  # Safe
end
```

The compiler isn't being mean. It's being your guardian angel.

3. Built-in HTTP Server

Ruby needs Puma, Unicorn, or Passenger. Crystal has it built-in:

```crystal
require "http/server"

server = HTTP::Server.new do |context|
  context.response.print "Hello, world!"
end

server.listen(8080)
```

No gems. No config. Just works.

HTTP::Server docs: https://crystal-lang.org/api/HTTP/Server.html

4. Built-in JSON Handling

Ruby needs require "json" (which is stdlib, but still). Crystal has it:

```crystal
require "json"

data = JSON.parse(%({"name": "John"}))
name = data["name"].as_s
```

JSON docs: https://crystal-lang.org/api/JSON.html

5. Built-in YAML

Ruby needs require "yaml". Crystal has it:

```crystal
require "yaml"

data = YAML.parse("name: John")
name = data["name"].as_s
```

YAML docs: https://crystal-lang.org/api/YAML.html

6. Built-in Logging

Ruby has Logger. Crystal has Log:

```crystal
require "log"

logger = Log.for("my_app")
logger.info { "Something happened" }
```

Log docs: https://crystal-lang.org/api/Log.html

7. Built-in Concurrency

Ruby has threads (complicated) and Ractors (new, still maturing). Crystal has fibers:

```crystal
spawn do
  # This runs in the background
  process_long_task
end

# Main thread continues immediately
```

Thousands of fibers. Minimal memory. Real concurrency.

Fiber docs: https://crystal-lang.org/api/Fiber.html

8. Built-in C Bindings

Need to use a C library? Crystal makes it easy:

```crystal
@[Link("ssl")]
lib LibSSL
  fun SSL_library_init : Int32
end

LibSSL.SSL_library_init
```

Ruby needs C extensions (painful). Crystal has FFI built-in.

C bindings docs: https://crystal-lang.org/reference/syntax_and_semantics/c_bindings/index.html

What Crystal Doesn't Have (And Why That's OK)

No Rails (Yet)

Ruby has Rails. Crystal has:

What you want Crystal alternative
Web framework Lucky (https://luckyframework.org/), Amber (https://amberframework.org/), Athena (https://athenaframework.org/)
ORM Granite (https://github.com/amberframework/granite), Jennifer (https://github.com/imdrasil/jennifer.cr)
Background jobs Mosquito (https://github.com/mosquito-cr/mosquito), Sidekiq.cr (https://github.com/maiha/sidekiq.cr)
Authentication You build it (or use shields (https://github.com/greyblake/shields.cr))

The ecosystem is smaller. But what exists is high quality.

No ActiveRecord

Ruby has ActiveRecord. Crystal has:

Feature Crystal option
SQL builder Clear (https://github.com/anykeyh/clear)
Query DSL Built into most ORMs
Migrations Built into Jennifer (https://github.com/imdrasil/jennifer.cr)

It's different. It's not worse. Just different.

No Gems (Shards Instead)

Ruby has RubyGems. Crystal has shards (https://crystal-lang.org/reference/the_shards_command/index.html):

```yaml
dependencies:
  lucky:
    github: luckyframework/lucky
```

Shards are simpler. They just point to git repos. No central server required.

What Rubyists Will Miss (And What Replaces It)

Ruby gem Crystal equivalent Notes
rails lucky or amber Full-featured frameworks
activerecord granite or jennifer ORMs with similar patterns
sidekiq mosquito Redis-backed jobs
devise Build your own Authentication is simpler with Crystal
rack Built-in HTTP::Server No need for middleware layer
puma Built-in server One less dependency
json Built-in No gem needed
yaml Built-in No gem needed
logger Built-in Log Different API, same purpose
dotenv cr-dotenv (https://github.com/gdotdesign/cr-dotenv) Or use ENV directly
rspec spectator (https://gitlab.com/arctic-fox/spectator) RSpec-like syntax
webmock webmock.cr (https://github.com/manastech/webmock.cr) Same name, same purpose
time Built-in Time Actually better than Ruby's

What Crystal Has That Ruby Doesn't

True Compilation

```bash
crystal build my_app.cr -o my_app
./my_app  # Runs anywhere, no Crystal needed
```

Single binary. No dependencies. No version conflicts.

Type Inference (Not Just Dynamic)

Ruby guesses types at runtime. Crystal guesses at compile time:

```crystal
def add(a, b)
  a + b  # Crystal figures out the types
end

add(5, 3)      # Works with Int32
add("a", "b")  # Works with String
```

It's not "static or dynamic". It's both.

Macros (Compile-time Code Generation)

Ruby has method_missing. Crystal has macros:

```crystal
macro define_getter(name)
  def {{name.id}}
    @{{name.id}}
  end
end

class User
  define_getter name
  define_getter age
end
```

Code that writes code. At compile time. No runtime performance cost.

Macro docs: https://crystal-lang.org/reference/syntax_and_semantics/macros/index.html

Enums

Ruby has symbols. Crystal has actual enums:

```crystal
enum Color
  Red
  Green
  Blue
end

color = Color::Red
```

Type-safe. Fast. Self-documenting.

Enum docs: https://crystal-lang.org/reference/syntax_and_semantics/enum.html

Named Tuples

Ruby has hashes. Crystal has both hashes AND named tuples:

```crystal
person = {name: "John", age: 25}  # Fast, immutable
person.name  # "John"
```

Perfect for options and configuration.

Tuple docs: https://crystal-lang.org/api/Tuple.html

Union Types

Ruby has "it could be anything". Crystal has actual union types:

```crystal
value = rand > 0.5 ? "hello" : 42
# value is String | Int32
```

The compiler tracks what's possible. You check once, then use safely.

The Trade-Offs (Honest Talk)

Learning Curve

Ruby: gentle slope, endless depth
Crystal: steep start, then smooth

The first week is brutal. The second week is better. By week three, you're faster than in Ruby because the compiler catches your mistakes.

Ecosystem Size

Ruby: 180,000+ gems
Crystal: ~2,000 shards

You'll build more yourself. But what you build will be exactly what you need.

Jobs

Ruby: everywhere
Crystal: niche

But niche means less competition. When companies need Crystal developers, they're desperate. I've seen it happen.

Projects Built By A Fellow Sufferer (That's Me)

While learning Crystal, I built a few things. They might help you:

- telegem (Ruby Telegram bot framework): https://gitlab.com/ruby-telegem/telegem
- tg-auth (Crystal Telegram authentication): https://github.com/gems/tgauth
- telecr (Crystal Telegram bot framework): https://github.com/telegem-cr/telecr
- telecr docs: https://telecr.gitlab.io

These are real projects, used by real people. They survived 64 pipelines. They'll work for you too.

Links You'll Actually Need

Official Docs

- [Crystal Language Home]: (https://crystal-lang.org/)
- [Crystal API Docs: ](https://crystal-lang.org/api/)
- [Crystal Reference: ](https://crystal-lang.org/reference/)
- [Getting Started Guide: ](https://crystal-lang.org/reference/getting_started/)
- [Crystal for Rubyists] (official): https://crystal-lang.org/reference/syntax_and_semantics/crystal_for_rubyists.html)
- [HTTP::Server docs:] (https://crystal-lang.org/api/HTTP/Server.html)
- [JSON docs:] (https://crystal-lang.org/api/JSON.html)
- [YAML docs:] (https://crystal-lang.org/api/YAML.html)
- [Log docs:] )https://crystal-lang.org/api/Log.html)
- [Fiber docs:] (https://crystal-lang.org/api/Fiber.html)
- [C bindings docs: ](https://crystal-lang.org/reference/syntax_and_semantics/c_bindings/index.html)
- [Macro docs:] (https://crystal-lang.org/reference/syntax_and_semantics/macros/index.html)
- Enum docs:] (https://crystal-lang.org/reference/syntax_and_semantics/enum.html)
- [Tuple docs:] (https://crystal-lang.org/api/Tuple.html)
- [Shards command: ](https://crystal-lang.org/reference/the_shards_command/index.html)

Essential Shards

- [Lucky Framework]: (https://luckyframework.org/)
- [Amber Framework:]( https://amberframework.org/)
- [Kemal: ](https://kemalcr.com/)
- [Granite ORM: ](https://github.com/amberframework/granite)
- [Jennifer ORM:] (https://github.com/imdrasil/jennifer.cr)
- [Mosquito:] (https://github.com/mosquito-cr/mosquito)
- [Sidekiq.cr:] (https://github.com/maiha/sidekiq.cr)
- [Spectator:] (https://gitlab.com/arctic-fox/spectator)
- [Webmock.cr:] (https://github.com/manastech/webmock.cr)
- [Ameba: ](https://github.com/crystal-ameba/ameba)
- [-dotenv:] (https://github.com/gdotdesign/cr-dotenv)
- [Clear:]( https://github.com/anykeyh/clear)
- [shields:]( https://github.com/greyblake/shields.cr)

Community

- [Crystal Forum: ](https://forum.crystal-lang.org/)
- [Crystal Discord]: (https://discord.gg/crystal-lang)
- [Reddit r/crystal_programming]: (https://reddit.com/r/crystal_programming)
- [GitHub Crystal Language]: (https://github.com/crystal-lang/crystal)
- [Crystal Weekly Newsletter]:( https://crystalweekly.com/)
- [Awesome Crystal](https://github.com/veelenga/awesome-crystal)

The Final Word

Ruby is a friend who lets you make mistakes and helps you fix them.

Crystal is a friend who stops you from making mistakes in the first place.

I need both kinds of friends. But for my production code, I choose the one that doesn't let me fail.

Now go write some Crystal. And when the compiler yells at you, remember: it's not personal. It's just saving your sleep schedule.
