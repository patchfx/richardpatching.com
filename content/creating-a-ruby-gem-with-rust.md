+++
title = "Creating a Ruby Gem with Rust"
date = 2020-05-22

[taxonomies]
tags = ["rust", "ruby"]
+++

In this post you will learn how to create a Rust library and access
it with a Rubygem.

I'm a big fan of the Ruby programming language, it's been a one of
my primary programming languages for the last 12 years. It's an
extremely expressive language that really lends itself to writing
elegant and easy to read code. One drawback however, is it's
performance. In fact it's probably the biggest criticism of the
language that I hear. If you are writing code where performance is
critical, then Ruby might not be the right tool for the job.
This doesn't necessarily mean you have to switch technologies
entirely, or create another microservice, you can in fact write
a native library and interface it with Ruby.

A language that I have grown to love over the last few years is
[Rust](https://rust-lang.org). It's a systems programming language
that's focused on performance and memory safety, without the
garbage collection. Ideal for creating libraries and tools where
performance is important.

In this tutorial we will create a very simple Rust library that will
print `Hello World from Rust!`. We will wrap this in a Rubygem and
have a Ruby interface to the Rust code.

# Create a new gem with bundler

Lets go ahead and create a new Ruby gem template using bundler.

```
bundle gem helloworld
cd helloworld
```

NOTE: There are a few fields in the created `helloworld.gemspec` with `TODO` tags.
You will need to edit those first before the gem can build.

# Create a new Rust library with Cargo

Rust has `cargo`, which you can think of as the equivalent to bundler with
Ruby. Using cargo we can create `crates`, which are just like gems.

To create a new crate with cargo is extremely easy.

```
cargo init --lib
```

Initialising the new Rust library creates a `Cargo.toml` file, this is much
like the Ruby `Gemfile` and contains information about the library and it's
dependencies.

Add the following line to the `Cargo.toml` file to tell Rust that this is a
dynamic system library and we will be using it from another language.

```
[lib]
crate-type=["cdylib"]
```

The `Cargo.toml` file should now look like the following.

```
[package]
name = "helloworld"
version = "0.1.0"
authors = ["Your Name <example@test.com>"]
edition = "2018"

[lib]
crate-type=["cdylib"]

[dependencies]
```

With our basic gem and crate templates in place, we can start connecting them
so that we can run some Rust code from Ruby.

# Writing some Rust code

This code is going to simply return `"Hello World from Rust!"`. Open the `src/lib.rs`
file remove the boiler plate code, replacing it with our hello world function.

```
#[no_mangle]
pub extern fn hello_world() {
    println!("Hello World from Rust!");
}
```

The above function, is public (`pub`) and accessible externally (`extern`).
It has no return type, but simply uses `println!` to write a string to our
output.

# Let's build the Rust library

We are going to build the Rust library and make it's shared object (`.so`)
file accessible to our Ruby gem.

```
task :rust_build do
  `cargo rustc --release`
  `mv -f ./target/release/libhelloworld.so ./lib/helloworld/helloworld.so`
end

task :build => :rust_build
task :default => :build
```

Our default `rake` task is set to build a release version of our Rust crate
and move it into the lib directory so that we can use it in our gem.

Running `rake` now should yield the following output.

```
$ rake
Finished release [optimized] target(s) in 0.01s
helloworld 0.1.0 built to pkg/helloworld-0.1.0.gem.
```

# Foreign Function Interface (FFI)

The Ruby FFI library is going to allow us to dynamically link to our Rust
crate and bind the functions we want to expose, in this case it will be
the `hello_world()` function.

Open the `helloworld.gemspec` file and add the FFI dependency.

`spec.add_runtime_dependency "ffi"`

Then run `bundle` to install. Now that we have our FFI dependency in place
we can write some ruby code to interface with our Rust crate!

Create a new file `lib/helloworld/ffi.rb` and add the following lines.

```
require 'ffi'

module Helloworld
  class FFI
    extend ::FFI::Library
    lib_name = "helloworld.#{::FFI::Platform::LIBSUFFIX}"
    ffi_lib File.expand_path(lib_name, __dir__)
    attach_function :hello_world, [], :void
  end
end
```

In the above code, we extend the FFI library, create a `lib_name` variable
that points to our `hellworld.so` file and then load it. Finally we attach
our Rust crate function `hello_world`, with no arguments (`[]`), and a
return type of `void`. Remember that Rust is a statically and strongly typed
language, as opposed to Ruby.

# The final step!

In our `lib/helloworld.rb` file add the following code

```
require "helloworld/ffi"
require "helloworld/version"

module Helloworld
  class Error < StandardError; end

  def self.say_hello
    FFI.hello_world
  end
end
```

We require our ffi file and then call the Rust crates `hello_world` function.
It's time to test the output! Run the `bin/console` and call the Ruby method
we created `Helloworld.say_hello` and you should see the output from the Rust
crate function!

```
$ bin/console
irb(main):001:0> Helloworld.say_hello
Hello World from Rust!
=> nil
```

Although this is an extremely basic example, it should set a good foundation
from which to explore Rust and Ruby further. There are many instances where
you might need extra performance, that you just don't get from Ruby and Rust
is an extremely good candidate for that.

You can view the full source code on [GitHub](https://github.com/patchfx/helloworld)
