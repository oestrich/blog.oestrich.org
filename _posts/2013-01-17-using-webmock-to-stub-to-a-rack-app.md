---
layout: post
categories:
- ruby
- rspec
- rails
title: Using Webmock to Stub to a Rack App
---

I have been integrating with outside services recently and decided to use webmock to stub the requests. Instead of creating a bunch of `stub_request` for each section of the service you want to stub, you can just stub the entire domain to a rack app. It's a lesser known feature of webmock because it's just briefly mentioned in their readme.

You just have to call `#to_rack` at the end of a `stub_request` instead of `#to_return`. In the rack app you can also validate that the requests you are sending contain the correct headers, are authenticated correctly, etc.

##### spec/spec_helper.rb

    RSpec.configure do |config|
      config.before do
        stub_request(:any, /outsideservice\.com/).to_rack(OutsideServiceApp.new)
      end
    end

##### spec/support/outside_service_app.rb

    class OutsideServiceApp
      def call(env)
        validate_request(env)

        case [env["REQUEST_METHOD"], env["PATH_INFO"]]
        when ["GET", "/orders"]
          [200, {}, [File.read("spec/fixtures/orders.json")]]
        else
          [404, {}, []]
        end
      end

      ...
    end

##### spec/outside_service_caller_spec.rb

    describe OutsideServceCaller do
      it "should load orders" do
        OutsideServiceCaller.orders.should == File.read("spec/fixtures/orders.json")
      end
    end
