---
layout: post
categories:
- ruby
- google cloud storage
title: Signing Google Cloud Storage URLs
description: How to sign Google Cloud Storage URLs so they are publicly accessible
date: 2016-01-29 10:30 AM
---

For my side project, [Worfcam][worfcam], I use Google Cloud Storage to host files. I have files that I want to be private most of the time, but allow anonymous access when I want. S3 has this by allowing you to sign a URL so it has an expiration. Luck for me Google Cloud Storage does this as well, the only problem was that the ruby SDK didn't do this for you like the S3 SDK does.

After a lot of searching I finally figured out how to generated signed URLs that expired for Google Cloud Storage.

##### signer.rb

```ruby
private_key = "..."
client_email = "..."
bucket = "..."
path = "..."

full_path = "/#{bucket}/#{path}"
expiration = 5.minutes.from_now.to_i

signature_string = [
  "GET",
  "",
  "",
  expiration,
  full_path,
].join("\n")

digest = OpenSSL::Digest::SHA256.new
signer = OpenSSL::PKey::RSA.new(private_key)
signature = Base64.strict_encode64(signer.sign(digest, signature_string))
signature = CGI.escape(signature)

"https://storage.googleapis.com#{full_path}?GoogleAccessId=#{client_email}&Expires=#{expiration}&Signature=#{signature}"
```

In the end this is pretty simple, but it took a long time to figure out exactly what I needed. This uses the private key that Google lets you download from the console, not the interoperability keys you can also downlown. This is very well explained [in a Cloud Storage support page][signed-urls-google], but it only lists Java, Python, Go, and C# as the languages.

I couldn't find anything for ruby so hopefully this helps someone else when they try to implement signed urls with ruby. This is also available [as a gist][gist].

[worfcam]: https://worfcam.com
[signed-urls-google]: https://cloud.google.com/storage/docs/access-control?hl=en#Signed-URLs
[gist]: https://gist.github.com/oestrich/44602a5262ad3c469681
