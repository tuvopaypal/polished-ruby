# Polished Ruby

## Chapter 1: Getting the Most out of Core Classes

1. How are nil and false different from all other objects? nil and false are the only objects Ruby treats as false in conditional expressions.
2. Are all standard arithmetic operations using two BigDecimal objects exact? No. Division is not exact if the division results in a repeating decimal.
3. Would it make sense for Ruby to combine symbols and strings? No. Symbols represent identifiers and strings represent text or data, so they serve different purposes.
4. Which uses less memory for the same data-hash, or Set? Hash, but only slightly. Set in Ruby 3.0 is implemented internally using a hash.
5. What are the only two core methods that return a new instance of Class? Class.new and Struct.new.

## Chapter 2: Designing Useful Custom Classes

1. Does creating a custom class make sense if you need both information hiding and custom behavior? In most cases, yes.
2. Which SOLID principle is almost impossible to implement in Ruby? The open-closed principle is almost impossible to implement in Ruby.
3. Is it useful to create classes that the user will not use directly? This depends. If it simplifies the implementation of a class the user uses, then yes.
4. How often does it make sense to use custom data structures in Ruby? Rarely. In most cases, you should use arrays and hashes, at least until you are dealing with a very large amount of data.

## Chapter 3: Proper Variable Usage

1. Is it always a good idea to use long descriptive names for local variables? No. However, it is generally a good idea if the scope of the local variable is very long, such as a local variable defined at the top of a long method or class definition containing many blocks.
2. When using instance variables for caching, why is it important that the object be frozen? Because if the object is not frozen, the cached value could become invalid, showing the previous value instead. It is possible to handle this by clearing caches when the object is modified.
3. A constant named SomeValue probably contains an instance of what type of Ruby class? A Module or Class.
4. When should you use class variables? Never.
5. Should you always avoid using global variables? Not always, but it is best to only use the built-in global variables and not add your own global variables.

## Chapter 4: Methods and Their Arguments

### Understanding that there are no class methods, only instance methods

Ruby does not have class methods or module methods as separate concepts – it only has instance methods. Every method that you would think of as a class or a module method is just an instance method of the class or module's singleton class.

### Delegating to other objects

Let's say you want to delegate A#foo to call b.foo. You can use either of the manual delegation approaches discussed previously, such as the explicit argument delegation approach:

```rb
class A
  attr_accessor :b

  def initialize(b)
    @b = b
  end

  def foo(...)
    b.foo(...)
  end
end
```

Using the forwardable library, you can handle this method delegation without defining a method yourself:

```rb
require 'forwardable'

class A
  extend Forwardable

  def_delegators :b, :foo
end
```

1. If class methods are instance methods, what class contains those instance methods? The class's singleton class, or a module that is included in it (for example, via Object#extend) or prepended to it.
2. How are method call frequency and method naming related? Methods called most frequently should generally have shorter names.
3. What's the best argument type to use for an argument that will rarely be used? An optional keyword argument, because using an optional positional argument will make adding future optional positional arguments awkward.
4. If you make a mistake with method or constant visibility, what gem helps you convert a public method or constant into a private one, while also issuing warnings if it's accessed via a public interface? The deprecate_public gem.
5. What's the best way to delegate all arguments to another method so that it works correctly in Ruby 2.6, 2.7, and 3.0? Accept *args and pass *args to the other method. After method definition, mark the method using ruby2_keywords if supported.

## Chapter 5: Handling Errors

### Handling errors with exceptions

One of the principles is that when you are designing an API, you should not only design the API to be easy to use, but you should also attempt to design the API to be difficult to misuse. This is the principle of misuse resistance. A method that does not raise an exception for errors is easier to misuse than a method that raises an exception for errors.

### Understanding more advanced retrying

```rb
require 'net/http'
require 'uri'

uri = URI("http://example.local/file")

retries = 0

begin
  Net::HTTP.get_response(uri)
rescue SocketError, SystemCallError, Net::HTTPBadResponse
  retries += 1
  raise if retries > 3
  retry
end
```

1. What is the main advantage of using return values to signal errors? Better performance, and it can be simpler if you always remember to check for errors.
2. What is the main advantage of using exceptions to signal errors? Unlike using return values, you must handle the error via an exception handler, so you won't silently ignore errors.
3. Why is it important not to retry transient errors immediately? In general, most errors that are transient will happen again if retried immediately. By waiting and then retrying, you are more likely to be successful.
4. When is a good time to add a subclass of an existing exception class? When you need to handle a particular type of error differently than other raised exceptions that currently use the same exception class.

## Chapter 6: Formatting Code for Easy Reading

### Enforcing consistency with RuboCop

RuboCop implements many checks, called cops, and most of the cops are enabled by default, even those not related to syntax. It can be tempting to use the RuboCop defaults, since it is otherwise daunting to go through every cop enabled by default and decide whether you want to enforce it. Thankfully, RuboCop has a solution for this, which is the configuration parameter, AllCops:DisabledByDefault. Using this configuration parameter, you can only enable the syntax checks that you believe will be helpful for your library.

1. Do all Ruby programmers want to enforce syntactic consistency? No.
2. If you are using RuboCop to enforce syntactic consistency, what's one RuboCop configuration parameter you should definitely use? AllCops:DisabledByDefault.
3. Why does enforcing arbitrary limits usually result in worse code? If there was a better approach that was within the arbitrary limit, it would have already been used.
4. What Ruby command-line option allows you to check a file for common formatting issues? ruby -wc
5. When should you worry about code formatting? When it negatively impacts the understandability of your code.

## Chapter 7: Designing Your Library

1. What should you focus on when first designing the library? You should focus on how the user will use your library.
2. If you currently don't have a need for flexibility in your library, is it a good idea to increase the size of your library to add flexibility? You should focus on how the user will use your library.
3. What is the main issue with having many similar methods with minor differences in behavior? It increases cognitive complexity for the user of the library.

## Chapter 8: Designing for Extensibility

1. What is the idiomatic way to add behavior to an individual object in Ruby? Using extend to include the module in the object's singleton class.
2. If you have a medium or large library, what's the advantage of designing a plugin system for it? It allows the user to only load the features they need, so they don't pay a performance or memory cost for features they don't use. Additionally, it generally makes maintenance easier for the library's maintainer.
3. What is the advantage of freezing the core classes when running a Ruby application? It ensures that no code attempts to modify the core classes while the application is running.

## Chapter 9: Metaprogramming and When to Use It

1. When is it a bad idea to implement an abstraction via metaprogramming? When it makes the code more difficult to understand, instead of easier to understand.
2. What is the most common reason to deal with singleton classes of singleton classes? When you need to do metaprogramming in a singleton class.
3. When should you use eval-based metaprogramming instead of block-based metaprogramming? Only when you must for performance reasons.
4. When should you use method_missing? Only when the instances of the class need to respond to any method or the number of methods they must respond to is very large.

## Chapter 10: Designing Useful Domain-Specific Languages

1. What is the main advantage of using a DSL for configuring libraries? It simplifies configuration for both the user and the maintainer by centralizing all configuration aspects in a single block.
2. How can you implement a DSL that works both as a normal block DSL and an instance_exec DSL? By checking the arity of the block, and if the block accepts an argument, yielding the DSL object. Otherwise, using instance_exec to evaluate the block in the context of the DSL object.
3. Of the various reasons given in this chapter for using a DSL, which is the most likely to cause problems for the user and the least likely to add value? Implementing a DSL purely to avoid code verbosity is the most likely to cause problems and the least likely to add value.

## Chapter 11: Testing to Ensure Your Code Works

### Realizing that 100% coverage means nothing

Line coverage is the simplest type of coverage. It allows you to check whether a line of code was ever executed during the testing process. This is important because any line without coverage during testing means the line was never tested at all. Now, just because the line was covered doesn't mean that the result of the line was actually tested. All it means is that at some point during testing, code somewhere on the line was executed.

Branch coverage takes the same idea as line coverage but takes it a step farther. It ensures that all branches in the code were taken. Suppose if you have the following Foo class with a method named branch:

```rb
class Foo
  attr_accessor :bar

  def branch(v)
    v > 1 ? bar : baz
  end

  def baz; raise; end
end
```

And you add a test for the method as follows:

```rb
describe Foo do

  it "#branch should return the value of bar" do
    foo = Foo.new
    foo.bar = 3

    (foo.branch(2)).must_equal 3
  end
end
```

This test is OK, but it is incomplete. From a line coverage perspective, this will show 100% line coverage, as all lines are covered. This will even consider the line with the baz definition covered, even though baz is never called. This is because the line is executed when the Foo class definition is evaluated. The line v > 1 ? bar : baz will also show as covered, even though the test only executes the bar call and not the baz call. This is where branch coverage comes in. With branch coverage enabled, the coverage will show that the else branch of code on the line was not covered (the branch taken when v is not greater than 1).

Method coverage is similar to branch coverage, but it only tells you whether the method was executed during the tests. Method coverage with the previous test would be able to tell you that the baz method was never called.

1. How do you check the syntax of a Ruby file and report errors and warnings? ruby -wc filename
2. Is behavior-driven development a good idea if the programmers are going to be writing the specifications? In general, using behavior-driven development is a waste of time if the programmers are writing the specifications, since the programmers could more easily write the executable test code directly, compared to writing the specifications and maintaining the code that converts the specification to executable test code.
3. Is it always bad to use abstractions in test code? Not always, it depends on the type of abstraction. Moving setup code to methods and using enumerables to define multiple test methods are both good uses of abstractions in test code.
4. Why might you prefer model testing to unit testing? Model testing is in general more reliable and less likely to result in false positives and false negatives compared to unit testing.
5. What does 100% code coverage mean? Nothing. But less than 100% code coverage means some code is not being tested at all.

## Chapter 12: Handling Change

1. What are three common reasons to refactor in a library? To simplify the library, to improve performance, and to add extensibility points. There are other good answers, such as being forced to deal with changes forced by external dependencies or to clean up buggy code.
2. What is the most important prerequisite before starting refactoring? A thorough test suite that you can rely on.
3. When does it make sense to extract multiple methods instead of a single method? When the multiple methods being extracted are called in separate places, and one extracted method is not the only place the other extracted method is called.
4. In the best case, what's the fastest approach to implementing a feature that requires refactoring? The cowboy approach of refactoring while implementing the feature. However, this approach is also the riskiest.
5. What keyword arguments should you pass to Kernel#warn for deprecation warnings? uplevel: 1 to show the caller's location and category: :deprecated to flag the warning as a deprecation warning.

## Chapter 13: Using Common Design Patterns

1. What design pattern does Ruby's garbage collector use? The object pool pattern.
2. How do you implement lazy evaluation if using the constant approach to implementing the singleton pattern? Use autoload to ensure the constant isn't loaded and evaluated until it is referenced.
3. When is it not a good idea to implement the null object pattern? Whenever you want to treat the null object differently than the real object, or when performance is critical.
4. Why would you want to use the hash approach to the visitor pattern instead of the case approach? For a large number of different types, the hash approach will perform better.
5. What's the significant difference between the adapter and strategy patterns? The adapter pattern wraps another object, while the strategy pattern does not. Both are designed to provide a unified interface to a different implementation.

## Chapter 14: Optimizing Your Library

1. What's the most important thing to do before optimizing your library? Make sure you really need to optimize. Only optimize after you have identified a bottleneck.
2. After you have identified a bottleneck, what steps should you take before optimizing your library? First, create a use case, then profile it to determine why it is slow, then create a benchmark to determine baseline performance.
3. If you are creating a lot of instances of a specific class, what is the fastest general way to speed that up? Move as much code as possible out of the class's initialize method.
4. If profiling your use case does not help you identify the slow code, where's the best place to look first? The best place to start optimizing if profiling doesn't help is to try to avoid object allocation as much as possible.

## Chapter 15: The Database Is Key

1. Why is database design the most important part of your application design? Because data is in general far more important than code and lasts longer than code.
2. When should you consider denormalizing your database? Only when you absolutely must for performance reasons.
3. What is a good reason to use a database trigger? There are many good answers to this question, but the most common is to enforce data consistency in related tables.
4. If your primary consideration when developing is how many external libraries you can use to save development time, which model layer is probably best? ActiveRecord.
5. Which model layer uses a fail-closed design for handling model errors during saving? Sequel

## Chapter 16: Web Application Design Principles

Rails come with many additional features, including the following:

-   Action Cable: A framework for real-time communication between the server and client using WebSockets
-   Action Mailbox: A framework for processing incoming emails
-   Action Mailer: A framework for sending emails
-   Action Text: A framework for editing and displaying rich text in web pages
-   Active Job: A framework for abstracting the handling of background jobs
-   Active Storage: A framework for processing uploaded files and media

Another advantage of Rails is that it focuses on convention over configuration. Rails is very much designed around the concept of staying on the rails, or doing things the way Rails wants you to do them. If you deviate from that approach, or go off the rails, you will have a significantly more difficult time using Rails. Productive use of Rails involves structuring your application to fit Rails, not the other way around.

1. Is it better to use a client-side or server-side development approach when designing an application that most involves data input via HTML forms? A server-side development approach.
2. Of the four most popular Ruby web frameworks, which is the fastest? Roda.
3. When should you prefer to use the nested approach for designing URL paths? When you can use information from earlier parts of the path during routing and/or request handling.
4. If your application only has a single user interface, should you consider using the island chain approach to structure the application? No.

## Chapter 17: Robust Web Application Security

For applications written in C, most security issues tend to be low-level security issues. These security issues are caused by things such as buffer overflows, integer overflows or underflows, and use-after-free (UAF) vulnerabilities. Ruby itself is mostly written in C, so a bug in Ruby itself could result in one of the previous security issues affecting Ruby.

### Performing access control at the highest level possible

Many security issues in Ruby web applications are due to missing authentication or authorization checks when processing a request. This is especially common in web frameworks that separate routing from request handling and use some type of conditional before hook for performing access control. Let's say you have a Rails controller that uses a before hook for access control:

```rb
class FooController < ApplicationController

  before_action :check_access

  def index
  end

  def create
  end

  private def check_access
  end
end
```

This is probably not likely to result in access control vulnerabilities since the access is checked for every action. However, let's say you set the before_action hook so that it's conditional, like so:

```rb
class FooController < ApplicationController

  before_action :check_access, only: [:create]
end
```

Here, you really need to be careful when adding any request handling methods to FooController. If they require access control and you don't add them to the :only option, then you have probably introduced a security vulnerability. This is because the before_action :only option uses a fail-open design. It's much better to use the before_action :except option instead:

```rb
class FooController < ApplicationController

  before_action :check_access, except: [:index, :bars]
end
```

It's still bad if you do not update the :except option when adding a request handling method to the controller. However, if you forget, you will end up requiring access control for a page that does not need it, which is a fail-closed design.

### Script injection

One way to mitigate the damage that a script injection attack can do is to set a strict Content-Security-Policy header for all the pages that are served by your application. For example, a Content-Security-Policy header with script-src self; will allow you to load JavaScript files from your site, but will not allow the use of inline JavaScript code. With a strict Content-Security-Policy header, the only way for an attacker to get their JavaScript running on your site would be to find a separate vulnerability that allows them to serve the JavaScript as a separate file from your site, which, in general, is much more difficult than exploiting a script injection vulnerability.

1. Why are integer overflow and underflow vulnerabilities less likely in Ruby compared to C? Ruby has an Integer type that supports integers of arbitrary sizes.
2. Why should you prefer whitelist security to blacklist security? Blacklist security is a fail-open design that is likely to lead to future security vulnerabilities, while whitelist security is a fail-closed design.
3. Why is conditional access control challenging when you're using Sinatra? Because Sinatra does not offer a simple way to implement a fail-closed design.
4. What response header should you use to mitigate script injection vulnerabilities? The Content-Security-Policy header, with a strict script_src setting that does not allow inline JavaScript.
5. For high-security environments where filesystem access limiting and system call limiting are required, what's the best operating system to use when deploying Ruby web applications? OpenBSD, since the pledge gem can be used to enforce both limited filesystem access and limited system call access.
