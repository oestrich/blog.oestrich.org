---
layout: post
title: Sinatra OAuth Workflow - Use This to Speed Up Your App Development
---

_This post was originally published on the_
[SmartLogic Blog](http://blog.smartlogicsolutions.com/2013/01/23/sinatra-oauth-workflow-use-this-to-speed-up-your-app-development/).

<img class="float-img" alt="With this Sinatra workflow, your app development will be as smooth as Frank" src="/images/sinatra.jpg" width="193" height="240" />

When you need to connect to multiple different [APIs](http://blog.smartlogicsolutions.com/2012/12/12/api-versioning-3-ways-to-architect-your-api-to-handle-versioned-requests/), it can take a long time to manage your OAuth workflow for each third-party app. On a recent [app development](http://smartlogicsolutions.com) project, I was getting really agitated using cURL for every step in the process while working on pulling in data from multiple apps. So I decided to do something about it. I'm a programmer, [damn it](https://twitter.com/cahooon).

While I don't have a magic solution, I created a [Sinatra](http://www.sinatrarb.com/) OAuth proxy app that works with Harvest. This sped up development for connecting with Harvest, and with small changes, the same code can also work for other applications. For example, I tested it out on Foursquare and the same code mostly worked.

This is the code I used to speed up OAuth and inspect the third party app's API.

[gist](https://gist.github.com/30350e793bc7900aa952)

##### server.rb
    require 'sinatra'
    require 'json'
    require 'faraday'
    require 'faraday_middleware'

    enable :sessions

    IDENTIFIER = "YOUR_IDENTIFIER"
    SECRET = "YOUR_SECRET"
    REDIRECT_URI = "http://localhost:3000/auth"
    AUTH_URL = "https://api.harvestapp.com/oauth2/authorize"
    TOKEN_URL = "https://api.harvestapp.com/oauth2/token"
    BASE_API_URL = "https://api.harvestapp.com/"

    # Middlware to insert "Accept: application/json" header
    class JsonMiddleware < Faraday::Middleware
      def call(env)
        env[:request_headers]["Accept"] = "application/json"
        @app.call(env)
      end
    end

    # Redirect to the API to start the OAuth handshake
    # http://www.getharvest.com/api/authentication-oauth2
    # http://tools.ietf.org/html/draft-ietf-oauth-v2-22#section-4.1
    get "/" do
      query_string = Rack::Utils.build_query({
        :client_id => IDENTIFIER,
        :redirect_uri => REDIRECT_URI,
        :state => "optional-csrf-token",
        :response_type => "code"
      })

      redirect "#{AUTH_URL}?#{query_string}"
    end

    # Harvest will redirect back here with the OAuth code you
    # use to get the access token.
    get "/auth" do
      query_string = Rack::Utils.build_query({
        :code => params[:code],
        :client_id => IDENTIFIER,
        :client_secret => SECRET,
        :redirect_uri => REDIRECT_URI,
        :grant_type => "authorization_code"
      })

      response = Faraday.post("#{TOKEN_URL}?#{query_string}")
      session[:token] = JSON.parse(response.body)["access_token"]
      redirect "/token"
    end

    # Removes favicon warnings
    get "/favicon.ico" do
    end

    # Display your access token
    get "/token" do
      session[:token]
    end

    # Proxy everything after / to Harvest
    #
    # http://localhost:3000/projects/1/entries?from=2013-01-10&to=2013-01-14
    # => https://api.harvest.com/projects/1/entries?from=2013-01-10&to=2013-01-14
    get "/*" do
      headers "Content-Type" => "text/plain"

      params.delete("captures")
      url = "/#{params.delete("splat").join}?"
      url += Rack::Utils.build_query(params)

      client = Faraday.new(:url => BASE_API_URL) do |faraday|
        faraday.request :url_encoded
        faraday.request :oauth2, session[:token]
        faraday.use JsonMiddleware
        faraday.adapter Faraday.default_adapter
      end

      response_body = client.get(url).body
      JSON.pretty_generate(JSON.parse(response_body))
    end

How do you handle OAuth without [cURLin'](http://blog.smartlogicsolutions.com/2012/07/12/curlin-for-docs/) all day long? Comment and let us know.

_For more like this, follow [@SmartLogic](http://twitter.com/smartlogic) and [@ericoestrich](http://twitter.com/ericoestrich) on Twitter._

[Image Source](http://www.flickr.com/photos/tom-margie/1546347753/sizes/s/in/photostream/)
