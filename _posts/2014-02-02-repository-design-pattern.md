---
layout: post
title: Repository Design Pattern
---
Last week I studied and began implementing the Repository Design Pattern in a Ruby application. The RDP involves placing a layer of abstraction between data storage and the rest of the app, which ultimately provides more flexibility with regards to how you want to store your data.

I've been working on a bank app in Ruby. It doesn't do anything particularly fancy yet--you can create and store customers and accounts, deposit to and withdraw from accounts, pay interest to the accounts, view transaction histories for accounts, etc. As I mentioned a few posts ago, one of my early challenges at 8th Light has been lowering my dependency on "the Rails way" of doing things--this issue came to light quickly with the bank app as I realized I wasn't quite sure how to persist objects like a person or an account. In Rails, I'd create models for these objects and corresponding tables for them in a SQL database, and if I wanted to save a Person or an Account I'd simply store a record in the database. Starting from scratch in pure Ruby, however, I can't immediately assume that I want to store my data in a SQL database. In fact, it became a requirement that I implement a data storage method that *isn't* SQL.

My first thought was to simply create an array for each "kind" of object and store those objects by simply shoveling (`<<`) them into that array after they get instantiated.

{% highlight ruby %}
# person.rb
class Person
  attr_reader :first_name, :last_name
  def initialize(params={})
    @first_name = params[:first_name]
    @last_name = params[:last_name]
  end
end

# person_repository.rb
module PersonRepository
  @people = Array.new

  def self.add(person)
    @people << person
  end
end
{% endhighlight %}

I instantiate a Person object with `p = Person.new(first_name: "Miles", last_name: "Davis")`, then store it in memory via `PersonRepository.add(p)`. A decent solution, but not very configurable. For example, what if sometime in the future we wanted to instead store our data in a Riak key/value store (not so coincidentally my goal for this coming week)? We'd have to completely re-write the PersonRepository code from scratch, and almost certainly change some things in other areas of the app as well. From a SOLID perspective, this violates the Open/Closed Principle (more on SOLID coming in a later post).

So, what can we do instead? Enter the Repository Pattern. We add an abstracted Repository later that allows us to add new types of repositories (memory, Riak, SQL, Redis, etc.) without requiring extensive modification of the code. Here's an outline:

{% highlight ruby %}
# person.rb
class Person
  attr_reader :first_name, :last_name, :id
  def initialize(params={})
    @id = nil
    @first_name = params[:first_name]
    @last_name = params[:last_name]
  end
end

# repository.rb
module Repository
  def self.register(type, repo)
    repositories[:type] = repo
  end

  def self.repositories
    @repositories ||= Hash.new
  end

  def self.for(type)
    repositories[type]
  end
end

# repositories/memory/person_repository.rb
module MemoryRepository
  class PersonRepository
    attr_reader :people

    def initialize
      @people ||= Hash.new
      @id = 1
    end

    def add(person)
      person.id = @id
      @people[@id] = person
      @id += 1
      return person
    end
  end
end
{% endhighlight %}

First, you register an instance of the kind of repository you want to use for the given model/noun: `Repository.register(:person, MemoryRepository::PersonRepository.new)`. This would be executed someplace like the rake console task before starting Pry, or maybe a spec_helper file before launching RSpec. The Repository module's @repositories hash now looks like this: `{:person => MemoryRepository::PersonRepository.new}`, so when we call `Repository.for(:person)` somewhere, we are returned our instance of the MemoryRepository::PersonRepository in which we're storing all our person objects.

What's great about this is if we want to change to, say, Riak storage, all we need to do (besides setting up the Riak repository code itself) is call `Repository.register(:person, RiakRepository::PersonRepository.new)`--that's it! The rest of the app doesn't care about and isn't affected by *how* the data is being stored; it simply references `Repository.for(:person)`, which now returns the Riak repo instance instead of the Memory repo instance. The abstracted Repository layer thus allows the app to be **open to extension** while remaining **closed for modification** (Open/Closed Principle).
