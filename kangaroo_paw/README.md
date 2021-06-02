# Kangaroo Paw

## Origin

A mix of reading, personal experience and exchanges with other Rails developers.

## Basics

The aim is to keep things simple in every class by sticking to simplicity that the single responsibility principle tend to foster. To do so we rely on several additional layers :

- commands : to act on resources
- decorators : to present resources' content
- form objects : to handle forms without exposing a whole model
- workers : to handle asynchronous work
- other code bits : to handle non Rails related things

As consequence we also have some guidelines about the Rails' layers :
- models : define relationships, scopes
- controllers : handle params, call on commands to do things, render is explicitely passed locals
- views : heavy use of partials, as little logic in here as possible

## In details

### Commands

**[related design pattern](https://refactoring.guru/design-patterns/command/ruby/example)**

Although there are some great gems out there we found that it's quite often enough to define one way to do those and stick to it.

Commands should done one thing or call several other commands. Their role is to act on resources : create, update, delete them. Their name should be quite clear about what they do : `SendConfirmationAfterOrderSuccess` for example. It's also best to namespace them as it helps with getting a context about the command : `Order::SendConfirmationOnSuccess`.

One method should be exposed, be it a class method or an instance one, but make it obvious and always use the same name : `perform` or `call` is often picked.

Commands should not be long, should not have many private methods. If done well their role and content is simple and obvious to the eye. Tests are, in turn, easy to write and simple to maintain.

### Decorators

**[related design pattern](https://refactoring.guru/design-patterns/decorator)

This is another classic pattern and Ruby comes with a great library to use : [Simple Delegator](https://ruby-doc.org/stdlib-2.5.1/libdoc/delegate/rdoc/SimpleDelegator.html). More elaborate ones exist but `SimpleDelegator` will get you a long way.

The goal is to use decorators to handle all the presentation work around a Model instance. A typical use is a `name` method that concatenates `first_name` and `last_name` together. Another could be returning the creation date of a resource in a formatted way. We don't like seeing tons of HTML in decorators' methods that's not the point here.

In general decoratored instances of a model are what is returned by a Controller to the view through render's `locals` parameter.

Again, decorators are easy to tests and since they can decorate an object even if it's not persisten their tests can be pretty fast.

### Form objects

**resource:** [Thoughbot's ActiveModel Form Objects](https://thoughtbot.com/blog/activemodel-form-objects)

A rarely seen but very powerful way to handle complex (and simple) forms with all the goodness of validations and a good place to store business logic related to the form too. The controller and views are left lighter and clearer.

This relies on inclusion of `ActiveModel::Model` in a simple Ruby class to be able to use `attr_accessor`, `validates` and few other tricks. The form is non the wiser and believe it's handling a resource. When the object is sent to the controller we can use the params to instanciate another instance of the form object, check the validation and other business logic through the exposed methods and proceed with calling the command.

All this make for good separation of responsability again and simplicity in the code. This in turn is easy to tests and maintain.

### Workers

**resource:** [ActiveJob Basics on Rails Guides](https://guides.rubyonrails.org/active_job_basics.html)

What ever the library used for this they should be a defacto inclusion in any RoR project to handle everything that can be put into asynchronous mode.

ActiveJob is a great way to handle the complexity and have a nice abstraction layer to be able to switch between underlying libraries without any difference in how things happen.

That being said the available backends are split among a few types :
- single process ([Sucker Punch](https://github.com/brandonhilkert/sucker_punch#active-job))
- redis backed ([Sidekiq](https://github.com/mperham/sidekiq/wiki/Active-Job), [Resque](https://github.com/resque/resque/wiki/ActiveJob), [Delayed Job](https://github.com/collectiveidea/delayed_job#active-job))
- PostgreSQL backed ([Queue classic](https://github.com/QueueClassic/queue_classic#active-job), [Que](https://github.com/que-rb/que#additional-rails-specific-setup))
- RabbitMQ backed ([Sneakers](https://github.com/jondot/sneakers/wiki/How-To:-Rails-Background-Jobs-with-ActiveJob))
- SQS backed ([Shoryuken](https://github.com/ruby-shoryuken/shoryuken/wiki/Rails-Integration-Active-Job))

Those should already give any team plenty of room to grow. Yet, if you want to use other tools such as Kafka there are also libraries to do that ([Ruby-kafka](https://github.com/zendesk/ruby-kafka), [Racecar](https://github.com/zendesk/racecar), [Karakfa](https://github.com/karafka/karafka).

### other code bits

And then there is often a need to write custom code such as API clients, cache layers, metrics abstractions, and other extra little things. Those should be kept out of the way of the application code base they serve. In many cases it's entirely possible to maintain those as separate libraries (private or public gems) and have the code base use them. In other cases `lib/` is the place to store them, with the relevant namespacing.


