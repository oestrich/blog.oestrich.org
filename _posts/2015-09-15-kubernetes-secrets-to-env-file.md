---
layout: post
categories:
- kubernetes
- ruby
title: Kubernetes Secrets to Environment File
description: Ruby snippet to convert a Kubernetes secrets folder into an environment file.
date: 2015-09-15 07:30 PM
---

When hosting a Rails application, I really like to push as much configuration into the environment as possible. This is why I like [dotenv][dotenv] and used it pretty extensively in [the bramble][bramble]. With the bramble I stuck the `.env` file inside of the container. Moving to Kubernetes I didn't want to continue with this.

Kubernetes has something called a secret that stores key/values and you mount them as a volume in the pod. This works great except I want to continue using dotenv so I don't write the entire application specifically for Kubernetes.

To manage this I wrote a simple script that converts a secrets folder into an environment file. I run this little script before starting `foreman`.

##### env.rb

```ruby
env = {}

Dir["#{ARGV[1]}/*"].each do |file|
  key = file.split("/").last
  key = key.gsub("-", "_").upcase
  env[key] = File.read(file).strip
end

File.open(ARGV[0], "w") do |file|
  env.each do |key, value|
    file.puts(%{export #{key}="#{value}"})
  end
end
```

```bash
ruby env.rb .env /etc/etc
```

The file looks at the folder passed in as the second argument. Secret key values have to be hyphenated so we convert hyphens to underscores and uppercase everything. Then write the values to the first file in a format that dotenv accepts.

## Secrets

To create the secret you upload the following YAML to kubernetes. Once the secret is created you mount them in a pod.

##### secrets.yml

```yaml
apiVersion: "v1"
kind: "Secret"
metadata:
  name: app-env
data:
  rails-env: cHJvZHVjdGlvbg==
  rack-env: cHJvZHVjdGlvbg==
  rails-serve-static-files: dHJ1ZQ==
```

```bash
kubectl create -f secrets.yml
```

This file creates the secret `app-env`. Values are base 64 encoded.

##### app-controller.yml

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: app
  labels:
    name: app
spec:
  replicas: 3
  selector:
    name: app
  template:
    metadata:
      labels:
        name: app
    spec:
      volumes:
        - name: environment-variables
          secret:
            secretName: app-env
      containers:
        - name: teashop
          image: smartlogic/app
          imagePullPolicy: Always
          ports:
            - containerPort: 5000
          volumeMounts:
            - name: environment-variables
              readOnly: true
              mountPath: /etc/env
```

This replication controller mounts the secret into `/etc/env`.

##### Dockerfile

To use the new script in your `Dockerfile` change the entrypoint to:

```docker
ENTRYPOINT ["./foreman.sh"]
```

With `foreman.sh` as:

```bash
#!/bin/bash
ruby env.rb .env /etc/env && foreman start ${BASH_ARGV[0]}
```

## Improvements

This isn't the nicest set up but it is a good stop gap until we can figure something else out. Some places to consider going forward are [Keywhiz][keywhiz] and [etcd][etcd]. Either of those should be a much better configuration management.

[dotenv]: https://github.com/bkeepers/dotenv
[bramble]: https://blog.oestrich.org/2015/04/the-bramble/
[keywhiz]: https://square.github.io/keywhiz/
[etcd]: https://github.com/coreos/etcd
