---
layout: post
categories:
- ruby
- elasticsearch
- rake
title: Reindexing Elasticsearch with Ruby
description: Reindex Elasticsearch via a rake task
---

For a project at work we needed to reindex a large [Elasticsearch][elasticsearch] index and couldn't do it via the `_reindex` API. The source needed to be processed slightly in the new index. We were reindexing to gain more shards.

This is the rake task that helped reindex. It is based on this [Elasticsearch Guide][es-guide].

## Usage

First create the new index via Postman.

```
POST /new_index
{
  'settings': {
    'index': {
      'number_of_shards': 3
    }
  }
}
```

Next run the rake task. This will handle the reindex from one to the other via scan/scroll and the bulk API. I ran this on my local machine to not deal with Heroku timeouts. I simply set the correct environment variables to have Chewy pick up the production Elasticsearch. This is risky but I was not worried because we were dealing with only a new index.

```bash
bundle exec rake elasticsearch:reindex[old_index,new_index]
```

This rake task comes with a nice progress bar to track how far along the reindex is.

Once the reindex is complete you can update the alias you use to have production start using the new index.

```
POST /_aliases
{
  'actions': [
    { 'remove': { 'index': 'old_index', 'alias': 'alias' } },
    { 'add': { 'index': 'new_index', 'alias': 'alias' } }
  ]
}
```

## Rake Task

```ruby
namespace :elasticsearch do
  desc "Reindex a index"
  task :reindex, [:old_index_name, :new_index_name] => [:environment] do |t, args|
    client = Chewy.client
    results = client.search({
      index: args[:old_index_name],
      scroll: '10m',
      body: {
        "query" => { "match_all" => {} },
        "sort" => ["_doc"],
        "size" => 1000,
      },
    })

    progressbar = ProgressBar.create({
      :title => "Documents (thousands)",
      :total => results["hits"]["total"] / 1000 + 1,
      :format => '%a |%B| %p%% %t %c of %C',
    })

    loop do
      break if results["hits"]["hits"].empty?

      bulk_body = results["hits"]["hits"].map do |result|
      source = result["_source"]

      # process the source

      {
        index: {
          _index: args[:new_index_name],
          _type: result["_type"],
          _id: result["_id"],
          data: source,
        },
      }
      end

      response = client.bulk(body: bulk_body)

      if response["errors"]
        raise "Problem reindexing - #{response.inspect}"
      end

      progressbar.increment

      results = client.scroll({
        scroll: '10m',
        scroll_id: results["_scroll_id"],
      })
    end

    progressbar.finish
  end
end
```

## Improvements

This rake task isn't perfect, but it gets most of the way there. Some future improvements will be better handling around timeouts when talking to Elasticsearch. It would also be good to deal with indexing errors when bulk importing into the new index.

[elasticsearch]: https://www.elastic.co/
[es-guide]: https://www.elastic.co/guide/en/elasticsearch/guide/current/reindex.html
