---
layout: post
categories:
- ruby
- s3
- aws
- images
title: S3 Upload Script for Images
---

I wanted a way to upload screenshots quickly to something I had control of. This is a small script that I placed in my `PATH` and the XFCE launcher. It uploads to S3 and auto deletes after a week.

## Setting up S3

Create a bucket with the same name as your domain, e.g. `img.example.com`. Create a CNAME for `img.example.com` to `img.example.com..s3.amazonaws.com.`.

To have photos automatically deleted after 7 days open the S3 properties for the bucket. Expand the lifecycle tab and add a rule. Select the whole bucket and permanently delete after 7 days.

##### S3 properties
![S3 properties for the bucket](/images/s3-lifecycle.png)

Place the following script in a folder in your `PATH`. I have `~/bin` in my path for such a case. You will need to change the `HOST`, `access_key_id`, and `secret_access_key`.

##### upload.rb
```ruby
#!/usr/bin/env ruby

require 'aws-sdk'
require 'securerandom'
require 'clipboard'
require 'uri'

HOST = "img.example.com"

AWS.config({
  :access_key_id => '...',
  :secret_access_key => '...',
})

if ARGV.length == 0
  puts "Please run with a file"
  sleep 5
  exit
end

s3 = AWS::S3.new
bucket = s3.buckets[HOST]

# So many different ways because when you drop an image on a
# launcher for XFCE it adds `file://` in front of it.
if ARGV[0] =~ /file:\/\//
  file = URI.decode(URI.parse(ARGV[0]).path)
elsif ARGV[0] =~ /\A\//
  file = Pathname.new(ARGV[0])
else
  file = File.join(Dir.pwd, ARGV[0])
end

file_name = "#{SecureRandom.uuid}#{File.extname(file)}"

object = bucket.objects[file_name]
object.write(File.read(file), {
  :acl => :public_read,
  :reduced_redundancy => true
})

url = "http://#{HOST}/#{file_name}"

Clipboard.copy url
puts url

sleep 5
```
