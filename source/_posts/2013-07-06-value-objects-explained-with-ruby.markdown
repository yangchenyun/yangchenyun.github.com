---
layout: post
title: "Value Objects Explained with Ruby"
date: 2013-07-06 20:42
comments: true
categories: ruby, design pattern, value objects
published: false
---

This article explains the concept of value objects. It first defines and demonstrates various kinds of value objects, then it explains the rule to construct valid ones and consequence of violation. At last, it shows several ways to implement value objects in Ruby. 

Although the examples are written in Ruby, the concept could be easily applied to other languages as well. 

* Table of Content
{:toc}

## What is a Value Object?
A value object as defined in [P of EAA][p-eqq-value-object] is:

     ...their notion of equality isn't based on identity, instead two value objects are equal if all their fields are equal. 

That means value objects which **have the same internal fields must equal** to each other. The value of all fields sufficiently **determines** the equality of a value object.

The simplest examples are the primitive objects - `Symbol`, `String`, `Integer`, `TrueClass`(`true`), `FalseClass`(`false`), `NilClass`(`nil`), `Range`, `Regexp` etc. The value of the object determines its equality. For example, **whenever** `1.0` appears in a program, it should be the equal[^1] to `1.0` because they have the same value.

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

Value objects could also be composed of multiple fields. For example, the [`IPAddr`][ipaddr-lib] class in the standard library have three fields, `@addr`, `@mask_addr` and `@family`. `@addr` and `@mask_addr` defines the IP addresses values and `@family` determines its type as IPv4 or IPv6. `IPAddr` objects which have the same fields are equal to each other.

```ruby
require 'ipaddr'

ipaddr1 = IPAddr.new "192.168.2.0/24"
ipaddr2 = IPAddr.new "192.168.2.0/255.255.255.0"

ipaddr1.inspect
#=> "#<IPAddr: IPv4:192.168.2.0/255.255.255.0>"
ipaddr2.inspect
#=> "#<IPAddr: IPv4:192.168.2.0/255.255.255.0>"
ipaddr1 == ipaddr2 # => true
```

Similarly, money, GPS data, tracking data, date range etc. are all proper candidates for value objects.

The above examples demonstrates the definition of value objects - objects whose equality is based on their internal fields other than their identity.

To guarantee value objects with the same fields will be equal to each other whenever it appears in a program, there is an implicit rule to follow when constructing values objects.

## Rule to Construct Value Objects
The rule to guarantee the equality of value objects across its life cycle is : **the attributes of a value object will remain unchanged from instantiation to the last state of its existence.** "...this is required for the implicit contract that two value objects created equal, should remain equal."[^2] Following this rule, value object should have an immutable interface. 

Sometimes, the need to create variants of value objects might break the rule if without careful implementation. Taking the following `Money` class for example.

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

The `usd` money value object is changed during its life cycle due to the changes to its `@amount` field.

The public setter method `amount=` violates the rule because When it is called, the value shifts from the object's original one.

The correct approach to create variants of value objects is to implement the setter method to initialize a new value object instead of modifying the current one:

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

In this way, once a `Money` value object is created, it will remain the initial state across its life cycle. New variants are created as different value objects instead of making changes to the original one.

## How to Implement a Value Object in Ruby?
Now, it is clear to implement a value object following the above definition and rule:

- Value objects have multiple attributes
- Attributes should be immutable across its life cycle
- Equality is determined by its attributes (and its type)

We've already seen the implementation of `Money` value objects with Ruby normal class syntax. Let's complete the implementation by adding methods for determining equality.

```ruby
class Money
  def ==(other_money)
    self.class == other_money.class &&
    amount == other_money.amount && 
    currency == other_money.currency
  end
  alias :eql? :==

  def hash
    [@amount, @currency].hash
  end
end

usd = Money.new(10, 'USD')
usd2 = Money.new(10, 'USD')
usd == usd2 # => true
```

`eql?` and `==` are standard Ruby methods to determine equality at the object level. By the definition of value objects, the comparison results of all the fields are applied. It is also required to distinguish `Money` from other objects which might have the same attributes. 

For example:

```ruby
AnotherMoney = Struct.new(:money, :currency)
other_usd = AnotherMoney.new(10, 'USD')
usd == other_usd # => false
```

That is accomplished by the line `self.class == other_money.class`.

`hash` method is standard Ruby methods to generate hash value for an object. From the Ruby doc, "...this function must have the property that a.eql?(b) implies a.hash == b.hash." So the implementation uses all the fields' value to generate the hash.

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

It is much more concise than the normal class definition. Attributes are declared through the `Struct.new` interface. Definition of `hash` and `==` methods are omitted because they are inherited from the `Struct` class.

However, one drawback of using `Struct` to define value objects is that they are mutable through the default setter methods and they allow missing of default attributes.

```ruby
usd = Money.new(10, 'USD')
usd.amount = 20
usd.inspect
# => <struct Money amount=10, currency="USD">

invalid_usd = Money.new(1)
invalid_usd.inspect
# => <struct Money amount=1, currency=nil>
```

To keep the conciseness of `Struct`-way but fix its problems, we could use the [`Value`](value-gem) gem which is a `Struct`-like class with all the above problems fixed. Here is an example from the library's demo page:

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

Now, working with value objects in Ruby should be easy and fun.

## Conclusion
Start with definition of value objects, this article shows the usage of value objects from primitive object to more complex domain-specific objects. It also dives into the implicit rule for consistent behavior of value objects during its life cycle.

At last, we've went through several different ways to implement the concept of values objects using Ruby's normal class definition and `Struct` class. Finally, we ended up with a useful gem `Values` to create value objects with ease and conciseness.

Does the explanation of value objects provide you inspiration for writing more elegant code? What's your opinions towards the usage of value objects?

[^1]: "Equal" is used in the meaning of equality(`==` or `eql?`) instead of identify(`equal?`) here.

[^2]: http://en.wikipedia.org/wiki/Value_object

## References
[Value Object][wiki-value-object] on Wikipedia.

Some [discussion][c2-value-object] [on][c2-value-object-hypotheses] [value objects][c2-immutable] at c2.com.

[p-eqq-value-object]: http://martinfowler.com/bliki/ValueObject.html
[ipaddr-lib]: https://github.com/ruby/ruby/blob/ruby_2_0_0/lib/ipaddr.rb#L350-352
[c2-value-object]: http://c2.com/cgi/wiki?ValueObject
[c2-value-object-hypotheses]: http://c2.com/cgi/wiki?ValueObjectHypotheses
[c2-immutable]: http://c2.com/cgi/wiki?ValueObjectsShouldBeImmutable
[value-gem]: https://github.com/tcrayford/Values
[wiki-value-object]: http://en.wikipedia.org/wiki/Value_object