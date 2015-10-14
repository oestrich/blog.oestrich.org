---
layout: post
categories:
- kubernetes
- rails
title: Kubernetes Build Script for Rails Apps
description: A basic build script to deploy a Rails app on Kubernetes
date: 2015-10-13 12:00 pm
---

We've so far seen how to put postgres inside of Kubernetes, here is how I was able to put a Rails app inside of Kubernetes. This is my deploy script that builds the docker container, and then pushes it out into the cluster. I'll show snippets first then the entire script at the end.

## Build Docker Image

```bash
build_dir=`mktemp -d`
git archive master | tar -x -C $build_dir
echo $build_dir
cd $build_dir

docker build -t $docker_image .
docker tag -f $docker_image $docker_tag
gcloud docker push $docker_tag
```

This creates a temp directory and creates a clean copy of the repo to build. I don't want to build in the working directory because it might contain files you don't want to build into the image. This makes sure it only ever contains what the `master` branch has.

It also pushes the image up tagged as the git sha. This lets us make sure we're always deploying the correct code, seen when we update the replication controllers.

## Optionally migrate

```bash
docker tag -f $docker_image "$docker_image:migration"
gcloud docker push "$docker_image:migration"
kubectl create -f config/kubernetes/migration-pod.yml
echo Waiting for migration pod to start...
sleep 10
while [ true ]; do
	result=`kubectl get pods migration | grep ExitCode`
	if [[ $result == *"ExitCode:0"* ]]; then
		break
	else
		sleep 1
	fi
done
kubectl delete pod migration
```

This section is done with a switch `-m`. It pushes to a special tag named `migration` and then creates a pod that will run `rake db:migrate`. It pushes to the special tag so that the migration pod can specify that branch and always have the latest code. Below is the yaml that is used to create the migration pod.

##### config/kubernetes/migration-pod.yml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: migration
  labels:
    name: migration
spec:
  restartPolicy: Never
  volumes:
    - name: environment-variables
      secret:
        secretName: app-env
  containers:
    - name: migration
      image: gcr.io/project/app:migration
      imagePullPolicy: Always
      args: ["bundle", "exec", "rake", "db:migrate"]
      volumeMounts:
        - name: environment-variables
          readOnly: true
          mountPath: /etc/env
```

## Update Replication Controllers

```bash
kubectl rolling-update app-web --image=$docker_tag --update-period="10s"
kubectl rolling-update app-worker --image=$docker_tag --update-period="10s"
```

This updates the replication controller's image. We set the update period to 10 seconds to speed the process up. The default time is 60 seconds. Kubernetes will take out one pod at a time and update to the new docker image. Once it completes the rolling update all pods will be running the newly deployed code.

## Update to loading kubernetes secrets

I had to update how I was [loading secrets from last time][secrets]. I replaced `foreman.sh` with `env.sh` and changed the docker `CMD`.

##### env.sh

```bash
#!/bin/bash
ruby env.rb .env /etc/env && source .env && $*
```

`$*` is worth pointing out. Bash uses it as a variable for all arguments. In the case of the docker file below it will expand out to:

```bash
ruby env.rb .env /etc/env && source .env && foreman start web
```

##### Dockerfile

```docker
ENTRYPOINT ["./env.sh"]
CMD ["foreman", "start", "web"]
```

## Full Script

```bash
#!/bin/bash
set -e

# Get options
migrate=''
deploy=''
while getopts 'md' flag; do
  case "${flag}" in
    m) migrate='true' ;;
    d) deploy='true' ;;
    *) error "Unexpected option ${flag}" ;;
  esac
done

sha=`git rev-parse HEAD`
docker_image="gcr.io/project/app"
docker_tag="$docker_image:$sha"

# Create an clean directory to build
build_dir=`mktemp -d`
current_dir=`pwd`

git archive master | tar -x -C $build_dir
echo $build_dir
cd $build_dir

# Build docker image and push
docker build -t $docker_image .
docker tag -f $docker_image $docker_tag
gcloud docker push $docker_tag

# Migrate if required
if [ $migrate ]; then
  docker tag -f $docker_image "$docker_image:migration"
  gcloud docker push "$docker_image:migration"
  kubectl create -f config/kubernetes/migration-pod.yml
  echo Waiting for migration pod to start...
  sleep 10
  while [ true ]; do
    result=`kubectl get pods migration | grep ExitCode`
    if [[ $result == *"ExitCode:0"* ]]; then
      break
    else
      sleep 1
    fi
  done
  kubectl delete pod migration
fi

if [ $deploy ]; then
  # Update images on kubernetes
  kubectl rolling-update app-web --image=$docker_tag --update-period="10s"
  kubectl rolling-update app-worker --image=$docker_tag --update-period="10s"
fi

# clean up build folder
cd $current_dir
rm -r $build_dir
```

[secrets]: {% post_url 2015-09-15-kubernetes-secrets-to-env-file %}
