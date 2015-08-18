---
layout: post
date: 2015-07-29 1:00 PM
categories:
- nginx
- ssl
title: Nginx in Docker with a Self-Signed Root Certificate
description: How to create a docker container for nginx for an SSL endpoint using a self signed root certificate.
---

For a side project at work we needed to get a simple SSL endpoint in front of [Bosun][bosun]. I went about this by sticking Nginx inside of a docker container with a self-signed root certificate. I picked a self-signed root certificate because we didn't want to pay for a trusted certificate for a small project and creating our own trust completely worked out well for the project.


## Generate the root certificate and certificates

This is taken from [ruby documentation][ruby-documentation] as it was the easiest way I found to generate a root certificate.

``` ruby
require 'openssl'

root_key = OpenSSL::PKey::RSA.new 2048 # the CA's public/private key
root_ca = OpenSSL::X509::Certificate.new
root_ca.version = 2 # cf. RFC 5280 - to make it a "v3" certificate
root_ca.serial = 1
root_ca.subject = OpenSSL::X509::Name.parse "/DC=org/DC=ruby-lang/CN=Ruby CA"
root_ca.issuer = root_ca.subject # root CA's are "self-signed"
root_ca.public_key = root_key.public_key
root_ca.not_before = Time.now
root_ca.not_after = root_ca.not_before + 2 * 365 * 24 * 60 * 60 # 2 years validity

ef = OpenSSL::X509::ExtensionFactory.new
ef.subject_certificate = root_ca
ef.issuer_certificate = root_ca

root_ca.add_extension(ef.create_extension("basicConstraints","CA:TRUE",true))
root_ca.add_extension(ef.create_extension("keyUsage","keyCertSign, cRLSign", true))
root_ca.add_extension(ef.create_extension("subjectKeyIdentifier","hash",false))
root_ca.add_extension(ef.create_extension("authorityKeyIdentifier","keyid:always",false))
root_ca.sign(root_key, OpenSSL::Digest::SHA256.new)

key = OpenSSL::PKey::RSA.new 2048
cert = OpenSSL::X509::Certificate.new
cert.version = 2
cert.serial = 2
cert.subject = OpenSSL::X509::Name.parse "/CN=example.com"
cert.issuer = root_ca.subject # root CA is the issuer
cert.public_key = key.public_key
cert.not_before = Time.now
cert.not_after = cert.not_before + 1 * 365 * 24 * 60 * 60 # 1 years validity
ef = OpenSSL::X509::ExtensionFactory.new
ef.subject_certificate = cert
ef.issuer_certificate = root_ca
cert.add_extension(ef.create_extension("keyUsage","digitalSignature", true))
cert.add_extension(ef.create_extension("subjectKeyIdentifier","hash",false))
cert.sign(root_key, OpenSSL::Digest::SHA256.new)

File.open("root.key", "w") do |file|
  file.write(root_key.to_pem)
end
File.open("root.crt", "w") do |file|
  file.write(root_ca.to_pem)
end

File.open("bosun.key", "w") do |file|
  file.write(key.to_pem)
end
File.open("bosun.crt", "w") do |file|
  file.write(cert.to_pem)
end
```


## Setup Nginx in Docker

##### Dockerfile

``` docker
FROM nginx
ADD ssl /etc/nginx/ssl
ADD sites /etc/nginx/sites
ADD nginx.conf /etc/nginx/nginx.conf
```

This is my normal nginx setup. Nothing special here.

##### siites/bosun.conf

``` nginx
upstream bosun {
  server 192.168.1.1:5000;
}

server {
  listen 443 ssl;

  ssl on;
  ssl_certificate /etc/nginx/ssl/bosun.crt;
  ssl_certificate_key /etc/nginx/ssl/bosun.key;

  location / {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://bosun;
  }

  client_max_body_size 4G;
  keepalive_timeout 10;
  server_tokens off;

  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
}
```

In this, the only thing that needs to specially pointed out is the `upstream` needs to point at your ip address or wherever the server lives.

## Build and Start Nginx

``` bash
docker build -t nginx-ssl .
docker run -t -i -p 4443:443 nginx-ssl
```

## Connecting via Faraday

``` ruby
require 'faraday'
require 'json'
require 'openssl'

store = OpenSSL::X509::Store.new
store.add_file("root.pem")

connection = Faraday.new('https://localhost:4443', {
  :ssl => {
    :cert_store => store,
  }
})
```

To setup faraday, you need to create a custom certificate store that contains the `root.pem` file. After the connection is created with the right certificate store, everything just works.

This is also a neat way to get a green SSL bar without having to pay. You can install your new root certificate in your browser and not have trust issues warning you about continuing. This is primarily useful for small side projects that only you will be visiting as anyone without the root certificate trusted will still see warnings.

[bosun]: http://bosun.org/
[ruby-documentation]: http://ruby-doc.org/stdlib-1.9.3/libdoc/openssl/rdoc/OpenSSL/X509/Certificate.html
