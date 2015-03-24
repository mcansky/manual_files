# For each and threads

Some times you don't remember why Rubyists prefer to use *each* rather than *for*. And then one day you end up facing the reason.

## The example

Let's say you want to go through a series of items to treat :

```ruby
queues = %w(one two)
pool = []

for queue in queues do
  pool << Thread.new { puts queue }
end

pool.each { |t| t.join }
```

Here is how it's going to look in irb :

```
irb(main):001:0> queues = %w(one two)
=> ["one", "two"]
irb(main):002:0> pool = []
=> []
irb(main):003:0>
irb(main):004:0* for queue in queues do
irb(main):005:1*   pool << Thread.new { puts queue }
irb(main):006:1> end
twotwo

=> ["one", "two"]
irb(main):007:0>
irb(main):008:0* pool.each { |t| t.join }
```

Yep the *puts* call outputs 'two' twice.

## Solution

So out of simple "ruby syntax" mind I replaced the for loop with an each :

```ruby
queues = %w(one two)
pool = []

queues.each do |queue|
  pool << Thread.new { puts queue }
end

pool.each { |t| t.join }
```

Here is the result in irb :

```
irb(main):010:0* queues = %w(one two)
=> ["one", "two"]
irb(main):011:0> pool = []
=> []
irb(main):012:0>
irb(main):013:0* queues.each do |queue|
irb(main):014:1*   pool << Thread.new { puts queue }
irb(main):015:1> end
twoone
=> ["one", "two"]
irb(main):016:0>
irb(main):017:0* pool.each { |t| t.join }
```

This time the output is "ok", it's not in the right order (oh threads) but the two names are printed properly.

## Another experiment with for

Just to understand a bit more what happened let's add a sleep in the for loop.

```ruby
queues = %w(one two)
pool = []

for queue in queues do
  pool << Thread.new { puts queue }
  sleep 1
end

pool.each { |t| t.join }
```

Let's run it in irb :

```
irb(main):018:0> queues = %w(one two)
=> ["one", "two"]
irb(main):019:0> pool = []
=> []
irb(main):020:0>
irb(main):021:0* for queue in queues do
irb(main):022:1*   pool << Thread.new { puts queue }
irb(main):023:1>   sleep 1
irb(main):024:1> end
one

two
=> ["one", "two"]
irb(main):025:0>
irb(main):026:0* pool.each { |t| t.join }
```

This time the output was correct, not even in a bad order.

## Reasons

The reasons are tied with the difference between for and each and the way threads work.

You can read http://stackoverflow.com/questions/5677081/ruby-what-is-the-difference-between-a-for-loop-and-an-each-loop for the difference between for and each. The gist of it is that *for* is a syntax construct that will keep its scope between each loop while *each* handles a method call with a copy of the data for each loop.

So why does the *for* with the sleep works ? That part demonstrates how threads work. Remember that the main thread of the program will run until the end or until it passes the hand to other threads. You can call other threads to run explictly by using the *join* call on them, but a non explicit call will happen if the main thread is ... sleeping. In that later case the scheduler will see the sleep call, pass the main thread in the background and grab the first thing it can to execute.

In the first example the for loop will create the sub threads by passing reference of the *queue* variable and then the main thread will continue until it reaches the *join* call and then run the sub threads. But by then, obviously the *queue* variable has changed to the last value of the loop.

The *sleep* call alters that process by letting the scheduler handle the sub threads before the queue value change with another go through the for loop.
