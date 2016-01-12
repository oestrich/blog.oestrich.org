---
layout: post
categories:
- kubernetes
- nginx
- linux
title: Hosting Your Own Kubernetes NodePort Load Balancer
description: How to create a NodePort load balancer
date: 2016-01-11 09:00 PM
---

I recently switched from using a regular `Loadbalancer` in [kubernetes][kubernetes] to using a `NodePort` load balancer. I switched because I wanted to get away from using the fairly expensive network load balancer in Google Cloud Compute in a personal project.

I did this by creating a new service for [nginx][nginx] inside of the cluster, and setting it to use a `NodePort` load balancer. Then creating a micro instance that hosts another nginx that proxies to the private IPs of the cluster nodes.

##### nginx-service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
      name: http
    - port: 443
      nodePort: 30443
      name: https
  selector:
    name: nginx
```

The import pieces here are `type` and the `nodePort` line inside of ports. This will cause kubernetes to listen on `30080` and `30443` respectively. This port has to be [between the range of 30000-32767][nodeport-range].

Once the service is defined as a `NodePort` load balancer you need to set up the micro instance.

##### /etc/nginx/sites-enabled/example

```nginx
upstream example-https {
  server 10.0.0.1:30443; # private node ip
  server 10.0.0.2:30443; # private node ip
}

server {
  # ... ssl setup ...

  location / {
    # ... proxy_pass configuration ...
    proxy_pass https://example-https;
  }

  # ... other nginx setup ...
}
```

Here we have a stripped down sample file `sites-enabled/example` that nginx will load and start serving. I have SSL set up and `proxy_pass` to the private IP of each kubernetes node. I do keep SSL between this front nginx and the nginx inside of the cluster.

There isn't anything special about this file so I've stripped out the unimportant pieces. You can see a more complete example of my nginx files in the post [hosting nginx in docker][nginx-docker].

## Drawbacks

There are a few drawbacks using this approach. I've had at least once the private IP change (I think the nodes rebooted themselves after a crash) and I had to update this frontend nginx. You also have to manually add in an new nodes. The normal load balancer would have handled each case for me.

Overall I've been happy with this setup. It was nice to see that kubernetes let me handle my own external load balancing.

[kubernetes]: http://kubernetes.io/
[nginx]: http://nginx.org/
[nodeport-range]: http://kubernetes.io/v1.0/docs/user-guide/services.html#type-nodeport
[nginx-docker]: {% post_url 2015-03-17-nginx-docker-container %}
