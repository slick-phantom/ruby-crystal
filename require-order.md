# 06 - Require Order and Project Structure

What Rubyists Need to Know (70% of This)

The Big Lie Ruby Told You

Ruby lied to you about requires. It told you order doesn't matter.

```ruby
# Ruby - this works fine
require "./bot"
require "./client"  # Even if bot needs client
```

Ruby loads everything into memory, then figures it out later. It's forgiving. It's flexible. It's also slow.

Crystal is not forgiving.

```crystal
# Crystal - order MATTERS
require "./client"  # Must come FIRST
require "./bot"     # Because bot uses Client
```

Why Order Matters

Crystal compiles files in the order you require them. When it sees:

```crystal
require "./bot"
```

It compiles bot.cr right then. If bot.cr mentions Client and client.cr hasn't been compiled yet, Crystal has no idea what Client is.

```crystal
# bot.cr
class Bot
  getter client : Client  # What's Client? Not compiled yet!
end
```

The compiler doesn't look ahead. It doesn't guess. It just fails.

The Simple Rule

Require dependencies before dependents.

```
# Wrong order
require "./core/bot"        # Bot needs Api::Client
require "./api/client"      # Client loads too late

# Right order
require "./api/client"      # Client loads first
require "./core/bot"        # Bot can use Client now
```

Project Structure That Works

After 64 pipelines, here's what actually works:

```
src/
├── telecr.cr              # Main entry point (requires everything)
├── api/
│   ├── client.cr          # Low-level API (required first)
│   └── types.cr           # Type definitions (required early)
├── core/
│   ├── bot.cr             # Depends on api, markup, etc.
│   ├── context.cr
│   ├── handler.cr
│   └── middleware.cr
├── markup/
│   ├── keyboard.cr
│   └── inline.cr
├── plugins/
│   ├── rate_limit.cr
│   └── upload.cr
├── session/
│   └── middleware.cr
└── webhook/
    └── server.cr
```

The Main Entry Point

Your telecr.cr should look like this:

```crystal
# telecr.cr - Main entry point
require "./api/*"        # Load API first (lowest level)
require "./core/*"       # Load core next
require "./markup/*"     # Then markup
require "./plugins/*"    # Then plugins
require "./session/*"    # Then session
require "./webhook/*"    # Finally webhook (depends on everything)

module Telecr
  VERSION = "0.1.0"
  # ...
end
```

Order matters. API code doesn't depend on anything else, so it goes first. Webhook code depends on everything, so it goes last.

The Star Require

require "./api/*" loads all .cr files in the api directory. But here's the catch: it loads them in ALPHABETICAL order.

```crystal
# If you have:
api/
├── client.cr
├── types.cr
└── utils.cr

# They load in this order:
# 1. client.cr (c before t and u)
# 2. types.cr (t after c)
# 3. utils.cr (u after t)
```

If client.cr needs something from types.cr, alphabetical order might not help you. In that case, require specific files:

```crystal
require "./api/types"    # Load types first
require "./api/client"   # Then client
require "./api/utils"    # Then utils
```

Circular Dependencies (The Nightmare)

Sometimes A needs B and B needs A. This is called a circular dependency.

```crystal
# bot.cr
require "./context"
class Bot
  def process(ctx : Context)  # Bot needs Context
  end
end

# context.cr
require "./bot"
class Context
  def initialize(@bot : Bot)  # Context needs Bot
  end
end
```

This will NOT work. Crystal can't resolve this. You have two options:

Option 1: Forward declarations

```crystal
# bot.cr
class Bot  # Don't require context here
  # ...
end

# context.cr
require "./bot"  # Require bot here
class Context
  def initialize(@bot : Bot)
  end
end
```

Option 2: Restructure your code

Move the circular parts to a third file, or use abstract classes/interfaces.

The Require Path Trick

When you're deep in subdirectories, paths can get confusing:

```crystal
# In src/core/bot.cr, you need to require something from src/api
require "../api/client"  # ../ goes up one level from core/
```

The path is relative to the CURRENT FILE, not the project root.

```
src/
├── core/
│   └── bot.cr           # Here, "../api/client" works
├── api/
│   └── client.cr
└── telecr.cr            # Here, "./api/client" works
```

I drew myself a map of relative paths after pipeline 22.

The Require Order Debug Technique

When you get "undefined constant" errors, it's usually require order. Here's how I debug:

1. Look at the error: "undefined constant Api::Client" means Api::Client wasn't loaded yet.
2. Find where it's used: In bot.cr, line 11.
3. Check require order: Is api/client.cr required BEFORE core/bot.cr?
4. Add debug prints:

```crystal
# At the top of api/client.cr
puts "LOADING: api/client.cr"

# At the top of core/bot.cr
puts "LOADING: core/bot.cr"
```

If you see "LOADING: core/bot.cr" before "LOADING: api/client.cr", you know the order is wrong.

The Shard Dependencies

Your shard.yml also affects require order indirectly:

```yaml
dependencies:
  ameba:
    github: crystal-ameba/ameba
```

These are loaded BEFORE your code. You can rely on them being available.

The Spec Helper

Your spec/spec_helper.cr needs to require everything:

```crystal
# spec/spec_helper.cr
require "../src/**"  # Loads everything recursively
require "spectator"
require "webmock"
```

The ** means "all files in all subdirectories". This loads everything, but the order is alphabetical by path. If your specs have circular dependencies, they'll fail.

My Problems With Require Order (30% of This)

Pipeline 8: The Api::Client Mystery

```
In src/core/bot.cr:11: undefined constant Api::Client
```

I spent an hour staring at this. The file existed. The namespace was correct. Everything looked fine.

The problem? I was requiring core/bot.cr before api/client.cr in my main file. Took me 45 minutes to notice.

Pipeline 17: The Circular Dependency Spiral

```
Error: can't find file './context' relative to '/src/core/bot.cr'
```

I had bot.cr requiring context.cr, and context.cr requiring bot.cr. Crystal just gave up. I had to split out an interface.

Pipeline 29: The Relative Path Confusion

```
Error: can't find file '../api/client' in /src/core/bot.cr
```

I was in src/core/bot.cr and used require "./api/client" (which looks in src/core/api/). Should have been require "../api/client". The dots matter.

Pipeline 41: The Star Require Alphabet Problem

```
Error: undefined constant Api::Types
```

client.cr needed types.cr, but ./api/* loaded client.cr first (alphabetical order). I had to split into explicit requires.

Pipeline 52: The Spec Helper Chaos

My specs passed locally but failed in CI. Turned out my local environment had files cached in a different order. The CI loaded files alphabetically, exposing a hidden dependency.

What Rubyists Actually Need To Remember

1. Require Order Matters

```
require "./low_level"    # Things that don't depend on anything
require "./middle_level" # Things that depend on low_level
require "./high_level"   # Things that depend on everything
```

2. Main Entry Point Controls Everything

Your main file should require everything in the right order. Users only need to require your main file.

```crystal
# User does:
require "telecr"
# Your main file handles all internal requires
```

3. Paths Are Relative To Current File

```crystal
require "./file"           # Same directory
require "../file"          # Parent directory
require "../../file"       # Grandparent directory
```

4. Circular Dependencies Are Evil

If A needs B and B needs A, you have a design problem. Fix it.

5. Use ** Carefully

```crystal
require "./**"  # Loads everything (order undefined)
```

Use this only in spec helpers or when order truly doesn't matter.

6. Debug With Print Statements

```crystal
puts "Loading: #{__FILE__}"
```

See what's loading and when.

7. The Error Message Tells You Where

"undefined constant Api::Client in src/core/bot.cr" means:

· api/client.cr isn't loaded yet
· Or it's loaded with the wrong namespace
· Or it's loaded after bot.cr

The Trade-Off

Ruby lets you require files in any order because it's lazy. It loads everything, then figures out dependencies at runtime.

Crystal requires order because it compiles ahead of time. It needs to know everything BEFORE it runs.

Ruby is easier to start with. Crystal is harder to mess up in production.

After 64 pipelines, I've learned to structure my projects with clear dependency layers. It's more work upfront, but I never get "undefined constant" errors in production.

Next: 07 - Common Errors Decoded
