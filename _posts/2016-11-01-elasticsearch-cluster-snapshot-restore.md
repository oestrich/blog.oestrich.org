---
layout: post
categories:
- elasticsearch
- backup
title: Elasticsearch Cluster Snapshot & Restore
description: How to take a snapshot in one cluster, and restore into another.
date: 2016-11-01 11:30 AM
---

We recently needed to do a cross cluster snapshot and restore for elasticsearch. We were hosted on Elastic Cloud and found out the hard way that the tooling in place there does not work. We did find out that Elasticsearch has a backup repository system in place that works very well. We would have saved ourselves some time if we had just started with this.

## Create a user in AWS IAM

First off create a user in AWS IAM that has access to S3. You can attach the full access or limit down to a single bucket as shown in the [Elastic Cloud documentation][elasticcloud-s3-restore].

```json
{
  "Statement": [
    {
      "Action": [
        "s3:*"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::bucket-name",
        "arn:aws:s3:::bucket-name/*"
      ]
    }
  ]
}
```

## Create the Repository

Create the repository in each cluster. This is also taken from that guide. You also need the [repository-s3][repository-s3] plugin installed in each cluster for this to work.

```bash
sudo bin/elasticsearch-plugin install repository-s3

curl -X PUT localhost:9200/_snapshot/bucket-name -d '{
  "type": "s3",
  "settings": {
    "bucket": "bucket-name",
    "region": "us-east1",
    "access_key": "AKIAYOURKEYHERE",
    "secret_key": "secret-key",
    "compress": true
  }
}'
```

## Snapshot

On the old cluster, [create a snapshot][snapshot]. You can check on the status as it processes.

```bash
curl -X PUT localhost:9200/_snapshot/bucket-name/snapshot-backup-name
curl -X GET localhost:9200/_snapshot/bucket-name/snapshot-backup-name/_status
```

There are a lot of options you can provide to the snapshot, including limiting to certain indices.

## Restore

On the new cluster, [restore the snapshot][restore]. You can easily view the status of the restore with regular [elasticsearch monitoring tools][kopf]. The index health with be shown as shards come online.

```bash
curl -X POST localhost:9200/_snapshot/bucket-name/snapshot-backup-name/_restore -d '{
  "indices": "one-index"
}'
```

## Conclusion

Hopefully this is of use to others. Snapshotting and restoring manually is a very simple process and was much easier than trying to figure out a custom solution from your elasticsearch host.

[elasticcloud-s3-restore]: https://www.elastic.co/guide/en/cloud/current/custom-repository.html
[repository-s3]: https://www.elastic.co/guide/en/elasticsearch/plugins/5.0/repository-s3.html
[snapshot]: https://www.elastic.co/guide/en/elasticsearch/guide/current/backing-up-your-cluster.html#_snapshotting_all_open_indices
[restore]: https://www.elastic.co/guide/en/elasticsearch/guide/current/_restoring_from_a_snapshot.html
[kopf]: https://github.com/lmenezes/elasticsearch-kopf
