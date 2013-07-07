---
layout: post
title: "Value Objects Explained with Ruby"
date: 2013-07-06 20:42
comments: true
categories: ruby, design pattern
published: false
---

This article explains the concept of value objects. It first defines and demostrates various kinds of value objects, then it explains the hypotheses to define a value object and various violations. At last, it will demostrates the application of value objects in Ruby. Despite the choice of language, the concept could be easily ported to other language as well. 

* Table of Content
{:toc}

## What is a Value Object?
A value object is defined in [P of EAA][p-eqq-value-object] as:

     ...their notion of equality isn't based on identity, instead two value objects are equal if all their fields are equal. 

That means two value objects which have the same internal fields **must be equal** to each other. The value of all fields sufficiently **determines** the equality of a value object.

The simplest examples are the primitive object types - Symbol, String, Integer, true/false, nil, Range, Regexp etc. The value of the object itself determines its equality. That mean, for example, **whenever *** `1.0` appears in a program, it should be taken as equal to `1.0` because they have the same value.

```ruby
var1 = :symbol
var2 = :symbol
var1 == var2  # => true

var1 = 'string'
var2 = 'string'
var1 == var2  # => true

var1 = 1.0
var2 = 1.0
var1 == var2  # => true

var1 = true
var2 = true
var1 == var2  # => true

var1 = nil
var2 = nil
var1 == var2  # => true

var1 = /reg/
var2 = /reg/
var1 == var2  # => true

var1 = 1..2
var2 = 1..2
var1 == var2  # => true

var1 == [1, 2, 3]
var2 == [1, 2, 3]
var1 == var2  # => true

var1 == { key: 'value'}
var2 == { key: 'value'}
var1 == var2  # => true
```

These are examples of value objects with one field. 

There are more complex value objects with multiple data fields. For example, the [`IPAddr`][ipaddr-lib] class in the standard library construct object with three fields, `@addr`, `@mask_addr` and `@family`. `@addr` and `@mask_addr` defines the IP addresses values and `@family` determines whether it is IPv4 or IPv6. `IPAddr` objects have the same fields are equal to each other.

```ruby
require 'ipaddr'

# construct with 
ipaddr1 = IPAddr.new "192.168.2.0/24"
ipaddr2 = IPAddr.new "192.168.2.0/255.255.255.0"
ipaddr1 == ipaddr2 # => true
```

Money, GPS data, tracking data, datetime etc. are all good candidates for value objects.

The above examples demostrates the definition of value objects - objects whose equality is based on their internal fields other than their identity.

To guarantee a value object with the same fields would be equal whenever it appears in the program, there is implicit rule to construct values objects.

## Rule to Construct Value Objects
The rule is that: **the attributes of a value object will remain unchanged from instantisation to the last state of its existence.** "...this is required for the implicit contract that two value objects created equal, should remain equal."[^1] Following this rule, value object will have an immutable interface. 

Sometimes, the need to create variants of value objects will break the rule without careful implementation. Taking the following `Money` class for example.

```ruby
class Money
  attr_accessor :currency, :amount

  def initialize(amount, currency)
    @amount = amount
    @currency = currency
  end
end

usd = Money.new(10, 'USD')
#<Money:0x007f987f283b50 @amount=10, @currency="USD">

usd.amount = 20
usd.inspect
#<Money:0x007f987f283b50 @amount=20, @currency="USD">
```

The public setter method `amount=` violates the rule because it changes the `@amount` attribute within a `Money` object. When it gets called on a value object, the value object shifts from its original value.

A better approach to create variants of value objects is to implement the setter method to initialize a new value object.

```ruby
class Money
  # remove the public setter interface
  attr_reader :currency, :amount

  def initialize(amount, currency)
    @amount = amount
    @currency = currency
  end

  # a setter method to return a new value object
  def amount=(other_amount)
    Money.new(other_amount, currency)
  end
end

usd = Money.new(10, 'USD')
usd.inspect
#<Money:0x007f9672753ba8 @amount=10, @currency="USD">

other_usd = (usd.amount = 20)
usd.inspect
#<Money:0x007f9672753ba8 @amount=10, @currency="USD">
```

In this way, once a `Money` value object is created, it will remain the initial state across its lifecycle. New variants are created as different value objects instead of making changes to the original one.

## How to Implement a Value Object in Ruby?
It is clear to implement a value object following the above definition and rule:
- Value objects have multiple attributes
- Attributes should be immutable across its life cycle
- Equality is determined by its attributes (and its type).

We've already seen the implementation of `Money` value objects with Ruby normal class syntax. Let's complete the implementation by adding methods to determine equality.

```ruby
class Money
  def ==(other_money)
    self.class == other_money.class &&
    amount == other_money.amount && 
    currency == other_money.currency
  end

  def hash
    [@amount, @currency].hash
  end
end

usd = Money.new(10, 'USD')
usd2 = Money.new(10, 'USD')
usd == usd2 # => true
```

The line `self.class == other_money.class` distinguishes `Money` from other objects which might have the same equality.

```ruby
AnotherMoney = Struct.new(:money, :currency)
other_usd = AnotherMoney.new(10, 'USD')
usd == other_usd # => false
```

Besides the normal class syntax, `Struct` as shown in the last example is a very concise way to build value objects. Here is an example to implement the same `Money` value objects using `Struct`:

```ruby
class Money < Struct.new(:amount, :currency)
  def amount=(other_amount)
    Money.new(other_amount, currency)
  end
end

usd = Money.new(10, 'USD')
usd2 = Money.new(10, 'USD')

usd.hash == usd2.hash # => true
usd == usd2 # => true
```

The `hash` and `==` methods are ommited because they are defined within the `Struct` class.

One drawback of using `Struct` to define value objects are that they are mutable through the default setter methods and they allow missing of default attributes.

To keep the conciseness of `Struct` but fix its problems, we could use the [`Value`](value-gem) library. Here is an example from the demo page:

```ruby
Point = Value.new(:x, :y)
Point.new(1)
# => ArgumentError: wrong number of arguments, 1 for 2
# from /Users/tcrayford/Projects/ruby/values/lib/values.rb:7:in `block (2 levels) in new
# from (irb):5:in new
# from (irb):5
# from /usr/local/bin/irb:12:in `<main>

p = Point.new(1, 2)
p.x = 1
# => NoMethodError: undefined method x= for #<Point:0x00000100943788 @x=0, @y=1>
# from (irb):6
# from /usr/local/bin/irb:12:in <main>
```

## Conclusion
Start with definition of value objects, this article shows the usage of value objects from primitive object types to more complex domain-specific objects. It also dives into the implicit rule for consistent behavior of value objects.

At last, we've went through several different ways to implement the concept of values objects using Ruby and ended with a useful gem to create value objects with ease and conciseness.

Does value objects provide you inspiration for more elegant code? What's your experience with value objects?

## References
[Value Object on Wikipedia][wiki-value-object]
[Value Object][c2-value-object], [Value Object Hypotheses][c2-value-object-hypotheses], [Value Objects Should Be Immutable][c2-immutable] at c2.com.

[^1]: http://en.wikipedia.org/wiki/Value_object


[p-eqq-value-object]: http://martinfowler.com/bliki/ValueObject.html
[ipaddr-lib]: https://github.com/ruby/ruby/blob/ruby_2_0_0/lib/ipaddr.rb#L350-352
[c2-value-object]: http://c2.com/cgi/wiki?ValueObject
[c2-value-object-hypotheses]: http://c2.com/cgi/wiki?ValueObjectHypotheses
[c2-immutable]: http://c2.com/cgi/wiki?ValueObjectsShouldBeImmutable
[value-gem]: https://github.com/tcrayford/Values
[wiki-value-object]: http://en.wikipedia.org/wiki/Value_object















