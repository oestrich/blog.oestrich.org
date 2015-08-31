---
layout: post
categories:
- kubernetes
- postgres
- google cloud
- docker
title: Running Postgres Inside Kubernetes With Google Container Engine
description: Running a Postgres container inside of kubernetes using the Google Cloud.
date: 2015-08-31 03:00 PM
---

I have been playing around with [Kubernetes][kubernetes] in the last week or so and have really started liking it. One of the main things I wanted to get running was [Postgres][postgres] inside of the cluster. This is particularly challenging because postgres requires a volume that sticks around with the [docker][docker] container. If it doesn't you will lose your data.

In researching this I found out kubernetes has a persistence manager that lets you mount [Google Compute][google-compute] disks. This also works for AWS EBS volumes if your cluster is hosted over there.

## Create the disk

``` bash
gcloud compute disks create pg-data-disk --size 50GB
```

Make sure this will be in the same zone as the cluster.

Next attach the disk to an already running instance, possibly a new instance. We only need to temporarily attach the disk so we can format it.

``` bash
gcloud compute instances attach-disk pg-disk-formatter --disk pg-data-disk
```

After the disk is attached, ssh into the instance and run the following commands. This will mount and then format the disk with ext4. Then we unmount the drive.

``` bash
sudo /usr/share/google/safe_format_and_mount -m "mkfs.ext4 -F" /dev/sdb /media/pg-data/
sudo umount /media/pg-data/
```

```bash
gcloud compute instances detach-disk pg-disk-formatter --disk pg-data-disk
```

## Set up Kubernetes

With all of this complete we can start on the kubernetes side. Create the following four files.

##### postgres-persistence.yml

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pg-data-disk
  labels:
    name: pg-data-disk
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
  gcePersistentDisk:
    pdName: "pg-data-disk"
    fsType: "ext4"
```

This creates a persistent volume that pods can mount. Set it up with the same information that you used to create the disk.

##### postgres-claim.yml

``` yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pg-data-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

This creates a claim on the persistent volume that pods will use to attach the volume. It should have the same information as above.

##### postgres-pod.yml

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres
  labels:
    name: postgres
spec:
  containers:
    - name: postgres
      image: postgres
      env:
        - name: DB_PASS
          value: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
      ports:
        - containerPort: 5432
      volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: pg-data
  volumes:
    - name: pg-data
      persistentVolumeClaim:
        claimName: pg-data-claim
```

There are a few important features of this file. We use the persistent claim in the `volumes` section and mount it in the `volumeMounts` section. Postgres doesn't have it's data dir at the top level of a mount so we move it lower with the environment variable `PGDATA`. You can also change the password with `DB_PASS`.

##### postgres-service.yml

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  labels:
    name: postgres
spec:
  ports:
    - port: 5432
  selector:
    name: postgres
```

This creates a service to easily access the postgres pod from other pods in the cluster. It creates a DNS entry pods can use to connect.

## Create resources on Kubernetes

``` bash
kubectl create -f postgres-persistence.yml
kubectl create -f postgres-claim.yml
kubectl create -f postgres-pod.yml
kubectl create -f postgres-service.yml
```

Run all of these commands and Postgres will be up and running inside your cluster.

## Improvements

This works great accept for creating new databases. I had to attach to the running docker container to create new databases. This isn't too bad, but could be a lot better.

It would also probably be better to create a replication controller instead of just a regular pod. That way if the pod died it would come back online by itself.

Overall kubernetes has been super fun to work with, especially the version hosted by [Google][google-container-engine].

[kubernetes]: http://kubernetes.io/
[docker]: https://www.docker.com/
[google-compute]: https://cloud.google.com/compute/
[google-container-engine]: https://cloud.google.com/container-engine/
[postgres]: http://www.postgresql.org/
