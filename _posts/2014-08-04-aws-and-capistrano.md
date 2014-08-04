---
layout: post
categories:
- aws
- ec2
- ruby
- rails
- capistrano
- deploy
title: Deploying on multiple EC2 servers with Capistrano
---

For [SmartChat](http://github.com/smartlogic/smartchat-api) we were deploying on EC2. There were three types of serveers to deploy and I used EC2 tags and capistrano to deploy to each server correctly without having to hardcode any of the servers.

Here is my EC2 console with a custom `Type` key.

![EC2 Console with tags](/images/ec2-console.png)

In the deploy script for production we can now use the AWS SDK to pull your instances down and deploy to them correctly.

Dotenv is used because AWS keys are stored in `.env` and capistrano won't load that normally.

##### config/deploy/production.md
    require 'aws-sdk'
    require 'dotenv'
    Dotenv.load

    set :rails_env, "production"

    primary = true

    ec2 = AWS::EC2.new
    ec2.instances.tagged("Type").tagged_values("web").each do |instance|
      next unless instance.status == :running

      # We only want one application server migrating the database
      server instance.public_dns_name, :web, :app, :db, :primary => primary

      primary = false
    end

    ec2.instances.tagged("Type").tagged_values("worker").each do |instance|
      next unless instance.status == :running
      server instance.public_dns_name, :worker, :app
    end

    ec2.instances.tagged("Type").tagged_values("scheduler").each do |instance|
      next unless instance.status == :running
      server instance.public_dns_name, :scheduler, :app
    end
