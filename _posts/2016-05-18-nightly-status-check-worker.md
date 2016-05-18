---
layout: post
categories:
- sidekiq
- ruby
- rails
- api
title: Nightly Status Check Worker
description: How to use a nightly status check worker to verify your integration is still valid.
date: 2016-05-18 12:00:00
---

Several of my projects involve integrating with APIs. In order to verify that all is well with our integrations we perform a nightly check up. In this check up we perform token validation and other sanity checks such as billing is valid. We check for anything that might change that would end up breaking our integration.

## Base Worker

```ruby
class NightlyCheckWorker
  include Sidekiq::Worker
  include Sidetiq::Schedulable

  recurrence { daily }

  def perform
    User.active.each do |user|
      CheckWorker.perform_async(user.id)
    end
  end
end
```

This worker is here to fan out to separate jobs for each active user. The `NightlyCheckWorker::CheckWorker` is listed below.

## Validation Class

```ruby
class NightlyCheckWorker
  class Validation
    include ActiveModel::Validations

    attr_reader :user

    def initialize(user)
      @user = user
    end

    def disable?
      false
    end

    # Hook to send mail on validation fail
    def deliver_notice
    end
  end
end
```

This class acts as an interface for the rest of validations. It has a basic constructor and a few methods with defaults that can be overridden.

It includes `ActiveModel::Validations` to allow each validation class to perform validations similar to normal active record objects.

## A Validator

```ruby
class NightlyCheckWorker
  class GoogleTokenValidator < Validation
    def disable?
      true
    end

    def message
      "Google token is invalid"
    end

    validate do
      response = user.google_client.get("/ping")
      case response.status
      when 401
        errors.add(:google, "token has been rejected"
      when 403
        errors.add(:google, "is invalid")
      end
    end
  end
end
```

This is a sample validator class. We can see that this should be disabled and gives a message we can use in when sending an email. Note that the validation code is just a sample and is almost certainly wrong. It is provided to show how we check a response and give different errors depending on the status or message contained within.

## NightlyCheckWorker::CheckWorker

```ruby
class NightlyCheckWorker
  class CheckWorker
    include Sidekiq::Worker

    def perform(user_id)
      validators.each do |validator_class|
        validator = validator_class.new(user)
        unless validator.valid?
          user.disable! if validator.disable?

          validator.deliver_notice
        end
      end

      update_user_info
    end

    private

    def validators
      [
        GoogleTokenValidator,
        # ...
      ]
    end
  end
end
```

This is the class that handles all of the work. The `#validators` method contains all of the validator classes and will loop over them performing checks. The only thing worth pointing out in this code is the last method call of `#perform`. I find it a good spot to include general caching in these workers. Every night we'll cache any counts or update information that might have gotten out of date.

## Improvements

One of the things that we changed after using this for a long time is saving the validation errors in the database. Previously they were always emailed to us and that was it. Now we're saving them in a JSON column attached to the user. I will leave this as an excercise for the reader.
