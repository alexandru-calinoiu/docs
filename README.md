---
home: true
heroImage: /eventide-icon-132.png
description: 'Microservices and Event Sourcing for Ruby'
actionText: Get Started →
actionLink: /examples/quickstart.md
features:
- title: Microservices
  details: Message-based services hosted in any number of operating system processes or machines, with actor-based pub-sub consumers, component hosting, message dispatching, and handlers
- title: Event Sourcing
  details: Business logic entities projected from event streams with both in-memory, first-level caching and second-level on disk caching
- title: Storage Options
  details: Support for Postgres and EventStore message stores and transports, depending on your performance and scale needs
footer: MIT Licensed | Copyright © 2015-present The Eventide Project
---

- - -

``` ruby
class Handler
  include Messaging::Handle

  handle Withdraw do |withdraw|
    account_id = withdraw.account_id

    account = store.fetch(account_id)

    time = clock.iso8601

    stream_name = stream_name(account_id)

    unless account.sufficient_funds?(withdraw.amount)
      return
    end

    withdrawn = Withdrawn.follow(withdraw)
    withdrawn.processed_time = time

    write.(withdrawn, stream_name)
  end
end

class Withdraw
  include Messaging::Message

  attribute :account_id, String
  attribute :amount, Numeric
  attribute :time, String
end

class Withdrawn
  include Messaging::Message

  attribute :account_id, String
  attribute :amount, Numeric
  attribute :time, String
  attribute :processed_time, String
end

class Account
  include Schema::DataStructure

  attribute :id, String
  attribute :balance, Numeric, default: 0

  def withdraw(amount)
    self.balance -= amount
  end

  def sufficient_funds?(amount)
    balance >= amount
  end
end

class Projection
  include EntityProjection

  apply Withdrawn do |withdrawn|
    account.id = withdrawn.account_id

    amount = withdrawn.amount

    account.withdraw(amount)
  end
end

class Store
  include EntityStore

  entity Account
  projection Projection
end
```
